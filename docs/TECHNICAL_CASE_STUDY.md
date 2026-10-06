# Technical Case Study

This case study walks through how Smart Transfer moves files between two Android phones: discovery and pairing, consent on the receiver, chunked HTTP upload on the local network, and the auxiliary systems (history, ads, billing, notifications) that surround the core path. Terminology here is conceptual; it avoids production identifiers such as collection names, route paths or analytics keys.

## Problem framing

Phone-to-phone transfer products often rely on cloud relays, which introduce cost, privacy questions and bandwidth limits. Smart Transfer instead keeps payloads on the LAN while still giving users familiar pairing affordances (QR codes and short numeric codes) that work even when multicast browse is flaky. The engineering challenge is to make that hybrid reliable on real Android networks: rotating hotspot IPs, permission denials, split APKs, partial files after process death, and user pauses mid-gigabyte video.

## Device identity and LAN addressing

Each install generates a stable device id and reads a human-readable name from platform APIs. Before advertising or connecting, the app enumerates IPv4 addresses suitable for peer HTTP, preferring Wi-Fi or hotspot interfaces over cellular when both exist. Discovery TXT records can carry explicit IP and port hints because some Android builds fail to connect via raw `.local` hostnames returned from automatic resolution.

## Discovery layer

A dedicated service type is registered over mDNS/NSD. While Receive is active, the receiver advertises its listener port together with id, display name and platform in TXT records. While Send is choosing a device, the sender browses the same type, merges results in a map keyed by peer id, and emits list updates on a broadcast stream. Browse and advertise are torn down aggressively when screens dispose to avoid ghost registrations after Wi-Fi toggles.

Permission to use nearby Wi-Fi devices is treated as part of discovery correctness: if the user permanently denies it, the UI explains that settings is the only recovery path, because repeating the in-app prompt will not show again.

## Pairing metadata in Firestore

When Firebase is initialised, Receive creates a document containing:

- A random six-digit numeric code indexed for lookup.
- Receiver id, display name and platform.
- LAN address and port once the listener binds.
- Status progressing through waiting, requested, accepted or rejected, transferring and completed.

The QR code encodes either a Firebase session token or, when cloud is unavailable, a compact local encoding of id, name, address, port and platform. Senders resolve codes by querying waiting documents or by reading the session id from a scan. After pairing completes, the document is deleted so codes do not linger.

Submitting a transfer request writes sender identity and a JSON summary of each file (name, size, mime type) into the same document so both sides can show consistent file lists before bytes flow. This metadata is small relative to media payloads and is not a substitute for uploading files to cloud storage—the dependency list includes a storage SDK, but the application logic described here routes file bytes only across LAN HTTP.

## HTTP listener on the receiver

The receiver runs an embedded HTTP server bound to the chosen IPv4 port. Responsibilities include:

- Holding at most one pending incoming request until the user accepts or rejects.
- Returning structured accept responses that include free bytes on disk and platform id so the sender can adapt (for example skipping app sends to non-Android peers).
- Serving byte-offset queries per file index so the sender knows how much is already on disk.
- Accepting upload bodies with offset and total query parameters, appending to partial files, emitting progress events, and detecting stalls when chunks stop arriving unless the sender has signalled pause.
- Honouring cancel, pause and resume signals via lightweight control endpoints so the UI can stop cleanly without orphaning half-written files.
- Mapping receiver-initiated cancels and storage failures to distinct status codes so the sender can show specific messaging instead of a generic failure.

When the user accepts, files land under a configured save directory and, for media types, are published into user-visible Downloads where platform APIs allow.

## HTTP client on the sender

The sender drives the state machine:

1. Connect to peer address and port with timeouts tuned for LAN but not instantaneous hotspots.
2. POST the file manifest and await accept/reject.
3. For each file in order, query existing byte count, open a random-access read on the source, stream from offset to end in 256 KiB chunks, flush after each chunk with a twenty-second watchdog, and update UI progress from bytes acknowledged.
4. On failure, classify peer cancel, receiver error, connection refused (process killed) and local cancel separately so retry UI can offer the right action.

Pause stops reading while keeping the session open; resume clears pause state on both sides. Cancel withdraws pairing before accept, or aborts mid-transfer with cooperative socket shutdown where possible.

## Transfer orchestration

A central controller coordinates UI routes, discovery instances, cloud session service, client, server (when receiving), notification foreground task, history persistence and analytics hooks. It tracks pairing method (nearby, QR or code), revision counters when Receive refreshes codes, and flags for LAN loss, pairing service timeouts and permission errors surfaced before discovery starts.

Speed displayed in the UI is derived from bytes over elapsed time with logic to ignore idle periods while paused so resumed transfers do not look artificially fast or slow. No particular megabyte-per-second figure is asserted in this portfolio.

## Phone Clone and batch preparation

Clone reuses Send pickers but allows multiple categories to be collected into one list before device selection. A preflight pass validates permissions and storage access per category, showing a preparing overlay while counts or folder walks run. Select All automates media and folders but deliberately skips apps, forcing a dedicated app screen so the batch always contains explicit package choices.

## APK packaging on the wire

Installed apps may ship as multiple split APK files. The sender encodes filenames with a marker, package name and original segment name so the receiver groups parts, orders installs, and drives the Android package installer session API through a thin native bridge. Legacy filename patterns remain supported for backward compatibility with older app versions.

## Persistence and recovery

Transfer history serialises structured entries to preferences as JSON strings, sorted newest-first on load. A separate pending-transfer record stores the sender’s file list and peer display name when a session ends abnormally; the home screen can offer resume within six hours before the record expires.

## Notifications and process visibility

Active transfers start a data-sync typed foreground service with a persistent notification updated as files progress, satisfying Android background execution rules while bytes move. Local notifications also integrate with Firebase Cloud Messaging for product messaging when configured.

## Premium and ads

Billing queries subscription SKUs, purchases through Play, and mirrors entitlement into preferences that ad widgets consult. Entitlement is client-side in the described release; server-side receipt validation would be a natural hardening step but is not part of this write-up.

Ads load through the mobile ads SDK with placement-specific cooldowns to avoid back-to-back interstitials and rewarded placements. Premium bypasses load/show paths entirely.

## Testing

The automated test suite contains a single widget test that pumps the root app widget and expects splash marketing strings. It does not exercise LAN transfer in CI. Manual testing dominates pairing and throughput validation on real hardware.

## Security and privacy posture (engineering-level)

Pairing codes and QR payloads expose enough information for another device on the same network—or anyone who can read the cloud document while it exists—to attempt a connection. The receiver must accept before bytes flow. Traffic between phones is not wrapped in application-layer TLS; users should use trusted networks. This case study does not label the system “encrypted” or “private by design”; see the privacy doc for repository scope.
