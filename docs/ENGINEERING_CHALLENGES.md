# Engineering Challenges

Each section follows a problem, context, approach and outcome pattern. Wording stays clear of exploit instructions and of performance marketing.

## Reliable LAN pairing when multicast and IPs disagree

**Problem.** Users expect QR and “nearby device” lists to “just work” on hotspot networks where mDNS browse returns empty, `.local` names fail to connect, or the receiver’s IP changes mid-session.

**Context.** Android NSD can resolve a service while TCP to the resolved hostname still fails on some OEM stacks. Receivers also switch between Wi-Fi and phone hotspot when toggling tethering.

**Approach.** Discovery advertises optional IPv4 and port TXT overrides alongside the standard port field. Connection code prefers explicit IPv4 strings from TXT or Firestore over hostname resolution. When Receive stays open, a background hook updates the cloud pairing document if the LAN address changes so QR and code lookups stay current. Three pairing entry points—browse, scan and typed code—share one controller method that validates permissions before starting browse, avoiding opaque empty lists when nearby Wi-Fi permission is denied.

**Outcome.** Senders can recover when one path fails (for example scan after an empty nearby list). Pairing still requires both devices on a reachable L2 network; cellular-only remote transfer is out of scope.

## Cooperative HTTP uploads with pause, stall and cancel semantics

**Problem.** A single long POST per file would freeze progress reporting, confuse pause with failure, and leave ambiguous half-files if the sender process dies.

**Context.** Users pause large sends; receivers low on storage need to abort with a reason the sender understands; Android may kill either app under memory pressure.

**Approach.** Uploads read from disk in 256 KiB chunks, flushing each chunk with a timeout so zero-byte progress can be distinguished from a dead socket. Pause sets a flag checked on both sides so stall watchdogs ignore intentional idle periods. Cancel distinguishes sender abort, receiver user cancel and receiver resource errors via different HTTP status semantics and optional reason headers. The receiver tracks the active upload response so a cancel can terminate the in-flight socket immediately instead of waiting for the next chunk boundary.

**Outcome.** UI can show pause versus error versus peer decline without guessing. Resume asks the receiver how many bytes were persisted and continues from offset.

## Accept/reject sheet races with early cancel

**Problem.** If the sender cancels while the accept sheet is visible, the receiver could wait until a long timeout expires.

**Context.** Pairing UX allows withdraw before accept; users change their mind quickly on crowded networks.

**Approach.** The listener keeps a pointer to the pending incoming request. A cancel endpoint arriving before accept auto-completes the pending completer as rejected and dismisses the sheet path in the controller.

**Outcome.** Neither side sits in a zombie pairing state after a withdrawn request.

## Split APKs as one logical app

**Problem.** Sending a single base APK misses split features; sending many raw APK files leaves the receiver with uninstallable fragments.

**Context.** Modern Android apps often ship multiple split APKs per package; filenames alone collide across packages.

**Approach.** The sender encodes package identity into a structured wire filename; the receiver groups parts by package, preserves order, and feeds them into an install session through native code. Legacy double-underscore numbered APK patterns remain recognised for older senders.

**Outcome.** Received apps appear as one install card with states pending, installing, installed or failed rather than a folder of unrelated APKs.

## Phone Clone batching without unsafe “select all apps”

**Problem.** Clone marketing promises “move your data”, but silently selecting every installed app is rarely desired and may include work profiles or unsupported packages.

**Context.** Clone UI uses checkboxes per category like a backup wizard; apps need per-package consent on Android.

**Approach.** Select All includes media categories and folders only. Apps tile routes to a manual checklist screen; preflight verifies install permissions separately. The same transfer engine runs once the aggregated list is built.

**Outcome.** Clone batches are large where automation is safe and explicit where the platform requires user choice.

## Foreground visibility during long transfers

**Problem.** Android restricts background network work; users switch apps while multi-gigabyte sends continue.

**Context.** Data-sync foreground service type matches file transfer use case on recent API levels.

**Approach.** When bytes actively move, a foreground task service starts with a persistent notification updated from progress streams; it stops when the session completes or aborts. Separate local notification helpers cover one-shot alerts where appropriate.

**Outcome.** Transfers remain eligible to continue while the user briefly leaves the app, subject to OEM battery policies not controlled by the app.

## Client-side Pro flag versus Play Billing

**Problem.** Premium must hide ads immediately after purchase without waiting for a slow UI rebuild across many ad widgets.

**Context.** Multiple ad surfaces (banner, native, interstitial, app open, rewarded) each gate on a shared singleton.

**Approach.** Billing callbacks write a boolean preference and notify listeners; ad widgets short-circuit load and show when premium is true. Restore purchases re-queries owned items on paywall open.

**Outcome.** Consistent ad-free experience for paying users in the common case. Server-side entitlement validation would tighten tamper resistance and is noted as a trade-off rather than documented as a vulnerability recipe.

## Persisted sender batch after process death

**Problem.** Wi-Fi drops or user force-stop during a large send loses in-memory file lists.

**Context.** Within-session retry already handles single-file failures; cold start needs a narrower recovery path.

**Approach.** On certain failure exits, serialise the selected files and peer display name to preferences with a timestamp; home UI offers resume into device choice for up to six hours, then discards stale entries.

**Outcome.** Users can reconnect without re-picking hundreds of photos, provided files still exist at original paths.

## Permission education versus Settings deep links

**Problem.** Android 13+ splits nearby Wi-Fi, notifications and all-files access across different settings pages; denial breaks features silently.

**Context.** First-run screen is informational; Settings must reflect live status.

**Approach.** Permission rows map to platform permission enums; Settings re-queries status on resume and opens the matching system screen on tap. Permanent nearby denial triggers a dedicated dialog with a settings shortcut immediately after the bulk request on first run.

**Outcome.** Support burden shifts from “why is nearby empty?” to actionable recovery steps.

## Analytics without leaking filenames

**Problem.** Product wants funnel metrics on pairing method and transfer outcome without publishing user file names to analytics backends.

**Context.** Central analytics wrapper wraps Firebase Analytics calls.

**Approach.** Events describe pairing method, result, failure reason categories and premium state—not individual file paths or contact identifiers in the patterns described here.

**Outcome.** Product telemetry supports debugging funnels while keeping payload content off analytics parameters in the architecture described.
