# Common Mistakes

> Mistakes that hurt your algorithm score, based on how the system actually works.

---

## Mistake 1: Posting Too Frequently

### The Problem

```
Author Diversity Penalty (per-response, by score-rank):
Your top post: 100%   →  ~76%  →  ~59%  →  ~47%  …→ ~20% floor
(percentages illustrative — real decay/floor are redacted)
```

### Real Example

**Bad approach:**
```
9:00 AM - Tweet about topic A
9:15 AM - Tweet about topic B
9:30 AM - Tweet about topic C
9:45 AM - Tweet about topic D

Result: your posts compete in the same feeds and your lower-scored ones get decayed
```

**Better approach:**
```
9:00 AM - Quality tweet
3:00 PM - Quality tweet
9:00 PM - Quality tweet

Result: Each post gets full scoring potential
```

### How to Fix

- Space posts 3-4+ hours apart
- Use threads instead of multiple posts
- Quality over quantity

**Source:** `author_diversity_scorer.rs`

---

## Mistake 2: Ignoring Negative Signals

### The Problem

| Your gains | Their cost |
|------------|-----------|
| 10 likes | 1 block = net negative |
| 20 likes | 1 report = net negative |

### Real Example

**Scenario:** You post controversial content for engagement
```
Result: 
+ 500 likes
+ 100 retweets
- 10 blocks
- 2 reports

Math: The blocks and reports may outweigh the positive engagement
```

### How to Fix

- Avoid content that triggers blocks (spam, harassment)
- Stay within ToS (avoid reports)
- Controversy for engagement is risky math

**Source:** `weighted_scorer.rs` - negative weights

---

## Mistake 3: Weak Hooks

### The Problem

The algorithm predicts engagement probability. A weak hook = low P(dwell), P(reply), P(like).

### Real Examples

**❌ Weak hooks:**
```
"Just wanted to share some thoughts..."
"New blog post!"
"Thoughts on X?"
"Happy Monday everyone!"
```

**✅ Strong hooks:**
```
"I made $100K from one tweet. Here's exactly how:"
"Unpopular opinion: [bold claim]"
"90% of people get [topic] wrong. Here's why:"
"The biggest mistake I see in [niche]:"
```

### How to Fix

- Lead with value or curiosity
- Be specific, not vague
- Create a reason to stop scrolling

---

## Mistake 4: No Engagement Hook

### The Problem

Posts without engagement invitation = lower P(reply), P(quote).

Reply is the top-weighted positive signal (well above a like).

### Real Example

**❌ No hook:**
```
"Just finished reading an interesting book about productivity."

[Ends with period. No invitation to engage.]
```

**✅ With hook:**
```
"Just finished reading an interesting book about productivity.

What's the best productivity book you've ever read? Looking for my next one 👇"

[Explicit invitation to reply]
```

### How to Fix

- End with a question
- Ask for opinions
- Invite specific responses
- Use 👇 or "Let me know"

---

## Mistake 5: Buying Followers

### The Problem

```
Fake/inactive followers:
├── Never engage with your content
├── Algorithm sees low engagement rate
├── Future posts get lower distribution
└── Negative spiral
```

### The Math

```
Account A: 1,000 real followers, 50 likes/post = 5% engagement
Account B: 10,000 fake followers, 50 likes/post = 0.5% engagement

Algorithm prefers Account A's engagement rate
```

### How to Fix

- Grow organically
- Focus on follower quality
- Better to have 1,000 engaged than 10,000 inactive

---

## Mistake 6: Off-Topic Content

### The Problem

The Two-Tower retrieval builds your embedding from your content history. Inconsistent topics = confused embedding = poor matching.

### Real Example

**❌ Confused embedding:**
```
Monday: Tech startup tips
Tuesday: Recipe for pasta
Wednesday: Political hot take
Thursday: Fitness advice
Friday: Crypto prediction

Result: Algorithm can't figure out who to show your content to
```

**✅ Clear embedding:**
```
Monday: SaaS growth tactics
Tuesday: Startup hiring tips
Wednesday: Founder mindset
Thursday: Fundraising insights
Friday: Product development

Result: Clear signal → matched to startup/SaaS audience
```

### How to Fix

- Pick 1-3 related topics
- Stay consistent
- If you have multiple interests, consider separate accounts

**Source:** `phoenix/recsys_retrieval_model.py`

---

## Mistake 7: Engagement Bait

### The Problem

Common engagement bait phrases often get:
1. Muted by users (muted keyword filter)
2. Flagged as spam (VF filter)
3. Trigger "Not interested" (negative signal)

### Examples to Avoid

```
❌ "Like if you agree!"
❌ "Retweet for good luck"
❌ "Follow for follow"
❌ "Drop a 🔥 if you want more"
```

### How to Fix

- Earn engagement through value
- Ask genuine questions
- Create shareable insights naturally

**Source:** `muted_keyword_filter.rs`, `vf_filter.rs`

---

## Mistake 8: Ignoring Replies

### The Problem

Not replying to comments:
1. Misses engagement opportunity (reply has high weight)
2. Signals low interest in community
3. People stop engaging

### Real Impact

```
Post with 50 comments, 0 author replies:
├── Conversation dies
├── Future engagement decreases
└── Algorithm sees declining interest

Post with 50 comments, 30 author replies:
├── Thread activity spikes
├── More people join conversation
└── Algorithm sees high engagement
```

### How to Fix

- Reply to every comment (especially first hour)
- Ask follow-up questions
- Keep conversations going
- Show you care about community

---

## Mistake 9: Link-First Posts

### The Problem

Posts that are just links:
1. Take people off-platform (less engagement)
2. No dwell time on the tweet
3. Low native engagement

### Real Example

**❌ Link-first:**
```
"Check out my new article: [link]"

Result: Click = leave X = no further engagement tracked
```

**✅ Value-first:**
```
"I spent 40 hours researching [topic]. Here are the 5 key insights:

1. [Insight]
2. [Insight]
3. [Insight]
4. [Insight]
5. [Insight]

Full article with data: [link]"

Result: Dwell time + potential replies + then maybe click
```

### How to Fix

- Lead with value, link at end
- Summarize key points in tweet
- Make the tweet standalone valuable

---

## Mistake 10: Posting at Dead Times

### The Problem

Age Filter removes old posts. Posts at dead times:
1. Get less initial engagement
2. Age out before peak hours
3. Never get momentum

### Real Example

```
Post at 3 AM:
├── 0 engagement first 4 hours
├── By peak time, post is "old"
├── Less distribution

Post at 9 AM:
├── Immediate engagement
├── Builds momentum
├── Algorithm amplifies
```

### How to Fix

- Know your audience's timezone
- Post at peak engagement hours
- Check analytics for your best times

**Source:** `age_filter.rs`

---

## Mistake Summary Table

| Mistake | Algorithm Impact | Fix |
|---------|------------------|-----|
| Post too frequently | Diversity penalty | Space 3-4+ hours |
| Trigger blocks | Strong negative weight | Stay authentic |
| Weak hooks | Low engagement prediction | Lead with value |
| No engagement hook | Low P(reply) | End with question |
| Buy followers | Low engagement rate | Grow organically |
| Off-topic content | Confused embedding | Niche down |
| Engagement bait | Muted/filtered | Earn engagement |
| Ignore replies | Miss engagement | Reply actively |
| Link-first posts | Low dwell time | Value first |
| Dead time posting | Age filter | Peak times |

---

**Back to:** [Rules](../rules/) | [Checklists](../checklists/)
