# Architecture

Diagram labels are conceptual and do not correspond to production identifiers.

## System overview

<p align="center">
  <img src="../assets/diagrams/system-overview.svg" alt="Two Flutter clients on Android exchange plain HTTP on the LAN; Firestore holds pairing metadata; Play and Firebase services sit outside the peer link" width="900" />
</p>

Smart Transfer is a single Flutter application targeting Android for release. Nearly all transfer logic executes on the two peer phones. Cloud Firestore participates only in pairing and session state—not in file payload relay. Google Play services supply billing, ads, and in-app updates; Firebase supplies analytics, crash reporting and push delivery.

## Transfer sequence

<p align="center">
  <img src="../assets/diagrams/transfer-flow.svg" alt="Sequence diagram from LAN listener and optional cloud record through request, chunked upload with resume, and finish" width="900" />
</p>

The sequence diagram in the repository README asset folder decomposes pairing, manifest request, per-file offset negotiation, chunked upload, side-channel control messages, and final verification into four phases. File bytes never traverse the cloud box in that figure.

## Layered structure

```mermaid
flowchart TD
    subgraph Presentation
        HOME["Home tabs: Send, Receive, Clone"]
        PICK["Category pickers and review"]
        CONN["Device choice, QR scan, sheets"]
        PROG["Progress, complete, partial, failed"]
        SET["Settings, history, premium"]
    end
    subgraph Orchestration
        CTRL["Transfer controller"]
        NAV["Route helpers and alerts"]
    end
    subgraph TransferServices
        DISC["LAN discovery"]
        SRV["Embedded HTTP listener"]
        CLI["HTTP sender client"]
        CLOUD["Cloud pairing service"]
        PAUSE["Pause and stall logic"]
    end
    subgraph Platform
        BR["Native install bridge"]
        NOTIF["Foreground task and notifications"]
        CAP["Capability and storage probes"]
    end
    subgraph Local
        PREF[("Preferences: history, pending, premium, locale")]
        FS[("App and Downloads storage")]
    end
    subgraph Remote
        FSDB["Firestore pairing docs"]
        FB["Analytics, Crashlytics, FCM"]
        PLAY["Play Billing and in-app updates"]
        ADS["Mobile ads SDK"]
    end
    HOME --> CTRL
    PICK --> CTRL
    CONN --> CTRL
    PROG --> CTRL
    SET --> PREF
    CTRL --> DISC
    CTRL --> CLOUD
    CTRL --> CLI
    CTRL --> SRV
    CTRL --> PAUSE
    CTRL --> NOTIF
    CTRL --> BR
    SRV --> FS
    CLI --> FS
    CLOUD --> FSDB
    CTRL --> FB
    SET --> PLAY
    SET --> ADS
```

| Layer | Responsibility |
|---|---|
| **Presentation** | Material screens for each step of send, receive and clone; modal sheets for connect, accept and alerts; widgets for progress rings, device tiles and storage summaries. |
| **Orchestration** | Single long-lived controller per active transfer session wiring streams from discovery, server and client into UI state; handles pairing method analytics and revision bumps when Receive regenerates codes. |
| **Transfer services** | mDNS advertise/browse, Firestore CRUD for sessions, Dart HTTP server on receiver, HTTP client on sender with chunked writes, QR codec, LAN address selection. |
| **Platform bridges** | Kotlin-side helpers for APK install sessions, unknown-app-source checks, and other APIs not exposed cleanly in pure Dart. |
| **Local stores** | Preferences for JSON history lists, pending sender batches, premium boolean, language code; filesystem paths for incoming partial and complete files. |
| **Remote integrations** | Firestore for pairing; Firebase SDKs for telemetry and push; Play Billing and Mobile Ads as separate Google SDK paths. |

## Pairing state flow

```mermaid
sequenceDiagram
    participant R as Receiving UI
    participant RS as HTTP listener
    participant MD as LAN discovery
    participant DB as Firestore pairing doc
    participant S as Sending UI
    participant CL as HTTP client
    R->>RS: Start listener on LAN port
    R->>MD: Advertise service with TXT
    R->>DB: Create doc with code and endpoint
    R-->>S: QR or numeric code visible
    alt Nearby browse
        S->>MD: Browse peers
        MD-->>S: Device list
    else QR or code
        S->>DB: Resolve session or code
        DB-->>S: LAN address and port
    end
    S->>CL: Open connection to peer
    CL->>RS: Submit file manifest
    RS-->>R: Show accept sheet
    R->>RS: User accepts or rejects
    RS-->>CL: Response with free space
    loop Each file
        CL->>RS: Query byte offset
        RS-->>CL: Existing length
        CL->>RS: Chunked upload body
    end
    RS->>R: Save files and offer installs
```

## Module boundaries (conceptual)

Feature folders group UI and services:

- **Transfer** — screens, widgets, controller, LAN client/server, discovery, QR codec, cloud session wrapper, gallery and folder pickers, APK encoding, preflight, notifications.
- **History** — models, list/detail screens, preferences-backed service.
- **Permissions** — first-run education screen and shared tile widgets reused from Settings.
- **Settings and premium** — toggles, policy viewer, billing UI.
- **Onboarding and splash** — first-run marketing and animation entry.
- **Core** — theme, routing, analytics wrapper, ads service, premium singleton, locale service.

Shared widgets provide buttons, headers and snack bars consistent across features.

## Data categories and sources

| Category | Source mechanism |
|---|---|
| Photos / videos | Gallery index via photo manager APIs with permission prompts |
| Music | Audio media store scan |
| Documents | System document picker and file paths |
| Apps | Installed package list with size metadata where available |
| Folders | Custom browser with storage access helper for all-files grant |

Each selected item normalises into a transfer file model carrying display name, byte size, mime type and platform-specific source path or content URI.

## Failure and edge routes

Dedicated screens and dialogs cover: pairing timeout, HTTP connection failure, receiver decline including low storage, stalled upload watchdog, LAN disconnect banner, partial completion with retry list, user cancel with QR invalidation messaging, and missing install permission for received APKs.

## What sits outside this app

There is no custom backend service for accounts or file relay in the described architecture. Firestore security rules and Firebase project configuration are not published in this portfolio. Entitlement verification for Pro is not delegated to a server in the release described here.
