---
name: Manage the allowed-indicator (false-positive) list in Cyware Intel Exchange
description: >-
  Add, verify and review indicators on the Intel Exchange allowed list so known-good infrastructure
  stops generating noise, using the documented reason vocabulary and supported indicator types.
api: openapi/cyware-intel-exchange-openapi.yml
generated: '2026-08-04'
method: generated
operations:
  - listSupportedAllowedIndicatorTypes
  - parseIoc
  - reasonsList
  - createReason
  - addIndicatorsToAllowedList
  - verifyAllowedIndicators
  - listAllowedIndicators
  - allowedIndicatorDetails
  - updateAllowedIndicator
  - deleteAllowedIndicator
  - bulkAction
---

# Manage the allowed-indicator list in Cyware Intel Exchange

The allowed list (historically "whitelist") suppresses known-good indicators. It is a high-blast-radius
control: anything on it stops being actioned, so treat every write as a security decision.

## Steps

1. **Check the indicator type is supported.**
   `GET /conversion/allowed_indicators/getoptions/` (`listSupportedAllowedIndicatorTypes`).
2. **Normalise the input.** `POST /conversion/allowed_indicators/parse-ioc/` (`parseIoc`) parses a raw
   string into typed indicators. Use it instead of splitting the string yourself.
3. **Pick a reason.** `GET /conversion/allowed_indicators/reasons/` (`reasonsList`). If no existing
   reason fits, `POST /conversion/allowed_indicators/reasons/` (`createReason`) — but prefer reusing
   an existing reason so reporting stays clean.
4. **Add.** `POST /conversion/allowed_indicators/` (`addIndicatorsToAllowedList`).
5. **Verify.** `POST /conversion/allowed_indicators/verify/` (`verifyAllowedIndicators`) before
   trusting that the entries took effect.
6. **Review.** `GET /conversion/allowed_indicators/` (`listAllowedIndicators`), then
   `GET /conversion/allowed_indicators/{indicator_id}/` (`allowedIndicatorDetails`) for one entry.
   Update with `PUT` (`updateAllowedIndicator`), remove with `DELETE` (`deleteAllowedIndicator`), or
   act in bulk with `POST /conversion/allowed_indicators/bulk-actions/` (`bulkAction`).

## Related surface

- Third-party feed indicators can be ignored separately:
  `POST /conversion/allowed_indicators/third_party_indicators/`
  (`addThirdPartyAllowedIndicatorToIgnoredList`),
  `GET` the same path (`listThirdPartyIgnoredIndicators`), and
  `DELETE /conversion/allowed_indicators/third_party_indicators/bulk-actions/`
  (`removeIndicatorFromThirdPartyIgnoredList`).
- Marking objects allowed straight from a threat-data list is a bulk action instead:
  `POST /ingestion/threat-data/bulk-action/{action_type}/` with `action_type=whitelist` /
  `un_whitelist` (`bulkAddRemoveAllowedIndicators`).

## Rules

- **Never add an indicator to the allowed list without an explicit human instruction.** This is the
  one flow in Intel Exchange where an agent mistake silently disables detection.
- Always attach a reason. A blank reason makes later review impossible.
- No idempotency key exists — re-POSTing the same indicators may create duplicates. Verify (step 5)
  rather than retrying.
- Rate limits are per Open API credential and are not signalled in headers; back off on `429`.
