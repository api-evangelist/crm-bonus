---
name: Load customer master data into Oto CRM
description: >-
  Upsert customers, their consent flags and the suppression list into the Oto CRM
  platform (CRM Bonus) through the Oto Data API, in staging first and then
  production, handling batch limits, validation errors and retry safety.
api: openapi/crm-bonus-oto-data-api-openapi.yml
operations:
  - login_auth_login_post
  - post_customers_v1_customers_post
  - post_blocked_v1_blocked_post
---

# Load customer master data into Oto CRM

Use this when a retailer's ERP, POS or marketing cloud needs to push its customer
base into Oto CRM so it can be segmented, messaged and matched against web
behaviour.

## Before you start

- You need a bearer token. There is **no self-service key page** — Oto/CRMBonus
  support issues it. Ask for staging and production separately; the two
  environments require different credentials.
- Base URL is `https://data-api.otocrm.com.br` for production and
  `https://data-api-hmg.otocrm.com.br` for staging (homologação). Writes in
  staging do not persist.
- **Always run staging first.** The provider's stated reason for the environment
  is that its schema validation surfaces bad source data before it reaches
  production.

## Step 1 — Get a token (only if you were given a username/password)

Call `login_auth_login_post` (`POST /auth/login`) with `{"username": …,
"password": …}`. It returns `{"access_token": …, "expires_in": 3600}`. Refresh
before the hour is up. If you were handed a long-lived token directly, skip this
step and use it as-is.

Send it on every subsequent call as `Authorization: Bearer <token>`. Omitting it
returns **403 `{"detail":"Not authenticated"}`** — note that it is 403, not 401,
so do not branch on 401 for a missing credential.

## Step 2 — Upsert customers

Call `post_customers_v1_customers_post` (`POST /v1/customers`) with the batch
envelope every operation on this API shares:

```json
{"data": [ { "customer_id": "…", "create_date": "…" }, … ]}
```

- `customer_id` and `create_date` are the only required fields.
- Records dedupe on **`customer_id` + `data_source`**.
- Ship the consent flags with the record — `email_permission`,
  `mobile_permission`, `whatsapp_permission`. They are how channel permission is
  carried; omitting them on a retry will clear them (see retry safety below).
- Account-specific `custom_field_NN` slots are configured with Oto in advance and
  then used for segmentation.

## Step 3 — Upsert the suppression list

Call `post_blocked_v1_blocked_post` (`POST /v1/blocked`) with the same envelope;
`customer_id` is required. This is the do-not-contact list. Treat it as the
highest-consequence write on the surface — an incorrect batch here contacts
people who opted out. Never let an agent write it without an explicit,
human-confirmed instruction.

## Batch limits

- Max **10,000 records** per request. Over that returns **413 Payload Too
  Large**. Chunk the array client-side.
- Max **200 requests per minute per source IP**. Excess responses are *delayed*
  first and eventually rejected with **429 Too many requests**. There is **no
  `RateLimit-*` or `Retry-After` header** on any response, so you cannot read
  remaining budget — pace at or under 200/min yourself and back off
  exponentially on a 429.

## Retry safety

Writes are upserts: a record whose primary key already exists is **replaced in
full**. That makes an identical replay safe and convergent — there is no
Idempotency-Key header and none is needed for that case.

The trap: because the replacement is total rather than a merge, replaying a
**trimmed** payload for an existing key **erases every field you left out**.
Always retry the exact original body. Never retry with a reduced field set.

## Handling errors

- **422** — validation failure. `detail` is an array of
  `{"loc": ["body", <record index>, "<field>"], "msg": …, "type": …}`. The index
  in `loc` points at the offending record inside your batch, so map it back to
  the source row and fix the source system rather than the payload.
- **403 `Not authenticated`** — missing/expired bearer token; re-authenticate.
- **404 `Not Found`** — wrong path. The API root itself is a 404; only the
  documented `/v1/*` and `/auth/login` paths exist.
- **500** — retry, then check <https://status.otocrm.com.br> ("API Pública"
  component) before escalating.

Errors are plain `application/json` `{"detail": …}` — **not** RFC 9457 problem
details, so do not look for `type`/`title` members.
