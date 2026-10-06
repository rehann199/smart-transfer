# Product Overview

Smart Transfer helps someone copy files from one Android phone to another without a cable or a desktop middleman. The experience is organised around three home tabs—Send, Receive and Phone Clone—plus History and Settings reachable from the shell. Everything in this document describes what the shipped Android app presents and what the production code is built to do. Store taglines about speed or security are not repeated here.

## Who it is for

People setting up a new phone, sharing large media with a friend nearby, or moving documents and sideloaded apps between two devices on the same Wi-Fi network or personal hotspot.

## First run

A language chooser offers eleven locales. An onboarding carousel introduces Send, Receive, Clone and History at a marketing level; the engineering portfolio treats those slides as UI copy only. A permissions setup screen lists four required grants with plain-language explanations: nearby devices for discovery, photos and videos for gallery-backed categories, all-files access for folders and mixed storage, and notifications for transfer progress. Users can defer with “Do it later” but transfer pickers will prompt again when a specific category needs access.

## Send flow

The Send tab opens a category grid: Photos, Videos, Music, Documents, Apps and Folders. Tiles show scanned counts where the app has already indexed storage (for example video count and total size). Tapping a category opens the appropriate picker—gallery timelines, audio library, document chooser, installed-app checklist or folder browser. After selection, the user reviews the batch, then lands on device choice.

Device choice supports three pairing paths:

- **Nearby** — a live list populated from LAN service discovery, refreshed while the screen is open.
- **QR** — camera scan of the receiver’s code; invalid codes are rejected before network work starts.
- **Numeric code** — manual entry of the six digits shown on the receiver; resolved through the cloud pairing document.

Connecting sheets show progress while the sender reaches the receiver’s HTTP listener. The receiver sees an incoming request sheet listing files and sizes and can accept or decline; a low-storage decline returns a specific reason to the sender.

During transfer, both sides show progress rings or bars, current file name, and controls to pause (sender) or cancel. Completion routes to a success screen with shortcuts to open Downloads or install received apps. Partial success and failure screens retain enough state to retry individual files or the whole batch.

## Receive flow

Receive starts the local HTTP listener, registers LAN advertisement, and optionally creates a cloud pairing record. The screen shows “Ready to Receive”, the device model name, a QR code, and the six-digit code. Leaving Receive or finishing a session refreshes codes so an old QR cannot be reused indefinitely. Endpoint updates propagate to Firestore when the LAN IP changes on the same screen.

## Phone Clone flow

Clone is a guided variant of Send aimed at new-device setup:

1. **Role** — user picks Old Phone (send data) or New Phone (receive data) with inline instructions to open the app on both devices.
2. **Content (old phone)** — multi-select grid across the same six categories. Select All covers photos, videos, music, documents and folders but not apps; apps require explicit picks because bulk “all apps” is not meaningful in code.
3. **Gate** — non-Pro users may see a rewarded-ad loading dialog before clone proceeds (visible in portfolio screenshots as “Loading ad”).
4. **Pairing and transfer** — identical pipeline to Send once the batch is fixed.

The new phone follows Receive behaviour while the old phone drives sending.

## History

History lists prior operations with mode (send, receive or clone), counterpart device label, status pill, timestamps and totals. Detail screens break out per-file outcomes. All records live in on-device preferences; Settings offers clear-all with a privacy note that history never leaves the phone unless the user copies files elsewhere manually.

## Settings

Grouped sections include:

- **Permissions** — live Allowed/Denied style status with navigation to system settings per row.
- **Transfer** — history entry point and clear-history action.
- **Help** — contact support entry.
- **Premium** — Pro paywall with plan selection and restore purchases.
- **General** — language, rate-us prompt, privacy policy viewer, more-apps cross-promo, version label.

## Apps and installs

On supported Android builds, Send Apps lists user-installed packages with sizes. Split APKs are encoded so the receiver reassembles parts before invoking the install session API. The user must grant install-unknown-apps for Smart Transfer; the app surfaces a dedicated message when that grant is missing.

## Monetisation surfaces

Free users see mediated ads in banners, native slots, interstitials (with cooldown spacing), app-open placements and rewarded gates for Phone Clone and extended transfer quotas. Pro subscription hides ad widgets by checking a locally stored premium flag after Play Billing reports a purchase. Exact prices and trial wording are store-managed and are not quoted here.

## What the app does not claim in this portfolio

- No statement that transfers are encrypted end-to-end; peer traffic uses plain HTTP on the LAN unless the operating system adds another layer.
- No throughput benchmarks; speed depends on Wi-Fi conditions, file sizes and device storage speed.
- No guarantee that every installed app can be cloned; OEM and Android version limits apply, and some categories show “not supported” messages when platform APIs refuse access.

## Languages

Arabic, German, English, French, Hindi, Italian, Japanese, Korean, Portuguese, Russian and Simplified Chinese strings ship in bundled tables selected from Settings or the first-run chooser.
