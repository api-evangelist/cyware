---
name: Publish and share intel through Cyware Intel Exchange collections and TAXII
description: >-
  Set up a STIX collection, publish intel into it, and check that subscribers are polling it —
  the sharing side of Intel Exchange that ISACs, ISAOs and CERTs run on.
api: openapi/cyware-intel-exchange-openapi.yml
generated: '2026-08-04'
method: generated
operations:
  - listCollections
  - createCollection
  - updateCollection
  - listRelationshipSdo
  - createRelationshipSdo
  - listSubscriberLogs
  - taxiiConfigurationDetails
  - retrieveCertificate
---

# Publish and share intel through Intel Exchange collections and TAXII

Intel Exchange's sharing model is TAXII 2.x: intel is published into **collections**, and
**subscribers** poll those collections. Data marking (TLP) on the collection controls what leaves.

## Steps

1. **Review the TAXII setup.** `GET /conversion/taxii/configuration/`
   (`taxiiConfigurationDetails`) for the server configuration, and
   `GET /conversion/certificate/` (`retrieveCertificate`) for the certificates in the platform —
   check `status` and `expiry_time` before assuming a sharing channel is healthy.
2. **List or create the collection.** `GET /publishing/collection/` (`listCollections`);
   `POST /publishing/collection/` (`createCollection`) with `name`, `description`, `polling`,
   `inbox` and `marking_config`. `marking_config` is the data-marking (TLP) type — it is an
   enumerated field, so read the allowed values from the spec rather than guessing.
3. **Adjust it.** `PUT /publishing/collection/{collection_id}/` (`updateCollection`).
4. **Work the intel package.** The shareable-intel surface carries the STIX objects and their
   relationships: `GET /ingestion/shareable-intel/{intel-id}/relationship/` (`listRelationshipSdo`),
   `POST` the same path (`createRelationshipSdo`), and per-relationship
   `GET`/`PUT`/`DELETE .../relationship/{relationship-id}/`.
5. **Confirm delivery.** `GET /publishing/subscriber/polling_logs/{subscriber_id}/`
   (`listSubscriberLogs`) shows what each subscriber actually pulled, with `response_code`,
   `request_method` and `collection_id` filters plus `timestamp_from` / `timestamp_to` in **epoch
   seconds**. This is the only delivery evidence — there is no outbound delivery receipt.

## Rules

- **Sharing is irreversible.** Once a subscriber polls a collection you cannot unsend it. Confirm the
  data marking and the collection's audience with a human before publishing.
- TAXII traffic has its own rate-limit family, separate from the Open API one — read
  `GET /rest-auth/rate-limit/taxii/` (`taxiiRateLimitConfigurationDetails`) before bulk publishing.
- All timestamps are epoch seconds. All list endpoints paginate with `page` / `page_size` (max 100)
  and return `{next, previous, page_size, total, results}`.
- No idempotency key exists; a retried collection create makes a second collection.
