---
title: "SQP Kanban — Three Bugs That Broke Deployment"
date: 2026-05-14
draft: false
tags: ["docker", "mongodb", "mongoose", "node", "debugging", "devops"]
description: "The three bugs I hit while deploying a Node.js app to my homelab — from Mongoose schema mismatches to a browser API that only works on HTTPS."
---

Deploying SQP Kanban looked simple on paper. In practice, three bugs made it interesting.

## Bug 1 — Mongoose Refuses to Save

**Symptom:** Adding projects and tasks worked in the UI, but nothing persisted after a page refresh. The save status was stuck on "Saving..." forever.

**Two problems hiding as one.** First, the save status update in `saveBoard()` was a bare template literal that didn't actually call anything:

```javascript
// This does nothing — it's just a floating string expression
`Saved ${new Date().toLocaleTimeString()}`
```

Second — and this was the real blocker — the Mongoose schemas for subdocuments (projects, tasks, columns) used the default `_id` type of `ObjectId`, but the frontend was generating IDs with `crypto.randomUUID()`, which produces UUIDs like `b14a56c5-2628-4138-990c-cb685fdf04ce`. MongoDB ObjectIds are 24-character hex strings. Mongoose threw a `CastError` on every save attempt.

**Fix:** Added `_id: { type: String }` to the project, task, and column schemas so Mongoose accepts any string as an ID. Also had to drop the existing board collection since the old data had ObjectId-type `_id` values:

```bash
docker exec -it sqp-mongo mongosh sqp-kanban --eval "db.boards.drop()"
```

## Bug 2 — Podman Compose Compatibility

**Symptom:** `podman-compose up -d` failed with `AttributeError: 'str' object has no attribute 'get'` and later tried to pull a non-existent image from Fedora's registry.

**Cause:** `podman-compose` doesn't handle some Docker Compose shorthands. The compact `build: .` syntax and a bare volume declaration (`sqp-mongo-data:` with no properties) both caused failures.

**Fix:** Used the expanded build syntax with explicit `context`, `dockerfile`, and `image` fields. Added `driver: local` to the volume declaration. Both changes are backward-compatible with Docker Compose.

## Bug 3 — crypto.randomUUID() Only Works on HTTPS

**Symptom:** The app worked perfectly on `localhost:4000` during development, but showed "Could not load board" when accessed via `http://192.168.1.241:4000` on Bitfrost.

**This one was sneaky.** The server logs showed MongoDB connected, the API returned valid JSON when tested with `curl`, and even the browser's Network tab showed a 200 response. But the page wouldn't render.

The breakthrough came from adding `console.error` to the catch block — `crypto.randomUUID()` is a secure-context-only browser API. It works on `https://` and `http://localhost` (which browsers treat as secure), but fails on `http://192.168.1.241` with `TypeError: crypto.randomUUID is not a function`.

**Fix:** Replaced all `crypto.randomUUID()` calls with a `generateId()` helper that tries the native API first and falls back to a manual UUID generator:

```javascript
function generateId() {
  if (typeof crypto !== "undefined" && crypto.randomUUID) {
    return crypto.randomUUID();
  }

  return "xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx".replace(/[xy]/g, function (c) {
    const r = (Math.random() * 16) | 0;
    const v = c === "x" ? r : (r & 0x3) | 0x8;
    return v.toString(16);
  });
}
```

## Takeaway

All three bugs shared a theme: code that works in development doesn't necessarily work in production. `localhost` is a special case for browsers. Docker and Podman aren't perfectly interchangeable. And Mongoose's default `ObjectId` type silently rejects anything that isn't a 24-character hex string. The config files took minutes to write — the debugging took hours.
