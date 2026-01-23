# X Algorithm Playbook

> **The definitive guide to maximizing your reach on X (Twitter), based on reverse-engineering the open-source algorithm.**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)
[![Stars](https://img.shields.io/github/stars/tang-vu/x-algorithm-playbook?style=social)](https://github.com/tang-vu/x-algorithm-playbook)

## Why This Exists

X (formerly Twitter) open-sourced their recommendation algorithm, and xAI [released an updated version](https://github.com/xai-org/x-algorithm) in January 2026. This playbook distills thousands of lines of code into **actionable rules** that anyone can follow to maximize their reach.

**No fluff. No guesswork. Just algorithm-backed strategies.**

---

## Quick Start

| If you want to... | Read this |
|-------------------|-----------|
| Get the essentials | [Golden Rules](rules/00-golden-rules.md) |
| Understand scoring | [Scoring System](rules/01-scoring-system.md) |
| Optimize before posting | [Pre-Post Checklist](checklists/pre-post-checklist.md) |
| Avoid penalties | [Avoiding Penalties](rules/05-avoiding-penalties.md) |
| Deep dive | [Action Weights Reference](reference/action-weights.md) |

---

## The Algorithm in 60 Seconds

```
Your Post Score = Σ (weight × P(action))

Where actions include:
├── POSITIVE: reply (+2×), like, retweet, quote, share, follow
├── NEGATIVE: block (-10×), mute, report (-20×), "not interested"
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
    └── algorithm-faq.md
```

---

## The Golden Rules (TL;DR)

1. **Replies are king** — Posts that generate replies score ~2× higher
2. **Avoid negative actions** — One block = -10× the value of a like
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
│  CANDIDATE SOURCING                                              │
│  ├── Thunder: Posts from accounts user follows (in-network)     │
│  └── Phoenix: ML-discovered posts (out-of-network)              │
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
│  SCORING (Phoenix ML Model)                                      │
│  ├── Predicts P(like), P(reply), P(retweet)... for 19 actions   │
│  ├── Weighted sum = final score                                  │
│  └── Author diversity penalty applied                            │
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

| Action | Weight | Impact |
|--------|--------|--------|
| Reply | ~2× | Highest positive signal |
| Quote Tweet | ~1.5× | Strong engagement |
| Retweet | ~1× | Good reach |
| Like | ~1× | Baseline positive |
| Follow Author | ~1× | High intent signal |
| Share (DM/Link) | ~0.5× | Moderate |
| Click | ~0.3× | Weak positive |
| Dwell Time | Variable | Longer = better |
| "Not Interested" | ~-1× | Negative signal |
| Mute | ~-5× | Strong negative |
| Block | ~-10× | Very strong negative |
| Report | ~-20× | Devastating |

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
