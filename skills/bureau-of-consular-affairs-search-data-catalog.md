---
name: Search the CA Data Catalog and pull a dataset
description: >-
  Find passport, visa, and adoption statistics datasets in the Bureau of Consular Affairs CKAN
  catalog, then read their rows out of the datastore.
api: openapi/bureau-of-consular-affairs-discovery-api-openapi.yml
operations:
  - packageSearch
  - packageShow
  - resourceShow
  - datastoreSearch
  - datastoreSearchSql
  - organizationList
  - tagList
---

# Search the CA Data Catalog and pull a dataset

The catalog is a stock CKAN deployment. Base: `https://cadatacatalog.state.gov/api/3/action`.

## Before you start

`cadatacatalog.state.gov` is fronted by a Cloudflare WAF that returned **HTTP 403 "Sorry, you have
been blocked"** to every automated request from API Evangelist's pipeline on 2026-09-05 — including
plain reads, across several User-Agent strings, on both HTTP and HTTPS. If you get a 403 HTML page,
that is the WAF, not an auth failure, and no key will fix it. Fall back to the human catalog at
`https://cadatacatalog.state.gov/` in a browser, or to the mirrored datasets published as ArcGIS
Feature Services.

## Steps

1. **Find datasets** — `packageSearch` with `q`, `rows` and `start`:

   ```
   GET /action/package_search?q=passport&rows=20
   ```

   Use `tagList` and `organizationList` first if you want to browse the controlled vocabulary rather
   than guess keywords.

2. **Open one dataset** — `packageShow` with the dataset's `name` (the URL slug) or `id`:

   ```
   GET /action/package_show?id=passportstatistics
   ```

   The result carries `resources[]`. Each resource is one file or table.

3. **Decide how to read the data.** If a resource has `datastore_active: true`, query it as rows.
   Otherwise fetch `resource.url` directly and parse the file yourself (`format` tells you what it is
   — CSV, XLSX, XML).

4. **Query rows** — `datastoreSearch` with the resource id:

   ```
   GET /action/datastore_search?resource_id=<id>&limit=100&offset=0
   ```

   For aggregation, `datastoreSearchSql` accepts a read-only SQL statement. Treat it as read-only —
   it is not a write path.

5. **Page.** CKAN's `limit`/`offset` pair; read `result.count` for the total.

## Errors

CKAN returns **HTTP 200 on failure**. The body is the discriminator:

```json
{"help":"...","success":false,"error":{"__type":"Not Found Error","message":"..."}}
```

Check `success` before touching `result`. The documented `__type` values are `Not Found Error`,
`Validation Error` and `Authorization Error`. Malformed requests may instead come back as 400, 409
or 500.

## Do not write

`packageCreate`, `packageUpdate` and `packageDelete` exist on this catalog and require a CKAN API
token in an `Authorization` header. Do not call them. There is **no idempotency key** on this API — a
retried `packageCreate` creates a second dataset — and `packageUpdate` has **no undo**.
