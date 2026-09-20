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

POSITIVE: copy-link share (20.0), reply/quote/DM-share (5.0), follow (4.0)…
NEGATIVE: report (−234), mute (−58.8), "not interested" (−43.2), block (−31.2)
```

**Key insight:** Phoenix predicts 64 action classes feeding ~26 weighted terms — real values now public. Weights multiply *predicted probabilities*, not raw counts. Maximize positive, avoid negative.

> 🆕 **Updated for the September 18, 2026 release:** real published action weights, the production Phoenix model, a 54-rule visibility-filtering engine, SimClusters candidates, VMRanker DPP reranking, a new-author cold-start boost, and Under the Hood transparency reports. **[See what changed →](reference/september-2026-update.md)**

---

## Quick Navigation

| Start Here | Go Deeper |
|------------|-----------|
| [10 Golden Rules](rules/00-golden-rules.md) | [Scoring System](rules/01-scoring-system.md) |
| [Pre-Post Checklist](checklists/pre-post-checklist.md) | [Action Weights](reference/action-weights.md) |
| [Common Mistakes](case-studies/common-mistakes.md) | [Filter System](reference/filter-system.md) |
| [What's New (Sep 2026)](reference/september-2026-update.md) | [Algorithm FAQ](reference/algorithm-faq.md) |

---

## The 10 Golden Rules

1. **Get sent, get replies** — Copy-link share (20.0) + replies (5.0; 20.0 from mutuals) lead
2. **Avoid negative actions** — Report −234, mute −58.8, not-interested −43.2, block −31.2
3. **Space your posts** — Author diversity: 2nd post in a feed scores ×0.625
4. **In-network first** — Verified ×0.75 out-of-network discount
5. **Video > Image > Text** — Verified 10s minimum; direct weight is small (0.07)
6. **Dwell time matters** — Longer content = higher engagement signal
7. **Don't trigger filters** — 17 pipeline filters + 54-rule visibility engine
8. **Engage authentically** — Algorithm tracks your interaction patterns
9. **Niche down** — Semantic IDs + SimClusters reward consistent topics
10. **Quality > Quantity** — One great post beats five mediocre ones

[Read full rules →](rules/00-golden-rules.md)

---

## Action Weights

| Action | Real weight | Impact |
|--------|-------------|--------|
| Share via copy link | **20.0** | Top positive |
| Reply · Quote · DM share | 5.0 | Strong |
| Follow author | 4.0 | Intent |
| Retweet | 1.0 | Good |
| Like | 0.5 | Baseline |
| Block | −31.2 | Very negative |
| "Not interested" / Mute | −43.2 / −58.8 | Harsher than block |
| Report | −234.0 | Most severe |

> **Real production values** — published Aug 2026 in `params/param.rs`. They scale predicted probabilities, not raw counts. [Details →](reference/action-weights.md)

[Full reference →](reference/action-weights.md)

---

## Repository Contents

```
x-algorithm-playbook/
├── rules/           # 7 core strategy guides
├── checklists/      # 3 actionable checklists
├── case-studies/    # 2 real-world examples
└── reference/       # 5 technical deep-dives
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
