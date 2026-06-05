---
layout: default
title: X Algorithm Playbook
---

<div align="center">

<h1>X Algorithm Playbook</h1>

<h3>The definitive guide to maximizing your reach on X (Twitter)</h3>

<p><strong>Based on reverse-engineering the open-source algorithm</strong></p>

<p>
  <a href="https://github.com/tang-vu/x-algorithm-playbook">
    <img src="https://img.shields.io/github/stars/tang-vu/x-algorithm-playbook?style=for-the-badge&logo=github" alt="GitHub stars">
  </a>
  <a href="LICENSE">
    <img src="https://img.shields.io/badge/License-MIT-blue?style=for-the-badge" alt="License">
  </a>
</p>

<hr>

</div>

## The Algorithm in 60 Seconds

```
Your Post Score = Σ (weight × P(action))

POSITIVE: reply (highest), quote, follow, retweet, like, share
NEGATIVE: report (most severe), block, mute, "not interested"
```

**Key insight:** The algorithm predicts 19 different user actions and weights them. Maximize positive, avoid negative.

> 🆕 **Updated for the May 15, 2026 release** (refreshed every 4 weeks): new `grox` content-understanding service, new out-of-network reach paths (Phoenix Topics, MoE, Who-to-Follow), and a downloadable mini Phoenix model. **[See what changed →](reference/may-2026-update.md)**

---

## Quick Navigation

| Start Here | Go Deeper |
|------------|-----------|
| [10 Golden Rules](rules/00-golden-rules.md) | [Scoring System](rules/01-scoring-system.md) |
| [Pre-Post Checklist](checklists/pre-post-checklist.md) | [Action Weights](reference/action-weights.md) |
| [Common Mistakes](case-studies/common-mistakes.md) | [Filter System](reference/filter-system.md) |
| [What's New (May 2026)](reference/may-2026-update.md) | [Algorithm FAQ](reference/algorithm-faq.md) |

---

## The 10 Golden Rules

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

[Read full rules →](rules/00-golden-rules.md)

---

## Action Weights

| Action | Relative (est.) | Impact |
|--------|-----------------|--------|
| Reply | Highest | Top positive |
| Quote Tweet | High | Strong |
| Retweet | Medium | Good |
| Like | Medium | Baseline |
| Block | Negative | Very negative |
| Report | Negative | Most severe |

> Exact weight values are **redacted** from the source — these are relative estimates. [Why →](reference/action-weights.md#the-exact-weight-values-are-redacted)

[Full reference →](reference/action-weights.md)

---

## Repository Contents

```
x-algorithm-playbook/
├── rules/           # 7 core strategy guides
├── checklists/      # 3 actionable checklists
├── case-studies/    # 2 real-world examples
└── reference/       # 4 technical deep-dives
```

---

## Source

This playbook is based on analysis of:

- [xAI X Algorithm](https://github.com/xai-org/x-algorithm)
- [Twitter Algorithm (Archive)](https://github.com/twitter/the-algorithm)

---

<div align="center">

<p><strong>Star this repo if it helped you grow on X!</strong></p>

<p><a href="https://github.com/tang-vu/x-algorithm-playbook">View on GitHub</a></p>

</div>
