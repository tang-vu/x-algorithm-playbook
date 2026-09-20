# What's New — August–September 2026 Releases

> The follow-up drops to the May 2026 release are the biggest yet: the **actual scoring weights are now public**, the entire **visibility-filtering and labeling stack** shipped, and the **production Phoenix model code** replaced the demo. This page covers the Aug 13, Aug 14, and Sep 18, 2026 dev notes (plus the July mutual-follow boost they documented).

---

## TL;DR

- 🔓 **The real weights are published** — `home-mixer/params/param.rs` now carries production defaults. Copy-link share = **20.0**, reply/quote/DM-share = **5.0**, like = **0.5**; report = **−234**, mute = **−58.8**, not-interested = **−43.2**, block = **−31.2**.
- 🚫 **Mute > block.** The old severity guess was wrong — mutes and "not interested" outweigh blocks per-weight.
- 🤝 **Mutual follows get reply weight 20** — the July "bidirectional boost" is live at +15.
- 🆕 **New-author boost is real code** — original posts from ≤1k-follower accounts can be lifted to ~slot 15–16.
- 👁️ **Visibility filtering fully open** — 28 shared + 26 recommendation-only rules, named, in `visibility-filtering/`.
- 🧩 **New OON source: SimClusters** — joins Phoenix retrieval; Phoenix MoE exists but is off by default.
- 🏭 **Phoenix is now the production model** — real JAX training + Rust serving + synthetic data; no more demo artifacts.
- 🗳️ **First law-driven filter** — `Brazil2026ElectionFilter`.
- 🔎 **Under the Hood** — you can now *see the labels on your own account*, including legal withholding by country (Sep 18).

**One-line takeaway:** *The guessing era is over — weights, filters, and labels are all auditable now. Optimize the real numbers, not folklore.*

---

## What Actually Changed

### August 13, 2026 — the transparency drop

**1. Production parameters published.** `home-mixer/params/param.rs` now ships the real config — all action weights, `OonWeightFactor` (0.75), `TopicOonWeightFactor` (0.5), `AuthorDiversityDecay` (0.5) / `Floor` (0.25), `MinVideoDurationMs` (10,000), cold-start thresholds, VMRanker settings, and every source/filter toggle. Cron jobs keep the defaults synced to production.

**2. The visibility stack shipped.** `visibility-filtering/` (the ALLOW/INTERSTITIAL/DROP rule engine) plus everything that produces its labels: `grox` classifiers, `media-model-proxy` + `clip` (media models), `agatha`/`bdsm`/`user-cred-v2` (account reputation), `scarecrow`/`botmaker`/`botmaker-rules` (event rules), `abuse-enforcement-service`, `safety-label-user-agg`, `takedowns`.

**3. SimClusters joined the sources.** `simclusters/` clusters accounts and posts by who-engages-with-what and feeds candidates alongside Thunder and Phoenix retrieval.

**4. Phoenix got real.** The demo transformer was replaced with the **production implementation**: JAX training, the Rust gRPC serving engine, checkpoint format, and synthetic data generators so you can run a real end-to-end training run. Production ranking config: emb 2560, 8 layers, GQA 20/4, history 1022, 64-action taxonomy, semantic IDs (6×256 codes) derived from each post's multimodal embedding.

**5. Pipeline structure clarified.** The **Post Pipeline** (`PhoenixCandidatePipeline`) does source → filter → score → select. A wrapping **Blending Pipeline** (`ForYouCandidatePipeline`) interleaves ads, Who-to-Follow, and prompts. `VMRanker` reranks via `vm-ranker/` (determinantal point process — sacrifices score for neighbor diversity).

### August 14, 2026 — weights explained + Brazil

- Official correction on how to read weights: **they scale predicted probabilities, not engagement counts** — so "report is 468× a like" does *not* mean one report erases 468 likes. `P(report)` is >1000× rarer at baseline, and bad-actor mass reporting mostly affects users similar to the bad actors.
- `Brazil2026ElectionFilter` went live: posts from accounts reported to Brazil's Electoral Court are dropped for non-followers (list updated Aug 27).

### September 18, 2026 — transparency tooling

- **Under the Hood** reports now show whether your account/posts were visibility-limited for **legal compliance** — e.g. withheld in a specific country after a legal demand, and *which* country.

### July 2026 (documented retroactively) — the mutual-follow boost

The `docs/BIDIRECTIONAL_BOOST_CHANGE.md` walkthrough: `BidirectionalFollowReplyWeightBoost` A/B'd Jul 10 → broad launch Jul 13 at +20 → tuned to **+15** on Jul 24. Original posts from mutual follows now get reply weight **20** (5.0 + 15.0). The tested dwell-boost variant was not shipped.

---

## What It Means For Your Reach

### 1. Copy-link sharing is the new top lever

`ShareViaCopyLinkWeight = 20.0` — the single largest weight — means the algorithm most rewards the action of someone **copying your post's URL to send it elsewhere** (group chats, DMs, off-platform). Content that gets *sent* beats content that gets liked by 40-to-1 in weight terms. Write things people forward: "send this to someone who…" content, sharp takes people attach their own comment to, reference posts.

### 2. Mutuals are the most boosted relationship

Reply weight for mutual-follow original posts = **20.0** (5 + 15 boost). Converting followers into *mutual* follows — replying, engaging back, being followable — multiplies your in-network edge. Note the boost only covers original posts, not your replies or retweets.

### 3. Small accounts have a real slot

The cold-start boost reserves ~**slot 15–16** for one eligible post per feed: original post, author ≤1,000 followers, post ≤48h old, <1,000 prior Home views. Under ~1k followers, *every* original post is a lottery ticket with the odds tilted — make each one count.

### 4. Your reputation is computed, not vibes

`agatha` (blocks/reports/spam-reports relative to favorites), `bdsm` (inauthentic behavior sequences), and `user-cred-v2` (PageRank over the follow+engagement graph) produce standing account labels that the 26 recommendation-only rules read. Followers can still see you while discovery dies — check **Under the Hood** if reach stalls.

### 5. In-network isn't automatically safe harbor

Replies and retweets by followed accounts still get the ×0.75 OON-style discount (`EnableOonRescoreForInNetworkRepliesRetweets`), and `AncillaryVFFilter` drops your reply/quote if its parent was dropped. Original posts carry the full weight; choose your parents carefully.

### 6. Diversity is enforced twice

Author-diversity decay (×0.625 → 0.25 floor) handles repeat *authors*; VMRanker's DPP handles repeat *content* — near-identical posts get pushed apart in the ranking. Varied beats repetitive twice over.

### 7. Posts now have a hard shelf life

`AgeFilter` drops anything **older than 48 hours** — the "evergreen thread" idea is dead; plan follow-ups or reposts inside the window.

---

## Still Redacted

- Grox LLM prompt files (`j2`)
- Some botmaker rules
- `NEGATIVE_SCORES_OFFSET` normalization constant value
- Production model *weights* (code is public; trained checkpoints are not)

---

## Sources

- [xai-org/x-algorithm](https://github.com/xai-org/x-algorithm) — dev notes (Aug 13/14, Sep 18) + README
- [`home-mixer/params/param.rs`](https://github.com/xai-org/x-algorithm/blob/main/home-mixer/params/param.rs) — production weight defaults
- [`home-mixer/scorers/ranking_scorer.rs`](https://github.com/xai-org/x-algorithm/blob/main/home-mixer/scorers/ranking_scorer.rs) — scoring order + boost logic
- [`visibility-filtering/rules/registry.rs`](https://github.com/xai-org/x-algorithm/blob/main/visibility-filtering/rules/registry.rs) — rule inventory
- [`docs/BIDIRECTIONAL_BOOST_CHANGE.md`](https://github.com/xai-org/x-algorithm/blob/main/docs/BIDIRECTIONAL_BOOST_CHANGE.md) — mutual-follow boost rollout
- [`phoenix/README.md`](https://github.com/xai-org/x-algorithm/blob/main/phoenix/README.md) — production model architecture

---

**Related:** [Action Weights](action-weights.md) · [Filter System](filter-system.md) · [Scoring System](../rules/01-scoring-system.md) · [May 2026 Update](may-2026-update.md)
