---
name: Check the U.S. travel advisory level for a country
description: >-
  Look up the current U.S. Department of State Travel Advisory level (1-4) for one or more countries,
  with the advisory date and a link to the human-readable advisory page. Anonymous, no key.
api: openapi/bureau-of-consular-affairs-arcgis-travel-advisory-levels-layer.json
operations:
  - FeatureServer/54/query
---

# Check a travel advisory level

The advisory data lives in an ArcGIS Feature Service, not in the CKAN catalog and not on
travel.state.gov (which blocks automated clients — see the warning at the bottom).

**Service**

```
https://services6.arcgis.com/R6wlO6UHmSzqm9Vs/arcgis/rest/services/Travel_Advisory_Levels__(2024)_View_Layer/FeatureServer/54/query
```

## Steps

1. **Query by ISO 3166-1 alpha-3 code.** Always prefer `ISO_3` over `NAME` — country names in this
   layer are not normalized and the layer contains rows whose `NAME` is null.

   ```
   GET .../FeatureServer/54/query
     ?where=ISO_3='AFG'
     &outFields=NAME,LEVEL_,URL,ISO_3,ADVDATE
     &returnGeometry=false
     &f=json
   ```

2. **Always send `returnGeometry=false`** unless you need the polygon. These are country boundary
   polygons; leaving geometry on turns a few hundred bytes into megabytes.

3. **Read the result.** `LEVEL_` is the advisory level: 1 Exercise Normal Precautions, 2 Exercise
   Increased Caution, 3 Reconsider Travel, 4 Do Not Travel. `URL` is the advisory page to cite to a
   human. `ADVDATE` is epoch milliseconds — convert before showing it.

4. **Check for empty rows.** The layer includes rows where every attribute is null (boundary
   fragments carrying no advisory). Filter them out rather than reporting "unknown level".

5. **Paging.** `maxRecordCount` is 2000. If you query all countries at once and the response carries
   `"exceededTransferLimit": true`, page with `resultOffset` and `resultRecordCount`.

## Errors

The service returns **HTTP 200 on failure**. A failed call looks like:

```json
{"error":{"code":400,"message":"Invalid URL","details":["Invalid URL"]}}
```

Test for the presence of an `error` key in the body. Do not branch on the status code.

## Watch out

- `travel.state.gov` sits behind a Cloudflare WAF that returns 403 to most automated clients. Do not
  scrape the advisory pages; use this service, or the RSS feed at
  `https://travel.state.gov/_res/rss/TAsTWs.xml`, which is not blocked.
- The advisory level can change at any time and there is no webhook. The RSS feed is the change
  signal: each item's description states what changed.
