# Technical Case Study

This document walks through how Pulse is built. Production source is private, so descriptions are conceptual and omit identifiers, constants and code.

## 1. Scope and shape

Pulse is a single Flutter application for Android and iOS. It is local-first: every screen reads from on-device storage, and a cloud copy of check-in data exists only for people who choose to sign in. The codebase is organised into presentation (screens, widgets, providers), services (sync, notifications, analytics, subscriptions, ads, insight rules), a local data layer, and a small set of pure scoring utilities.

About 35.5k lines across 148 hand-written Dart files in `lib/`, excluding generated serialization code and localization output. There are 14 test files.

Wellness posture: the app is a self-reflection tool. It does not diagnose or treat and says so in the interface. The engineering below is about software behaviour, not health outcomes.

## 2. Start-up pipeline

Cold start has several competing costs: local storage, Firebase, connectivity, locale, ad SDK initialisation and a possible app-open ad. The approach:

1. A deliberately tiny splash app runs first, with the logo pre-decoded, so the native splash hands off to something that paints immediately.
2. After the first frame, local storage, Firebase, connectivity, locale, home preloading and ad configuration initialise concurrently.
3. Every wait is bounded by a timeout, so a slow network or ad SDK cannot hold the user on the splash indefinitely.
4. Free users with connectivity may see an app-open ad, then the real app mounts under a Riverpod scope. Premium users skip the ad step.
5. On iOS, analytics collection is held until the tracking-transparency prompt resolves (section 8).

Start-up milestones are recorded as performance traces. No timing numbers are published here because none were captured for publication.

## 3. The scoring pipeline

The scoring layer is plain Dart with no framework dependencies, which keeps it unit-testable.

**Inputs.** A baseline of 12 questions (three dimensions, four questions each) on a 0-6 scale, and a daily check-in of three questions on the same scale, one per dimension.

**Steps, conceptually.**
1. Normalise each answer onto a 0-100 range.
2. Treat exhaustion and disengagement as higher-is-worse. Invert the efficacy answer into an inefficacy value so all three point the same direction.
3. Combine the three dimensions with fixed weights into a raw score.
4. Smooth each dimension against the previous smoothed state with an exponential moving average. The first daily check-in is seeded from the baseline. With no baseline, it seeds from the day's own values.
5. Compute the displayed score from the smoothed dimensions, map it to one of four bands, and name the highest smoothed dimension as the dominant driver, with a fixed tie-break order.

**Why smooth.** A raw score from three taps swings with the day. Smoothing means one hard day moves the number only part of the way and the band changes more slowly than mood does. Skipped days create no score; the last known smoothed state carries forward.

**What it is not.** This is the app's own custom method. The three-dimension structure echoes what is often discussed in burnout literature, but the item selection, daily short form, weights, smoothing and band edges are design choices of this app. It is not a validated clinical instrument and not an implementation of a named standard instrument. Exact weights, smoothing factor and thresholds are intentionally not published here. A natural next step before any stronger claim would be an independent review of question wording and weighting.

**Tests.** The calculator has unit tests for normalisation, weighting, band edges, tie-breaking, baseline averaging and the smoothing update.

## 4. State management and navigation

State is held in Riverpod providers (classic notifier and future providers rather than a code-generation path). Providers orchestrate services and expose the current score state, recent check-ins, premium status and sync status.

Navigation uses go_router with a stateful shell of four tabs (Today, Patterns, Outlook, Settings). A redirect gate keeps people in onboarding until it completes, and lets them reach a few routes such as the privacy policy or paywall mid-flow. Route observers feed screen-level analytics.

A small "sync tick" provider acts as an invalidation signal: when cloud listeners write new data into local storage, dependents re-read local storage. The UI never reads cloud state directly.

## 5. Persistence and sync

**Local.** Hive boxes hold settings, profile, baseline, daily check-ins, completed actions, weekly insights and pattern cards. Shared preferences hold lightweight values such as the saved locale. Journal text is stored on the device; adding encryption at rest is planned.

**Cloud (signed-in only).** The cloud store is organised per user. Writes go cloud-first and then local. Real-time listeners write changes back into local storage. A first-sync rule decides direction: if the cloud has data, local data is merged up and cloud then wins; if not, local data is pushed up. Large pushes are batched. A generation counter voids in-flight syncs after sign-out, and a safety upload runs before sign-out or reset so a user does not lose recent entries.

**Guests.** Check-in, baseline and journal data stay on the device unless the person signs in. Push-messaging registration and remote ad configuration use cloud services for every install, and analytics and crash reporting run as described in section 10.

**Connectivity.** Actions that need the network (sign-in, purchase, restore, sign-out, reset, deletion) are guarded by a verified-connectivity check with a dialog. Offline behaviour is described here only at that level.

## 6. Accounts and deletion

Sign-in is Google or Apple, or no account. On iOS, Apple sign-in uses the native sheet with a hashed nonce; on Android it uses Firebase's provider flow. Account deletion re-authenticates with the same provider, deletes the user's cloud documents collection by collection, removes the auth user and push token, signs out and clears local data. Deletion in a client loop is not transactional, and analytics data held by the analytics provider is outside its scope. Those caveats are acknowledged rather than hidden.

## 7. Rule-based insights and micro-actions

- **Weekly summary:** requires a minimum number of check-ins in the last seven days, derives a trend label from first versus last score, and renders a templated sentence.
- **Pattern cards:** several detectors (weekday peak, end-of-week drift, driver dominance, volatility, missed-day clusters) run over a recent window once enough days exist.
- **Micro-actions:** a static, hand-written library of short actions localized in every supported language. Selection filters by band, driver and duration, excludes recently used actions and falls back progressively.
- **Localization after the fact:** insights and cards persist a type and metadata (plus an English fallback sentence) and rebuild their sentence from localized templates at render time, so changing language re-renders history.

None of this uses a language model or any machine-learning model. The Outlook tab is a simple trend-based projection that needs a longer history. It presents the projected trend with a risk-style label and a probability-style figure computed from it; this is a presentation of a simple extrapolation, not a clinical or predictive risk assessment.

## 8. Platform integration

- **Reminders.** Android schedules a self-renewing one-off background task each day (avoiding exact-alarm permission); iOS uses a native repeating local notification. Notification text is localized from the saved locale, including from a background isolate.
- **iOS tracking prompt.** Analytics collection is disabled natively at launch. A native bridge requests the prompt once, even if triggered from both native and Dart code. Events are queued while it is pending and replayed in order when it resolves, with a timeout.
- **Ads.** A mediation SDK through a locally maintained plugin fork with a native preload cache. Remote configuration decides which placements are active and fails closed on missing values. Premium users skip ads everywhere.
- **Billing.** The Flutter in-app purchase plugin over Play Billing and the App Store. A silent restore on launch refreshes the premium state kept on the device. Entitlement checks are client-side; server-side receipt verification is the planned next step.
- **Updates.** Play's flexible in-app updates on Android; a version check on iOS.

## 9. Localization

The app uses Flutter's generated localizations with 20 locale resource files, each carrying the same set of keys. Layout direction flips for right-to-left languages. Questionnaire items, bands, micro-actions, insight templates, notification text and the privacy policy are all localized. Translation quality was not reviewed.

## 10. Observability

An analytics layer wraps the Firebase SDK. It validates event and parameter shapes against platform limits, queues events until initialisation is allowed, observes routes and overlays automatically, and records ad lifecycle in a single event shape. Crashes go to crash reporting as typed non-fatals and fatals, and custom performance traces cover start-up, local storage init, sync and ad loading. Analytics should be read as covering product-usage events, and readers should consult the store privacy disclosures for what is collected.

## 11. Tests

14 test files: the scoring calculator, the analytics layer (validator, session, events, service, observers, widgets and ad hooks, start-up and performance), ad placement configuration and ad loading (including fail-closed parsing), a screen-cover tracker, a paywall widget test, motion and a default widget test. Areas without dedicated tests include sync, authentication, billing, insight rules and notification scheduling. No claim is made here about passing status or coverage.

## 12. Trade-offs and next steps

| Area | Current state | Natural next step |
|---|---|---|
| Local data | Journal text is stored on the device | Add encryption at rest |
| Entitlement | Client-side checks | Server-side receipt verification |
| Test coverage | Present for scoring and analytics only | Cover sync, billing and insight rules |
| Short daily form | A design choice, not independently reviewed | Independent review of question wording and weighting |
| Ads in wellness flow | Present for free users | Revisit placement density |
