# X Algorithm Playbook

> **The definitive guide to maximizing your reach on X (Twitter), based on reverse-engineering the open-source algorithm.**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)
[![Stars](https://img.shields.io/github/stars/tang-vu/x-algorithm-playbook?style=social)](https://github.com/tang-vu/x-algorithm-playbook)

## Why This Exists

X (formerly Twitter) open-sourced their recommendation algorithm, and xAI [ships regular updates](https://github.com/xai-org/x-algorithm) — latest: **September 18, 2026** (refreshed every ~4 weeks with developer notes). This playbook distills thousands of lines of code into **actionable rules** that anyone can follow to maximize their reach.

**No fluff. No guesswork. Just algorithm-backed strategies.**

> 🆕 **New in the Aug–Sep 2026 updates:** the **real action weights are now public** (copy-link share 20.0, reply 5.0, report −234.0…), the **production Phoenix model** shipped, a **visibility-filtering engine** (54 rules incl. recommendation-only drops), **SimClusters** OON source, **VMRanker** DPP reranking, a **new-author cold-start boost**, and **Under the Hood** transparency reporting. **[See what changed →](reference/september-2026-update.md)**

---

## Quick Start

| If you want to... | Read this |
|-------------------|-----------|
| Get the essentials | [Golden Rules](rules/00-golden-rules.md) |
| Understand scoring | [Scoring System](rules/01-scoring-system.md) |
| Optimize before posting | [Pre-Post Checklist](checklists/pre-post-checklist.md) |
| Avoid penalties | [Avoiding Penalties](rules/05-avoiding-penalties.md) |
| Deep dive | [Action Weights Reference](reference/action-weights.md) |
| See the latest changes | [September 2026 Update](reference/september-2026-update.md) |

---

## The Algorithm in 60 Seconds

```text
Your Post Score = Σ (weight × P(action))

Where actions include (real weights):
├── POSITIVE: share via copy link (20.0), reply (5.0), quote (5.0),
│             DM share (5.0), follow (4.0), share (2.0), retweet (1.0),
│             like (0.5), click family (0.05–0.4), dwell (0.004–0.05)
├── NEGATIVE: report (−234), mute (−58.8), "not interested" (−43.2),
│             block (−31.2), not dwelled (−0.02)
└── BOOST:   reply on a mutual-follow original post → 20.0
```

**Key insight:** Phoenix predicts 64 action classes (24 named feed actions + continuous dwell signals); ~26 weighted terms feed the score. Weights multiply *predicted probabilities per viewer*, not raw counts. Your goal: maximize positive action probability, avoid negative signals.

---

## Repository Structure

```text
x-algorithm-playbook/
├── rules/                  # Core strategies (start here)
│   ├── 00-golden-rules.md      ← The 10 most important rules
│   ├── 01-scoring-system.md    ← How your posts are scored
│   ├── 02-content-optimization.md
│   ├── 03-engagement-tactics.md
│   ├── 04-timing-frequency.md
│   ├── 05-avoiding-penalties.md
│   └── 06-growth-strategies.md
│
├── checklists/             # Actionable checklists
│   ├── pre-post-checklist.md
│   ├── weekly-audit-checklist.md
│   └── profile-optimization.md
│
├── case-studies/           # Real examples
│   ├── viral-thread-anatomy.md
│   └── common-mistakes.md
│
└── reference/              # Technical deep-dives
    ├── action-weights.md
    ├── filter-system.md
    ├── september-2026-update.md ← What changed in the latest release
    ├── may-2026-update.md       ← Superseded (weights were still redacted then)
    └── algorithm-faq.md
```

---

## The Golden Rules (TL;DR)

1. **Get sent, get replies** — Copy-link share (20.0) and replies (5.0; 20.0 from mutuals) are the top signals
2. **Avoid negative actions** — Report −234, mute −58.8, not-interested −43.2, block −31.2
3. **Space your posts** — Author diversity: your 2nd post in a feed scores ×0.625 (floor ×0.25)
4. **In-network first** — Out-of-network content takes a verified ×0.75 discount
5. **Video > Image > Text** — Verified 10s minimum; direct video weight is small (0.07)
6. **Dwell time matters** — Continuous dwell term (0.004) rewards longer reads
7. **Don't trigger filters** — 17 pipeline filters + a 54-rule visibility engine can hide you
8. **Engage authentically** — Algorithm tracks your interaction patterns
9. **Niche down** — Semantic IDs + SimClusters reward consistent topics
10. **Quality > Quantity** — One great post beats five mediocre ones

[Read the full Golden Rules →](rules/00-golden-rules.md)

---

## How the Scoring System Works

The algorithm uses a **Grok-based transformer model** (Phoenix) to predict engagement:

```text
┌─────────────────────────────────────────────────────────────────┐
│                    YOUR POST ENTERS THE SYSTEM                   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  CANDIDATE SOURCING                                             │
│  ├── Thunder: in-network posts from accounts you follow         │
│  ├── Phoenix Retrieval: ML-discovered out-of-network posts      │
│  ├── SimClusters: engagement-cluster candidates (new Aug 2026)  │
│  ├── Phoenix Topics · Phoenix MoE (off by default)              │
│  └── Who-to-Follow · Ads · Prompts                              │
│  (grox classifies + semantic IDs tag every post first)          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  FILTERING (17 pre-scoring filters can remove your post)         │
│  ├── Age filter (>48h — verified)                                │
│  ├── Muted keywords                                              │
│  ├── Blocked/muted authors                                       │
│  └── Spam/safety + visibility-filtering verdicts                 │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  SCORING (ranking_scorer.rs — verified order)                   │
│  ├── Phoenix Scorer: predicts 64 action classes + dwell         │
│  ├── Weighted sum: Σ (weight × P(action)) — real weights        │
│  ├── Cold-start boost: small authors lifted to ~slot 15–16      │
│  ├── Author diversity: ×0.75·0.5^k + 0.25                       │
│  ├── OON discount: ×0.75 (×0.5 on topic surfaces)               │
│  └── VMRanker: DPP rerank for feed diversity (top 150)          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  RANKING + BLENDING: posts interleaved with ads/prompts/WTF      │
└─────────────────────────────────────────────────────────────────┘
```

[Deep dive into scoring →](rules/01-scoring-system.md)

---

## Action Weights (Real Values)

> Since August 2026, the **exact production weights are public** in `home-mixer/params/param.rs`. They multiply *predicted probabilities* — not raw action counts.

| Action | Real weight | Impact |
|--------|-------------|--------|
| Share via copy link | **20.0** | Top positive signal |
| Reply (mutual follow) | **20.0** | 5.0 + 15.0 bidirectional boost |
| Reply · Quote · DM share | **5.0** | Strong engagement |
| Follow Author | 4.0 | High intent signal |
| Share | 2.0 | Moderate |
| Retweet | 1.0 | Good reach |
| Like | 0.5 | Baseline positive |
| Click / link / media | 0.05–0.4 | Weak positive |
| Dwell Time | 0.004–0.05 | Longer = better (continuous) |
| Block | −31.2 | Strong negative |
| "Not Interested" | −43.2 | Harsher than block |
| Mute | −58.8 | Harsher than block |
| Report | **−234.0** | Most severe |

[Full action reference →](reference/action-weights.md)

---

## Contributing

Found an insight? Want to add a case study? PRs welcome!

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

---

## Sources

- [X Algorithm Source Code (xAI)](https://github.com/xai-org/x-algorithm) — Primary source for this playbook
- [X Algorithm (Twitter Archive)](https://github.com/twitter/the-algorithm)
- [X Algorithm ML Components](https://github.com/twitter/the-algorithm-ml)
- Original analysis from codebase review

---

## License

MIT License - Use freely, attribution appreciated.

---

<p align="center">
  <b>Star this repo if it helped you grow on X!</b>
  <br><br>
  <a href="https://github.com/tang-vu/x-algorithm-playbook">⭐ Star on GitHub</a>
</p>
