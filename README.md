# Survey

A single-page survey app. Questions are read live from a Google Spreadsheet,
responses are saved to Cloud Firestore, and the Results tab visualizes them.
The survey runs in **rounds**: the admin begins a round, responses collect in
that round's own Firestore collection, and closing the round archives it —
past rounds stay selectable (by date) in the Results tab.

- **Live site:** https://student-survey-hksyu.web.app
- **Hosting / database:** Firebase (Hosting + Firestore) — project `student-survey-9b441`

## How it works

- **Questions** come from a Google Spreadsheet. In the page's *Survey settings*
  panel you set the spreadsheet link and a **sheet index** (0 = first tab). The
  tab's name is looked up at runtime and used as the survey's *base name*.
  - Sheet layout: first row is the headers `ID, Type, Question, Option1, Option2, …`;
    every following row is one question.
  - `Type` can be `Single-Choise`, `Multi-Choise`, `scale`, or `text`.
  - The spreadsheet must be shared as **Anyone with the link → Viewer**.
- **Rounds** — the survey only accepts responses while a round is open:
  - The admin presses **Begin survey** (settings panel) to open a round. Each
    round writes to its own collection named `<base>__<YYYY-MM-DD_HHMM>`
    (e.g. `Sheet1__2026-08-17_1432`).
  - **Close survey** stops submissions immediately. The round's collection is
    kept as-is — that *is* the archive — and its meta doc records the close
    date and response count.
  - Round bookkeeping lives in the `_surveys` collection: one doc per round
    (doc id = the round's collection name) with
    `{ base, status: 'open'|'closed', openedAt, closedAt, responseCount }`.
  - Closure is enforced by the **security rules** (not just the UI): a
    collection only accepts creates while its `_surveys` doc says `open`.
  - First **Begin survey** after upgrading: if the old plain `<base>`
    collection already holds responses, it is registered as a closed round so
    that data stays visible in Results.
- **Responses** are written as one document per submission
  (`{ answers, createdAt, uid }`, doc id = anonymous uid → one response per
  person *per round*) in the open round's collection.
- **Results** are **admin-only**: the Results tab appears only after logging in
  under *Survey settings*. It has a **round selector** — pick any round
  (labelled by date) and it reads every document in that round's collection and
  charts it. Note that charts always use the *current* sheet questions; if you
  change the questions between rounds, old rounds render against the new
  question list.

## Security note

The admin login is client-side (SHA-256 password gate) — presentation only.
That gate is what hides the Results tab: the *page* won't show results to
non-admins, but the underlying data stays publicly readable in Firestore
(`allow read: if true`), so a determined visitor could still fetch raw
responses — or open/close rounds — via the API. Responses themselves can't be
edited or deleted (one per uid). If stronger protection matters, add real
Firebase Auth for the admin, restrict reads and `_surveys` writes to that uid
in the rules, and aggregate results server-side.

## Project layout

```
public/index.html        the whole app (HTML + CSS + JS, no build step)
firebase.json            Firebase Hosting + Firestore config
firestore.rules          public read; create only into open rounds; _surveys meta
firestore.indexes.json   (none)
.firebaserc              default project
```

## Deploy

```
firebase deploy --only hosting            # deploys public/ to student-survey-hksyu.web.app
firebase deploy --only firestore:rules    # REQUIRED for the rounds feature to work
```

> ⚠️ After deploying the new rules, the survey is **closed by default** —
> nobody can submit until the admin presses **Begin survey** in the settings
> panel. (GitHub Actions auto-deploys *hosting* only on push to `main`; the
> rules must be deployed manually with the command above.)

## Prerequisites

- The **Google Sheets API** must be enabled on the project and allowed on the
  Firebase web API key (used for the read-only sheet-name lookup).
- Firestore database created (Native mode).
- **Anonymous authentication** enabled (used for one-response-per-person and
  for writing round meta docs).
