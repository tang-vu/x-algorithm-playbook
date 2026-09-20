# Algorithm FAQ

> Frequently asked questions about the X algorithm, answered with code references.

---

## General Questions

### Q: Is the X algorithm really open source?

**A:** Yes. Twitter first open-sourced the recommendation code in April 2023, xAI shipped a major rewrite on **May 15, 2026**, and — crucially — the **August–September 2026 releases** made almost everything real ([github.com/xai-org/x-algorithm](https://github.com/xai-org/x-algorithm), Apache-2.0): the **production Phoenix model** (JAX training + Rust serving), the **actual action weights** in `home-mixer/params/param.rs`, the **visibility-filtering rule engine**, SimClusters, VMRanker, and the new-author boost. The main things still withheld: full-size model weights and some anti-spam rule internals (`scarecrow`, `botmaker` prompts/rules) to reduce gaming.

**Source:** [github.com/xai-org/x-algorithm](https://github.com/xai-org/x-algorithm) · [github.com/twitter/the-algorithm](https://github.com/twitter/the-algorithm)

---

### Q: How often does the algorithm change?

**A:** xAI now publishes updates to the open-source repo **every 4 weeks, with developer notes** explaining what changed. The fundamental mechanics (Grok-based scoring, filtering, two-tower retrieval) are stable; specific weights, sources, and model configs evolve release to release. Re-check the repo's developer notes monthly.

---

### Q: Does paying for X Premium help reach?

**A:** Based on the public code, there's no explicit "premium boost." However, premium features (longer posts, edit button) may indirectly help engagement. The algorithm focuses on engagement signals, not subscription status.

---

## Scoring Questions

### Q: Which action matters most?

**A:** Now verified by real weights (`params/param.rs`): **share-via-copy-link = 20.0** is the single biggest weight — someone copying your post's link to send it elsewhere. Next: **reply, quote, DM share = 5.0 each**, and a reply on an original post to a **mutual follow** is effectively **20.0** (5.0 + 15.0 `BidirectionalFollowReplyWeightBoost`). A like is 0.5 — 1/40th of a copy-link share.

**Source:** `home-mixer/params/param.rs`

---

### Q: How bad is getting blocked?

**A:** Bad — but *not the worst*. Real values: block = **−31.2**, while **mute = −58.8** and **"not interested" = −43.2** are harsher, and **report = −234.0** dwarfs everything. Important caveat: weights multiply *predicted probabilities per viewer*, not counts — a single block doesn't erase N likes. What kills reach is consistently making viewers predict-blockable.

**Source:** `home-mixer/params/param.rs` — `BlockAuthorWeight`, `MuteAuthorWeight`, `NotInterestedWeight`, `ReportWeight`

---

### Q: Are the exact action weights public?

**A:** **Yes — since the August 2026 release.** `home-mixer/params/param.rs` publishes the production defaults: `ShareViaCopyLinkWeight=20.0`, `ReplyWeight=5.0`, `QuoteWeight=5.0`, `ShareViaDmWeight=5.0`, `FollowAuthorWeight=4.0`, `RetweetWeight=1.0`, `FavoriteWeight=0.5`, `ReportWeight=−234.0`, `MuteAuthorWeight=−58.8`, `NotInterestedWeight=−43.2`, `BlockAuthorWeight=−31.2`, plus diversity (0.5/0.25), OON (0.75), cold-start, and VMRanker params. (This supersedes the May-era claim that values were redacted — the `params` module is now in the repo.)

**Source:** `home-mixer/params/param.rs` — [full table](action-weights.md)

---

### Q: Does "view" count matter?

**A:** Views/impressions are not directly weighted. What matters is engagement (actions taken). However, dwell time (how long someone looks at your post) IS tracked and weighted.

---

### Q: Are all retweets equal?

**A:** The algorithm looks up the original tweet ID for retweets when scoring. Your retweet's score is influenced by the original content's predicted engagement.

**Source:** `phoenix_scorer.rs`

---

## Reach Questions

### Q: Why did my reach suddenly drop?

**A:** Possible causes:

1. **Getting blocked/muted** by multiple users (author socialgraph filter)
2. **Content topic shift** (embedding mismatch)
3. **Algorithm changes** (happens regularly)
4. **Posting frequency** (author diversity penalty)
5. **Lower engagement rate** (signals declining interest)

---

### Q: How do I appear on non-followers' feeds?

**A:** Through the Two-Tower retrieval system (Phoenix). Your content embedding is matched with user embeddings via similarity search. Being consistent in your niche creates a cleaner embedding that matches better.

**Source:** `phoenix/` (two-tower retrieval)

---

### Q: What are the out-of-network candidate sources?

**A:** As of September 2026: **Phoenix Retrieval** (two-tower similarity over semantic IDs), **SimClusters** (accounts/posts clustered by engagement patterns — added Aug 2026), **Phoenix Topics** (topical discovery), **Phoenix MoE** (exists but `EnablePhoenixMOESource=false` — an A/B experiment), **Who-to-Follow**, and **Ads/Prompts** at the blending layer. Practically: sharp, consistent topics open all doors; MoE isn't live for most users yet.

**Source:** `home-mixer/sources/`, `simclusters/`, `phoenix/`

---

### Q: Does posting time matter?

**A:** Yes. The Age Filter removes posts older than **48 hours** (`MaxPostAgeHours=48`, verified). Posts need early engagement to be distributed widely. Posting when your audience is online is crucial.

**Source:** `home-mixer/filters/age_filter.rs`, `params/param.rs`

---

### Q: Do hashtags help?

**A:** Not directly mentioned in scoring code. Hashtags may help with:

1. Content embedding (topic signal)
2. Search discovery
3. User muted keyword filters (could hurt if hashtag is muted)

Use relevant hashtags sparingly.

---

## Content Questions

### Q: Are threads better than single tweets?

**A:** Threads have advantages:

1. One published post = one author-position (vs N separate posts each taking a position in the diversity decay)
2. Higher dwell time (people read multiple tweets)
3. More engagement opportunities per thread

**Note:** the author-diversity scorer only groups by `author_id` — it has **no special thread logic**. The benefit comes from publishing one post instead of many, not from a thread exemption in the code.

---

### Q: Does video get boosted?

**A:** Less than you'd think, per the real weights. The minimum duration gate is exactly **10 seconds** (`MinVideoDurationMs=10_000`, verified), but `VqvWeight=0.0` right now — the dedicated video-quality-view bonus is dormant. The live video terms are `VideoOpenWeight=0.07` and dwell time. Video earns reach through engagement, not a video bonus.

**Source:** `home-mixer/params/param.rs` — `MinVideoDurationMs`, `VqvWeight`, `VideoOpenWeight`

---

### Q: Do images help?

**A:** Photo Expand is tracked as a positive action, but it's low-weight. Images help more by:

1. Stopping scroll (attention)
2. Increasing dwell time
3. Making content shareable

---

### Q: What about external links?

**A:** The code doesn't show explicit link penalties. However:

- Links take users off-platform (less engagement)
- Native content generally performs better
- Link previews may affect scroll behavior

---

## Technical Questions

### Q: What is "candidate isolation"?

**A:** A key ML architecture decision. When the transformer scores candidates, each post can only "see" the user context—not other candidate posts. This ensures your score doesn't depend on what other posts are in the batch.

**Source:** `phoenix/grok.py` - `make_recsys_attn_mask()`

---

### Q: What is the Two-Tower model?

**A:** A retrieval system with two neural networks:

1. **User Tower:** Encodes user + history into embedding
2. **Candidate Tower:** Encodes posts into embeddings
3. **Matching:** Dot product similarity finds relevant posts

**Source:** `phoenix/` (two-tower retrieval)

---

### Q: What is `grox` / content understanding?

**A:** `grox` is the content-understanding service that runs **classifiers and embedders** over every post during hydration — *before* scoring. It produces topic labels, embeddings, and safety signals; Phoenix then quantizes the multimodal embedding into a **semantic ID** (6×256 codes) that becomes the post's identity for retrieval. Key takeaway: there is **no manual keyword/hashtag boost** for relevance — clarity is what gets you matched to the right audience.

**Source:** `grox/`, `phoenix/`

---

### Q: How does the author diversity penalty work?

**A:** Within one feed response, your posts are sorted by score, and each extra post from the same author is attenuated by its rank — decaying toward a floor (never to zero). **Real values** (Aug 2026): `decay=0.5`, `floor=0.25`:

```text
Post 1: 100%   Post 2: 62.5%   Post 3: 43.75%   Post 4: 34.4%   …→ 25% floor
```

**Formula:** `multiplier = 0.75 × 0.5^position + 0.25` (`AuthorDiversityDecay`/`AuthorDiversityFloor` in `params/param.rs`, applied in `ranking_scorer.rs`). Steeper than previously assumed — your second post loses over a third of its score.

---

### Q: What is OON penalty?

**A:** Out-of-network content (from accounts the viewer doesn't follow) is multiplied by **`OonWeightFactor=0.75`** (verified real value; `TopicOonWeightFactor=0.5` on topic surfaces). Since Aug 2026, **in-network replies and retweets take the same ×0.75** (`EnableOonRescoreForInNetworkRepliesRetweets=true`) — only original posts keep full score.

**Source:** `home-mixer/scorers/ranking_scorer.rs`, `home-mixer/params/param.rs`

---

### Q: Can my post be visible to followers but invisible to everyone else?

**A:** **Yes — verified mechanism.** The visibility-filtering engine has 28 shared rules + **26 rules that fire only on recommendations to non-followers** (spam-high-recall, NSFW, DMCA'd media, do-not-amplify labels, legal takedowns, account-state labels…). That's the "OON ceiling": followers see you, recommendations don't carry you. Replies/quotes of a dropped post get dropped too (`AncillaryVFFilter`). Check **Settings → Under the Hood** — since Sep 18, 2026 it shows visibility-impacting labels on your account, including legal-compliance withholding.

**Source:** `visibility-filtering/rules/registry.rs`, `under-the-hood/`

---

## Myth Busting

### Myth: Posting more = more reach

**Fact:** Author diversity penalty means your 2nd, 3rd posts get progressively lower scores. Quality > quantity.

---

### Myth: Likes are the most important metric

**Fact:** Verified — a like is 0.5 while a copy-link share is 20.0 (40×) and a reply 5.0 (10×). Likes are near the *bottom* of the positive hierarchy.

---

### Myth: The algorithm is completely random

**Fact:** It's a deterministic ML system based on predicted engagement. Understanding it helps.

---

### Myth: Buying followers helps reach

**Fact:** Followers who don't engage provide no positive signals. Low engagement rate may hurt your distribution.

---

### Myth: You can't recover from low reach

**Fact:** The algorithm is forward-looking. Consistent high-quality content will improve your signals over time.

---

## Still Have Questions?

Open an issue on this repo with the `question` label.

---

**Back to:** [Rules](../rules/) | [Checklists](../checklists/)
