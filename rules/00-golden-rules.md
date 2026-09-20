# The 10 Golden Rules of X Algorithm

> These are the most important rules distilled from the X algorithm source code — **updated with the real published weights** (Aug–Sep 2026 releases). Master these and you'll outperform 95% of accounts.

---

## Rule 1: Get Sent, Get Replies 👑

**Why:** The two biggest levers are now verified numbers: **share-via-copy-link = 20.0** (someone copying your post's URL to send it elsewhere — the single highest weight) and **reply = 5.0**, which becomes **20.0** on original posts to *mutual follows* thanks to the July 2026 bidirectional boost. Write content people *forward* and *answer*.

**Do:**

- End posts with questions
- Write "send this to someone who…" content people copy-link into chats
- Make controversial (but not offensive) takes
- Ask for opinions and experiences

**Don't:**

- Post statements with no engagement hook
- Make content that requires no response
- Optimize for likes — a like is worth 1/40th of a copy-link share

**Algorithm source:** `ranking_scorer.rs` + `params/param.rs` — `ShareViaCopyLinkWeight=20.0`, `ReplyWeight=5.0`, `BidirectionalFollowReplyWeightBoost=15.0`

---

## Rule 2: Avoid Negative Actions At All Costs ⚠️

**Why:** Negative actions carry strong **negative** weight and subtract from your score. The real order (by weight) is **report (−234) ≫ mute (−58.8) > not-interested (−43.2) > block (−31.2) > not-dwelled (−0.02)** — mutes and "not interested" are *harsher* than blocks, the opposite of the old assumption. Caveat: these multiply *predicted probabilities*, so a single report doesn't mechanically erase N likes.

**Do:**

- Stay in your niche
- Be authentic and helpful
- Avoid rage-bait that triggers blocks

**Don't:**

- Spam replies on unrelated posts
- Be overly political or divisive
- Post content that might get reported

**Algorithm source:** `params/param.rs` — `ReportWeight=−234.0`, `MuteAuthorWeight=−58.8`, `NotInterestedWeight=−43.2`, `BlockAuthorWeight=−31.2`

---

## Rule 3: Space Your Posts (Author Diversity Penalty) ⏰

**Why:** Within a single feed response, the algorithm decays your extra posts from the same author by score-rank, toward a floor. Real values: `decay=0.5`, `floor=0.25` → `multiplier = 0.75×0.5^k + 0.25`:

- Your top post: 100%
- 2nd: 62.5% · 3rd: 43.75% · 4th: 34.4% · …→ 25% floor

**Do:**

- Post 3-4 hours apart (fewer of your posts compete in the same response)
- Use threads instead of separate posts (one published post = one author-position)
- Quality over quantity

**Don't:**

- Post 5 tweets in an hour
- Think more posts = more reach

**Algorithm source:** `ranking_scorer.rs` — `AuthorDiversityDecay=0.5`, `AuthorDiversityFloor=0.25`

---

## Rule 4: In-Network First 🏠

**Why:** Out-of-network content is multiplied by **0.75** (`OonWeightFactor` — real value; 0.5 on topic surfaces). And since Aug 2026, even **in-network replies and retweets** take the same ×0.75 (`EnableOonRescoreForInNetworkRepliesRetweets=true`) — only *original posts* keep full score. Your followers see you first, and your original posts see them best.

**Do:**

- Focus on building genuine followers
- Engage with your existing audience
- Create content your followers want to share

**Don't:**

- Buy followers (they don't engage = negative signal)
- Ignore your existing audience chasing new ones

**Algorithm source:** `ranking_scorer.rs` + `params/param.rs` — `OonWeightFactor=0.75`, `TopicOonWeightFactor=0.5`

---

## Rule 5: Video > Image > Text (With Caveats) 🎬

**Why:** Verified minimum video duration is exactly **10 seconds** (`MinVideoDurationMs=10_000`). But note the current weights: `VqvWeight=0.0` (the video-quality-view bonus is dormant) — the live video signal is `VideoOpenWeight=0.07`. Video still helps via dwell time and engagement, not via a big dedicated weight.

**Do:**

- Create videos longer than 10 seconds
- Add captions (silent scrolling)
- Hook in first 3 seconds
- Native upload (not YouTube links)

**Don't:**

- Post super short clips that don't qualify
- Assume video alone wins — shares/replies still dominate
- Post without thumbnails

**Algorithm source:** `params/param.rs` — `MinVideoDurationMs=10_000`, `VqvWeight=0.0`, `VideoOpenWeight=0.07`

---

## Rule 6: Dwell Time Is a Signal ⏱️

**Why:** The algorithm tracks both binary dwell (did they stop?) and continuous dwell time (how long?).

**Do:**

- Write longer, engaging content
- Use threads for complex topics
- Format for readability (spacing, bullets, emojis)
- Create content worth re-reading

**Don't:**

- Post one-liners only
- Create skimmable-only content

**Algorithm source:** `phoenix_scorer.rs` + `params/param.rs` — `DwellWeight=0.05`, `ContDwellTimeWeight=0.004`

---

## Rule 7: Don't Get Filtered 🚫

**Why:** Two filtering layers can COMPLETELY HIDE your content: **17 pre-scoring pipeline filters** (incl. a hard 48-hour age limit) plus the **visibility-filtering rule engine** — 28 rules that apply to everyone and **26 extra rules that only fire on recommendations to non-followers** (spam-high-recall, NSFW, DMCA media, do-not-amplify labels…). Your post can look fine to followers while being invisible to everyone else.

**Filters that can block you:**

1. Age filter (>48h — hard cutoff)
2. Duplicate/self/dedup filters
3. Muted keyword filter
4. Blocked/muted author filter
5. Visibility rules: spam, NSFW, legal takedowns, account labels
6. Previously seen/served filters
7. AncillaryVF — your reply dies if the parent post was dropped

**Do:**

- Post at peak times for freshness
- Avoid spammy keywords ("DM me", "link in bio" spam)
- Don't post the same thing twice

**Don't:**

- Use flagged keywords
- Try to game the system with duplicate content

**Algorithm source:** `home-mixer/filters/` + `visibility-filtering/rules/registry.rs` — [full reference](../reference/filter-system.md)

---

## Rule 8: Engage Authentically 🤝

**Why:** The algorithm uses your engagement history to build your "user embedding." If you engage in your niche, you'll be matched with similar content creators and audiences.

**Do:**

- Like/reply to posts in your niche
- Build relationships with similar creators
- Be genuinely helpful in replies

**Don't:**

- Mass-like random content
- Use engagement pods (pattern detected)
- Ignore your community

**Algorithm source:** `phoenix/recsys_retrieval_model.py` - Two-Tower model

---

## Rule 9: Niche Down for Retrieval 🎯

**Why:** Retrieval matches user embeddings to post embeddings — and since Aug 2026 posts also carry **semantic IDs** (quantized codes from the multimodal embedding): same-topic posts literally share ID prefixes, so a crisp topic IS your post's identity in the index. `grox` classifies/embeds every post at publish time, and **SimClusters** adds a second OON door that groups you with accounts/posts by engagement overlap. All doors read the same signal: topical consistency.

**Do:**

- Focus on 1-3 related topics
- Be unambiguous about what each post is about (clear language, on-topic media)
- Build topical authority

**Don't:**

- Post random content across many topics
- Bury the topic in vague phrasing the classifier can't read
- Confuse your audience (and the algorithm)

**Algorithm source:** `phoenix/` (two-tower retrieval + semantic IDs) · `grox/` (content understanding) · `simclusters/` (engagement clusters)

---

## Rule 10: Quality > Quantity 📊

**Why:** All the rules above compound. One viral post beats 10 mediocre ones because:

- No author diversity penalty
- More replies per post
- Higher dwell time
- More authentic engagement

**Do:**

- Spend time crafting content
- Study what works
- Iterate and improve

**Don't:**

- Spray and pray
- Post just to post

---

## Quick Reference Card

| Rule | Key Metric | Action |
|------|------------|--------|
| 1 | Shares (20.0) + replies (5.0) | Write forwardable, debatable posts |
| 2 | Report −234 / mute −58.8 | Stay authentic, don't annoy |
| 3 | Post frequency (×0.625 on #2) | 3-4 hr spacing |
| 4 | OON ×0.75 | Build genuine following, post originals |
| 5 | Media type | Native video > 10s (small direct weight) |
| 6 | Dwell time (0.004 cont.) | Longer content |
| 7 | Filters + VF rules | Avoid spam patterns, check Under the Hood |
| 8 | Engagement | Interact in niche |
| 9 | Topic focus (SIDs + SimClusters) | Niche down |
| 10 | Quality | Fewer, better posts |

---

**Next:** [Understanding the Scoring System →](01-scoring-system.md)
