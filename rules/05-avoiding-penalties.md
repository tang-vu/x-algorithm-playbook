---
description: "The negative weights and filters that destroy reach — reports, mutes, blocks, and the 54-rule visibility-filtering engine."
---

# Avoiding Penalties

> How to avoid the negative actions and filters that destroy your reach.
>
> *Last verified: **September 20, 2026** against [xai-org/x-algorithm](https://github.com/xai-org/x-algorithm) @ [`8b25829`](https://github.com/xai-org/x-algorithm/commit/8b25829717a4f104dd04403ee7d0253c5fedb1b7) (Sep 18, 2026 release).*

---

## The Negative Action Weights

These actions carry **negative** weight and SUBTRACT from your score — **real values** from `params/param.rs` (Aug 2026):

| Action | Real weight | Impact |
|--------|-------------|--------|
| Report | **−234.0** | Most severe by far |
| Mute author | **−58.8** | Severe — *harsher than block!* |
| "Not Interested" | **−43.2** | Severe — *harsher than block!* |
| Block author | **−31.2** | Strong, but NOT the worst negative |
| Not dwelled | −0.02 | Mild (skipped past fast) |

**Reality check:** weights multiply *predicted probabilities*, not counts — `−234` doesn't mean one report cancels 468 likes; `P(report)` is just extremely rare. Two lessons: (1) the real severity order is **mute/not-interested > block**, so content that merely *annoys* people costs more than content that gets blocked; (2) negative signals are per-viewer predictions — consistently triggering them is what kills reach, not one bad actor.

---

## What Triggers Negative Actions

### Block Triggers

| Behavior | Risk Level |
|----------|------------|
| Spam replies | 🔴 High |
| Unsolicited DMs | 🔴 High |
| Harassment | 🔴 High |
| Extreme views | 🟡 Medium |
| Over-tagging | 🟡 Medium |
| Off-topic engagement | 🟢 Low |

### Mute Triggers

| Behavior | Risk Level |
|----------|------------|
| Posting too frequently | 🔴 High |
| Off-brand content | 🟡 Medium |
| Engagement bait | 🟡 Medium |
| Repetitive content | 🟢 Low |

### Report Triggers

| Behavior | Risk Level |
|----------|------------|
| ToS violations | 🔴 Automatic |
| Misinformation | 🔴 High |
| Harassment | 🔴 High |
| Spam patterns | 🟡 Medium |

### "Not Interested" Triggers

| Behavior | Risk Level |
|----------|------------|
| Irrelevant content | 🟡 Medium |
| Clickbait | 🟡 Medium |
| Low quality | 🟢 Low |

---

## The Filters That Can Hide Your Content

Two layers can remove your post — **19 pre-scoring pipeline filters** + the **visibility-filtering rule engine** (28 shared rules + 26 that only apply to recommendations to non-followers). [Full reference →](../reference/filter-system.md)

### 1. Age Filter

- Posts older than **48 hours** are dropped (`MAX_POST_AGE` in `home-mixer/params/config.rs`) — verified
- **Avoid:** Posting at dead times

### 2. Drop Duplicates Filter

- Duplicate tweet IDs removed
- **Avoid:** Posting same content twice

### 3. Core Data Hydration Filter

- Posts missing text or author removed
- **Avoid:** N/A (technical issue)

### 4. Self Tweet Filter

- Your own posts removed from your feed
- **Avoid:** N/A (expected behavior)

### 5. Retweet Deduplication Filter

- Multiple retweets of same content deduplicated
- **Avoid:** N/A (expected behavior)

### 6. Ineligible Subscription Filter

- Paywalled content from non-subscribed users removed
- **Avoid:** N/A (subscription feature)

### 7. Previously Seen Posts Filter

- Posts user already saw are removed
- **Avoid:** N/A (expected behavior)

### 8. Previously Served Posts Filter

- Already-served posts in session removed
- **Avoid:** N/A (expected behavior)

### 9. Muted Keyword Filter ⚠️

- Posts with user's muted keywords hidden
- **Avoid:** Spam words, controversial terms

### 10. Author Socialgraph Filter ⚠️

- Posts from blocked/muted authors hidden
- **Avoid:** Getting blocked/muted

### 11. OON/NSFW/subscription/new-user filters

- OON retweet+reply dedup, OON NSFW (SimClusters), ineligible subscriptions, new-user min-engagement, inventory holdouts, plus post-selection `DedupConversationFilter`
- **Avoid:** N/A mostly (expected behavior)

### 12. VF Filter (Visibility Filtering) ⚠️ — post-selection

- A whole second engine (`visibility-filtering/`): spam, violence, gore, deleted posts, **legal takedowns**, NSFW, account-state labels → verdicts **allow / interstitial / drop**
- **26 rules fire only for non-followers** — the "OON ceiling": visible to followers, invisible in recommendations
- Replies/quotes of a dropped post get dropped too (`AncillaryVFFilter`)
- **Avoid:** ToS violations, spam labels, sensitive media without marking

**Most important to avoid:** #9, #10, #12 — plus account-level labels from `scarecrow`/`botmaker`/`user-cred-v2`. Check **Settings → Under the Hood** to see labels applied to your account/posts.

---

## Muted Keywords to Avoid

Common words that get muted:

### Promotional Spam

```text
❌ "DM me"
❌ "Link in bio"
❌ "Follow for follow"
❌ "Giveaway" (spam context)
❌ "Free money"
```

### Crypto/Scam Adjacent

```text
❌ "100x"
❌ "Guaranteed returns"
❌ "Not financial advice"
❌ "WAGMI" (depending on audience)
```

### Engagement Bait

```text
❌ "Like if you agree"
❌ "Retweet to win"
❌ "Follow + RT"
```

### Political/Divisive (Niche-Dependent)

```text
⚠️ Political keywords (muted by many)
⚠️ Controversial names
⚠️ Culture war terms
```

---

## Content Safety Guidelines

### Stay Within ToS

| ✅ Safe | ❌ Dangerous |
|---------|--------------|
| Strong opinions | Personal attacks |
| Debate | Harassment |
| Criticism | Threats |
| Adult topics (marked) | Explicit content (unmarked) |

### Avoid Automated Detection

Patterns that may trigger spam detection:

```text
❌ Same reply on multiple posts
❌ Mass following/unfollowing
❌ Suspicious posting patterns
❌ Coordinated behavior (pods)
❌ Link spamming
```

---

## Protecting Your Reputation

### Engagement Hygiene

| ✅ Do | ❌ Don't |
|-------|---------|
| Engage meaningfully | Reply with "Great post!" only |
| Space your engagement | Mass-engage in short bursts |
| Stay in your niche | Randomly engage everywhere |
| Build relationships | Transactional engagement |

### Handling Conflict

```text
When someone attacks you:
├── Don't engage (engagement signals interest)
├── Block if necessary (protects your experience)
├── Move on (don't create drama)
└── Never escalate (risk getting blocked by others)
```

---

## Recovery from Penalties

### If Your Reach Dropped

1. **Audit recent content** - Did you trigger negative actions?
2. **Check for pattern changes** - Posting frequency, content type
3. **Wait and observe** - Sometimes it's algorithmic noise
4. **Slowly rebuild** - Focus on high-quality, safe content

### Signs of Penalty

| Symptom | Possible Cause |
|---------|----------------|
| Sudden reach drop | Multiple blocks/mutes |
| Posts not showing | Filter triggered |
| Low engagement on quality content | Embedding confusion |

---

## Quick Reference

### Pre-Post Safety Check

- [ ] No spam keywords
- [ ] No ToS violations
- [ ] Not engaging with controversial topics recklessly
- [ ] Not over-tagging people
- [ ] Content matches my usual topics

### Engagement Safety Check

- [ ] Adding genuine value in replies
- [ ] Not spamming big accounts
- [ ] Not using engagement bots/pods
- [ ] Staying within my niche

---

**Next:** [Growth Strategies →](06-growth-strategies.md)
