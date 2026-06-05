# Content Optimization

> How to create content that maximizes your algorithm score.

---

## The Content Hierarchy

Based on algorithm weights, prioritize content that generates:

```
1. 💬 Replies (highest weight)
2. 🔄 Quote Tweets
3. 👤 Profile Visits → Follows
4. 🔁 Retweets
5. ❤️ Likes
6. ⏱️ Dwell Time
7. 🖱️ Clicks
```

---

## Content Types Ranked

### Tier 1: Conversation Starters

**Best for:** Replies, Quote Tweets

| Type | Example | Why It Works |
|------|---------|--------------|
| Questions | "What's your hot take on X?" | Directly invites replies |
| Controversial opinions | "Unpopular opinion: [take]" | Triggers agree/disagree |
| Fill-in-the-blank | "The best [topic] is ___" | Low friction to reply |
| Polls + discussion | "Vote + explain in replies" | Double engagement |
| Debates | "A or B? Defend your choice" | Creates camps |

### Tier 2: Shareable Value

**Best for:** Retweets, Bookmarks, Shares

| Type | Example | Why It Works |
|------|---------|--------------|
| Threads | "10 lessons from [experience]" | High dwell time, saves |
| Tutorials | "How to do X (step-by-step)" | Bookmark-worthy |
| Curated lists | "50 tools for [niche]" | Save for later |
| Data/stats | "[Surprising statistic]" | Easy to share |
| Cheat sheets | "Everything about X in one image" | Visual + useful |

### Tier 3: Authority Builders

**Best for:** Follows, Profile Clicks

| Type | Example | Why It Works |
|------|---------|--------------|
| Original insights | "[Unique perspective on industry]" | Shows expertise |
| Predictions | "Here's what I think will happen" | Positioning |
| Behind-the-scenes | "How I [achieved X]" | Personal + valuable |
| Lessons learned | "I made this mistake so you don't" | Relatable expertise |

---

## Hook Optimization

The first line determines if people stop scrolling.

### High-Performing Hook Formulas

| Formula | Example |
|---------|---------|
| Contrarian | "Everyone is wrong about [topic]" |
| Promise | "This one change 10x'd my [metric]" |
| Curiosity gap | "I spent 100 hours studying [X]. Here's what I found:" |
| Direct value | "Save this for later:" |
| Story | "Last week, something crazy happened..." |
| Challenge | "I bet you can't [do X]" |
| Specificity | "3 things I learned from [specific experience]" |

### Hooks to Avoid

| ❌ Weak Hook | ✅ Better Version |
|-------------|-------------------|
| "Thoughts on X?" | "The biggest mistake people make with X:" |
| "Check out this article" | "Key insight from [article]: [insight]" |
| "New blog post!" | "[Specific takeaway from blog post]" |
| "Happy Monday!" | [Don't post this] |

---

## Format Optimization

### For Dwell Time

```
✅ DO:
• Use line breaks (spacing helps readability)
• Use bullet points
• Use emojis sparingly as visual markers
• Create scannable structure
• Thread long content (keeps them reading)

❌ DON'T:
• Wall of text
• No formatting
• Bury the value
```

### For Engagement

```
✅ DO:
• End with question or CTA
• Tag relevant people (sparingly)
• Use images that add value
• Reply to your own tweet with additional context

❌ DON'T:
• End with period (full stop)
• Tag randomly for reach
• Use stock photos
• Ignore the replies
```

---

## Media Guidelines

### Images

| ✅ Do | ❌ Don't |
|-------|---------|
| Screenshots (tweets, data) | Generic stock photos |
| Diagrams and charts | Low-resolution images |
| Memes (if on-brand) | Too much text in image |
| Before/after | Irrelevant visuals |
| Carousels for multi-point content | Single slides with paragraphs |

### Videos

| Requirement | Why |
|-------------|-----|
| > 10 seconds | VQV weight only applies above minimum duration |
| Native upload | Better reach than YouTube links |
| Captions | Most watch on mute |
| Hook in first 3s | That's your scroll-stop moment |
| Vertical format | Mobile-first consumption |

### Threads

| Best Practice | Reason |
|---------------|--------|
| First tweet is the hook | Determines if people read on |
| 5-15 tweets ideal | Sweet spot for dwell time |
| Number your tweets | "1/10" shows commitment |
| End with value + follow CTA | Capture intent |
| Space within thread | Easier to read |

---

## Content Understanding (grox)

> 🆕 **May 2026 update.** Before any scoring happens, a content-understanding service called **`grox`** runs **classifiers and embedders** over your post. It decides *what your post is about* — the topic labels, the embedding used for retrieval, and the safety signals. The ranking model then learns relevance from engagement; **there is no manual keyword or hashtag boost.**

**The practical rule:** write so the *machine* understands your topic on the first pass, not just humans.

| ✅ Helps `grox` read you | ❌ Confuses `grox` |
|-------------------------|--------------------|
| One clear topic per post | Three unrelated ideas in one post |
| Plain, specific language ("B2B SaaS churn") | Vague subtweets ("this changes everything 👀") |
| On-topic media that matches the text | Random meme unrelated to the point |
| Consistent vocabulary across your posts | Topic-hopping that smears your embedding |
| Saying the thing directly | Burying the subject in irony/ambiguity |

**Why it pays off:**

```
Clear post → clean embedding → matches the RIGHT audience
             → also eligible for Phoenix Topics + MoE discovery
             → higher P(reply/like) because it reached people who care

Vague post → noisy embedding → matched to no one in particular
             → low engagement → algorithm stops distributing it
```

**Hashtags & keywords:** they are *not* a ranking boost. Their only value now is as **topic signal** that helps `grox` classify you (and for human search). One or two relevant, real-word hashtags help comprehension; stuffing them adds noise and can trip the **muted-keyword filter**. (See [Filter System](../reference/filter-system.md).)

> Bottom line: clarity *is* distribution. The clearer your post's topic, the more reach doors ([Phoenix Topics, MoE](06-growth-strategies.md#may-2026-reach-paths)) it can walk through.

---

## Content Calendar Strategy

Based on the Author Diversity Penalty (per-response decay by score-rank; example impacts below are illustrative, since the real decay/floor are [redacted](../reference/action-weights.md#the-exact-weight-values-are-redacted)):

| Frequency | Score Impact (illustrative) | Recommendation |
|-----------|-----------------------------|----------------|
| 1 post/day | ~100% | Safe, sustainable |
| 2 posts/day | ~76% on the lower-scored one | Space 6+ hours apart |
| 3 posts/day | ~59% on the 3rd | Consider threading instead |
| 4+ posts/day | Diminishing returns | Quality suffers |

**Pro tip:** A thread is one published post — one author-position instead of many.

---

## Quick Checklist

Before posting, ask:

- [ ] Does my hook stop the scroll?
- [ ] Will people reply to this?
- [ ] **Could a classifier tell exactly what this post is about?** (grox clarity)
- [ ] Is this formatted for easy reading?
- [ ] Does the media add value (and match the topic)?
- [ ] Am I providing unique value?
- [ ] Have I spaced this from my last post?
- [ ] Is there a clear engagement hook at the end?

---

**Next:** [Engagement Tactics →](03-engagement-tactics.md)
