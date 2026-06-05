# The 10 Golden Rules of X Algorithm

> These are the most important rules distilled from the X algorithm source code. Master these and you'll outperform 95% of accounts.

---

## Rule 1: Replies Are King 👑

**Why:** Reply is treated as the highest-value positive signal in the scoring formula. (The exact multiplier is **redacted** — the "~2×" quoted around the web is an estimate, not a value from the code.)

**Do:**
- End posts with questions
- Make controversial (but not offensive) takes
- Create "fill in the blank" posts
- Ask for opinions and experiences

**Don't:**
- Post statements with no engagement hook
- Make content that requires no response

**Algorithm source:** `weighted_scorer.rs` - `REPLY_WEIGHT`

---

## Rule 2: Avoid Negative Actions At All Costs ⚠️

**Why:** Negative actions carry strong **negative** weight and actively subtract from your score — the scorer sums negative signals separately and offsets your score downward (verified). The commonly-assumed severity order is **report > block > mute > not-interested**; the exact multipliers are redacted, but a few of these can erase a lot of positive signal.

**Do:**
- Stay in your niche
- Be authentic and helpful
- Avoid rage-bait that triggers blocks

**Don't:**
- Spam replies on unrelated posts
- Be overly political or divisive
- Post content that might get reported

**Algorithm source:** `weighted_scorer.rs` - `BLOCK_AUTHOR_WEIGHT`, `REPORT_WEIGHT`

---

## Rule 3: Space Your Posts (Author Diversity Penalty) ⏰

**Why:** Within a single feed response, the algorithm decays your extra posts from the same author by score-rank, toward a floor (verified formula: `(1−floor)×decay^position + floor`):
- Your top post: 100%
- next: ~76% · next: ~59% · …→ ~20% floor

> Numbers are illustrative (assume decay≈0.7, floor≈0.2) — the real `decay`/`floor` are redacted from the source.

**Do:**
- Post 3-4 hours apart (fewer of your posts compete in the same response)
- Use threads instead of separate posts (one published post = one author-position)
- Quality over quantity

**Don't:**
- Post 5 tweets in an hour
- Think more posts = more reach

**Algorithm source:** `author_diversity_scorer.rs`

---

## Rule 4: In-Network First 🏠

**Why:** Out-of-network content is multiplied by `OON_WEIGHT_FACTOR` (a penalty factor; exact value redacted, understood < 1.0). Your followers see you first.

**Do:**
- Focus on building genuine followers
- Engage with your existing audience
- Create content your followers want to share

**Don't:**
- Buy followers (they don't engage = negative signal)
- Ignore your existing audience chasing new ones

**Algorithm source:** `oon_scorer.rs`

---

## Rule 5: Video > Image > Text (With Caveats) 🎬

**Why:** Video Quality View (VQV) gets its own weight bonus, BUT only if video exceeds minimum duration threshold.

**Do:**
- Create videos longer than ~10 seconds
- Add captions (silent scrolling)
- Hook in first 3 seconds
- Native upload (not YouTube links)

**Don't:**
- Post super short clips that don't qualify
- Post without thumbnails

**Algorithm source:** `weighted_scorer.rs` - `VQV_WEIGHT`, `MIN_VIDEO_DURATION_MS`

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

**Algorithm source:** `phoenix_scorer.rs` - `DWELL_WEIGHT`, `CONT_DWELL_TIME_WEIGHT`

---

## Rule 7: Don't Get Filtered 🚫

**Why:** 12 different filters can COMPLETELY HIDE your content before scoring even happens.

**Filters that can block you:**
1. Age filter (too old)
2. Duplicate filter
3. Self-tweet filter
4. Muted keyword filter
5. Blocked/muted author filter
6. Spam filter (VF)
7. Previously seen filter
8. Subscription eligibility filter

**Do:**
- Post at peak times for freshness
- Avoid spammy keywords ("DM me", "link in bio" spam)
- Don't post the same thing twice

**Don't:**
- Use flagged keywords
- Try to game the system with duplicate content

**Algorithm source:** `home-mixer/filters/`

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

**Why:** The Two-Tower retrieval system matches user embeddings to content embeddings via dot product similarity. Clear, consistent topics = stronger embedding signal. As of May 2026, a content-understanding service (`grox`) classifies and embeds every post **before** scoring — so how cleanly your post is *understood* directly decides which audiences it can reach. Crisp topics also open the new **Phoenix Topics** retrieval door.

**Do:**
- Focus on 1-3 related topics
- Be unambiguous about what each post is about (clear language, on-topic media)
- Build topical authority

**Don't:**
- Post random content across many topics
- Bury the topic in vague phrasing the classifier can't read
- Confuse your audience (and the algorithm)

**Algorithm source:** `phoenix/` (two-tower retrieval) · `grox/` (content understanding)

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
| 1 | Replies | Ask questions |
| 2 | Blocks/Reports | Stay authentic |
| 3 | Post frequency | 3-4 hr spacing |
| 4 | Follower quality | Build genuine following |
| 5 | Media type | Native video > 10s |
| 6 | Dwell time | Longer content |
| 7 | Filters | Avoid spam patterns |
| 8 | Engagement | Interact in niche |
| 9 | Topic focus | Niche down |
| 10 | Quality | Fewer, better posts |

---

**Next:** [Understanding the Scoring System →](01-scoring-system.md)
