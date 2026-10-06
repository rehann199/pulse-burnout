# Privacy and Source

## What this repository is

A portfolio showcase for a production application. It describes the engineering of Pulse: Burnout & Stress Check; it is not a source distribution.

## What the app is, in one paragraph

Pulse is a self-reflection and tracking tool for work-related stress. It does not diagnose or treat any condition and it is not medical advice. The app itself shows a short wellness-insights-only disclaimer on the check-in and result screens. Nothing in this repository should be read as a statement about health outcomes.

## What is published

- Written descriptions of architecture, design decisions and trade-offs
- Original diagrams created for this showcase
- Real screenshots from the Android build, captured with sample/placeholder data
- The app icon, re-encoded

## What is intentionally not published

- Production source code, in whole or in part
- Scoring weights, smoothing factor, band thresholds and other tuning values
- The full question set and micro-action library
- Build and signing configuration, certificates, keystores and keys
- Firebase and Google service configuration, API keys and tokens
- Ad network identifiers, placement and unit IDs, and account details
- Billing product identifiers and subscription pricing
- Internal endpoints, collection names, database paths and security-rule contents
- Version control history of the production repository
- User data of any kind

## Figures

Figures are limited to: about 35.5k lines across 148 hand-written Dart files in `lib/` (generated and localization code excluded), 20 locale resource files and 14 test files. Diagram labels are conceptual and do not correspond to production identifiers. No user counts, ratings, performance figures or health outcomes are stated, because none are claimed.

## Honest statements about the app's data handling

These are descriptions of the design, not certifications.

- Data is stored on the device by default. Check-in, baseline and journal data stay on the device unless the person signs in. Push-messaging registration and remote ad configuration use cloud services for every install, and analytics and crash reporting run as described below.
- Signing in with Google or Apple adds a per-user cloud copy using Firebase services.
- Account deletion is available inside the app.
- Local storage is not additionally encrypted by the app. Hardening it is listed as a next step.
- Analytics, crash reporting and advertising SDKs are part of the app. This repository does not claim that any category of data is excluded from them. The store privacy disclosures and the in-app policy are the authoritative description of collection.
- No encryption, compliance or certification claim is made.

## Images

- **Screenshots** are real captures from the Android build, cropped to the app content, with sample/placeholder data and metadata removed. Free-tier ad slots appear as placeholders in some of them. The result image is a cropped per-dimension view of a sample check-in and is not a real person's result. The baseline value on the home screenshot is sample data.
- **Paywall** is cropped to the heading and benefits list; prices are not reproduced in this repository.
- **App icon** is the product icon, resized.
- **Diagrams** are original vector illustrations.

## Access

Source access for hiring or due-diligence purposes can be discussed at the owner's discretion.
