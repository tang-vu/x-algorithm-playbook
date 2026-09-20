---
description: "Every filter and visibility rule that can hide your post — the 19-stage pipeline plus the 54-rule visibility-filtering engine."
---

# Filter System Reference

> Complete guide to every layer that can hide your content — pipeline filters **and** the visibility-filtering rule engine, now fully public since the August 13, 2026 release.
>
> *Last verified: **September 20, 2026** against [xai-org/x-algorithm](https://github.com/xai-org/x-algorithm) @ [`8b25829`](https://github.com/xai-org/x-algorithm/commit/8b25829717a4f104dd04403ee7d0253c5fedb1b7) (Sep 18, 2026 release).*

---

## The Two Filtering Layers

Filtering happens in **two separate systems**, and they answer different questions:

```text
┌──────────────────────────────────────────────────────────────────┐
│ LAYER 1 — home-mixer filters (home-mixer/filters/)                │
│ "Is this post eligible for THIS feed THIS time?"                  │
│ Age, duplicates, muted keywords, already-seen, subscription…      │
│ Runs inside the request path, before/after scoring                │
└──────────────────────────────────────────────────────────────────┘
┌──────────────────────────────────────────────────────────────────┐
│ LAYER 2 — visibility-filtering (visibility-filtering/)            │
│ "May this post be shown to this viewer AT ALL?"                   │
│ Safety labels, account state, legal takedowns, NSFW, spam…        │
│ Answers ALLOW / INTERSTITIAL / DROP via a rule engine             │
└──────────────────────────────────────────────────────────────────┘
```

**Key insight:** a dropped post gets zero visibility regardless of quality — and since Aug 2026 the *labeling systems that feed the drops* are public too, so you can see exactly what gets accounts/posts flagged.

---

## Layer 1: Pipeline Filters

### Pre-scoring filters (19 wired, in order)

These run before the ML scorer (`home-mixer/filters/`):

| # | Filter | Removes |
|---|--------|---------|
| 1 | `DropDuplicatesFilter` | Same post returned by multiple sources |
| 2 | `CoreDataHydrationFilter` | Posts whose text/metadata failed to load |
| 3 | `AgeFilter` | **Posts older than 48 hours** (`MAX_POST_AGE`, `params/config.rs`) |
| 4 | `SelfTweetFilter` | Your own posts |
| 5 | `OONRetweetReplyFilter` | Reposts/replies from non-followed accounts; replies with missing parent |
| 6 | `OONNsfwSimclustersFilter` | SimClusters-sourced posts by adult-flagged authors (OON only) |
| 7 | `RetweetDeduplicationFilter` | Repeated reposts of the same post |
| 8 | `IneligibleSubscriptionFilter` | Subscriber-only posts the viewer can't access |
| 9 | `PreviouslySeenPostsFilter` | Posts already shown |
| 10 | `PreviouslySeenPostsBackupFilter` | Same, from a second impressions record |
| 11 | `PreviouslyServedPostsFilter` | Posts served earlier this session |
| 12 | `ViewerMutedKeywordFilter` | Posts matching the viewer's muted keywords (upstream README calls it `MutedKeywordFilter`) |
| 13 | `AuthorSocialgraphFilter` | Posts from accounts the viewer blocks/mutes |
| 14 | `Brazil2026ElectionFilter` | Posts from accounts reported to Brazil's Electoral Court — unless the viewer follows the account |
| 15 | `VideoFilter` | Video posts when the request excludes video |
| 16 | `TopicIdsFilter` | Posts outside requested topics / in excluded topics |
| 17 | `NewUserMinEngagementFilter` | For new accounts: OON posts under an engagement bar (default off) |
| 18 | `InventoryHoldoutFilter` | A configured % held out per post+viewer (default off) |
| 19 | `FavHoldoutFilter` | Per-post impression holdout that scales with the post's fav count — gated by `EnableFavHoldout=false` (default off) |

The upstream README documents 17 of these; the code wires two more: **`Brazil2026ElectionFilter`** (added Aug 14, 2026 — removes posts from accounts reported to Brazil's Electoral Court for the 2026 election unless the viewer follows the account; the first **law-driven filter published in the repo**, list updated Aug 27) and **`FavHoldoutFilter`** (shipped in the same file as the inventory holdout; default off). Expect more jurisdiction-specific rules like Brazil's.

Also on disk for other surfaces/modules: `following_viewer_muted_keyword_filter`, `following_retweet_deduplication_filter`, `self_reply_chain_filter`, `popular_topics_author_dedup_filter`, `push_to_home_dedup_filter`, `invalid_conversation_module_filter`, `result_size_filter`, `ad_adjacent_served_filter`.

### Post-selection filters (3)

After ranking fixes the order:

| Filter | Removes |
|--------|---------|
| `VFFilter` | Posts visibility-filtering answered **DROP** for |
| `AncillaryVFFilter` | ⚠️ Posts whose **parent / quoted / reposted post was itself dropped** — your clean reply dies with a dropped parent |
| `DedupConversationFilter` | Extra branches of the same conversation |

---

## Layer 2: Visibility Filtering (the rule engine)

`visibility-filtering/` returns one of three verdicts per (post, viewer) pair:

```text
ALLOW          → show normally
INTERSTITIAL   → show behind a tap-through warning (adult/graphic media)
DROP           → do not show
```

**How the rules run** (`rules/registry.rs`):

- Rules evaluate **in order**; the **first DROP wins** and ends evaluation.
- Interstitials don't end evaluation — a later drop still removes the post.
- Two policies matter for For You: `TimelineHome` (**28 shared rules**) and `TimelineHomeRecommendations` (shared **+ 26 recommendation-only rules** that fire only for OON posts from accounts you don't follow — and can *only* drop).

### Shared rules (apply to everything, in order)

| Group | Rules (real names) |
|-------|--------------------|
| Author state | `SuspendedAuthorRule`, `DeactivatedAuthorRule`, `ErasedAuthorRule`, `OffboardedAuthorRule`, `ProtectedAuthorDropRule` |
| Viewer relationship | `ViewerBlocksAuthorRule`, `ViewerMutesAuthorRule`, `MutedRetweetsRule` |
| Post labels | `PdnaTweetLabelRule`, `BounceTweetLabelRule`, `SpamTweetLabelRule`, `ForEmergencyUseOnlyDropRule`, `FosnrHatefulConductDropRule`, `FosnrViolentSpeechDropRule`, `FosnrAbuseDropRule`, `FosnrCivicIntegrityDropRule`, `NullcastedTweetDropRule`, `DropStaleTweetsRule` |
| Legal | `DropLegalTakendownPostRule`, `DropLocalLawsTakendownPostRule` (country-law withholding) |
| Sensitive viewers | `SensitiveViewerLoggedOutDropRule`, `SensitiveViewerUnderageDropRule`, `SensitiveViewerNoStatedAgeDropRule` |
| Exclusive content | `DropExclusiveTweetContentRule` |
| Interstitials (last) | `NsfwHighPrecisionInterstitialRule`, `GoreAndViolenceInterstitialRule`, `NsfwCardImageInterstitialRule`, `NsfwAuthorInterstitialRule` |

### Recommendation-only rules (OON posts — 26 more ways to get dropped)

These **only** fire when your post is recommended to a non-follower — the same post stays visible to your followers:

| Theme | Rules |
|-------|-------|
| Media rights/geo | `DropTweetsWithDmcaMediaRule`, `DropTweetsWithGeoRestrictedMediaRule` |
| NSFW author | `DropNsfwUserAuthorRule`, `DropNsfwAdminAuthorRule`, `NsfwNearPerfectAuthorRule`, `NsfwAvatarImageRule`, `NsfwBannerImageRule` |
| NSFW post | `TweetNsfwUserDropRule`, `TweetNsfwAdminDropRule`, `NsfwHighRecallDropRule`, `NsfwHighPrecisionOonDropRule`, `NsfwCardImageOonDropRule`, `NsfwTextTweetLabelDropRule`, `GoreAndViolenceOonDropRule` |
| Spam/abuse | `SpamHighRecallDropRule`, `MaliciousUrlOonDropRule`, `AbusiveHighRecallRule`, `FosnrAbuseInsultsOonDropRule`, `DoNotAmplifyOonDropRule`, `DoNotAmplifyNonFollowerRule` |
| Account labels | `NsfwHighRecallUserLabelRule`, `NsfwHighPrecisionUserLabelRule`, `SpamHighRecallUserLabelRule`, `CompromisedUserLabelRule`, `ReadOnlyUserLabelRule`, `ImpersonationHighPrecisionUserLabelRule` |

**Strategic read:** followers forgive; recommendations don't. Borderline content keeps in-network reach but loses all discovery — an "OON ceiling" that's invisible in your follower engagement.

---

## Where the Labels Come From (Labeling Path)

Labels aren't set by the feed pipeline — they're produced continuously by separate systems, stored, and read back on the request path:

| System | Produces |
|--------|----------|
| `grox/` | Post classifiers — spam, adult, violent media — + text/image embeddings |
| `media-model-proxy/` | Image/video models: adult, violence & gore, hateful symbols, subject matter, known-media matching |
| `clip/` | The image+text embedding model the media classifiers consume |
| `agatha/` | **Account reputation** — blocks, reports, spam reports *relative to favorites*; spam-suspension & adult-content labels |
| `bdsm/` | Inauthentic/abusive behavior from an account's action sequence over time |
| `user-cred-v2/` | PageRank-style credibility score over follow + engagement graph |
| `scarecrow/` + `botmaker/` + `botmaker-rules/scarecrow/` | Event-driven rules: "on this event, if these conditions hold, apply this label" |
| `abuse-enforcement-service/` | Acts on account model scores: label, challenge, or suspend |
| `safety-label-user-agg/` | Labels an account for what *its posts* collected |
| `takedowns/` | Legal/local-law takedown reasons (post + author level) |

Not public: the Grox LLM prompt files and some botmaker rules (deliberately withheld to limit gaming).

---

## Check Your Own Labels: Under the Hood

xAI now ships a transparency tool — **Under the Hood** — that shows aggregate stats on the visibility-impacting labels applied to *your* account and posts. As of Sep 18, 2026 it also reports **legal-compliance withholding** (e.g. which country your post was withheld in).

**Use it:** if your reach collapsed with no obvious cause, check for labels before changing strategy. The builder/serving code is in `under-the-hood/`.

---

## Filter Avoidance Checklist

```text
□ No spam/bait keywords (ViewerMutedKeywordFilter, SpamTweetLabelRule)
□ Nothing that earns blocks/mutes (AuthorSocialgraph, ViewerBlocks/Mutes)
□ Media is clean — no borderline NSFW/gore (interstitials + OON drops)
□ Account standing clean — check Under the Hood monthly
□ No DMCA/geo-restricted media (recommendations drop it)
□ Not replying under content likely to be dropped (AncillaryVFFilter)
□ Fresh post — hard 48h ceiling (AgeFilter)
□ Original posts for reach — replies/RTs get the OON-style discount
```

---

## Source Files

| Component | Where |
|-----------|-------|
| Pipeline filters | `home-mixer/filters/` |
| VF rule engine & registry | `visibility-filtering/` → `rules/registry.rs` |
| VF client & label types | `visibility-filtering-client/` |
| Label producers | `grox/`, `media-model-proxy/`, `clip/`, `agatha/`, `bdsm/`, `user-cred-v2/` |
| Label rules/enforcement | `scarecrow/`, `botmaker/`, `botmaker-rules/`, `abuse-enforcement-service/`, `safety-label-user-agg/`, `takedowns/` |
| Your labels report | `under-the-hood/` |

---

**Next:** [Algorithm FAQ →](algorithm-faq.md)
