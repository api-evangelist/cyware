---
name: Run a Cyware Orchestrate playbook and read the result
description: >-
  Find a playbook, run it, follow the run log to completion, and pull the per-node results — plus the
  direct app-action path when a single integration action is all that is needed.
api: openapi/cyware-orchestrate-openapi.yml
generated: '2026-08-04'
method: generated
operations:
  - testConnectivity
  - productReleaseVersion
  - getPlaybook
  - openPlaybook
  - runPlaybook
  - listPlaybookRunLogs
  - getPlaybookResult
  - getPlaybookDetailRunLog
  - getNodeResultDetails
  - bulkTerminateApiView
  - getApps
  - getAppActions
  - listAppInstances
  - executeAction
---

# Run a Cyware Orchestrate playbook and read the result

## Before you start

- Base URL is the tenant's own Orchestrate host with the API at `/soarapi`; paths are `/v1/...`.
- Auth is the same signed-query-string scheme as Intel Exchange but with lowercase parameter names:
  `access_id`, `expires` (epoch, current time + up to 30 seconds) and
  `signature` = Base64(HMAC-SHA1(secret_key, `"<access_id>\n<expires>"`)).
- Sanity-check the connection first: `GET /v1/test_connectivity/` (`testConnectivity`) and
  `GET /v1/release_version/` (`productReleaseVersion`).

## Steps

1. **Find the playbook.** `GET /v1/playbook/filter/` (`getPlaybook`) to search, or
   `GET /v1/playbook/` (`openPlaybook`) to open one. Resolve to a `playbook_unique_id` before running
   anything — never run by name.
2. **Run it.** `POST /v1/playbook/run/` (`runPlaybook`).
3. **Follow the run.** `GET /v1/playbook/playbook-result/` (`listPlaybookRunLogs`) or
   `GET /v1/playbook/playbook-result/filter/` (`getPlaybookResult`) to locate the
   `playbook_result_unique_id`, then
   `GET /v1/playbook/playbook-result/{playbook_result_unique_id}/` (`getPlaybookDetailRunLog`) for
   the detailed log. Poll on an interval; there is no callback or event stream for completion.
4. **Read a step.** `GET /v1/playbook/node-results/{node_unique_id}/` (`getNodeResultDetails`).
   Large payloads download separately via
   `GET /v1/playbook/node-results/export/{node_result_unique_id}/` (`nodeResultDownload`) and
   `GET /v1/playbook/playbook-result/export/{playbook_result_unique_id}/`
   (`playbookResultDownload`).
5. **Stop runaway runs.** `POST /v1/playbook/playbook-result/bulk-terminate/`
   (`bulkTerminateApiView`).

## Single action instead of a playbook

`GET /v1/integrations/apps/` (`getApps`) → `GET /v1/integrations/app-actions/` (`getAppActions`) →
`GET /v1/integrations/app-instances/` (`listAppInstances`) to choose the configured instance →
`POST /v1/integrations/actions/execute/` (`executeAction`).

## Rules

- **Running a playbook executes real actions against real security infrastructure** — blocking IPs,
  isolating hosts, opening tickets, sending mail. Require explicit human approval before
  `runPlaybook` or `executeAction`, and never select the playbook or the instance on the user's
  behalf from a fuzzy name match.
- **No idempotency key exists.** A retried `runPlaybook` starts a second execution. On a timeout,
  search the run log (step 3) before retrying.
- Events arriving through a webhook trigger are the other way playbooks start — see
  `asyncapi/cyware-orchestrate-webhooks.yml`.
- The open-source Cyware MCP server binds several of these flows as tools, but it calls internal
  routes (`playbooks/`, `execute/`, `/soarapi/integrations/sync-exec-action/`) rather than these
  documented `/v1/` paths. If you are writing an integration, use the documented paths above.
