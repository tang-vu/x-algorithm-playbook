# Action Weights Reference

> Complete reference for all 19 actions the algorithm predicts and weights.

---

## Overview

The X algorithm predicts the probability of 19 different user actions and combines them with weights:

```
Final Score = Σ (weight_i × P(action_i))
```

> ✅ The **19 action heads** are confirmed in the downloadable mini Phoenix model config (May 2026 release). The repo's developer notes name 15 of them explicitly: favorite, reply, repost, quote, click, profile click, video view, photo expand, share, dwell, follow author, not interested, block author, mute author, report.

---

## The Exact Weight Values Are Redacted

**Verified against `home-mixer/scorers/weighted_scorer.rs` (May 2026 release).** This is the most important thing to understand about "weights":

- The code multiplies each predicted probability by a per-action weight (`p::FAVORITE_WEIGHT`, `p::REPLY_WEIGHT`, `p::BLOCK_AUTHOR_WEIGHT`, …) and sums them.
- **Those weight *values* are NOT in the open-source repo.** They live in a `params` module that is **not published** — `home-mixer/lib.rs` declares 13 modules and `params` is not one of them; there is no `params.rs` and no build script. The code references the constants; the numbers are stripped.

**What this means:** every specific multiplier in this playbook (e.g. "reply ~2×", "block −10×", "report −20×") is an **illustrative estimate / relative ordering — NOT a value extracted from the code.** Treat them as direction, not as ground truth. Anyone claiming a precise ratio (e.g. "one reply = 150 likes") is inferring, not quoting the source.

### What the source DOES verify

```rust
// weighted_scorer.rs — compute_weighted_score() sums exactly 19 terms:
combined =  favorite·FAVORITE_W  + reply·REPLY_W       + retweet·RETWEET_W
          + photo_expand·…       + click·…             + profile_click·…
          + vqv·vqv_weight       + share·…             + share_via_dm·…
          + share_via_copy_link·… + dwell·…            + quote·QUOTE_W
          + quoted_click·…       + dwell_time·CONT_DWELL_TIME_W   // continuous
          + follow_author·…
          + not_interested·NOT_INTERESTED_W            // negative
          + block_author·BLOCK_AUTHOR_W                // negative
          + mute_author·MUTE_AUTHOR_W                  // negative
          + report·REPORT_W;                           // negative

// apply(score, weight) = score.unwrap_or(0.0) * weight  → missing predictions count as 0
```

Verified facts (no numbers needed):

- **19 weighted terms**, with `favorite, reply, retweet, quote, share, share_via_dm, share_via_copy_link, click, quoted_click, profile_click, vqv, photo_expand, dwell, dwell_time, follow_author` positive and **`not_interested, block_author, mute_author, report` negative**.
- **Video (`vqv`) only gets weight if `video_duration_ms > MIN_VIDEO_DURATION_MS`** — otherwise its weight is `0.0`. (The duration threshold value is also redacted.)
- **`dwell_time` is continuous** (uses `CONT_DWELL_TIME_WEIGHT`), separate from the binary `dwell`.
- After summing, an **offset/normalization** is applied (`offset_score()` + `normalize_score()`): negative-dominant scores are rescaled via `NEGATIVE_WEIGHTS_SUM / WEIGHTS_SUM` and shifted by `NEGATIVE_SCORES_OFFSET`, so the final score stays well-behaved.

---

## Positive Actions

Actions that INCREASE your post's score:

| # | Action | Code Name | Description | Relative Weight |
|---|--------|-----------|-------------|-----------------|
| 1 | **Like** | `favorite` | User likes the post | ⭐⭐ Medium |
| 2 | **Reply** | `reply` | User replies to post | ⭐⭐⭐ High |
| 3 | **Retweet** | `retweet` | User retweets | ⭐⭐ Medium |
| 4 | **Quote Tweet** | `quote` | User quote tweets | ⭐⭐⭐ High |
| 5 | **Share** | `share` | User shares post | ⭐ Low-Medium |
| 6 | **Share via DM** | `share_via_dm` | User shares via direct message | ⭐ Low |
| 7 | **Share via Link** | `share_via_copy_link` | User copies link | ⭐ Low |
| 8 | **Click** | `click` | User clicks on post | ⭐ Low |
| 9 | **Profile Click** | `profile_click` | User clicks author's profile | ⭐⭐ Medium |
| 10 | **Follow Author** | `follow_author` | User follows after seeing post | ⭐⭐⭐ High |
| 11 | **Photo Expand** | `photo_expand` | User expands photo | ⭐ Low |
| 12 | **Video View** | `video_quality_view` | User watches video (quality) | ⭐⭐ Medium |
| 13 | **Dwell** | `dwell` | User stops to read (binary) | ⭐ Low |
| 14 | **Dwell Time** | `dwell_time` | Time spent reading (continuous) | ⭐ Variable |
| 15 | **Quoted Click** | `quoted_click` | User clicks quoted content | ⭐ Low |

### Notes on Positive Actions

- **Reply** and **Quote Tweet** are widely treated as the highest-value positive signals (the "~2×" is an estimate, not a code value)
- **Follow Author** is high-intent signal (user wants more)
- **Video Quality View** only counts if video exceeds minimum duration — **verified** (`vqv_weight = 0` below `MIN_VIDEO_DURATION_MS`)
- **Dwell Time** is continuous (`CONT_DWELL_TIME_WEIGHT`) — **verified**

---

## Negative Actions

Actions that DECREASE your post's score:

| # | Action | Code Name | Description | Relative Weight |
|---|--------|-----------|-------------|-----------------|
| 16 | **Not Interested** | `not_interested` | User clicks "Not interested" | ❌ Negative |
| 17 | **Mute Author** | `mute_author` | User mutes you | ❌❌ Strong Negative |
| 18 | **Block Author** | `block_author` | User blocks you | ❌❌❌ Very Strong |
| 19 | **Report** | `report` | User reports post | ❌❌❌❌ Devastating |

### Negative Weights — Illustrative Only

> ⚠️ **These numbers are NOT from the code** (the values are redacted — see above). They're a widely-cited *relative* intuition for teaching purposes. What the source guarantees is only that these four actions carry **negative** weight and that report/block are treated as the most severe.

| Action | Illustrative relative impact | Verified from source? |
|--------|------------------------------|-----------------------|
| Not Interested | ≈ −1× a like | Sign only (negative) |
| Mute | ≈ −5× a like | Sign only (negative) |
| Block | ≈ −10× a like | Sign only (negative) |
| Report | ≈ −20× a like | Sign only (negative) |

**Takeaway (safe to rely on):** negative actions actively subtract from your score, and the algorithm separates positive and negative sums with an offset — so a few blocks/reports can erase a lot of positive signal. The *exact* ratio is unknown.

---

## Weight Categories

### Tier 1: Conversation Signals (Highest Value)
```
reply, quote, follow_author
```
These indicate deep engagement and intent.

### Tier 2: Amplification Signals (High Value)
```
retweet, share, share_via_dm
```
User willing to put your content in front of others.

### Tier 3: Interest Signals (Medium Value)
```
favorite, profile_click, video_quality_view
```
User is interested but passive.

### Tier 4: Attention Signals (Low Value)
```
click, photo_expand, dwell, quoted_click
```
Basic attention metrics.

### Tier -1: Negative Signals (Subtract)
```
not_interested, mute, block, report
```
Actively hurts your score.

---

## Special Cases

### Video Quality View (VQV)

```rust
// Only applies if video exceeds minimum duration
if video_duration_ms > MIN_VIDEO_DURATION_MS {
    apply(vqv_score, VQV_WEIGHT)
} else {
    // VQV weight = 0, not counted
}
```

**Implication:** Short video clips don't get the VQV bonus.

### Dwell Time (Continuous)

Unlike other binary actions, dwell time is continuous:

```
dwell_score: Binary (did user stop?)
dwell_time: Continuous (how long in seconds?)
```

Both are weighted and contribute to score.

---

## Algorithm Source Files

| Component | Where |
|-----------|-------|
| Weight application (19-term sum + offset) | `home-mixer/scorers/weighted_scorer.rs` |
| Score extraction (Phoenix Scorer) | `home-mixer/scorers/phoenix_scorer.rs` |
| 19 action heads | `phoenix/` model config |
| **Weight VALUES (`*_WEIGHT` constants)** | **`params` module — redacted / not published** |

---

## Strategic Implications

### Optimize For (Priority Order)

1. **Replies** - Ask questions, create debate
2. **Quote Tweets** - Create quotable insights
3. **Follows** - Provide unique ongoing value
4. **Retweets** - Make shareable content
5. **Likes** - Be generally engaging

### Avoid At All Costs

1. **Reports** - Stay within ToS
2. **Blocks** - Don't annoy or spam
3. **Mutes** - Don't over-post or be off-brand
4. **"Not Interested"** - Stay relevant to audience

---

## Quick Reference Card

```
MAXIMIZE:
├── Reply (⭐⭐⭐)
├── Quote (⭐⭐⭐)
├── Follow (⭐⭐⭐)
├── Retweet (⭐⭐)
├── Like (⭐⭐)
└── Video View (⭐⭐) [if > min duration]

MINIMIZE (severity order — exact weights redacted):
├── Report (❌❌❌❌ most severe)
├── Block (❌❌❌)
├── Mute (❌❌)
└── Not Interested (❌)
```

---

**Next:** [Filter System →](filter-system.md)
