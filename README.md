# X Algorithm Playbook

> **The definitive guide to maximizing your reach on X (Twitter), based on reverse-engineering the open-source algorithm.**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)
[![Stars](https://img.shields.io/github/stars/tang-vu/x-algorithm-playbook?style=social)](https://github.com/tang-vu/x-algorithm-playbook)

## Why This Exists

X (formerly Twitter) open-sourced their recommendation algorithm, and xAI [shipped a major update](https://github.com/xai-org/x-algorithm) on **May 15, 2026** — now refreshed **every 4 weeks** with developer notes. This playbook distills thousands of lines of code into **actionable rules** that anyone can follow to maximize their reach.

**No fluff. No guesswork. Just algorithm-backed strategies.**

> 🆕 **New in the May 2026 update:** a content-understanding service (`grox`), new out-of-network reach paths (Phoenix Topics, Phoenix MoE, Who-to-Follow), a unified end-to-end pipeline, and a downloadable mini Phoenix model. **[See what changed →](reference/may-2026-update.md)**

---

## Quick Start

| If you want to... | Read this |
|-------------------|-----------|
| Get the essentials | [Golden Rules](rules/00-golden-rules.md) |
| Understand scoring | [Scoring System](rules/01-scoring-system.md) |
| Optimize before posting | [Pre-Post Checklist](checklists/pre-post-checklist.md) |
| Avoid penalties | [Avoiding Penalties](rules/05-avoiding-penalties.md) |
| Deep dive | [Action Weights Reference](reference/action-weights.md) |
| See the latest changes | [May 2026 Update](reference/may-2026-update.md) |

---

## The Algorithm in 60 Seconds

```
Your Post Score = Σ (weight × P(action))

Where actions include:
├── POSITIVE: reply (highest), quote, follow, retweet, like, share
├── NEGATIVE: report (most severe), block, mute, "not interested"
└── NEUTRAL: click, dwell time, profile view
```

**Key insight:** The algorithm predicts 19 different user actions and weights them. Your goal is to maximize positive action probability while avoiding negative signals.

---

## Repository Structure

```
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
    ├── may-2026-update.md      ← What changed in the latest release
    └── algorithm-faq.md
```

---

## The Golden Rules (TL;DR)

1. **Replies are king** — Reply is the top-weighted positive signal
2. **Avoid negative actions** — Blocks/reports carry strong negative weight
3. **Space your posts** — Author diversity penalty kicks in after 1st post
4. **In-network first** — Your followers see you before non-followers
5. **Video > Image > Text** — But only if video exceeds minimum duration
6. **Dwell time matters** — Longer content = higher engagement signal
7. **Don't trigger filters** — 12 filters can completely hide your content
8. **Engage authentically** — Algorithm tracks your interaction patterns
9. **Niche down** — Consistent topics improve retrieval matching
10. **Quality > Quantity** — One great post beats five mediocre ones

[Read the full Golden Rules →](rules/00-golden-rules.md)

---

## How the Scoring System Works

The algorithm uses a **Grok-based transformer model** (Phoenix) to predict engagement:

```
┌─────────────────────────────────────────────────────────────────┐
│                    YOUR POST ENTERS THE SYSTEM                   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  CANDIDATE SOURCING                                             │
│  ├── Thunder: in-network posts from accounts you follow         │
│  ├── Phoenix Retrieval: ML-discovered out-of-network posts      │
│  ├── Phoenix Topics + Phoenix MoE: topical / expert discovery   │
│  └── Who-to-Follow · Ads · Prompts                              │
│  (grox content-understanding tags + embeds every post first)    │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  FILTERING (12 filters can remove your post)                     │
│  ├── Age filter (too old)                                        │
│  ├── Muted keywords                                              │
│  ├── Blocked/muted authors                                       │
│  └── Spam/safety filters                                         │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  SCORING (sequential scorers)                                   │
│  ├── Phoenix Scorer: predicts 19 actions via Grok transformer   │
│  ├── Weighted Scorer: Σ (weight × P(action)) = base score       │
│  ├── Author Diversity Scorer: attenuates repeated authors       │
│  └── OON Scorer: down-weights out-of-network content            │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  RANKING: Top-scored posts shown first                           │
└─────────────────────────────────────────────────────────────────┘
```

[Deep dive into scoring →](rules/01-scoring-system.md)

---

## Action Weights (Simplified)

> The algorithm sums 19 weighted action probabilities, but the **exact weight values are redacted** (not in the open-source repo). The column below is **relative intuition, not code values** — see [why →](reference/action-weights.md#the-exact-weight-values-are-redacted).

| Action | Relative (est.) | Impact |
|--------|-----------------|--------|
| Reply | Highest | Top positive signal |
| Quote Tweet | High | Strong engagement |
| Follow Author | High | High intent signal |
| Retweet | Medium | Good reach |
| Like | Medium | Baseline positive |
| Share (DM/Link) | Low–Med | Moderate |
| Click | Low | Weak positive |
| Dwell Time | Variable | Longer = better (continuous) |
| "Not Interested" | Negative | Negative signal |
| Mute | Negative | Strong negative |
| Block | Negative | Very strong negative |
| Report | Negative | Most severe |

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
