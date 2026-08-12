---
name: Load transactions and catalog into Oto CRM
description: >-
  Push the retail backbone — stores, sellers, products, orders and order items —
  into Oto CRM (CRM Bonus) through the Oto Data API in the right dependency
  order, so orders resolve against real stores, sellers and SKUs.
api: openapi/crm-bonus-oto-data-api-openapi.yml
operations:
  - post_stores_v1_stores_post
  - post_sellers_v1_sellers_post
  - post_products_v1_products_post
  - post_orders_v1_orders_post
  - post_order_items_v1_order_items_post
---

# Load transactions and catalog into Oto CRM

Use this for the recurring sync that keeps Oto's view of a retailer's operation
current: the shops, the staff, the SKUs, and the sales.

## Order matters

Orders and order items reference stores, sellers and products by id. Load the
dimensions before the facts:

1. `post_stores_v1_stores_post` — `POST /v1/stores`. Required: `id`, `name`.
   Carries `cnpj`, `type`, `cluster`, geography, `ecommerce_id`/`store_vtex` for
   cross-system matching, and `custom_field_NN` slots.
2. `post_sellers_v1_sellers_post` — `POST /v1/sellers`. Required: `id`, `name`.
   `store` binds the seller to a store; `disabled` retires one without deleting
   history.
3. `post_products_v1_products_post` — `POST /v1/products`. Required: `sku`, `id`,
   `name`. `sku` is the key that order items and Oto Tags interactions join on —
   get it consistent across systems or the joins silently miss.
4. `post_orders_v1_orders_post` — `POST /v1/orders`. Required: `customer_id`,
   `date`, `id`, `store`, `total`. Also carries `canceled`, `status`,
   `payment_method`, `payment_type`, `installments` and `custom_field_NN`.
5. `post_order_items_v1_order_items_post` — `POST /v1/order_items`. Required:
   `date`, `id`, `store`, `sku`, `seller_id`, `quantity`, `price`. `id` is the
   **order** id; `seq` distinguishes lines within it; `list_price` vs `price`
   carries the discount.

Customers should already be loaded — see the *Load customer master data* skill.
An order references `customer_id`.

## The shared contract

Every one of these is a `POST` taking the same envelope:

```json
{"data": [ … ]}
```

- Authenticate with `Authorization: Bearer <token>` (support-issued, or a JWT
  from `POST /auth/login`). Missing credential returns **403**, not 401.
- **10,000 records** max per request → **413** over the limit.
- **200 requests/minute per source IP** → responses are throttled, then **429**.
  No rate-limit headers are returned; pace yourself.
- Success returns `{"success": …, "requestId": …}`. Log `requestId` — it is the
  only correlation identifier available, and it comes from the server, not from
  you.

## Retry safety

All writes are upserts keyed on the required fields, and a matching record is
**replaced in full**. Replaying the same body is safe. Replaying a partial body
wipes the omitted fields — always retry the original payload verbatim.

## Cancellations, not deletions

There is no delete operation anywhere on this API. Reverse a sale by re-upserting
the order with `canceled` set, and the line with `order_item_canceled` set. An
agent that "cannot find the delete endpoint" is not missing documentation — the
model is soft-cancel by design.

## Validate in staging first

Run the full sequence against `https://data-api-hmg.otocrm.com.br` before
production. It runs the same validation and does not persist. Read the **422**
`detail[].loc` array — `["body", <record index>, "<field>"]` — to map each
failure back to the exact source row.
