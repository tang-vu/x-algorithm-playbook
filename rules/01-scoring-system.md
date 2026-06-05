# The X Scoring System Explained

> How your posts are ranked in the For You feed, based on the actual algorithm code.

---

## The Scoring Formula

Every post is scored using this formula:

```
Final Score = Σ (weight_i × P(action_i))
```

Where:
- `P(action_i)` = Probability of user taking action (predicted by ML model)
- `weight_i` = Importance weight for that action

**Example:**
```
Score = (0.3 × P(like)) + (0.6 × P(reply)) + (0.4 × P(retweet)) + ...
                         - (0.5 × P(block)) - (1.0 × P(report))
```

---

## The 19 Predicted Actions

The Phoenix ML model predicts probability for each action:

### Positive Actions (Increase Score)

| Action | Description | Relative Weight |
|--------|-------------|-----------------|
| `favorite` | Like the post | ⭐⭐ |
| `reply` | Reply to the post | ⭐⭐⭐ (Highest) |
| `retweet` | Retweet | ⭐⭐ |
| `quote` | Quote tweet | ⭐⭐⭐ |
| `share` | General share | ⭐ |
| `share_via_dm` | Share via DM | ⭐ |
| `share_via_copy_link` | Copy link | ⭐ |
| `click` | Click on post | ⭐ |
| `profile_click` | Click author's profile | ⭐⭐ |
| `follow_author` | Follow after seeing post | ⭐⭐⭐ |
| `photo_expand` | Expand photo | ⭐ |
| `video_quality_view` | Watch video (quality) | ⭐⭐ |
| `dwell` | Stop and read | ⭐ |
| `dwell_time` | Time spent reading | ⭐ (continuous) |
| `quoted_click` | Click on quoted content | ⭐ |

### Negative Actions (Decrease Score)

| Action | Description | Relative Weight |
|--------|-------------|-----------------|
| `not_interested` | "Not interested" button | ❌ |
| `mute_author` | Mute the author | ❌❌ |
| `block_author` | Block the author | ❌❌❌ |
| `report` | Report the post | ❌❌❌❌ (Devastating) |

---

## The 7-Stage Pipeline

Before scoring even happens, your post flows through a fixed pipeline (orchestrated by `home-mixer`):

```
1. Query Hydration      → assemble the viewer's recent engagement history
2. Candidate Sourcing   → pull posts from every source (below)
3. Candidate Hydration  → enrich with author, media, content-understanding data
4. Pre-Scoring Filters  → drop ineligible posts (10 filters)
5. Scoring              → run the sequential scorers (below)
6. Selection            → sort by score, take top-K
7. Post-Selection       → final safety + dedup pass (2 filters)
```

---

## Where Candidates Come From

Your post can enter a feed through several sources — not just the two-tower retrieval:

| Source | Network | What it is |
|--------|---------|------------|
| **Thunder** | In-network | Realtime in-memory store of posts from accounts the viewer follows |
| **Phoenix Retrieval** | Out-of-network | Two-tower ANN similarity search (millions → hundreds) |
| **Phoenix Topics** | Out-of-network | Topical discovery — posts matched to topics the viewer engages with |
| **Phoenix MoE** | Out-of-network | Mixture-of-experts retrieval for specialized interest matching |
| **Who-to-Follow** | Out-of-network | Author/account suggestions |
| **Ads / Prompts** | Mixed | Promoted + system content |

**Why this matters:** out-of-network reach is no longer one funnel. **Phoenix Topics** and **Phoenix MoE** are distinct doors — a post with a crisp, consistent topic can be picked up by Topics even when generic retrieval misses it. (See [Growth Strategies](06-growth-strategies.md#may-2026-reach-paths).)

---

## How Content Is Understood: `grox`

New in May 2026: a dedicated **content-understanding service** (`grox`) runs **classifiers and embedders** over every post during hydration — *before* scoring. It turns raw text/media into the topic signals, embeddings, and safety labels the rest of the system reads.

**Strategic consequence:** there is **no manual keyword/hashtag feature engineering** for relevance. The model learns relevance from engagement sequences, and `grox` decides what your post is *about*. Clear, on-topic, well-understood content embeds cleanly and matches the right audiences; vague or off-niche content embeds noisily and matches poorly. (See [Content Optimization](02-content-optimization.md#content-understanding-grox).)

---

## The ML Model: Phoenix

### How It Works

```
┌─────────────────────────────────────────────────────────────────┐
│                         PHOENIX SCORER                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  INPUT:                                                          │
│  ├── User embedding (who is viewing)                             │
│  ├── User history (up to 127 recent engagements)                 │
│  └── Candidate posts (up to 64 per batch)                        │
│                                                                  │
│  PROCESSING:                                                     │
│  └── Grok-based Transformer (ported from Grok-1)                 │
│      ├── Attention mechanism                                     │
│      └── Candidate isolation (posts can't see each other)        │
│                                                                  │
│  OUTPUT:                                                         │
│  └── P(action) for each of 19 actions                            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Released Model Architecture (mini config)

The May 2026 update ships a runnable, downloadable Phoenix model (~2.8 GB via Git LFS) so the architecture is no longer a guess:

| Parameter | Value |
|-----------|-------|
| Embedding dimension | 128 |
| Transformer layers | 4 |
| Attention heads | 4 |
| Key size | 32 |
| Widening factor | ×2 |
| History sequence length | 127 |
| Candidate sequence length | 64 |
| User / Item / Author vocab | 1,000,000 each |
| Predicted action heads | 19 |

> The transformer is **ported from xAI's open-source Grok-1** and adapted for recommendations. Production uses a larger config, but the mechanics are identical. Run it yourself with `phoenix/run_pipeline.py` (retrieval → ranking in one entry point).

### Candidate Isolation

Key insight: **Your post's score doesn't depend on what other posts are in the batch.**

This means:
- Scores are consistent and cacheable
- You can't "hide" behind other posts
- Each post is judged independently

---

## The Scoring Pipeline

### Step 1: Weighted Scorer

```rust
// From weighted_scorer.rs
combined_score = 
    apply(favorite_score, FAVORITE_WEIGHT)
  + apply(reply_score, REPLY_WEIGHT)        // ~2× other weights
  + apply(retweet_score, RETWEET_WEIGHT)
  + apply(quote_score, QUOTE_WEIGHT)
  + apply(share_score, SHARE_WEIGHT)
  + apply(click_score, CLICK_WEIGHT)
  + apply(profile_click_score, PROFILE_CLICK_WEIGHT)
  + apply(vqv_score, vqv_weight)            // Only if video > min duration
  + apply(dwell_score, DWELL_WEIGHT)
  + apply(dwell_time, CONT_DWELL_TIME_WEIGHT)
  + apply(follow_author_score, FOLLOW_AUTHOR_WEIGHT)
  // Negative weights
  + apply(not_interested_score, NOT_INTERESTED_WEIGHT)  // Negative
  + apply(block_author_score, BLOCK_AUTHOR_WEIGHT)      // ~-10×
  + apply(mute_author_score, MUTE_AUTHOR_WEIGHT)        // ~-5×
  + apply(report_score, REPORT_WEIGHT)                  // ~-20×
```

### Step 2: Author Diversity Scorer

Penalizes multiple posts from same author in one feed:

```rust
// From author_diversity_scorer.rs
multiplier = (1 - floor) × decay^position + floor

// Example with decay=0.8, floor=0.2:
// Post 1: 1.0 (100%)
// Post 2: 0.84 (84%)
// Post 3: 0.67 (67%)
// Post 4: 0.54 (54%)
// ...converges to 0.2 (20%)
```

### Step 3: OON Scorer

Penalizes out-of-network content:

```rust
// From oon_scorer.rs
if !in_network {
    score *= OON_WEIGHT_FACTOR  // Less than 1.0
}
```

---

## Practical Implications

### What This Means for You

| Algorithm Behavior | Your Strategy |
|-------------------|---------------|
| Replies weighted highest | Create discussion-starting content |
| Blocks are -10× likes | Avoid controversial content that triggers blocks |
| Author diversity penalty | Space posts, use threads |
| OON penalty | Build quality followers first |
| Video needs min duration | Make videos 10+ seconds |
| Dwell time tracked | Create longer, engaging content |

### The Score Optimization Hierarchy

```
1. Maximize P(reply)     → Ask questions, create debate
2. Maximize P(quote)     → Create quotable, shareable insights
3. Maximize P(follow)    → Provide unique value
4. Maximize P(retweet)   → Make content worth sharing
5. Maximize P(like)      → Be likeable (baseline)
6. Minimize P(block)     → Don't annoy people
7. Minimize P(report)    → Stay within guidelines
```

---

## Source Files

| Component | Where |
|-----------|-------|
| Feed orchestration (pipeline) | `home-mixer/` |
| Weighted / diversity / OON scorers | `home-mixer/scorers/` |
| Phoenix ML (retrieval + ranking) | `phoenix/` |
| Grok transformer | `phoenix/` (ported from Grok-1) |
| Unified inference entry point | `phoenix/run_pipeline.py` |
| Content understanding (classifiers + embedders) | `grox/` |
| Realtime in-network post store | `thunder/` |
| Reusable pipeline framework | `candidate-pipeline/` |

---

**Next:** [Content Optimization →](02-content-optimization.md)
