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

## The ML Model: Phoenix

### How It Works

```
┌─────────────────────────────────────────────────────────────────┐
│                         PHOENIX SCORER                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  INPUT:                                                          │
│  ├── User embedding (who is viewing)                             │
│  ├── User history (128 recent engagements)                       │
│  └── Candidate posts (up to 32)                                  │
│                                                                  │
│  PROCESSING:                                                     │
│  └── Grok-based Transformer                                      │
│      ├── Attention mechanism                                     │
│      └── Candidate isolation (posts can't see each other)        │
│                                                                  │
│  OUTPUT:                                                         │
│  └── P(action) for each of 19 actions                            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

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

| Component | File |
|-----------|------|
| Weighted scoring | `home-mixer/scorers/weighted_scorer.rs` |
| Author diversity | `home-mixer/scorers/author_diversity_scorer.rs` |
| OON penalty | `home-mixer/scorers/oon_scorer.rs` |
| Phoenix ML | `home-mixer/scorers/phoenix_scorer.rs` |
| Grok transformer | `phoenix/grok.py` |
| Ranking model | `phoenix/recsys_model.py` |

---

**Next:** [Content Optimization →](02-content-optimization.md)
