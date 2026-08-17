# Commissary Bagger Queue

A live queue-management system for commissary baggers — customers (baggers) join a virtual line from their phone, and staff run the floor from a separate admin panel. Built as two static single-file web apps backed by a shared Firebase Realtime Database, no custom backend server.

**Live customer app:** https://baggerqueue.web.app/
*(This repo's GitHub Pages copy at `reidsterking.github.io/Commissary-App` is a backup mirror only — the link above is the one customers actually use.)*

The admin panel runs on a separate Firebase Hosting site and is not published in this public repo, since it holds the staff password and direct queue-management controls.

## What it does

### Customer app (`index.html`)

- **Join the queue** with a name and a bagger number (digits only, up to 3). Both must match an entry on the admin-managed roster — if they don't, the customer is offered a one-tap "submit a request" instead of just being turned away, which shows up in the admin panel for approval or denial.
- Each roster identity (name + number) can only have **one active spot in the queue at a time** — a second device can't take over or duplicate an already-active person's spot.
- **Live queue view**, grouped into "At the Register," "In Line," and "On Break," each showing join time and live position.
- **Self-service status changes**: move yourself to the register, to the back of the line, or to break, with one tap.
- The person at the **front of the line shows a skip count** ("Skipped 2x") if others have been called to a register ahead of them while they waited.
- **Custom icon** next to your name — set any single emoji, or up to two letters, in place of your initials.
- **Status notifications**: an alert (and optional sound) when you reach spot #1 or #2 in line, or when an admin moves you. Real OS-level notifications work on desktop and Android Chrome; a plain browser tab on iPhone Safari can't do this (no push infrastructure behind a static site), so iPhone users only see in-page alerts. A mute button silences the sound without affecting the alert itself.
- **Installable as an app** (PWA) — "Add to Home Screen" gets a real app icon on both iOS and Android, no app store needed.
- Tapping your own name lets you leave the queue or cancel a pending-removal timer; tapping someone else's name starts (or cancels) a 10-minute pending-removal timer for them, visible to everyone with a live countdown.
- A join-password gate protects the queue from randoms — the password itself, and whether the queue is open at all, are both controlled live from the admin panel.

### Admin panel (separate hosting site, not in this repo)

- **Bagger roster** — add or remove the name/number pairs allowed to join; a number can't be silently assigned to two different names (you're warned and offered to reassign it).
- **Pending profile requests** — approve or deny self-service join requests from workers who weren't yet on the roster.
- **Live queue management** — drag-and-drop reordering (marks whoever was dragged as "moved by admin"), send anyone to Register/Break/back of the Line, kick, block, or fix a name/number typo.
- **Block list** — block by name or by device; blocks auto-expire after 45 minutes.
- **Auto-kick**, checked continuously while the panel is open: anyone at the front of the line who's been skipped 3 times, OR anyone sitting in line/break for 45+ minutes with no status change, is removed automatically. (People currently at a register are exempt from the inactivity rule.)
- **Activity log** — the panel shows the last 10 events; the full history is kept and auto-trimmed to the most recent 100 entries (checked continuously while the panel is open), with a manual "clear now" button and a full CSV export that isn't limited by the 100-entry cap.
- **Daily reset** — clears the live queue for a new day, logged distinctly from the general "kick everyone" action; the roster and block list are left untouched by both.
- **Queue open/closed toggle**, a **"log everyone out"** action that forces every browser (even ones already past the password screen) to re-enter the join password, and a **password editor with a QR code** pointing at the customer join URL, so staff can post a printed sign that's always current.
- A continuous self-healing "guardian" check re-removes blocked people and stale entries even if something slips through a race condition, as long as an admin device has the panel open.

## Data model

Everything lives in one Firebase Realtime Database, fully client-writable — there's no server-side validation, so all the rules above are enforced in the app code on both ends, not by the database itself.

| Path | Contents |
|---|---|
| `people/{clientId}` | One entry per active queue participant: `name`, `baggerNumber`, `status` (`line` \| `register` \| `break`), `ts`, `order`, `icon`, `skippedCount`, `pendingRemovalAt`, `movedByAdmin` |
| `roster/{name__number}` | Approved bagger identities: `name`, `baggerNumber`, `addedAt` |
| `rosterRequests/{name__number}` | Pending self-service join requests awaiting admin approval |
| `blockedNames/{name}`, `blockedClients/{clientId}` | Active blocks, each with a `blockedAt` timestamp for the 45-minute auto-expiry |
| `activityLog/{pushId}` | Every join/move/kick/block/skip/reset event, `type` + `name` + `ts`, auto-trimmed to the most recent 100 |
| `siteStatus/enabled` | Whether the queue is open to new joins |
| `siteStatus/password` | Current customer join password |
| `siteStatus/authVersion` | Bumped to force every browser back to the password screen |

## Setup

1. **Create a Firebase project** at https://console.firebase.google.com/, then create a Realtime Database.
   For quick testing, database rules of `{"rules": {".read": true, ".write": true}}` allow the app to work out of the box — tighten this before relying on it for anything sensitive, since it currently trusts every client completely.

2. **Get your Firebase config** from Project Settings → SDK setup, and paste it into the `firebaseConfig` object near the top of `index.html` (and the admin panel's `index.html`, kept separately).

3. **Host the customer app.** This repo is set up to deploy to GitHub Pages as a backup. For a real deployment, Firebase Hosting with two hosting targets (one for the customer app, one for the admin panel, each with its own site) keeps the admin panel off the public web app's domain and out of this public repo.

4. **Set the admin password** in the admin panel's own `index.html` (`ADMIN_PASSWORD` constant) before deploying it anywhere — that file is intentionally not committed to this public repository.

## Notes on the architecture

This is deliberately a static, backend-free app — every rule (roster checks, block expiry, auto-kick, activity log trimming) runs in client-side JavaScript rather than on a server. That keeps hosting free and setup simple, but it means enforcement only happens while a browser tab is actually open and running the code: the self-healing "guardian" checks, the auto-kick sweep, and the activity log trim all depend on the admin panel being open somewhere. It also means the Realtime Database rules are the only real security boundary — the passwords in the app UI are a courtesy gate for typical customers, not a substitute for locking down the database itself if that matters for your deployment.
