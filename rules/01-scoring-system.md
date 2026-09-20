# The X Scoring System Explained

> How your posts are ranked in the For You feed, based on the actual algorithm code — **now with real published weights** (August 2026 release).

---

## The Scoring Formula

Every post is scored using this formula:

```text
Final Score = Σ (weight_i × P(action_i))
```

Where:

- `P(action_i)` = probability that *this specific viewer* takes the action (predicted by the Phoenix transformer)
- `weight_i` = that action's weight — **real values are public** in `home-mixer/params/param.rs`

**Example (real weights):**

```text
Score = 0.5×P(like) + 5.0×P(reply) + 1.0×P(retweet) + 20.0×P(copy_link_share) + ...
        − 31.2×P(block) − 234.0×P(report) − 58.8×P(mute) − 43.2×P(not_interested)
```

> ⚠️ **Crucial:** weights multiply *predicted probabilities*, not raw counts. "Report = −234" does **not** mean one report cancels 468 likes — `P(report)` is >1000× rarer than `P(like)` at baseline. [Full explanation →](../reference/action-weights.md#the-1-misconception-weights-scale-probabilities-not-counts)

---

## The Predicted Actions

Phoenix's action taxonomy (64 heads in the model config) feeds ~26 weighted terms, grouped in the repo's docs into five families:

| Group | Actions |
|-------|---------|
| **Engagement** | favorite (0.5) · reply (5.0) · repost (1.0) · quote (5.0) · share (2.0) · share via DM (5.0) · share via copy link (**20.0**) |
| **Clicks** | post click (0.4) · profile click (0.0) · link open (0.2) · photo expand (0.05) · video open (0.07) · quoted-post click (0.05) |
| **Attention** | video quality view (0.0) · dwell (0.05) · dwell time (0.004, continuous) · click dwell (0.0) · active seconds (0.0) |
| **Author** | follow author (4.0) · post unexplored (0.02, in-network only) |
| **Negative** | not interested (−43.2) · mute author (−58.8) · block author (−31.2) · report (−234.0) · not dwelled (−0.02) |

**Plus a conditional boost:** original posts from **mutual follows** get reply weight **20.0** (5.0 + 15.0 `BidirectionalFollowReplyWeightBoost`, shipped July 2026).

[Full weight table with params →](../reference/action-weights.md)

---

## The Pipeline: Post Pipeline + Blending Pipeline

The For You feed is built by two nested pipelines in `home-mixer`:

```text
POST PIPELINE (PhoenixCandidatePipeline)
1. Query Hydration      → viewer's recent engagement history (the model's main input),
                          following list, blocks/mutes, muted keywords, seen posts, topics
2. Candidate Sourcing   → sources queried in parallel (below)
3. Candidate Hydration  → post text/media, author details + account labels, language,
                          engagement counts, subscription status…
4. Pre-Scoring Filters  → 17 filters drop ineligible posts (incl. the 48h AgeFilter)
5. Scoring              → PhoenixScorer → RankingScorer → VMRanker (below)
6. Selection            → TopKScoreSelector: sort by score, keep top K
7. Post-Selection       → VFFilter (visibility verdicts) → AncillaryVFFilter
                          (drops replies/quotes of dropped posts) → DedupConversationFilter

BLENDING PIPELINE (ForYouCandidatePipeline)
8. BlenderSelector      → interleaves ranked posts with ads, Who-to-Follow, prompts
9. Side Effects         → served-post recording, cache refresh, event logging
```

Each stage can be toggled via feature-switch params in `home-mixer/params/param.rs`.

---

## Where Candidates Come From

| Source | Network | What it is | Status |
|--------|---------|------------|--------|
| **Thunder** (`thunder/`) | In-network | Realtime in-memory store of followed accounts' posts | ✅ on (max 1,200) |
| **Phoenix retrieval** (`phoenix/`) | Out-of-network | Two-tower similarity search over a checkpoint-baked index (`phoenix-rankall/`) | ✅ on (max 1,000) |
| **SimClusters** (`simclusters/`) | Out-of-network | Clusters accounts+posts by engagement patterns | ✅ on (new Aug 2026) |
| **Phoenix Topics** (`phoenix_topics_source.rs`) | Out-of-network | Topic-matched discovery | exists in code |
| **Phoenix MoE** (`phoenix_moe_source.rs`) | Out-of-network | Mixture-of-experts retrieval | ⚠️ `EnablePhoenixMOESource=false` (A/B experiment) |
| **TweetMixer** | Out-of-network | Legacy source | ⚠️ off by default |
| **Ads / Who-to-Follow / Prompts / push-to-home** | Blending layer | Added by the Blending Pipeline, not scored as posts | ✅ on |

**Why this matters:** OON reach has two live doors — Phoenix embedding similarity and SimClusters' engagement-based clusters. Both reward the same thing: a clear, consistent topic.

---

## How Content Is Understood: `grox` + Semantic IDs

Two systems decide *what your post is* before ranking:

- **`grox/`** — classifiers (spam, adult, violent media) plus text/image embeddings, run at publish time.
- **Semantic IDs** — Phoenix encodes each post's multimodal embedding into residual-quantized codes (6 levels × 256). Posts on the same topic **share SID prefixes**, so the models generalize to brand-new posts with zero engagement history — a crisp topic literally becomes your post's identity in the index.

**Strategic consequence:** no keyword/hashtag boost exists. `grox` + SIDs decide what your post is *about*; engagement decides the rest. Clear topic → clean identity → matched to people who care → higher P(reply/share). (See [Content Optimization](02-content-optimization.md#content-understanding-grox-semantic-ids).)

---

## The ML Model: Phoenix (production code, since Aug 2026)

The shipped tree is now the **real production stack** — JAX training, Rust gRPC serving, synthetic-data generators (no artifact download needed):

```text
┌─────────────────────────────────────────────────────────────────┐
│                         PHOENIX RANKER                           │
├─────────────────────────────────────────────────────────────────┤
│  INPUT:                                                          │
│  ├── User token (country, language, age bracket, apps…           │
│  │    — NO learned per-user ID; you are what you engaged with)   │
│  ├── History: up to 1,022 recent engagements (+ dwell times)     │
│  └── Candidates: up to 64 per batch                              │
│      (each = hashed IDs + semantic IDs + context features:       │
│       post age, local hour, timezone, surface)                   │
│                                                                  │
│  MODEL: Transformer, candidate isolation                         │
│  └── prod config: emb 2560 · 8 layers · GQA 20/4 · key 128       │
│      vocab 100M user / 100M item / 30M author · 64 action heads  │
│                                                                  │
│  OUTPUT: P(action) per candidate + dwell-time regression         │
└─────────────────────────────────────────────────────────────────┘
```

### Candidate Isolation

**Your post's score doesn't depend on what other posts are in the batch** — candidates attend only to the viewer context, never to each other. Scores are consistent and cacheable; you can't hide behind other posts, and a strong batch can't drag you down.

### Retrieval side

Two-tower: user tower (history → embedding, no user-ID embedding in prod) vs. candidate tower (post SIDs + hashed author). Trained contrastively with **favorites as the positive signal** — retrieval is optimized for what people *like*, not what they click.

---

## The Scoring Pipeline (RankingScorer order)

After `PhoenixScorer` produces the action probabilities:

```rust
// ranking_scorer.rs — verified order (default path)
1. weighted score  = offset( Σ positive·w − Σ negative·w )   // 26 terms
2. cold start      = AuthorColdStart lifts ONE small-author post to ~slot 15–16
                     (original post · ≤1k followers · ≤48h · <1k Home views)
3. author diversity = score × (0.75·0.5^k + 0.25)            // k = rank among your posts
4. OON discount    = score × 0.75   (×0.5 on topic surfaces;
                     also applied to in-network replies/retweets)
5. VMRanker        = DPP rerank over embeddings (theta 0.65, top 150)
                     — trades a little score for neighbor diversity
```

Real diversity curve: post #1 ×1.0, #2 ×0.625, #3 ×0.4375, #4 ×0.344 … floor ×0.25.

---

## Practical Implications

| Algorithm Behavior (now verified) | Your Strategy |
|-----------------------------------|---------------|
| Copy-link share = 20.0, top weight | Write "send this to…" posts people forward |
| Reply/quote/DM-share = 5.0 | Create discussion-starting content |
| Mutual follows: reply term 20.0 | Turn followers into mutuals |
| New-author slot ~15–16 for ≤1k-follower accounts | Post original content while small — every post is a boosted ticket |
| Diversity ×0.625 on your 2nd post in a feed | Space posts, use threads |
| OON ×0.75 (and in-net replies/RTs too) | Original posts > replies/RTs for reach |
| Video gate = 10s; VQV weight currently 0 | Video bonus is small now — video_open only 0.07 |
| VMRanker penalizes similar neighbors | Vary your content; don't repost near-duplicates |

### The Score Optimization Hierarchy (updated for real weights)

```text
1. Maximize P(copy-link share / DM share) → forwardable, "send this" content
2. Maximize P(reply) + P(quote)           → questions, debatable takes  (×4 on mutuals)
3. Maximize P(follow author)              → serial value worth subscribing to
4. Maximize P(retweet) + P(like)          → shareable, likeable baseline
5. Grow dwell_time                        → threads, long reads
6. Minimize P(report) ≫ P(mute), P(not-interested), P(block), P(not_dwelled)
```

---

## Source Files

| Component | Where |
|-----------|-------|
| Feed orchestration | `home-mixer/` |
| **Production weights** | `home-mixer/params/param.rs` |
| Weighted sum + diversity + OON + boosts | `home-mixer/scorers/ranking_scorer.rs` |
| New-author boost | `home-mixer/scorers/author_cold_start.rs` |
| Phoenix predictions | `home-mixer/scorers/phoenix_scorer.rs` |
| DPP rerank | `vm-ranker/` |
| Model train/serve (JAX + Rust) | `phoenix/` |
| Retrieval index | `phoenix-rankall/`, `phoenix-rankall-strato/` |
| Cluster candidates | `simclusters/` |
| Content understanding | `grox/` |
| In-network store | `thunder/` |
| Visibility verdicts | `visibility-filtering/` |

---

**Next:** [Content Optimization →](02-content-optimization.md)
