# Timing & Frequency

> When to post and how often, based on algorithm mechanics.

---

## The Author Diversity Penalty

The most important timing factor in the algorithm.

### How It Works

When multiple posts from the same author appear in one feed session:

```
Post #1: 100% of calculated score
Post #2: ~76% of calculated score
Post #3: ~59% of calculated score
Post #4: ~47% of calculated score
...
Post #N: Converges to ~20% floor
```

**Formula:**
```
multiplier = (1 - floor) × decay^position + floor
```

### What This Means

| Posting Frequency | Impact |
|-------------------|--------|
| 1 post/day | Optimal per-post score |
| 2 posts/day | 2nd post significantly penalized |
| 3+ posts/day | Rapidly diminishing returns |
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

Threads are the exception to frequency limits:

```
Thread = 1 "author slot" regardless of length

Benefits:
├── High dwell time (people read through)
├── No diversity penalty for thread tweets
├── Multiple engagement points
└── Easy to bookmark/save
```

---

## The Age Filter

Posts older than a threshold are filtered out entirely.

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
