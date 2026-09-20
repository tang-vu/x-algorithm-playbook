# Timing & Frequency

> When to post and how often, based on algorithm mechanics.

---

## The Author Diversity Penalty

A key reason not to flood the feed — though it works differently than most people think.

### How It Works (verified from `ranking_scorer.rs` — real values published Aug 2026)

It is a **per-response** de-duplication, not a clock timer. Within a single For You response, your posts are sorted by score; each additional post **from the same author** gets a decayed multiplier based on its rank (position 0 = your top post, full weight):

```
multiplier = (1 - 0.25) × 0.5^position + 0.25      // decay=0.5, floor=0.25 — REAL values
```

- Your **highest-scored** post keeps full weight; your 2nd, 3rd… decay toward a `floor` (so an author is attenuated, **never removed to zero**, by this scorer).
- `position` is rank **by score** among your posts in that response — not chronological.

**Real values** (`AuthorDiversityDecay=0.5`, `AuthorDiversityFloor=0.25`):

```
Post #1: 100%   Post #2: 62.5%   Post #3: 43.75%   Post #4: 34.4%   …→ 25% floor
```

The decay is *steeper* than previously assumed — your second post loses over a third of its score.

### What This Means

| Posting Frequency | Impact |
|-------------------|--------|
| 1 post/day | Optimal per-post score |
| 2 posts/day | 2nd post scores at 62.5% |
| 3+ posts/day | 3rd at 43.75%, →25% floor |
| Threads | Counts as ONE author slot (good!) |

---

## Optimal Posting Strategy

### General Guidelines

| Frequency | Recommendation |
|-----------|----------------|
| Minimum | 1 post/day for consistency |
| Optimal | 1-2 quality posts/day |
| Maximum | 3 posts/day (spaced 4+ hours) |
| Threads | Can be longer, counts as 1 post |

### Spacing Your Posts

If posting multiple times per day:

```
❌ Bad: 3 posts within 1 hour
✅ Good: Post at 9am, 2pm, 7pm

Minimum spacing: 3-4 hours
Ideal spacing: 6+ hours
```

### Thread Strategy

Threads are a smart way to ship more content without multiplying your author-positions:

```
A thread is ONE post you publish → one author-position,
not N separate posts competing in the same response.

Benefits:
├── High dwell time (people read through)
├── One author-position instead of N (less diversity decay)
├── Multiple engagement points
└── Easy to bookmark/save
```

> Note: this is about *how many separate posts you publish*, not a special rule for threads inside the scorer — the diversity scorer only sees `author_id`, not thread structure.

---

## The Age Filter

Posts older than a threshold are filtered out entirely — **verified: 48 hours** (`MaxPostAgeHours=48`).

### Implications

| Factor | Action |
|--------|--------|
| Posts "expire" | Time your posts for maximum initial reach |
| Freshness matters | Post when your audience is online |
| Viral potential | Early engagement determines longevity |

### The First Hour

The most critical window for any post:

```
0-15 min: Initial distribution to followers
15-60 min: Algorithm measures engagement
1+ hour: Decision point for wider distribution
```

**Strategy:** Be available to engage in replies during the first hour.

---

## Finding Your Audience's Peak Times

### General Peak Times (US-centric)

| Time (ET) | Activity |
|-----------|----------|
| 8-9 AM | Morning scroll |
| 12-1 PM | Lunch break |
| 5-7 PM | Evening commute |
| 9-10 PM | Night scrolling |

### Finding YOUR Peak Times

Your audience may differ. Check:

```
1. X Analytics → Posts → Best times
2. Experiment with different times
3. Track engagement rates by post time
```

### Time Zone Considerations

| Audience Type | Strategy |
|---------------|----------|
| US-focused | ET 8am-10pm window |
| Europe-focused | CET business hours |
| Global | Multiple windows or overlap (2-4pm ET) |
| Niche-specific | When your niche is active |

---

## Weekly Cadence

### Sample Weekly Schedule

| Day | Posts | Type |
|-----|-------|------|
| Monday | 1-2 | Thread or insight |
| Tuesday | 1-2 | Engagement-focused |
| Wednesday | 1-2 | Value content |
| Thursday | 1-2 | Controversial/discussion |
| Friday | 1 | Fun/casual |
| Saturday | 0-1 | Optional |
| Sunday | 1 | Planning thread |

### Rest Days

Taking breaks is fine:

```
✅ Consistency matters more than daily posting
✅ Quality doesn't require 7 days/week
✅ Occasional breaks don't hurt algorithmic standing
❌ Long absences (2+ weeks) may reduce initial reach
```

---

## Consistency vs. Volume

### The Algorithm Learns Your Pattern

```
Consistent posting → Algorithm learns when to check for your content
Erratic posting → Less predictable distribution

Choose a sustainable cadence you can maintain.
```

### Quality Trade-offs

| Approach | Pros | Cons |
|----------|------|------|
| Daily posting | Consistency, more shots | Quality may suffer |
| 3x/week | Higher quality per post | Less visibility |
| 1x/week | Maximum quality | Easy to forget about you |

**Recommendation:** 5-7 posts/week, evenly spaced.

---

## Quick Reference

### Posting Checklist

- [ ] Has it been 3+ hours since my last post?
- [ ] Is this peak time for my audience?
- [ ] Will I be available to engage for the first hour?
- [ ] Is this a quality post worth the author slot?

### Timing Rules

| Rule | Details |
|------|---------|
| Space posts 3-4+ hours | Avoid diversity penalty |
| Post at peak times | Check your analytics |
| Be present first hour | Early engagement matters |
| Threads > multiple posts | Better use of author slot |
| Consistency > volume | Sustainable cadence wins |

---

**Next:** [Avoiding Penalties →](05-avoiding-penalties.md)
