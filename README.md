# I'm at 1% — Live Battery Assistance (self-hosted server)

A real Node.js server for the app: no Firebase, no third-party backend.
Everyone who opens the site — from any device, anywhere — sees the same
live data (accounts, online status, requests, chat).

## What's inside
- `server.js` — Express API + WebSocket server + JSON-file storage (`data.json`, created automatically)
- `public/index.html` — the whole front-end (map, dashboard, requests, "Who To Help", chat, profile)
- Auth: email + password, hashed with bcrypt, sessions via JWT
- Realtime: a WebSocket pushes presence/request/message updates to every connected browser instantly

## Run it locally
```bash
npm install
npm start
```
Then open http://localhost:3000 — that's it, front-end and API are served from the same place.

## Deploy it for free (pick one)

### Render.com
1. Push this folder to a GitHub repo.
2. Render → New → Web Service → connect the repo.
3. Build command: `npm install`  ·  Start command: `npm start`
4. Add an environment variable `JWT_SECRET` set to any long random string (Render can generate one).
5. Deploy. Render gives you a public `https://yourapp.onrender.com` URL — share that.

### Railway.app
1. Push to GitHub, then Railway → New Project → Deploy from GitHub repo.
2. Railway auto-detects Node and runs `npm start`.
3. Add the `JWT_SECRET` variable in the project's Variables tab.
4. Railway gives you a public URL under Settings → Networking.

### Fly.io / any VPS
Any host that runs `node server.js` and exposes `$PORT` works the same way —
`npm install && npm start`, set `JWT_SECRET`, done.

## What's new in v2
- **Security**: random JWT secret auto-generated if `JWT_SECRET` is unset; rate limiting (auth + API + messages); security headers; strict input validation; 8+ char passwords; no user enumeration on login; change-password (signs out other sessions) and delete-account.
- **Privacy**: locations and presence are memory-only (never written to disk) and expire ~90 s after a user goes quiet; requests are only visible to their owner, the accepting helper, and available helpers (for pending ones).
- **Reliability**: debounced atomic writes flushed on shutdown, corrupt-file backup, presence sweeper, WebSocket ping/pong, coalesced live updates, graceful SIGTERM, `/healthz`, `DATA_DIR` for persistent disks.
- **Features**: request lifecycle (Pending → Accepted → Resolved / Cancelled / auto-Expired after 6 h), one open request per user, unread badges, last-message previews, read receipts, typing indicator, helper push toasts + browser notifications, distance from you, reconnect resync and offline banner, installable PWA (manifest + service worker).
- **Fewer dependencies**: `uuid` removed (uses `crypto.randomUUID`).

## Important notes
- **Persistence**: set `DATA_DIR` to a mounted disk path (e.g. `/var/data`) on Render/Railway/Fly so `data.json` survives redeploys.
- **HTTPS** is required for location sharing; Render/Railway provide it automatically.
- **Scale**: JSON storage suits a class/campus pilot. For more, move `db` to SQLite/Postgres.
- **Still on the wishlist**: password reset by email, token in an HttpOnly cookie, SRI hashes for the Leaflet CDN files, automated tests.

## API summary
| Method | Path | Purpose |
|---|---|---|
| POST | /api/auth/register · /login | `{token,user}` |
| GET / PUT / DELETE | /api/me | profile · update · delete account |
| POST | /api/me/password | change password → new token |
| POST | /api/location · /heartbeat · /offline | presence |
| GET | /api/users | others' public profile/presence |
| GET / POST | /api/requests | visible requests · create |
| POST | /api/requests/:id/accept · /resolve · /cancel | lifecycle |
| GET | /api/conversations | last message + unread per person |
| GET / POST | /api/messages | thread (`?with=ID`) · send |
| POST | /api/messages/read | mark thread read |
| GET | /healthz | health check |
| WS | /ws?token=… | `users:update`, `requests:update`, `request:new`, `message:new`, `message:read`, `typing` |
