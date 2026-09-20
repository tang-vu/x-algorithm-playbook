---
description: "Which content formats and hooks maximize your algorithm score — real X algorithm weights for video, images, links, and dwell time."
---

# Content Optimization

> How to create content that maximizes your algorithm score.
>
> *Last verified: **September 20, 2026** against [xai-org/x-algorithm](https://github.com/xai-org/x-algorithm) @ [`8b25829`](https://github.com/xai-org/x-algorithm/commit/8b25829717a4f104dd04403ee7d0253c5fedb1b7) (Sep 18, 2026 release).*

---

## The Content Hierarchy

Based on the **real published weights** (`params/param.rs`), prioritize content that generates:

```text
1. 🔗 Copy-link shares  (20.0 — the single biggest weight)
2. 💬 Replies           (5.0 — and 20.0 on mutual-follow original posts)
2. 📩 DM shares          (5.0)
2. 🔄 Quote Tweets       (5.0)
3. 👤 Follows            (4.0)
4. 📤 Shares             (2.0)
5. 🔁 Retweets           (1.0)
6. ❤️ Likes              (0.5)
7. ⏱️ Dwell Time         (0.004 continuous + 0.05 binary)
8. 🖱️ Clicks             (0.4 / 0.2 / 0.07 / 0.05 family)
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

**Best for:** Copy-link shares (20.0!), DM shares (5.0), Retweets, Bookmarks

| Type | Example | Why It Works |
|------|---------|--------------|
| Threads | "10 lessons from [experience]" | High dwell time, saves |
| Tutorials | "How to do X (step-by-step)" | Bookmark-worthy |
| Curated lists | "50 tools for [niche]" | Save for later |
| Data/stats | "[Surprising statistic]" | Easy to share |
| Cheat sheets | "Everything about X in one image" | Visual + useful |
| "Send this" posts | "Send this to someone who [relatable thing]" | Directly targets the 20.0 copy-link weight |

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

```text
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

```text
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
| > 10 seconds | `MinVideoDurationMs=10_000` — the verified gate (VQV heads currently weight 0; video_open = 0.07 is the live term) |
| Native upload | Better reach than YouTube links |
| Captions | Most watch on mute |
| Hook in first 3s | That's your scroll-stop moment |
| Vertical format | Mobile-first consumption |

> Reality check (Aug 2026 weights): the direct video weight is small (video_open 0.07, VQV 0.0). Video earns its keep through **dwell time + shares**, not a dedicated bonus — don't post video for the "video boost," post it because it holds attention.

### Threads

| Best Practice | Reason |
|---------------|--------|
| First tweet is the hook | Determines if people read on |
| 5-15 tweets ideal | Sweet spot for dwell time |
| Number your tweets | "1/10" shows commitment |
| End with value + follow CTA | Capture intent |
| Space within thread | Easier to read |

---

## Content Understanding (grox + Semantic IDs)

> Before any scoring happens, **`grox`** runs classifiers (spam, adult, violent media) and embedders over your post, and Phoenix assigns it a **semantic ID** — residual-quantized codes (6×256) from its multimodal embedding. Same-topic posts share SID prefixes, which is how brand-new posts with zero engagement get matched to the right audience. **There is no manual keyword or hashtag boost.**

**The practical rule:** write so the *machine* understands your topic on the first pass, not just humans.

| ✅ Helps `grox` read you | ❌ Confuses `grox` |
|-------------------------|--------------------|
| One clear topic per post | Three unrelated ideas in one post |
| Plain, specific language ("B2B SaaS churn") | Vague subtweets ("this changes everything 👀") |
| On-topic media that matches the text | Random meme unrelated to the point |
| Consistent vocabulary across your posts | Topic-hopping that smears your embedding |
| Saying the thing directly | Burying the subject in irony/ambiguity |

**Why it pays off:**

```text
Clear post → clean embedding + clean SID → matches the RIGHT audience
             → eligible for Phoenix retrieval + SimClusters + topic surfaces
             → higher P(reply/share) because it reached people who care

Vague post → noisy embedding → matched to no one in particular
             → low engagement → algorithm stops distributing it
```

**Hashtags & keywords:** they are *not* a ranking boost. Their only value now is as **topic signal** that helps `grox` classify you (and for human search). One or two relevant, real-word hashtags help comprehension; stuffing them adds noise and can trip the **muted-keyword filter**. (See [Filter System](../reference/filter-system.md).)

> Bottom line: clarity *is* distribution. The clearer your post's topic, the more reach doors ([Phoenix retrieval, SimClusters, Topics](06-growth-strategies.md#out-of-network-reach-doors)) it can walk through.

---

## Content Calendar Strategy

Based on the Author Diversity Penalty — now with **real values** (decay 0.5, floor 0.25; your Nth post in one feed response gets `0.75×0.5^(N−1) + 0.25`):

| Frequency | Score Impact (real) | Recommendation |
|-----------|---------------------|----------------|
| 1 post/day | 100% | Safe, sustainable |
| 2 posts/day | 62.5% on the lower-scored one | Space 6+ hours apart |
| 3 posts/day | 43.75% on the 3rd | Consider threading instead |
| 4+ posts/day | →25% floor, diminishing returns | Quality suffers |

**Pro tip:** A thread is one published post — one author-position instead of many.

---

## Quick Checklist

Before posting, ask:

- [ ] Does my hook stop the scroll?
- [ ] Will people reply to this?
- [ ] **Would someone copy the link and send it to a friend?** (20.0 — top weight)
- [ ] **Could a classifier tell exactly what this post is about?** (grox + SID clarity)
- [ ] Is this formatted for easy reading?
- [ ] Does the media add value (and match the topic)?
- [ ] Am I providing unique value?
- [ ] Have I spaced this from my last post?
- [ ] Is there a clear engagement hook at the end?

---

**Next:** [Engagement Tactics →](03-engagement-tactics.md)
