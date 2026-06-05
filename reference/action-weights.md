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

- **Reply** and **Quote Tweet** have the highest weights (~2× others)
- **Follow Author** is high-intent signal (user wants more)
- **Video Quality View** only counts if video exceeds minimum duration
- **Dwell Time** is continuous (more time = more signal)

---

## Negative Actions

Actions that DECREASE your post's score:

| # | Action | Code Name | Description | Relative Weight |
|---|--------|-----------|-------------|-----------------|
| 16 | **Not Interested** | `not_interested` | User clicks "Not interested" | ❌ Negative |
| 17 | **Mute Author** | `mute_author` | User mutes you | ❌❌ Strong Negative |
| 18 | **Block Author** | `block_author` | User blocks you | ❌❌❌ Very Strong |
| 19 | **Report** | `report` | User reports post | ❌❌❌❌ Devastating |

### Estimated Negative Weights

Based on code analysis:

| Action | Estimated Impact |
|--------|------------------|
| Not Interested | -1× a like |
| Mute | -5× a like |
| Block | -10× a like |
| Report | -20× a like |

**Key insight:** One block undoes 10 likes. One report undoes 20 likes.

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
| Weight application (Weighted Scorer) | `home-mixer/scorers/` |
| Score extraction (Phoenix Scorer) | `home-mixer/scorers/` |
| 19 action heads | `phoenix/` model config |
| Predicted-action definitions | `xai_recsys_proto` |

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

MINIMIZE:
├── Report (❌❌❌❌ -20×)
├── Block (❌❌❌ -10×)
├── Mute (❌❌ -5×)
└── Not Interested (❌ -1×)
```

---

**Next:** [Filter System →](filter-system.md)
