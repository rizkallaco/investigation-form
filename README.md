# investigation-form — Collections Field-Investigation Form

**Status: ACTIVE.** Single-page Arabic (RTL) form — "نموذج التحصيل" — used by Rizkalla
collections field agents to log customer/guarantor visits. The entire app is one static
`index.html` (vanilla HTML/JS, no build step, no dependencies).

## What it captures / where it goes

Form fields: visit type, collector name, **card number — exactly 14 digits**, status,
comment and address are required. Promised payment date is optional and uses a date-only
input.

On submit it POSTs JSON to the Firebase Cloud Function:

```
https://us-central1-rizpay-abb7d.cloudfunctions.net/collectionTicketSubmit
```

(project `rizpay-abb7d`; the function source lives in the sibling
`customer-installments-portal` repository).

## Geolocation is enforced

- On page load the browser requests high-accuracy GPS; the **submit button stays
  disabled until coordinates are captured** (hidden `location` field, format
  `"lng,lat"` — longitude first).
- No location permission = no submission (client-side hard block). Agents must allow
  location; the page must therefore be served over **HTTPS** (or localhost), since
  browsers block geolocation on plain HTTP.
- After a successful submit the form resets and re-fetches location.

## Run / deploy

- Local test: just open `index.html` (geolocation works on `file://`/localhost in most
  browsers) or serve statically.
- Production: GitHub Pages at `https://rizkallaco.github.io/investigation-form/`.

No env vars, no secrets, no credentials anywhere in this project.

## Git — own repo, do not use the parent workspace

This folder is its **own GitHub repo**: `https://github.com/rizkallaco/investigation-form.git`.
Commit here directly, never through any parent workspace repo.

## Gotchas

- The 14-digit "card number" (رقم البطاقة) is the customer card/national-style ID used
  to match the customer downstream — validation is `pattern="[0-9]{14}"` plus a JS
  regex check; changing it breaks matching in the backend flow.
- Collector names are hardcoded in the dropdown — staff changes require editing
  `index.html` and redeploying.
- Backend contract: JSON body keys `visitType`, `collectorName`, `cardNumber`,
  `status`, `promisedDate`, `comment`, `address`, `location` — keep in sync with
  `collectionTicketSubmit`.

## Sibling projects (new names)

- `branch-callcentre-form` — the other agent-facing form (installments) posting to a
  cloud function on the same Firebase project.
- `riz-sms-service` — collections SMS side of the same collections workflow.
