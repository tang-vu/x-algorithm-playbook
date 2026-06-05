# Filter System Reference

> Complete guide to the 12 filters that can hide your content before scoring.

---

## How Filtering Works

Before your post is even scored, it passes through filters:

```
Your Post → FILTERS → Scoring → Ranking → Feed
              │
              └── Removed posts never get scored
```

**Key insight:** A filtered post gets ZERO visibility, regardless of quality.

> ✅ **Confirmed in the May 2026 release.** The repo applies **10 pre-scoring filters** (`DropDuplicates`, `CoreDataHydration`, `Age`, `Selfpost`, `RepostDeduplication`, `IneligibleSubscription`, `PreviouslySeen`, `PreviouslyServed`, `MutedKeyword`, `AuthorSocialgraph`) and **2 post-selection filters** (`VFFilter`, `DedupConversation`) — the 12 below.

---

## The 12 Filters

### Filter 1: Drop Duplicates Filter

**What it does:** Removes duplicate tweet IDs from the candidate pool.

**When it triggers:**
- Same tweet appears from multiple sources

**How to avoid:** N/A (system deduplication)

**Source:** `drop_duplicates_filter.rs`

---

### Filter 2: Core Data Hydration Filter

**What it does:** Removes posts missing essential data (author_id, tweet_text).

**When it triggers:**
- Technical issues with post data
- Corrupted or incomplete posts

**How to avoid:** N/A (technical filter)

**Source:** `core_data_hydration_filter.rs`

---

### Filter 3: Age Filter ⚠️

**What it does:** Removes posts older than maximum age threshold.

**When it triggers:**
- Post is too old

**How to avoid:**
- Post at peak times for immediate engagement
- Build engagement quickly after posting

**Source:** `age_filter.rs`

---

### Filter 4: Self Tweet Filter

**What it does:** Removes your own tweets from your For You feed.

**When it triggers:**
- You see your own tweet candidate

**How to avoid:** N/A (expected behavior)

**Source:** `self_tweet_filter.rs`

---

### Filter 5: Retweet Deduplication Filter

**What it does:** Keeps only first occurrence when multiple people retweet same content.

**When it triggers:**
- Same original content retweeted by multiple followed accounts

**How to avoid:** N/A (expected behavior)

**Source:** `retweet_deduplication_filter.rs`

---

### Filter 6: Ineligible Subscription Filter

**What it does:** Removes subscription-only posts from non-subscribers.

**When it triggers:**
- Post is behind subscription paywall
- Viewer isn't subscribed to creator

**How to avoid:** N/A (subscription feature)

**Source:** `ineligible_subscription_filter.rs`

---

### Filter 7: Previously Seen Posts Filter

**What it does:** Removes posts the user has already seen.

**When it triggers:**
- User already saw this post

**How to avoid:** N/A (expected behavior)

**Source:** `previously_seen_posts_filter.rs`

---

### Filter 8: Previously Served Posts Filter

**What it does:** Removes posts already served in current session.

**When it triggers:**
- Post was already shown in current scroll session

**How to avoid:** N/A (expected behavior)

**Source:** `previously_served_posts_filter.rs`

---

### Filter 9: Muted Keyword Filter ⚠️ IMPORTANT

**What it does:** Removes posts containing keywords the user has muted.

**When it triggers:**
- Post contains any of user's muted words/phrases

**How to avoid:**
- Avoid commonly muted spam keywords
- Stay away from controversial trigger words
- Don't use engagement bait phrases

**Common muted keywords:**
```
Promotional:
- "DM me"
- "Link in bio"
- "Follow for follow"
- "Giveaway"

Crypto/Scam:
- "100x"
- "Not financial advice"
- "Easy money"

Engagement Bait:
- "Like if you agree"
- "Retweet to win"

Political (varies by user):
- Various political terms
```

**Source:** `muted_keyword_filter.rs`

---

### Filter 10: Author Socialgraph Filter ⚠️ CRITICAL

**What it does:** Removes posts from authors the user has blocked or muted.

**When it triggers:**
- Viewer has blocked you
- Viewer has muted you

**How to avoid:**
- Don't spam or annoy users
- Stay on-topic
- Engage authentically
- Don't mass-tag or mass-reply

**Source:** `author_socialgraph_filter.rs`

---

### Filter 11: VF Filter (Visibility Filtering) ⚠️ CRITICAL

**What it does:** Removes posts flagged for safety issues.

**Categories filtered:**
- Spam
- Violence
- Gore
- Deleted posts
- ToS violations

**When it triggers:**
- Content violates X policies
- Automated spam detection
- User reports

**How to avoid:**
- Stay within Terms of Service
- Don't post harmful content
- Avoid spam patterns
- Don't try to game the system

**Source:** `vf_filter.rs`

---

### Filter 12: Dedup Conversation Filter

**What it does:** Keeps only highest-scored post per conversation thread.

**When it triggers:**
- Multiple posts from same conversation thread
- Multiple branches of same discussion

**How to avoid:** N/A (expected behavior)

**Source:** `dedup_conversation_filter.rs`

---

## Filter Priority

Filters that YOU can control:

| Priority | Filter | Control Level |
|----------|--------|---------------|
| 🔴 High | Author Socialgraph (block/mute) | Avoid being blocked |
| 🔴 High | VF Filter (safety) | Stay within ToS |
| 🟡 Medium | Muted Keyword | Avoid spam words |
| 🟢 Low | Age Filter | Post at peak times |

---

## Pre-Scoring vs Post-Scoring Filters

### Pre-Scoring (Before ML Model)
```
1. Drop Duplicates
2. Core Data Hydration
3. Age
4. Self Tweet
5. Retweet Deduplication
6. Ineligible Subscription
7. Previously Seen
8. Previously Served
9. Muted Keyword
10. Author Socialgraph
```

### Post-Scoring (After Selection)
```
11. VF Filter (visibility/safety)
12. Dedup Conversation
```

---

## Filter Avoidance Checklist

```
□ Not using spam keywords
□ Not engaging in ways that trigger blocks
□ Compliant with X Terms of Service
□ Not posting duplicate content
□ Posting at active times (avoid age filter)
□ Authentic engagement (avoid spam detection)
```

---

**Next:** [Algorithm FAQ →](algorithm-faq.md)
