# Product Overview

## What Pulse is

Pulse: Burnout & Stress Check is a mobile app for daily self-reflection about work-related stress and burnout. A person answers a short questionnaire once, then three quick questions each day. The app turns the answers into a smoothed 0-100 score, one of four bands, and a "dominant driver" among three dimensions: exhaustion, disengagement and inefficacy. Around that score it adds a 7-day chart, weekly summaries, pattern cards, a short daily suggestion, an optional journal note and a daily reminder.

It runs on Android and iOS from one Flutter codebase and is published on Google Play and the App Store (links in the [README](../README.md)). Screenshots in this repository are from the Android build.

## What Pulse is not

This matters more than any feature list.

- Pulse is a self-reflection and tracking tool. It does not diagnose or treat any condition and it is not a substitute for professional advice.
- The app carries its own disclaimer. A one-line note on the daily check-in screen and on the result screen says it provides wellness insights only and is not medical advice and points people to a healthcare professional for medical concerns. The in-app privacy policy and the About screen carry the same not-medical-advice statement.
- The score comes from the app's own custom method. It echoes the three-dimension structure often discussed in burnout literature, but it is not a validated clinical instrument and not an implementation of any named standard instrument.
- There is no health-platform or wearable integration, and the app is not a crisis service.

## The daily loop

1. **Today.** The home tab shows the latest score and band (or the baseline before the first check-in), a button to start the daily check, today's micro-action and an upgrade prompt for free users.
2. **Check-in.** Three questions on a 0-6 tap scale, one per dimension, with a progress bar and back navigation. An optional fourth step offers a journal note that can be skipped.
3. **Result.** The result screen shows the smoothed score and band, the three dimension values with the primary driver highlighted, and a micro-action.
4. **Patterns.** A 7-day bar chart with week navigation; tapping a day reveals its score, dimension values and journal note. A weekly summary, pattern cards and (for premium users) a weekly insights list sit below.
5. **Reminder.** A once-daily local notification at a user-chosen time brings the person back, and tapping it opens the check-in directly.

Missing a day is treated neutrally. No score is invented for skipped days, the last known value carries forward, and the home copy for a missed day is deliberately gentle. The streak counter in Settings is a plain count of consecutive days and there is no streak-loss messaging.

<table>
  <tr>
    <td align="center"><img src="../assets/screenshots/pulse-home-today.webp" width="230" alt="Today home screen with a baseline score card and a Start daily check button"/><br/><sub><b>Today</b><br/>Baseline card (sample data) and the daily check button</sub></td>
    <td align="center"><img src="../assets/screenshots/pulse-daily-checkin-question.webp" width="230" alt="Daily check-in question on a 0 to 6 scale"/><br/><sub><b>Daily check-in</b><br/>0-6 scale, wellness disclaimer in the footer</sub></td>
    <td align="center"><img src="../assets/screenshots/pulse-daily-result-drivers.webp" width="230" alt="Per-dimension view of a sample check-in: three dimension bars with the primary driver highlighted, and a micro-action card"/><br/><sub><b>Result</b><br/>Per-dimension view of a sample check-in</sub></td>
  </tr>
</table>

## Onboarding

Onboarding is a short guided sequence rather than a sign-up wall. A six-step intro flow covers an explanation of the three dimensions, a language picker, a few profile questions (kind of work, role, a self-rating) that are not used in the score, and an optional sign-in with Google or Apple, with "continue without account" always available. The 12-question baseline follows. The baseline can be retaken later from Settings without losing check-in history.

<table>
  <tr>
    <td align="center"><img src="../assets/screenshots/pulse-onboarding-save-progress.webp" width="230" alt="Save your progress screen with Google, Apple and continue without account options"/><br/><sub><b>Save progress</b><br/>Optional sign-in; "continue without account" is available on the same screen</sub></td>
    <td align="center"><img src="../assets/screenshots/pulse-journal-note-optional.webp" width="230" alt="Optional journal note step"/><br/><sub><b>Journal note</b><br/>Optional, skippable, stored with the day</sub></td>
    <td align="center"><img src="../assets/screenshots/pulse-splash.webp" width="230" alt="Splash screen with the Pulse logo"/><br/><sub><b>Splash</b><br/>Shown while services initialise</sub></td>
  </tr>
</table>

## Features in detail

| Feature | Behaviour |
|---|---|
| **Baseline** | 12 questions, three dimensions with four questions each, 0-6 scale. Seeds the smoothing for the first daily check-in |
| **Daily check-in** | Three questions, one per dimension, 0-6 scale. Designed to take under a minute |
| **Score and bands** | A smoothed 0-100 value mapped to four bands. Colours are soft rather than alarm-style |
| **Drivers** | The dimension with the highest value is surfaced as the primary driver; all three values are shown |
| **Journal** | Optional free-text note per day, shown again when the day is opened in Patterns |
| **Micro-actions** | A static, hand-written library of short actions (one to fifteen minutes) selected by band and driver, avoiding recently used ones. Content is fixed, not generated |
| **Patterns** | Rule-based pattern cards (for example weekday peaks or high volatility) built once enough days exist, and a weekly summary with a trend label |
| **Weekly reflection** | An optional Sunday reflection with a few prompts and preset tags; stored but not scored |
| **Outlook** | A premium tab with a simple trend-based projection once enough history exists. It is not machine learning. The tab presents the projected trend with a risk-style label and a probability-style figure computed from it. This is a presentation of a simple extrapolation, not a clinical or predictive risk assessment |
| **Reminders** | One daily local notification, on by default, with a user-selectable time and an explicit permission prompt |
| **Gentle note** | After several consecutive days in the highest band, a soft card suggests slowing down and talking to someone trusted. It is a suggestion, not crisis support, and it does not assess or detect any condition |
| **Account** | Optional Google or Apple sign-in for a cloud copy and restore on a new device; in-app account deletion; local reset |
| **Languages** | 20 locale resource files, selectable in onboarding and Settings, with right-to-left layout handling |

## Free and premium

Free users see ads, delivered through a mediation SDK, around the check-in flow and on the tab shell. Premium is a subscription sold through the native stores. It removes ads and unlocks the Outlook tab, the weekly insights list, longer history and fuller micro-action personalisation. The screenshot below is cropped to the paywall heading and benefits list; prices are not reproduced in this repository.

<p align="center">
  <img src="../assets/screenshots/pulse-premium-paywall.webp" width="230" alt="Premium paywall heading with the list of premium benefits"/><br/>
  <sub><b>Premium paywall: benefits list</b></sub>
</p>

Ads inside a wellness flow are a deliberate monetisation trade-off and the app skips them entirely for premium users. The [trade-offs section of the engineering challenges](ENGINEERING_CHALLENGES.md) discusses how the ad stack is constrained.

## Privacy posture as built

- Data is stored on the device by default. Check-in, baseline and journal data stay on the device unless the person signs in. Push-messaging registration and remote ad configuration use cloud services for every install, and analytics and crash reporting run as described below.
- Signing in adds a per-user cloud copy and restore on another device.
- Account deletion is available in the app.
- Analytics, crash reporting and advertising SDKs are present; the stores' privacy disclosures and the in-app policy are the authoritative description of what is collected.

See [PRIVACY_AND_SOURCE.md](PRIVACY_AND_SOURCE.md) for what this repository publishes.
