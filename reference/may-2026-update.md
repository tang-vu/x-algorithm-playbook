# What's New — May 15, 2026 Release

> The biggest open-source drop since the algorithm went public. This page summarizes what changed in [`xai-org/x-algorithm`](https://github.com/xai-org/x-algorithm) and — more importantly — **what it means for your reach.**

---

## TL;DR

- 📅 **Released May 15, 2026** · Apache-2.0 · updated **every 4 weeks** with developer notes.
- 🧠 **New `grox` content-understanding service** — classifiers + embedders decide *what your post is about* before it's scored.
- 🚪 **New out-of-network reach doors** — Phoenix **Topics**, Phoenix **MoE**, **Who-to-Follow** (not just generic retrieval).
- ⚙️ **Runnable end-to-end pipeline** — a single `phoenix/run_pipeline.py` (retrieval → ranking).
- 📦 **Downloadable mini Phoenix model** (~2.8 GB via Git LFS) — the architecture is no longer guesswork.
- ✅ **Confirms the core mechanics** this playbook already taught: 19 action heads, weighted scoring, candidate isolation, author-diversity + OON scorers, 12 filters.

**One-line takeaway:** *Topic clarity is now distribution.* The clearer your post's topic, the more reach doors it can walk through.

---

## What Actually Changed

### New repository components

| Component | What it does | Why you care |
|-----------|--------------|--------------|
| `grox/` | Content understanding — classifiers + embedders over every post | Decides what your post is *about* → which audiences it can reach |
| `phoenix/` | ML retrieval + ranking (Grok-based transformer) | The scorer that ranks you |
| `home-mixer/` | Orchestrates the 7-stage feed pipeline | Where sourcing → filtering → scoring → selection happen |
| `thunder/` | In-memory realtime store of in-network posts | How followers see you instantly |
| `candidate-pipeline/` | Reusable framework wiring the sources together | Where the new reach doors plug in |

### New candidate sources

Out-of-network reach is no longer a single funnel:

- **Phoenix Retrieval** — generic two-tower similarity search.
- **Phoenix Topics** — topical discovery; surfaces posts that map cleanly to a topic the viewer engages with.
- **Phoenix MoE** — mixture-of-experts retrieval for specialized / deep-niche interests.
- **Who-to-Follow** — account suggestions.
- **Ads / Prompts** — promoted + system content.

→ See [Growth Strategies → May 2026 Reach Paths](../rules/06-growth-strategies.md#may-2026-reach-paths).

### The model is now downloadable

A pre-trained **mini Phoenix** model ships via Git LFS, so the released architecture is concrete:

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
| Predicted action heads | **19** |
| Artifacts size | ~2.8 GB (Git LFS) |

The transformer is **ported from xAI's open-source Grok-1**. Run it end-to-end with:

```bash
uv run phoenix/run_pipeline.py --artifacts_dir artifacts/oss-phoenix-artifacts
```

> Production uses a larger config, but the mechanics — candidate isolation, weighted action heads, two-tower retrieval — are identical to what you can now run locally.

---

## What It Means For Your Reach

### 1. Clarity beats cleverness

`grox` has to *understand* your post before it can match it to an audience. A vague subtweet embeds noisily and reaches no one; a clear, on-topic post embeds cleanly and is eligible for **Topics** and **MoE** discovery on top of generic retrieval.

→ [Content Optimization → Content Understanding (grox)](../rules/02-content-optimization.md#content-understanding-grox)

### 2. There are now several reach doors — topic clarity opens all of them

```
Sharp, consistent topic  →  clean embedding
                         →  Phoenix Retrieval + Topics + MoE eligible
                         →  reaches people who actually care
                         →  higher P(reply/like) → more distribution

Topic-hopping            →  smeared embedding
                         →  matched to no one → distribution stops
```

### 3. Hashtags/keywords are *signal*, not *boost*

No keyword or hashtag gives a ranking bump. Their only job now is helping `grox` classify your topic (and human search). One or two real-word, relevant tags help; stuffing adds noise and risks the **muted-keyword filter**.

### 4. The fundamentals didn't change — they got confirmed

`weighted_scorer.rs` confirms the scoring **structure**: 19 weighted action probabilities summed, replies/quotes/follows positive, not-interested/block/mute/report negative, video gated by a minimum duration, then an offset/normalization. Keep doing the [Golden Rules](../rules/00-golden-rules.md).

### 5. The exact weights are STILL redacted

Even with a runnable model and full pipeline open-sourced, the **weight *values*** (`REPLY_WEIGHT`, `BLOCK_AUTHOR_WEIGHT`, `MIN_VIDEO_DURATION_MS`, …) are **not published** — they live in a `params` module absent from the repo (`home-mixer/lib.rs` declares 13 modules; `params` isn't one). So any precise multiplier ("reply = 2×", "one reply = 150 likes", "block = −10×") is an **inference, not a source value.** This playbook now labels every such number as an estimate. What you *can* trust: the set of weighted actions, their signs, the video-duration gate, and the relative priority order.

---

## Sources

- [xai-org/x-algorithm](https://github.com/xai-org/x-algorithm) — primary source (May 15, 2026 release)
- [phoenix/README.md](https://github.com/xai-org/x-algorithm/blob/main/phoenix/README.md) — model architecture + pipeline
- All facts on this page are grounded in the repo's README and developer notes; exact production weights remain redacted.

---

**Related:** [Scoring System](../rules/01-scoring-system.md) · [Growth Strategies](../rules/06-growth-strategies.md) · [Content Optimization](../rules/02-content-optimization.md) · [Algorithm FAQ](algorithm-faq.md)
