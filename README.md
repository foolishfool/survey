# Survey

A single-page survey app. Questions are read live from a Google Spreadsheet,
responses are saved to Cloud Firestore, and the Results tab visualizes them.

- **Live site:** https://student-survey-hksyu.web.app
- **Hosting / database:** Firebase (Hosting + Firestore) — project `student-survey-9b441`

## How it works

- **Questions** come from a Google Spreadsheet. In the page's *Survey settings*
  panel you set the spreadsheet link and a **sheet index** (0 = first tab). The
  tab's name is looked up at runtime and used as the Firestore collection.
  - Sheet layout: first row is the headers `ID, Type, Question, Option1, Option2, …`;
    every following row is one question.
  - `Type` can be `Single-Choise`, `Multi-Choise`, `scale`, or `text`.
  - The spreadsheet must be shared as **Anyone with the link → Viewer**.
- **Responses** are written to Firestore as one document per submission
  (`{ answers, createdAt }`) in the collection named after the sheet.
- **Results** reads every document in that collection and charts it.

## Project layout

```
public/index.html        the whole app (HTML + CSS + JS, no build step)
firebase.json            Firebase Hosting + Firestore config
firestore.rules          public read + create; no client update/delete
firestore.indexes.json   (none)
.firebaserc              default project
```

## Deploy

```
firebase deploy --only hosting     # deploys public/ to student-survey-hksyu.web.app
firebase deploy --only firestore:rules
```

## Prerequisites

- The **Google Sheets API** must be enabled on the project and allowed on the
  Firebase web API key (used for the read-only sheet-name lookup).
- Firestore database created (Native mode).
