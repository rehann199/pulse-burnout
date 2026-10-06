# Architecture

Conceptual diagrams of the Pulse application. Labels describe responsibilities, not production identifiers.

## System overview

<p align="center">
  <img src="../assets/diagrams/system-overview.svg" alt="System overview: on-device Flutter app, optional Firebase services and monetisation components" width="900" />
</p>

The app is local-first. Everything on the left, inside the device boundary, works without an account. Firebase services and the ad and billing components are layered on top and can be absent (signed-out user, premium user, offline) without changing how screens read data.

## Layers

```mermaid
flowchart TD
    subgraph Presentation
        S[Screens and widgets]
        R[go_router<br/>tab shell + onboarding gate]
    end
    subgraph State
        P[Riverpod providers]
    end
    subgraph Domain
        C[Scoring utilities<br/>pure Dart]
        I[Insight and pattern rules]
        M[Micro-action selection]
    end
    subgraph Services
        SY[Sync service]
        NO[Notification service]
        AN[Analytics service]
        SU[Subscription service]
        AD[Ad services]
    end
    subgraph Data
        H[(Hive boxes)]
        F[(Cloud store)]
    end
    S --> P
    R --> P
    P --> C
    P --> I
    P --> M
    P --> SY
    P --> NO
    P --> SU
    S --> AD
    S --> AN
    C --> H
    I --> H
    M --> H
    H --> P
    SY <--> F
    SY --> H
```

| Layer | Responsibility |
|---|---|
| **Presentation** | Four-tab shell (Today, Patterns, Outlook, Settings), onboarding, check-in, result, paywall and settings screens; custom-painted charts |
| **State** | Riverpod providers expose the current score state, recent check-ins, premium status, sync status and onboarding progress |
| **Domain** | Pure scoring utilities, weekly summary and pattern rules, micro-action selection. No framework imports, so they are directly unit-testable |
| **Services** | Cloud sync, local notifications, analytics, subscriptions, ad orchestration, connectivity checks, update prompts |
| **Data** | Hive boxes as the only read path for the UI; a per-user cloud store for signed-in users |

## One check-in, end to end

```mermaid
sequenceDiagram
    participant U as Person
    participant UI as Check-in screens
    participant P as Provider
    participant C as Scoring utilities
    participant H as Hive (local)
    participant F as Cloud copy
    U->>UI: answer three questions (0-6)
    UI->>P: submit answers
    P->>H: read last smoothed state or baseline
    P->>C: normalise, weight, smooth, band, driver
    C-->>P: score, band, dominant driver
    alt signed in
        P->>F: write check-in (cloud first)
    end
    P->>H: write check-in
    P-->>UI: check-in saved
    opt no note yet today
        U->>UI: optional journal note
        UI->>H: store note with the day
    end
    P-->>UI: update current state
    UI-->>U: result screen with score and per-dimension drivers
```

The cloud write exists only for signed-in users, and a signed-in journal note is also written to the cloud. The UI always renders from local state afterwards.

## Sync model

```mermaid
flowchart LR
    subgraph Device
        UI[UI] --> LS[(Local store)]
        LS --> UI
    end
    subgraph Cloud
        CS[(Per-user documents)]
    end
    UI -- write, signed-in only --> CS
    CS -- snapshot listeners --> LS
    LS -->|"first sync: push if cloud empty"| CS
    CS -->|"first sync: cloud wins if it has data"| LS
```

Key behaviours in words:

- **One read path.** The UI reads local storage only. Listener callbacks update local storage and then nudge dependent providers to re-read.
- **First-sync rule.** An empty cloud receives the local data. A populated cloud is merged into and then becomes the source of truth.
- **Stale-sync guard.** A generation counter invalidates sync work that started before a sign-out.
- **Safety upload.** Before sign-out or reset, a final upload runs so recent entries are not lost.
- **Guests.** Check-in, baseline and journal data stay on the device unless the person signs in. Push-messaging registration and remote ad configuration use cloud services for every install, and analytics and crash reporting run as described in the technical case study.

## Start-up sequence

```mermaid
flowchart TD
    A[Native splash] --> B[Minimal Flutter splash app]
    B --> C{After first frame}
    C --> D[Local storage init]
    C --> E[Firebase init]
    C --> F[Connectivity + locale]
    C --> G[Ad configuration]
    D & E & F & G --> H{Free user, online, ad SDK ready?}
    H -- yes --> I[App-open ad with timeout]
    H -- no --> J[Mount main app]
    I --> J
```

Each wait has an upper bound. On iOS, analytics collection and ad initialisation additionally follow the tracking-transparency prompt (see below).

## Analytics ordering on iOS

```mermaid
sequenceDiagram
    participant N as Native layer
    participant D as Dart layer
    participant Q as Event queue
    participant A as Analytics SDK
    N->>N: disable collection at launch
    D->>Q: events logged while prompt pending
    N->>N: request tracking prompt once
    N-->>D: answer
    D->>A: enable collection
    Q->>A: replay queued events in order
```

A timeout prevents the queue from waiting forever if the prompt never resolves.

## Component responsibilities

| Component | Notes |
|---|---|
| **Scoring utilities** | Normalisation, weighting, exponential smoothing, band mapping and driver selection. Deterministic and independent of storage |
| **Insight rules** | Weekly summary and pattern detectors. Persist a type and metadata; sentences are built from localized templates at render time |
| **Micro-action selector** | Filters a static library by band, driver and duration, excludes recent actions, falls back progressively |
| **Sync service** | Singleton for signed-in users. Batched writes, listeners, generation counter, safety upload |
| **Notification service** | Daily reminder with a user-selected time; scheduling differs per platform; taps deep-link into the check-in |
| **Subscription service** | Store products, purchase and restore flows, local entitlement with a silent refresh on launch |
| **Ad services** | Mediation SDK, placement switches and a global switch from remote configuration, cooldowns and a watchdog for stuck full-screen ads |
| **Analytics service** | Validation, queueing, session tracking, route and overlay observation, typed crash non-fatals, performance traces |

## Honest limits of the architecture

- Entitlement checks are client-side; server-side receipt verification is the planned next step.
- Journal text is stored on the device; adding encryption at rest is planned.
