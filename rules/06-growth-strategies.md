# Growth Strategies

> Long-term strategies for sustainable growth based on algorithm mechanics.

---

## The Two Audiences

Understanding how the algorithm finds you new followers:

### In-Network (Followers)

```
Your post → Directly to followers' For You feeds
Advantage: No OON penalty, direct delivery
Strategy: Keep them engaged, avoid unfollows
```

### Out-of-Network (Discovery)

```
Your post → Phoenix Two-Tower Retrieval / SimClusters → Matched to similar users
Challenge: verified ×0.75 discount (OonWeightFactor; ×0.5 on topic surfaces)
Strategy: Strong engagement signals, clear niche
```

> Note: since Aug 2026, in-network **replies and retweets also take the ×0.75** (`EnableOonRescoreForInNetworkRepliesRetweets=true`) — only *original posts* get the full in-network score.

---

## The Two-Tower Retrieval System

How the algorithm decides to show your content to non-followers:

```
┌─────────────────────────────────────────────────────────────────┐
│                    TWO-TOWER RETRIEVAL                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  USER TOWER                       CANDIDATE TOWER               │
│  ├── User engagement history      ├── Your post content         │
│  ├── Topics they engage with      ├── Your post features        │
│  └── → User Embedding [D]         └── → Post Embedding [D]      │
│              │                              │                    │
│              └──────── Dot Product ─────────┘                    │
│                          │                                       │
│                   Similarity Score                               │
│                          │                                       │
│              Top-K Similar Posts Selected                        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Implications for Growth

| Factor | Strategy |
|--------|----------|
| Your embedding is built from your history | Be consistent in your topics |
| User embeddings are built from their history | Target people who engage with similar content |
| Dot product similarity | The clearer your niche, the stronger the match |

---

## Out-of-Network Reach Doors

> Updated for the Aug–Sep 2026 code. There are **multiple out-of-network doors**, not just generic retrieval — each rewards slightly different things, and one new door (SimClusters) joined in August.

| Door | What gets you in | How to optimize |
|------|------------------|-----------------|
| **Phoenix Retrieval** | Embedding similarity between your post's semantic ID and a user's engagement history | Strong, consistent niche embedding |
| **SimClusters** 🆕 | Your account/post clusters with accounts the user already engages with | Consistent niche → you cluster with the right neighborhood |
| **Phoenix Topics** | Your post maps cleanly to a topic the user engages with | One sharp topic per post; let `grox` classify you correctly |
| **Phoenix MoE** | A specialist "expert" matches your interest area | ⚠️ off by default (`EnablePhoenixMOESource=false`) — A/B experiment, don't rely on it |
| **Who-to-Follow** | You're suggested as an account worth following | High follow-through rate, clear profile + bio, consistent value |
| **Thunder (in-network)** | Followers, instantly | Keep existing followers engaged; avoid unfollows/mutes |

### What changed strategically

```
OLD mental model:  "go viral in retrieval or stay invisible"
NEW mental model:  several doors, each opened by TOPIC CLARITY + CONSISTENCY
  • Phoenix retrieval reads your post's semantic ID (embedding identity)
  • SimClusters reads WHO engages with you (engagement-neighborhood identity)

→ A mid-engagement post on a SHARP topic can still get OON reach
  via Topics / cluster match, even if generic retrieval would skip it.
→ Topic-hopping smears your embedding AND your cluster — shuts ALL the doors.
```

### How to play it

1. **Pick a topic per post and commit to it** — clarity is what `grox`, SIDs, and Topics read.
2. **Engage inside your niche's neighborhood** — SimClusters groups you by who engages with you; replies/engagement from in-niche accounts strengthen the right cluster.
3. **Optimize your profile for Who-to-Follow** — clear bio, consistent feed, obvious "what I'm about."
4. **Don't dilute** — one off-topic post weakens the signal every door depends on.

See [Scoring System → Where Candidates Come From](01-scoring-system.md#where-candidates-come-from) for how these sources feed the pipeline.

---

## Niche Strategy

### Why Niching Down Works

```
Broad topics → Weak embedding signal → Poor matching
Specific niche → Strong embedding signal → Better matching

Example:
❌ "Business tips" (too broad)
✅ "SaaS pricing strategies" (specific)
```

### Building Topical Authority

| Level | Example | Strategy |
|-------|---------|----------|
| Category | Tech | Too broad |
| Niche | Startups | Better |
| Sub-niche | B2B SaaS | Ideal |
| Micro-niche | SaaS pricing | Very strong signal |

### Multi-Topic Approach

If you must cover multiple topics:

```
✅ Related topics (SaaS + Pricing + Growth)
❌ Unrelated topics (SaaS + Cooking + Politics)

The algorithm will struggle to match unrelated content.
```

---

## Follower Quality Over Quantity

### Why Quality Matters

```
100 engaged followers > 10,000 inactive followers

Because:
├── In-network engagement boosts your posts
├── No engagement = no signal = algorithm ignores
├── Low engagement rate hurts future distribution
└── Fake followers = negative signal when they don't engage
```

### Building Quality Followers

| ✅ Do | ❌ Don't |
|-------|---------|
| Create for your ideal follower | Chase vanity metrics |
| Engage with target audience | Buy followers |
| Convert through value | Follow-for-follow |
| Let them find you organically | Force growth |

---

## The Engagement Flywheel

Sustainable growth comes from compound engagement:

```
Great Content
    │
    ▼
Engagement (replies, retweets, quotes)
    │
    ▼
Algorithm boosts reach
    │
    ▼
New people discover you
    │
    ▼
Some follow
    │
    ▼
Larger in-network audience
    │
    ▼
More engagement on next post
    │
    └──────────────────────────┐
                               ▼
                        Repeat cycle
```

### Breaking the Cold Start

🆕 **Verified new-author boost** (`author_cold_start.rs`, Aug 2026): the algorithm *reserves a slot* for small accounts. If your post is:

- an **original post** (not a reply or repost), AND
- from an author with **≤1,000 followers**, AND
- **≤48 hours old** with **<1,000 Home impressions**,

…then the single best eligible post can be **lifted straight to ~feed slot 15–16** (`ColdStartSlotMin=15/Max=16`, `EnableViewerColdStart=true`). Every original post while you're small is a boosted lottery ticket — the algorithm is literally looking for new voices to test.

```
1. Engage heavily with people in your niche
2. Provide massive value in replies
3. Build relationships with similar-sized accounts
4. Get mentioned/quoted by others
5. Post ORIGINAL content while under 1k followers (cold-start boost)
6. Slowly build in-network base
```

---

## Content Pillars Strategy

### The 4-Pillar Framework

| Pillar | Purpose | Frequency |
|--------|---------|-----------|
| Educational | Build authority | 40% |
| Engagement | Drive conversation | 30% |
| Personal | Build connection | 20% |
| Promotional | Convert followers | 10% |

### Example Content Mix

```
Monday: Educational thread
Tuesday: Question (engagement)
Wednesday: Personal story
Thursday: Industry insight (educational)
Friday: Fun/casual content
Saturday: Rest or engagement
Sunday: Educational thread
```

---

## Viral Mechanics

### What Makes Content Spread

From the algorithm's perspective, viral content has (real weights):

```
High P(copy_link_share ·20.0) + High P(reply ·5.0) + High P(quote ·5.0)
  + High P(DM share ·5.0) + High P(retweet ·1.0)
  + Low P(report ·−234 / mute ·−58.8 / not_interested ·−43.2 / block ·−31.2)
```

### Viral Content Types

| Type | Why It Works |
|------|--------------|
| Contrarian takes | Triggers debate (replies, quotes) |
| Relatable struggles | "Me too" responses |
| Surprising data | "Did you know?" shares |
| Useful frameworks | Bookmark + share |
| Story threads | Dwell time + engagement |

### Viral Cautions

```
⚠️ Viral doesn't always mean good
⚠️ Controversial viral → blocks/mutes
⚠️ One viral post ≠ sustained growth
⚠️ Viral for wrong audience = wrong followers
```

---

## Long-Term Strategy

### The 1-Year Playbook

| Phase | Months | Focus |
|-------|--------|-------|
| Foundation | 1-3 | Niche, voice, consistency |
| Growth | 4-6 | Engagement, community |
| Authority | 7-9 | Thought leadership |
| Monetization | 10-12 | Convert following to value |

### Metrics That Matter

| Metric | What It Tells You |
|--------|-------------------|
| Engagement rate | Content resonance |
| Reply ratio | Conversation quality |
| Follower growth rate | Sustainable growth |
| Profile visits → follows | Conversion |
| Negative action rate | Content safety |

---

## Quick Wins

### Immediate Actions

1. **Clarify your niche** - What are you THE person for?
2. **Audit your content** - Does it match your niche?
3. **Engage daily** - 30 min of meaningful engagement
4. **Post consistently** - 1 quality post/day
5. **Reply to replies** - Keep conversations going

### 30-Day Challenge

```
Week 1: Establish posting consistency
Week 2: Focus on engagement hooks
Week 3: Build relationships with 10 peers
Week 4: Analyze what worked, double down
```

---

**Next:** See [Checklists](../checklists/) for actionable daily workflows.
