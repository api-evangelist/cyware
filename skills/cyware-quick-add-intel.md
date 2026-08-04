---
name: Create intel in Cyware Intel Exchange with Quick Add Intel
description: >-
  Turn an IOC list or free-text report into a STIX intel package in Intel Exchange, confirm the
  ingestion completed, and inspect the objects it created.
api: openapi/cyware-intel-exchange-openapi.yml
generated: '2026-08-04'
method: generated
operations:
  - createParseIocsTask
  - quickAddIntel
  - createIntelViaOpenApi
  - retrieveQuickAddIntelStatus
  - retrieveQuickAddIntelRelationObjects
  - listCountryCodes
  - listRegions
---

# Create intel in Cyware Intel Exchange with Quick Add Intel

## Before you start

Same auth contract as every Intel Exchange call: `AccessID` + `Expires` (max 30 seconds ahead) +
HMAC-SHA1 `Signature`, all in the query string. Base URL is `https://<tenant-host>/ctixapi`.

## Steps

1. **Extract indicators from free text (optional).**
   `POST /conversion/quick-intel/free-text/` (`createParseIocsTask`) parses IOCs out of a block of
   text and returns a task. Use this rather than writing your own IOC regex.
2. **Create the intel.** `POST /conversion/quick-intel/create-stix/` (`quickAddIntel`) converts the
   supplied details into STIX and ingests them.
   `POST /conversion/quick-intel/open-api/` (`createIntelViaOpenApi`) is the Open-API-credential
   variant of the same flow.
3. **Confirm ingestion.** `GET /conversion/quick-intel/receive-report/`
   (`retrieveQuickAddIntelStatus`) returns the status of the submission. Do not treat a 2xx on step 2
   as proof the objects exist — ingestion is asynchronous.
4. **Inspect what was created.**
   `GET /ingestion/threat-data/report/{report_id}/relations/`
   (`retrieveQuickAddIntelRelationObjects`) lists the objects and relationships the report produced.
5. **Vocabularies.** When a field needs a country or region value, read the allowed set from
   `GET /conversion/quick-intel/vocabs/country_enum/` (`listCountryCodes`) and
   `GET /conversion/quick-intel/vocabs/region_ov/` (`listRegions`). Never invent an enum value.

## Rules

- **No idempotency key exists on this API.** A retried create can produce duplicate intel. If a
  create times out, do not re-POST — check step 3 first, then search
  (`POST /ingestion/threat-data/list/`) before re-submitting.
- For structured bundles use the import surface (`POST /conversion/import/...`) or the detailed STIX
  submission section instead; Quick Add Intel is for lightweight, analyst-shaped input.
- Confidence scores, TLP and tags applied at creation are the tenant's defaults unless you set them
  explicitly. Do not assume a default.
