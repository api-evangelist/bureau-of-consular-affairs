---
name: Find the U.S. embassy or consulate serving a location
description: >-
  Look up a U.S. diplomatic post by country or city and return its address, phone, emergency phone,
  email and website — including which post actually provides consular services for that location.
api: openapi/bureau-of-consular-affairs-arcgis-embassy-consulate-locations-layer.json
operations:
  - FeatureServer/0/query
---

# Find the U.S. embassy or consulate serving a location

**Service**

```
https://services6.arcgis.com/R6wlO6UHmSzqm9Vs/arcgis/rest/services/Embassy_and_Consulate_Locations_View_Layer/FeatureServer/0/query
```

## Steps

1. **Query by country.** The country code field here is `GENC3` (the U.S. Government GENC scheme),
   **not** `ISO_3` as on the advisory layer. If you already have an ISO alpha-3 code, match on
   `Country` by name as a fallback and verify the result — the bureau publishes no GENC-to-ISO
   crosswalk.

   ```
   GET .../FeatureServer/0/query
     ?where=Country='France'
     &outFields=FullName,LocType,ConSvcProv,City,Address1,Address2,TelNo,EmgcyTelNo,EmailAddr,Website,PostCode
     &returnGeometry=false
     &f=json
   ```

2. **Read `ConSvcProv` before you answer.** A post's own record is not necessarily the post that
   provides consular services for that location — `ConSvcProv` names the one that does. Answering
   with the nearest post rather than the servicing post is the most common way to get this wrong.

3. **Use `LocType`** to distinguish an Embassy from a Consulate General, a Consular Agency or a
   Virtual Presence Post, and `EmbassyStatus` to see whether the post is currently operating.

4. **For an emergency, surface `EmgcyTelNo`, not `TelNo`.** They are different numbers and the
   emergency line is the one that is answered after hours.

5. **Geography.** `Lat` and `Long_` are plain numeric fields on the record, so you can return
   coordinates without asking for geometry. `google_directions` carries a prebuilt directions link.

## Errors

Same envelope as every ArcGIS service here: HTTP 200 with an `error` object in the body. Check the
body.
