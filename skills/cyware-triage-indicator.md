---
name: Triage an indicator in Cyware Intel Exchange
description: >-
  Search threat data with CQL, open the matching threat data object, read its relations, enrich it
  with a configured enrichment tool, and record the verdict as an analyst TLP/score plus a tag.
api: openapi/cyware-intel-exchange-openapi.yml
generated: '2026-08-04'
method: generated
operations:
  - listThreatData
  - listThreatDataObjectDetails
  - listRelationsOfThreatDataObject
  - retrieveConfiguredTools
  - enrichThreatData
  - retrieveEnrichmentStatus
  - bulkAddTlp
  - bulkAddAnalystScore
  - bulkAddRemoveAllowedIndicators
---

# Triage an indicator in Cyware Intel Exchange

## Before you start

- The base URL is the tenant's own Intel Exchange host with the API mounted at `/ctixapi` — there is
  no shared SaaS endpoint. Get it from the tenant admin.
- Every request carries three query parameters: `AccessID`, `Expires` (epoch seconds, **at most 30
  seconds in the future**) and `Signature` = URL-encode(Base64(HMAC-SHA1(secret_key,
  `"<AccessID>\n<Expires>"`))). Re-sign every request; a signature is not reusable and clock skew
  over 30 seconds fails with 401.
- Timestamps in and out are **epoch seconds**, never ISO 8601.
- Paths end in a trailing slash. List responses are `{next, previous, page_size, total, results}`
  with `page` and `page_size` (max 100) query parameters.

## Steps

1. **Find the object.** `POST /ingestion/threat-data/list/` (`listThreatData`) with a Cyware Query
   Language (CQL) query in the body. If you do not know CQL, the open-source Cyware MCP server ships
   the grammar as a tool (`cql-ctix-grammar-rules`); do not guess operators.
   `POST /ingestion/threat-data/ai-search/` (`aiAssistedSearch`) accepts a natural-language search
   instead.
2. **Open the object.** `GET /ingestion/threat-data/{object_type}/{object_id}/basic/`
   (`listThreatDataObjectDetails`). Threat data objects are addressed by the **pair**
   `(object_type, object_id)` — `object_type` is the STIX type (`indicator`, `malware`,
   `threat-actor`, …), not part of the id.
3. **Read the graph.** `GET /ingestion/threat-data/{object_type}/{object_id}/relations/`
   (`listRelationsOfThreatDataObject`) for what this object is connected to. Never assert a
   relationship the API did not return.
4. **Pick an enrichment tool.** `GET /integration/apps/` (`retrieveConfiguredTools`) lists the tools
   the tenant has actually configured. Only tools in this list can be used.
5. **Enrich.** `GET /integration/apps/update/threatdata/` (`enrichThreatData`), then poll
   `GET /ingestion/enrichment/status/{object_id}/` (`retrieveEnrichmentStatus`) until enrichment
   completes. Enrichment tools have per-tenant quotas — stop polling on a non-2xx rather than
   retrying in a loop.
6. **Record the verdict.** Bulk endpoints take an explicit `ids` list and are the right unit of work
   even for one object:
   - analyst TLP: `POST /ingestion/threat-data/bulk-action/analyst_tlp/` (`bulkAddTlp`)
   - analyst score: `POST /ingestion/threat-data/bulk-action/analyst_score/` (`bulkAddAnalystScore`)
   - tag / watchlist / false-positive / deprecate:
     `POST /ingestion/threat-data/bulk-action/{action_type}/` (`bulkAddRemoveAllowedIndicators`)
     with `action_type` one of `add_tag`, `watchlist`, `un_watchlist`, `false_positive`,
     `un_false_positive`, `manual_review`, `deprecate`, `un_deprecate`, `whitelist`, `un_whitelist`.

## Rules

- **There is no idempotency key on this API.** Bulk actions are safe to re-apply because they set
  state on a named id list; creation calls are not. Never blind-retry a create.
- Bulk calls also accept `all_objects: true` with a filter. Treat that as a destructive, unbounded
  operation: require explicit human confirmation and never derive the filter yourself.
- Rate limits are tenant-configured per credential (documented example: 100/minute, 5000/hour,
  30000/day) and are **not** signalled in response headers. Back off on `429` and read
  `GET /rest-auth/rate-limit/openapi/` to see the configured ceiling.
- Errors are undocumented vendor JSON — there is no `application/problem+json`. Expect bare `400`,
  `401`, `403`, `404`, `429`, `500`.
