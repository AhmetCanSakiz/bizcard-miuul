---
name: bizcard-conventions
description: Project conventions for the BizCard digital business card app - how components should be structured (single file, function components, demo data location) and the exact JSON contract for the "Kartı Kaydet" (Save Card) and "Toplantı Talep Et" (Request Meeting) webhook actions. Use this whenever adding, editing, or reviewing components in BizCard's HTML/React files, whenever demo/sample data is touched, or whenever wiring up the save-card or meeting-request buttons to a webhook (e.g. a Power Automate flow). Trigger even if the user just says "add a button" or "hook this up" without naming the contract explicitly.
---

# BizCard Conventions

BizCard is a course project for non-developer business people ([[CLAUDE.md]] at the
project root). The code must stay simple and readable — no build tools, no npm, no
bundler. Keep that in mind before suggesting anything that would need one.

## Component rules

1. **Single file.** The whole app lives in one HTML file (currently `react.html`).
   Do not split components into separate `.jsx`/`.tsx` files — there is no bundler to
   stitch them back together, and splitting files would make the project harder for a
   non-developer to follow. Multiple components (`Avatar`, `ContactList`, `ProfileCard`,
   etc.) can and should exist side by side inside the same `<script type="text/babel">`
   block.

2. **Function components only.** Every component is a plain JS function using hooks
   (`useState`, `useMemo`, ...) — never a class component. This matches the existing
   `Avatar` / `ContactList` / `ProfileCard` components in `react.html`.

3. **Demo data lives in `src/data/card.js`, not inline in the component file.**
   - Create `src/data/card.js` as a plain (non-module) script. It should declare the data
     as a top-level `const`, e.g.:
     ```js
     const CARD_DATA = {
       name: "Ahmet Can Sakız",
       title: "Eğitmen & Miuul",
       company: "",
       contacts: [ /* ... */ ],
       socials: [ /* ... */ ],
     };
     ```
   - Load it with a classic `<script src="src/data/card.js"></script>` tag placed
     **before** the `<script type="text/babel">` block in the HTML file.
   - Why this works without a bundler: classic (non-module) `<script>` tags on the same
     page share one top-level scope, so a `const` declared in `card.js` is directly
     visible to the Babel script that follows it — no `import`/`export`, no build step.
   - Never use `import`/`export` or `<script type="module">` for this — it breaks the
     zero-build setup and won't work if the page is ever opened via `file://`.
   - The component file references the data as `CARD_DATA` (or destructures from it) —
     it should never hardcode the demo profile inline once `card.js` exists.
   - Per the project's constraints, this is still placeholder/demo data, not real contact
     info — keep it obviously fake unless the user says otherwise.

## Webhook data contract

The card currently has no backend ([[CLAUDE.md]]: "Bu aşamada backend/veritabanı yok").
The two visitor-facing actions — **"Kartı Kaydet"** (a visitor saves/shares their
interest in this card) and **"Toplantı Talep Et"** (a visitor requests a meeting) — are
designed to POST a JSON payload to a webhook URL later (most likely a Power Automate
HTTP-trigger flow, given the user's background). Until a real endpoint exists, buttons
should build this exact payload and log it / show a confirmation toast instead of
calling `fetch` against a placeholder URL.

Both payloads share an envelope:

```json
{
  "action": "save_card | request_meeting",
  "timestamp": "2026-07-23T18:30:00+03:00",
  "source": "bizcard-web",
  "visitor": { "...": "see per-action shape below" }
}
```

- `timestamp` is ISO 8601, local offset included (matches `new Date().toISOString()`
  behavior is fine too — don't overthink the format, just be consistent).
- `source` is always the literal string `"bizcard-web"` — it lets a downstream flow
  tell BizCard submissions apart from other webhook callers later.

### `save_card`

Visitor wants to save/register interest in the card owner's contact info.

```json
{
  "action": "save_card",
  "timestamp": "2026-07-23T18:30:00+03:00",
  "source": "bizcard-web",
  "visitor": {
    "name": "string, required",
    "email": "string, required",
    "phone": "string, optional",
    "company": "string, optional"
  }
}
```

### `request_meeting`

Visitor wants to book time with the card owner.

```json
{
  "action": "request_meeting",
  "timestamp": "2026-07-23T18:30:00+03:00",
  "source": "bizcard-web",
  "visitor": {
    "name": "string, required",
    "email": "string, required",
    "phone": "string, optional"
  },
  "meeting": {
    "preferredDate": "YYYY-MM-DD, optional",
    "preferredTime": "string, optional (e.g. \"14:00\" or \"afternoon\")",
    "message": "string, optional — free-text note from the visitor"
  }
}
```

### Rules for building these payloads in code

- Only `name` and `email` are required on `visitor` — everything else is optional and
  should be omitted from the JSON (not sent as `null` or `""`) when the visitor didn't
  fill it in.
- Keep the two actions as separate payload shapes (don't merge them into one generic
  "contact form" object) — they represent different intents and a downstream Power
  Automate flow will likely route them differently (e.g. one to a CRM row, one to a
  calendar-booking flow).
- If asked to actually send the webhook, don't invent a real URL — ask the user for the
  Power Automate HTTP-trigger URL (or wherever it should point) rather than guessing one.
