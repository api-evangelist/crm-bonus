---
name: Load loyalty, NPS and Oto Tags web behaviour into Oto CRM
description: >-
  Push cashback credits, NPS responses and the Oto Tags web event stream into Oto
  CRM (CRM Bonus), including binding anonymous web sessions to known customers
  with hashed identifiers.
api: openapi/crm-bonus-oto-data-api-openapi.yml
operations:
  - post_cashback_v1_cashback_post
  - post_nps_v1_nps_post
  - post_tag_hit_v1_tag_hits_post
  - post_tag_interaction_v1_tag_interactions_post
  - post_tags_ids_v1_tag_ids_post
---

# Load loyalty, NPS and Oto Tags web behaviour

The engagement layer on top of the transactional core: what a customer is owed,
what they think, and what they did on the site.

## Cashback credits

`post_cashback_v1_cashback_post` — `POST /v1/cashback`. Required: `customer_id`,
`order_date`, `order_id`, `order_store`, `credit`, `valid_until`. Optional
`seller_id` and `canceled`.

This operation moves **customer-redeemable value**. Treat it as the
highest-consequence write after the suppression list:

- Never let an agent originate a `credit` amount. The amount comes from the
  retailer's own promotion engine; the agent's job is transport.
- `valid_until` is required — an expiry is not optional in this model.
- Reverse a credit by re-upserting with `canceled` set; there is no delete.

## NPS responses

`post_nps_v1_nps_post` — `POST /v1/nps`. Required: `id`, `event_ts`,
`customer_id`, `order_date`, `order_id`, `order_store`. Carries `score`, ten
`rating_01`…`rating_10` sub-scores, and a free-text `comment`. The comment is
customer-authored text — do not summarise, translate or rewrite it on the way
in; load it verbatim.

## Oto Tags — the web event stream

Three endpoints, loaded in this order:

1. `post_tag_hit_v1_tag_hits_post` — `POST /v1/tag_hits`. Required: `event_ts`,
   `user_id`, `session_id`. Pageviews, with `url`, `referer`, `user_agent` and
   `utm_source`/`utm_medium`/`utm_campaign` attribution.
2. `post_tag_interaction_v1_tag_interactions_post` — `POST /v1/tag_interactions`.
   Required: `event_ts`, `user_id`, `session_id`, `type`, `id`. Product-level
   interactions; `sku` joins to the catalog you loaded with the transactions
   skill.
3. `post_tags_ids_v1_tag_ids_post` — `POST /v1/tag_ids`. Required: `event_ts`,
   `user_id`. **Identity resolution** — binds an anonymous `user_id` to a known
   customer.

### Identity resolution: prefer the hashes

`/v1/tag_ids` accepts `customer_id`, `mkt_cloud_id` **and** hashed forms —
`customer_id_sha256`, `email_sha256`, `email_md5`. Send the hashed identifiers
where your side can compute them; matching then works without transmitting
cleartext PII. Agree the normalisation (lowercase, trim) with Oto first, because
a hash mismatch fails silently — it simply does not match, and nothing errors.

## The shared contract

Same as every other operation on this API: `{"data": [ … ]}`,
`Authorization: Bearer <token>`, **10,000 records** max per request (**413** over),
**200 requests/minute per source IP** (throttled, then **429**, with no
rate-limit headers to read). Success returns `{"success": …, "requestId": …}`.

Writes are upserts keyed on the required fields and replace a matching record in
full — identical replays are safe, partial replays erase omitted fields.

Event endpoints are the highest-volume part of this surface. Batch to the 10,000
ceiling, keep to one in-flight request at a time per source IP, and back off
exponentially on 429 — there is no header telling you how much budget is left.

## Check status before escalating

`https://status.otocrm.com.br` publishes separate components for **API Pública**,
**Integração de Dados** and **Oto Tags**, so a Tags-only failure is visible
independently of the rest of the API.
