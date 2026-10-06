# Engineering Challenges

Each section follows Problem, Approach, Outcome, with the trade-offs kept visible. Tuning constants and identifiers are intentionally omitted.

## 1. A noisy self-report becomes a steadier, explainable score

**Problem.** Three taps a day vary with the day itself. Showing the raw value would make the number jump and invite over-reading of any single answer.

**Approach.**
- Normalise answers to a common range and invert the efficacy answer so all three dimensions are higher-is-worse.
- Combine them with fixed weights.
- Smooth each dimension against the previous state with an exponential moving average. The first daily check-in is seeded from the 12-question baseline so day one is not a cold start.
- Derive the band and the dominant driver from the smoothed values, with a deterministic tie-break order.
- Keep the whole pipeline in pure Dart with no storage or UI imports.

**Outcome.** One deterministic pipeline covered by unit tests for normalisation, weighting, band edges, tie-breaks, baseline averaging and the smoothing update. A bad day moves the score part of the way rather than all of it.

**Trade-offs.** The method is the app's own. It echoes the three-dimension structure often discussed in burnout literature, but it is not a validated clinical instrument and not an implementation of a named standard instrument. The short daily form is a simplified check-in, not a substitute for the longer baseline. An independent review of the question wording and weighting would be the right step before any stronger claim, and exact constants are deliberately not published.

## 2. Local-first for everyone, cloud copy for some

**Problem.** People without an account need the full experience offline-friendly and private by default. Signed-in people expect restore on a new device. Both must share one UI without branching everywhere.

**Approach.**
- Make local storage the only read path. The UI never reads cloud state directly.
- For signed-in users, write cloud-first and then local; attach real-time listeners that write remote changes back into local storage and then trigger dependent providers to re-read.
- Define a first-sync rule: push local data if the cloud is empty, otherwise merge up and let the cloud win.
- Batch large writes, guard against overlapping syncs, and use a generation counter so work started before a sign-out cannot write afterwards.
- Run a safety upload before sign-out or reset.

**Outcome.** A single sync path, restore-on-login, and explicit handling for the "new account with no data" case. Sync timings and failures are recorded as performance traces.

**Trade-offs.** Offline behaviour for signed-in accounts is described conservatively: sensitive actions check connectivity first and local data stays readable. For signed-in users the cloud write is attempted first; if it fails the error is recorded and the entry is still saved locally.

## 3. Account deletion that removes the data the app holds

**Problem.** Store expectations and the privacy policy require in-app deletion, but data spans several cloud collections and Firebase requires a recent sign-in before deleting an account.

**Approach.**
- Re-authenticate with the same provider the person used (Google or Apple, with a fresh nonce for Apple).
- Delete each cloud collection and the user document, delete the auth user, remove the push token, sign out of the provider and reset local storage.
- A friendly "please sign in again" path handles stale credentials.
- A separate local reset exists for guests and does not touch the cloud.

**Outcome.** End-to-end deletion from inside the app.

**Trade-offs.** A client-side deletion loop is not transactional, and data already held by the analytics provider is outside it. A server-side deletion function is the planned next step.

## 4. Ordering analytics, ads and the iOS tracking prompt

**Problem.** On iOS, data collection and advertising initialisation should not start before the person answers the tracking-transparency prompt. The prompt can be triggered from both native and Dart code, so it must be requested reliably and only once.

**Approach.**
- Disable analytics collection natively at launch.
- Add a native bridge that de-duplicates the request across both entry points.
- Queue events (with a cap) while the prompt is pending and replay them in order once it resolves.
- Add a timeout so a prompt that never resolves cannot hang the pipeline.

**Outcome.** Deterministic start-up ordering across Firebase, the prompt and the ad SDK.

**Trade-offs.** Analytics and advertising SDKs are present in the app, so the store privacy disclosures and in-app policy are the authoritative statement of what is collected. This document makes no claim that any category of data is excluded.

## 5. An ad stack that can be switched off remotely and fails closed

**Problem.** Several placements, a wellness context where intrusive ads are undesirable, and no appetite for shipping an app update to turn something off.

**Approach.**
- Fork the mediation plugin locally to add a native preload cache so inline banner and native ad slots can be warmed before their widgets mount.
- Hold per-placement switches and one global switch in remote configuration, parsed so that only an explicit true enables anything and a missing or invalid value is off.
- Add per-format cooldowns across placements and a watchdog for a stuck full-screen ad.
- Skip ads for premium users everywhere and when offline.
- Test the configuration parsing, including the fail-closed cases.

**Outcome.** Ad behaviour adjustable without a release, and a conservative default when configuration is unknown.

**Trade-offs.** Ads appear around and inside the check-in flow for free users. That is a monetisation choice that sits uneasily beside a calm wellness product, and revisiting placement density is a sensible improvement.

## 6. Subscriptions without a backend

**Problem.** Offer a premium subscription on two stores without operating servers.

**Approach.**
- Model products per store, with a trial offer, and let the store price override fallback display prices.
- Keep premium state on the device and refresh it from the store on every launch; default to no entitlement when state is unknown.
- Avoid counting trial purchases as revenue in analytics.

**Outcome.** A working subscription lifecycle with purchase, restore and lapse handling.

**Trade-offs.** Entitlement checks are client-side; adding server-side receipt verification is the planned next step.

## 7. Rule-driven insight text that survives a language change

**Problem.** Weekly summaries and pattern cards are persisted. If they were stored as finished sentences, switching language would leave history in the old language.

**Approach.** Persist a type and metadata (counts, driver, confidence, plus an English fallback sentence) and rebuild the sentence from localized templates when rendering. The rules themselves are plain thresholds over stored check-ins, with minimum-data requirements before anything is shown.

**Outcome.** Changing language re-renders existing history across all 20 locale resource files. The same idea applies to reminder text, which a background isolate localizes from the saved locale.

**Trade-offs.** Rules are simple and transparent by design. They are not statistical models and make no health claims. Translation quality was not reviewed.

## 8. Start-up under competing initialisation

**Problem.** Local storage, Firebase, connectivity, locale and ad SDK setup compete at cold start, and an ad SDK can be slow.

**Approach.** Show a minimal splash app first, run heavy initialisation concurrently after the first frame, bound every wait with a timeout and record milestones as traces.

**Outcome.** An instrumented cold-start timeline. No performance figures are published because none were captured for publication.

## 9. Keeping claims honest around a forecast

**Problem.** A premium Outlook tab suggests forecasting, and forecasting language invites over-claiming.

**Approach.** The tab shows a simple trend-based projection derived from recent smoothed scores and requires a longer history before it appears. It is described here as a simple projection, with no claim of machine learning, prediction accuracy or health outcomes. The tab presents the projected trend with a risk-style label and a probability-style figure computed from it. This is a presentation of a simple extrapolation, not a clinical or predictive risk assessment.

**Outcome.** A feature whose description stays close to its implementation.

## Summary of trade-offs

| Area | Current | Natural next step |
|---|---|---|
| Local data at rest | Journal text is stored on the device | Add encryption at rest |
| Entitlement | Client-side checks | Server-side receipt verification |
| Account deletion | Client-side loop | Server-side deletion function |
| Scoring | Custom method, short daily form | Independent review of question wording and weighting |
| Tests | Tests exist for scoring and analytics | Add sync, billing and insight-rule tests |
| Ads | Present in wellness flow for free users | Reduce placement density |
