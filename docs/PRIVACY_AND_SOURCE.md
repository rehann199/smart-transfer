# Privacy and Source

## What this repository is

A portfolio showcase for a production Android application distributed on Google Play as Smart Transfer. It describes software engineering for local device-to-device file copying. It is not a source distribution and makes no claims about transfer speed, encryption or data safety beyond what the implementation actually does.

## What is published

- Written descriptions of architecture, pairing, transfer behaviour, design decisions and trade-offs
- Original vector diagrams in `assets/diagrams/` (system overview and transfer sequence)
- Real screenshots from the Android build, cropped to app content (system status bar and ad-placeholder strips removed), captured with sample and placeholder data
- The app icon as WebP in `assets/branding/`
- Static code-size counts in the README (Dart files and lines under the application source tree, plus one widget test)

## What is intentionally not published

- Production source code, in whole or in part
- Build and signing configuration, certificates and keystores
- Firebase and Google service configuration files, API keys and tokens
- Ad unit identifiers, placement identifiers and mediation account details
- Subscription product identifiers, price points and trial durations
- Internal Firestore collection and field names, security rules and indexes
- Analytics event names, parameter keys and sampling configuration
- Exact HTTP route tables, native method channel names and internal class identifiers
- Version control history of the production repository
- User data of any kind

Line, file and test counts are static counts; no test results, performance figures, download numbers, user counts or store ratings are claimed. Diagram labels are conceptual and do not correspond to production identifiers.

## Notes on figures

Dart file and line counts exclude bundled language tables in a separate count. The eleven user-facing locales match the supported locale list in the generated localisation entrypoint. These numbers are snapshots for this portfolio, not live metrics.

## About the app's data handling

This repository does not restate or replace the application's privacy policy or the data safety section on the store listing; those documents are the authoritative sources for end users.

At the engineering level, the following properties apply to the described release:

- **File payloads** move directly between two devices on a local network using plain HTTP initiated by the app. They are not uploaded to cloud storage as part of the core transfer pipeline described here.
- **Pairing documents** in Cloud Firestore hold short-lived metadata: human-readable device names, a numeric code, LAN address and port, session status and a manifest summary (file names and sizes) while a session is active. Documents are deleted when sessions close.
- **Transfer history** and **pending resume batches** stay in on-device preferences until the user clears them or uninstalls.
- **Purchases** interact with Google Play Billing; entitlement for ad removal is mirrored to local preferences in the described release.
- **Advertisements** use Google's mobile ads SDK. Ad requests may share device and app identifiers per SDK behaviour and consent choices.
- **Firebase** integrates Analytics, Crashlytics and Cloud Messaging. Events and crash reports follow SDK configurations shipped in the app; this portfolio does not enumerate which fields are collected or excluded.
- **Notifications** include transfer progress foreground service updates and may include marketing pushes via Cloud Messaging when enabled.

No claim is made here that all processing is anonymous, that LAN traffic is encrypted by the app, or that the product complies with a specific regulation. Users on untrusted networks should treat plain HTTP peer traffic accordingly. Questions about data categories and retention belong to the published privacy policy and store disclosures.

## Third-party technology

Names such as Flutter, Dart, Android, Firebase, Firestore and Google Play appear only to identify platforms and SDKs. Trademarks belong to their respective owners.

## Contact and corrections

This repository is maintained as a static portfolio. Factual corrections about described behaviour should be backed by the production codebase or store listings before text changes.
