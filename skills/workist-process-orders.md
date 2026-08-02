---
name: Process orders through Workist
description: >-
  Upload an order document to Workist, wait for WorKI's AI extraction to
  finish, pull the structured result into your ERP, and mark it imported.
api: openapi/workist-integrations-openapi-original.yml
operations: [orders_create, orders_list, orders_retrieve, orders_mark_imported_update]
generated: '2026-07-21'
method: generated
---

# Process orders through Workist

Base URL `https://api.workist.com/api/v1` - every call needs
`Authorization: Bearer <token>` (token created in the Workbench under
Integrations > API Integrations). See `../authentication/workist-authentication.yml`.

## Steps

1. **Upload the order** - `orders_create` (`POST /orders`) with the incoming
   document. A `202 Accepted` means WorKI is processing it asynchronously.
2. **Poll for finished records** - `orders_list` (`GET /orders`) with
   `imported=False&finished=True` (optionally scope with `channel`,
   `date_from`/`date_to` in ISO 8601). Pagination is page-number style:
   follow `next` until null; records are in `items[]`.
3. **Fetch the extraction** - `orders_retrieve` (`GET /orders/{id}`) returns
   the processing record with the extracted `Order` (header fields +
   `line_items[]`, matched `customer`, delivery address, contact person).
4. **Import into your ERP**, then **mark it imported** -
   `orders_mark_imported_update` (`PUT /orders/{id}/mark_imported`). Call it
   exactly once per record: Workist publishes no idempotency-key contract
   (`../conventions/workist-conventions.yml`), so the imported flag is your
   dedup boundary.

## Rules

- There are no webhooks - this is a pull integration; poll on a schedule.
- Errors are plain HTTP statuses (400/401/404/500) - see
  `../errors/workist-problem-types.yml`. On 401, re-check the bearer token.
- The same flow works for invoices, delivery notes, order confirmations,
  RFQs, property bills, and lists of services (same envelope per document type).
