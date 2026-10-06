<div align="center">

<img src="assets/branding/app-icon.webp" alt="Pulse: Burnout & Stress Check app icon" width="110" />

# Pulse: Burnout & Stress Check

**A local-first Flutter app for daily work-stress self-reflection: a 12-question baseline, a three-question daily check-in, and a smoothed score with per-dimension drivers.**

**Android · iOS**<br/>
Flutter · Dart · Riverpod · Hive · Firebase

![Flutter](https://img.shields.io/badge/Flutter-Dart-02569B?logo=flutter&logoColor=white)
![Android](https://img.shields.io/badge/Android-3DDC84?logo=android&logoColor=white)
![iOS](https://img.shields.io/badge/iOS-000000?logo=apple&logoColor=white)
![Firebase](https://img.shields.io/badge/Firebase-FFCA28?logo=firebase&logoColor=black)
![Riverpod](https://img.shields.io/badge/Riverpod-state-0F766E)

<table>
  <tr>
    <td valign="middle"><img src="assets/screenshots/pulse-home-today.webp" width="250" alt="Today home screen with a baseline score card and a Start daily check button" /></td>
    <td valign="middle"><img src="assets/screenshots/pulse-daily-checkin-question.webp" width="250" alt="Daily check-in question on a 0 to 6 scale with the in-app wellness disclaimer" /></td>
    <td valign="middle"><img src="assets/screenshots/pulse-daily-result-drivers.webp" width="300" alt="Per-dimension view of a sample check-in: three dimension bars with the primary driver highlighted, and a micro-action card" /></td>
  </tr>
</table>
<sub>Left to right: Today home, daily check-in question, and a per-dimension view of a sample check-in (cropped to the drivers and micro-action cards).<br/>Real screenshots from the Android build, cropped to the app content, captured with sample/placeholder data, metadata removed.</sub>

</div>

---

## Get the App

<p>
  <a href="https://play.google.com/store/apps/details?id=app.pulse.daily&hl=en"><img alt="Get it on Google Play" src="https://img.shields.io/badge/Google%20Play-Get%20the%20app-414141?style=for-the-badge&logo=googleplay&logoColor=white" /></a>
  <a href="https://apps.apple.com/pk/app/pulse-burnout-stress-check/id6770918012"><img alt="Download on the App Store" src="https://img.shields.io/badge/App%20Store-Download-0D96F6?style=for-the-badge&logo=apple&logoColor=white" /></a>
</p>

> **What this is, and is not.** Pulse is a self-reflection and tracking tool for work-related stress. It does not diagnose or treat anything, it is not a substitute for professional advice, and the app itself states that it provides wellness insights only and is not medical advice. This repository describes the software engineering, not health outcomes.

## Product at a Glance

| | |
|---|---|
| **What it is** | A daily self-check for work-related burnout and stress: answer a short questionnaire, see a smoothed 0-100 score, a band and the dimension driving it |
| **Check-in shape** | A one-time 12-question baseline, then three quick daily questions on a 0-6 scale, one for each of exhaustion, disengagement and inefficacy |
| **Scoring** | The app's own custom method: weighted combination of three dimensions, smoothed over time with an exponential moving average, mapped to four bands. Not a validated instrument, not a named standard implementation |
| **Around the score** | Optional journal note, per-driver identification, a static library of short micro-actions, 7-day chart and pattern cards, daily reminders, a simple day-count streak |
| **Data** | Local-first on the device (Hive); optional Google or Apple sign-in adds a cloud copy and restore on a new device; in-app account deletion. Push registration, remote ad configuration, analytics and crash reporting use cloud services for every install |
| **Business model** | Ads for free users through a mediation SDK; premium subscription through the native stores |
| **Platforms** | Android and iOS, one Flutter codebase (screenshots here are from the Android build) |
| **Localization** | 20 locale resource files, including right-to-left layout handling |
| **Code size** | About 35.5k lines across 148 hand-written Dart files in `lib/` (generated and localization code excluded) |
| **Tests** | 14 test files: scoring calculator, analytics layer, ad configuration and loading, paywall widget, screen tracking, motion and start-up |

## Live Product Showcase

<table>
  <tr>
    <td align="center"><img src="assets/screenshots/pulse-journal-note-optional.webp" width="230" alt="Optional journal note step with Skip and Continue buttons"/><br/><sub><b>Optional journal note</b><br/>Final step of the check-in; can be skipped</sub></td>
    <td align="center"><img src="assets/screenshots/pulse-onboarding-save-progress.webp" width="230" alt="Save your progress screen with Google, Apple and continue without account options"/><br/><sub><b>Save progress sign-in</b><br/>Optional account with Google or Apple, or continue without one</sub></td>
  </tr>
  <tr>
    <td align="center"><img src="assets/screenshots/pulse-premium-paywall.webp" width="230" alt="Premium paywall heading with the list of premium benefits"/><br/><sub><b>Premium paywall: benefits list</b><br/>Cropped to the heading and benefits; prices are not reproduced in this repository</sub></td>
    <td align="center"><img src="assets/screenshots/pulse-splash.webp" width="230" alt="Pulse splash screen with the app logo"/><br/><sub><b>Splash</b><br/>Lightweight first screen while services start</sub></td>
  </tr>
</table>

Placeholder ad slots visible in some screens belong to the free tier. All screenshots are from the Android build.

## How a Check-in Becomes a Score

<p align="center">
  <img src="assets/diagrams/checkin-flow.svg" alt="Flow from baseline and daily check-in through scoring and smoothing to bands, drivers, storage and outputs" width="900" />
</p>

The scoring is deliberately small and explainable: normalise each answer, combine three dimensions with fixed weights, smooth each over time so one hard day does not whipsaw the number, and map the result to one of four bands. The structure of three dimensions echoes what is often discussed in burnout literature. The weights, the short daily form, the smoothing and the bands are the app's own design, and the score is a self-reflection aid rather than a clinical measure. Weights and thresholds are intentionally not published here. Details: [docs/TECHNICAL_CASE_STUDY.md](docs/TECHNICAL_CASE_STUDY.md).

## Engineering Highlights

### A noisy daily self-report becomes a steadier number
- **Problem:** three taps a day swing with the day; a raw score would jump around and feel arbitrary.
- **Approach:** normalise, invert efficacy so all three dimensions point the same way, combine with weights, then smooth each dimension with an exponential moving average seeded from the baseline. Band and dominant driver come from the smoothed values.
- **Outcome:** one deterministic, dependency-free scoring pipeline with unit tests for normalisation, weighting, bands, tie-breaks and smoothing.

### Local-first, with the cloud as an optional copy
- **Problem:** people without an account must get the full experience, while signed-in people expect multi-device restore, and both must share one UI.
- **Approach:** the UI reads only from local storage. For signed-in users, writes go to a per-user cloud store, snapshot listeners refresh local data, a first-sync merge rule decides which side wins, and a stale-sync guard plus a safety upload protect sign-out and reset.
- **Outcome:** one read path for every user, with sync layered on top.

### Analytics and ads that wait for the iOS tracking prompt
- **Problem:** analytics collection and ad initialisation must not start before the user answers the iOS tracking-transparency prompt, and the prompt itself must be requested exactly once.
- **Approach:** a native bridge de-duplicates the request, analytics events are queued while it is pending and replayed in order afterwards, and a timeout prevents the queue from hanging.
- **Outcome:** deterministic start-up ordering across Firebase, the tracking prompt and the ad SDK.

### More
| Area | What was built |
|---|---|
| **Account deletion** | Provider-specific re-authentication, deletion of the user's cloud documents, removal of the auth user and push token, then a full local reset |
| **Ads** | Mediation SDK through a locally maintained plugin fork with a native preload cache; remote per-placement and global switches that fail closed when a value is missing |
| **Subscriptions** | Native billing through the stores with a launch-time silent restore; premium state kept on the device and refreshed from the store |
| **Rule-based insights** | Weekly summaries and pattern cards are templated, rule-driven text rebuilt in the current language at render time |
| **Localization** | 20 locale resource files; layout direction flips for right-to-left languages; reminder text follows the saved language |
| **Observability** | Event analytics with a validating, queuing layer, crash reporting and performance traces around start-up, sync and ad loading |

Problem, approach, outcome and trade-offs in full: [docs/ENGINEERING_CHALLENGES.md](docs/ENGINEERING_CHALLENGES.md).

## Architecture

```mermaid
flowchart TD
    UI[Screens and go_router navigation<br/>Today, Patterns, Outlook, Settings] <--> ST[Riverpod providers]
    ST --> SC[Scoring layer<br/>weighting + smoothing]
    ST --> RU[Rule-based insights<br/>patterns, weekly summary, micro-actions]
    SC --> LS[(Local store: Hive)]
    RU --> LS
    LS --> ST
    ST <-->|signed-in users only| CS[Cloud copy<br/>Firestore, per-user]
    AU[Google / Apple sign-in<br/>Firebase Auth] --> CS
    ST --> RM[Reminders<br/>local notifications]
    UI --> AN[Analytics layer<br/>validate, queue, replay]
    AN --> FB[Analytics, crash reporting, performance]
    UI --> AD[Ad mediation SDK<br/>free tier]
    UI <--> BI[Native billing<br/>premium subscription]
```

More diagrams and component responsibilities: [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Technology Stack

| Layer | Technology |
|---|---|
| App framework | Flutter / Dart |
| State management | Riverpod |
| Navigation | go_router with a tabbed shell and an onboarding redirect |
| Local persistence | Hive, plus shared preferences for lightweight settings |
| Models | Immutable models with generated serialization |
| Cloud | Firebase Auth, Cloud Firestore, Cloud Messaging, Analytics, Crashlytics, Performance Monitoring |
| Sign-in | Google; Sign in with Apple; or no account |
| Charts | Custom-painted charts, no charting library |
| Notifications | Local notifications; scheduled background work on Android; native repeating trigger on iOS |
| Ads | Mediation SDK via a locally maintained plugin fork |
| Billing | Store billing through the Flutter in-app purchase plugin |
| Localization | Flutter generated localizations, 20 locale files |
| Tooling | Code generation, lints, mocking for tests |

## Project Scope

Pulse is a production application on Android and iOS. This portfolio focuses on the engineering: the scoring pipeline, the local-first data model and sync, platform integration details and the monetisation and observability layers. It makes no claims about health outcomes, user numbers or ratings.

## Documentation

| Read | For |
|---|---|
| [Product overview](docs/PRODUCT_OVERVIEW.md) | What the app does, screens and the daily loop |
| [Technical case study](docs/TECHNICAL_CASE_STUDY.md) | End-to-end technical write-up |
| [Architecture](docs/ARCHITECTURE.md) | Diagrams and component responsibilities |
| [Engineering challenges](docs/ENGINEERING_CHALLENGES.md) | Problem, approach, outcome and trade-offs |
| [Privacy and source](docs/PRIVACY_AND_SOURCE.md) | What is and is not published |

## Source Code

This repository is a portfolio showcase for a production application. Production source code is maintained privately. See [RIGHTS.md](RIGHTS.md) for usage terms.
