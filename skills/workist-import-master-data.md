---
name: Import ERP master data into Workist
description: >-
  Keep Workist's validation data in sync with your ERP - run a batched
  master data import (customers, addresses, articles, contacts) in chunks
  of up to 1000 records.
api: openapi/workist-integrations-openapi-original.yml
operations: [imports_create, imports_data_create, imports_start_create, imports_retrieve]
generated: '2026-07-21'
method: generated
---

# Import ERP master data into Workist

WorKI validates extracted documents against your ERP master data, so keep it
fresh. Base URL `https://api.workist.com/api/v1`, bearer-token auth.
Docs: https://docs.workist.com/en/docs/api-documentation/master-data-api/

## Steps (batched flow)

1. **Create the import run** - `imports_create`
   (`POST /master-data/imports`) with
   `{"lookup_definition_id": "<id>", "batched": true}`. Keep the returned
   import id.
2. **Append data in chunks** - `imports_data_create`
   (`POST /master-data/imports/{id}/data`), at most **1000 elements per
   batch**, repeating until all records are uploaded.
3. **Start ingestion** - `imports_start_create`
   (`POST /master-data/imports/{id}/start`) to ingest everything appended.
4. **Check status** - `imports_retrieve` (`GET /master-data/imports/{id}`)
   until `status` is `SUCCESS` (response also carries `lookup_type`).

## Rules

- Single-shot alternatives exist per data family (e.g.
  `clients_imports_create` at `POST /master-data/clients/imports?replace=True`,
  `articles_imports_create`, `addresses_imports_create`) - customers require
  `client_id1` + `company_name`; articles' `order_units` must be an array.
- Customers are the only required master data group; addresses, articles,
  contacts, conversion factors, framework contracts, and order/offer
  matching data are optional.
- 202 responses mean asynchronous processing - poll step 4 rather than
  retrying the POST (no idempotency-key contract exists).
