---
description: "Complete table of the X algorithm's real published action weights — positive, negative, and conditional boosts with source params."
---

# Action Weights Reference

> Complete reference for the actions the algorithm predicts and the **real, published weights** it applies to them.
>
> *Last verified: **September 20, 2026** against [xai-org/x-algorithm](https://github.com/xai-org/x-algorithm) @ [`8b25829`](https://github.com/xai-org/x-algorithm/commit/8b25829717a4f104dd04403ee7d0253c5fedb1b7) (Sep 18, 2026 release).*

---

## Overview

The Phoenix model predicts a probability for each action a viewer might take on a post. `RankingScorer` (`home-mixer/scorers/ranking_scorer.rs`) folds those predictions into one number:

```text
Final Score = Σ (weight_i × P(action_i))
```

> ✅ **As of the August 13, 2026 release, the actual weight values are public** in [`home-mixer/params/param.rs`](https://github.com/xai-org/x-algorithm/blob/main/home-mixer/params/param.rs) (defaults synced from production; last sync stamp 2026-09-18). Every number on this page is a real code value unless marked otherwise.

---

## The #1 Misconception: Weights Scale Probabilities, Not Counts

Before the table — the caveat the xAI team themselves added to the code (Aug 14, 2026 dev notes + comments in `param.rs`/`ranking_scorer.rs`):

- Each weight multiplies **the model's predicted probability that *this viewer* takes the action** (or a continuous value like dwell time). It does **not** multiply raw engagement counts.
- So the correct reading of "report = −234 vs like = 0.5" is **not** "1 report cancels 468 likes". `P(report)` is >1000× rarer than `P(like)` at baseline — the big weight exists so a rare, high-signal prediction can move the ranking at all.
- Corollary: **coordinated mass-reporting barely works.** Predictions are personalized — bad actors' reports mostly shift the score for *similar* users. And an action only counts if it happens on a post **served in the Home Timeline** — navigating directly to a post (e.g. via a group chat link) has no ranking impact.

What the weights *do* tell you: the relative value of a *predicted* action, and the sign.

---

## Positive Weights (real values)

| Action | Code / param | Weight | Notes |
|--------|--------------|--------|-------|
| **Share via copy link** | `ShareViaCopyLinkWeight` | **20.0** | 🏆 Highest single weight — someone copying your post's URL to send it elsewhere |
| **Reply** | `ReplyWeight` | **5.0** | **20.0** for mutual follows' original posts (see boost below) |
| **Share via DM** | `ShareViaDmWeight` | **5.0** | Private sharing counts heavy |
| **Quote** | `QuoteWeight` | **5.0** | Same tier as reply |
| **Follow author** | `FollowAuthorWeight` | **4.0** | High-intent signal |
| **Share** | `ShareWeight` | **2.0** | Generic share |
| **Retweet** | `RetweetWeight` | **1.0** | 2× a like, far below a reply |
| **Like** | `FavoriteWeight` | **0.5** | Baseline positive |
| **Click (post)** | `ClickWeight` | **0.4** | Expanding/clicking the post |
| **Open link** | `OpenLinkWeight` | **0.2** | Clicking a link in the post |
| **Video open** | `VideoOpenWeight` | **0.07** | Opening the video player |
| **Photo expand** | `PhotoExpandWeight` | **0.05** | |
| **Dwell (binary)** | `DwellWeight` | **0.05** | Stopped scrolling |
| **Quoted click** | `QuotedClickWeight` | **0.05** | Clicking the quoted post |
| **Post unexplored** | `PostUnexploredWeight` | **0.02** | In-network only by default (`PostUnexploredWeightInNetworkOnly=true`) |
| **Dwell time (continuous)** | `ContDwellTimeWeight` | **0.004** | Per-unit continuous — longer reads keep adding |
| **Profile click** | `ProfileClickWeight` | **0.0** | Tracked, currently unweighted |
| **VQV (video quality view)** | `VqvWeight` | **0.0** | Gated by 10s minimum duration, and currently weight 0 anyway |
| **Quoted VQV** | `QuotedVqvWeight` | **0.0** | Same gate |
| **Click dwell time** | `ContClickDwellTimeWeight` | **0.0** | Currently off |
| **Active seconds (5m residual)** | `ContActiveSecs5mResidualNormWeight` | **0.0** | Currently off |

## Negative Weights (real values)

| Action | Code / param | Weight | Rank |
|--------|--------------|--------|------|
| **Report** | `ReportWeight` | **−234.0** | Most severe |
| **Mute author** | `MuteAuthorWeight` | **−58.8** | 2nd — *harsher than block* |
| **"Not interested"** | `NotInterestedWeight` | **−43.2** | 3rd — *also harsher than block* |
| **Block author** | `BlockAuthorWeight` | **−31.2** | Mildest negative per-weight |
| **Not dwelled** | `NotDwelledWeight` | **−0.02** | Tiny tax for being scrolled past |

> ⚠️ **The old intuition was wrong about ordering.** Severity by weight is **report ≫ mute > not-interested > block**. Mutes and "not interested" votes outweigh blocks — consistent with their job of catching "this content annoyed me" signals, not just trust violations. Still remember the caveat: these multiply *predicted probabilities*, which for negatives are tiny to begin with.

---

## The Bidirectional Follow Boost (mutuals)

Shipped July 2026 (A/B from Jul 10; broad launch Jul 13 at +20; tuned to **+15** on Jul 24):

```rust
// ranking_scorer.rs — reply_weight_for()
// Original post (not reply, not retweet) AND author is a mutual follow:
reply_weight = 5.0 + 15.0 = 20.0
```

| Param | Value |
|-------|-------|
| `BidirectionalFollowReplyWeightBoost` | **15.0** |
| `BidirectionalFollowDwellWeightBoost` | 0.0 (tested, not shipped) |
| `EnableBidirectionalFollowHydration` | true |

**Meaning:** for people you *mutually* follow, your original posts get a reply term worth 4× the normal reply weight — mutuals are the single most boosted relationship in the scorer. ([Official diff walkthrough](https://github.com/xai-org/x-algorithm/blob/main/docs/BIDIRECTIONAL_BOOST_CHANGE.md))

---

## After the Weighted Sum

`RankingScorer` then applies, in order (default path):

### 1. Score offset

`offset_score(pos − neg)`: positive scores get shifted by `NEGATIVE_SCORES_OFFSET`; negative-dominant scores are rescaled by `negative_sum / total_sum` first — so heavy-negative posts compress instead of going infinitely negative.

### 2. New-author (cold start) boost — `author_cold_start.rs`

A genuine boost for small accounts. Eligible = **original post** (not reply/RT) from an author with **≤ 1,000 followers**, post **≤ 48h** old, **< 1,000 Home views so far**, and ranked inside the top 85% of candidates. The best eligible post is lifted to the score at ~**slot 15–16** of the feed — roughly one small-author post per response.

| Param | Value |
|-------|-------|
| `ColdStartFollowerCap` | 1,000 followers |
| `ColdStartImpressionThreshold` | 1,000 Home views |
| `ColdStartMaxPostAgeSecs` | 172,800 (48h) |
| `ColdStartSlotMin/Max` | 15 / 16 |
| `LowImpressionsMaxPositionRatio` | 0.85 |
| `EnableViewerColdStart` | true |

### 3. Author diversity decay

Real values now known: `AuthorDiversityDecay = 0.5`, `AuthorDiversityFloor = 0.25`.

```text
multiplier(k) = (1 − 0.25) × 0.5^k + 0.25     // k = your post's rank among your posts, by score

k=0 (your best):  1.000
k=1:              0.625
k=2:              0.4375
k=3:              0.3438
k=4:              0.2969
…→ floor:         0.25   (never below 25%)
```

### 4. Out-of-network discount

| Param | Value | Applies to |
|-------|-------|------------|
| `OonWeightFactor` | **0.75** | All OON posts; **also in-network replies & retweets** (`EnableOonRescoreForInNetworkRepliesRetweets=true`) |
| `TopicOonWeightFactor` | **0.5** | OON posts on topic-surface requests |

### 5. VMRanker (diversity rerank)

`vm-ranker/` — a determinantal point process over post embeddings that reorders the scored list, trading a little score for less similarity between neighbors (`VMRankerDppTheta = 0.65`, top 150 ranks). Posting content that's *different* from what's already in the feed helps; near-duplicates get spread apart.

---

## Video duration gate

`MinVideoDurationMs = 10_000` → **10 seconds**, exactly. The gate feeds the VQV heads (both currently weight 0.0, so the gate is dormant) — but it's the verified source of the "video > 10s" rule of thumb.

---

## Quick Reference Card

```text
MAXIMIZE (real weights):
├── Share via copy link  20.0   ← biggest single lever
├── Reply                 5.0   (20.0 from mutual follows)
├── Share via DM          5.0
├── Quote                 5.0
├── Follow author         4.0
├── Share                 2.0
├── Retweet               1.0
├── Like                  0.5
└── Click/dwell family    ≤0.4

MINIMIZE (real weights):
├── Report              −234.0   most severe
├── Mute                 −58.8
├── Not interested       −43.2
├── Block                −31.2
└── Not dwelled           −0.02  scroll-past tax

REMEMBER: weight × predicted probability, not raw counts.
```

---

## Source Files

| Component | Where |
|-----------|-------|
| **Weight values (production defaults)** | `home-mixer/params/param.rs` |
| Weighted sum, offset, diversity, OON, cold-start wiring | `home-mixer/scorers/ranking_scorer.rs` |
| New-author boost | `home-mixer/scorers/author_cold_start.rs` |
| DPP rerank | `vm-ranker/` |
| Action predictions | `phoenix/` (64-action taxonomy; prod ranking cfg: 8 layers, emb 2560, history 1022, candidates 64) |

---

**Next:** [Filter System →](filter-system.md)
