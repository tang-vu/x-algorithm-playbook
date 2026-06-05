# Algorithm FAQ

> Frequently asked questions about the X algorithm, answered with code references.

---

## General Questions

### Q: Is the X algorithm really open source?

**A:** Yes. Twitter first open-sourced the recommendation code in April 2023, and **xAI shipped a major rewrite on May 15, 2026** ([github.com/xai-org/x-algorithm](https://github.com/xai-org/x-algorithm), Apache-2.0). The 2026 release is the primary source for this playbook: it includes a runnable end-to-end pipeline, the `grox` content-understanding service, candidate sourcing, and a **downloadable mini Phoenix model** (~2.8 GB via Git LFS) you can actually run. Core logic for scoring, filtering, and ranking is public; the exact production weights and the full-size model are still redacted.

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

**A:** Based on code analysis, **Reply** has the highest positive weight (~2× other actions), followed by Quote Tweet and Follow.

**Source:** `weighted_scorer.rs`

---

### Q: How bad is getting blocked?

**A:** Very bad. Block appears to have approximately -10× the weight of a like. One block can undo the positive signal from 10 likes.

**Source:** `weighted_scorer.rs` - `BLOCK_AUTHOR_WEIGHT`

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

### Q: What are the new out-of-network sources in the May 2026 update?

**A:** Out-of-network reach is no longer a single funnel. The 2026 release exposes several candidate sources: **Phoenix Retrieval** (generic similarity search), **Phoenix Topics** (topical discovery), **Phoenix MoE** (mixture-of-experts for specialized interests), **Who-to-Follow**, and **Ads/Prompts**. Practically: a post with a sharp, consistent topic can be surfaced by Topics/MoE even when generic retrieval would miss it. Topic clarity is now a multi-door reach lever.

**Source:** `candidate-pipeline/`, `phoenix/`

---

### Q: Does posting time matter?

**A:** Yes. The Age Filter removes posts older than a threshold. Posts need early engagement to be distributed widely. Posting when your audience is online is crucial.

**Source:** `age_filter.rs`

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
1. Count as ONE "author slot" (no diversity penalty within thread)
2. Higher dwell time (people read multiple tweets)
3. More engagement opportunities per thread

**Source:** Author diversity logic in `author_diversity_scorer.rs`

---

### Q: Does video get boosted?

**A:** Video has a dedicated score (Video Quality View), but ONLY if the video exceeds a minimum duration threshold. Short clips don't get this bonus.

**Source:** `weighted_scorer.rs` - `VQV_WEIGHT`, `MIN_VIDEO_DURATION_MS`

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

**A:** New in May 2026: `grox` is a dedicated content-understanding service that runs **classifiers and embedders** over every post during hydration — *before* scoring. It produces the topic labels, embeddings, and safety signals the rest of the pipeline reads. Key takeaway: there is **no manual keyword/hashtag boost** for relevance. `grox` decides what your post is *about*, and the model learns relevance from engagement. Clear, on-topic content embeds cleanly and matches the right audiences; vague content embeds noisily and reaches no one.

**Source:** `grox/`

---

### Q: How does the author diversity penalty work?

**A:** When multiple posts from the same author appear:

```
Post 1: 100% score
Post 2: ~76% score  
Post 3: ~59% score
...converges to ~20% floor
```

**Formula:** `multiplier = (1 - floor) × decay^position + floor`

**Source:** `author_diversity_scorer.rs`

---

### Q: What is OON penalty?

**A:** Out-of-network (OON) content (from accounts you don't follow) is multiplied by `OON_WEIGHT_FACTOR` (less than 1.0). This gives in-network content an advantage.

**Source:** `oon_scorer.rs`

---

## Myth Busting

### Myth: Posting more = more reach

**Fact:** Author diversity penalty means your 2nd, 3rd posts get progressively lower scores. Quality > quantity.

---

### Myth: Likes are the most important metric

**Fact:** Replies have ~2× the weight of likes in the scoring formula.

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
