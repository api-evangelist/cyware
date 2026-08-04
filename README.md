# Cyware

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

Cyware is a New York-headquartered cybersecurity company, founded in 2016, that builds an AI-powered
threat intelligence and security operations platform for enterprise SOC teams, ISACs and ISAOs,
government agencies, CERTs and MSSPs. The Cyware Intelligence Suite spans Intel Exchange (its threat
intelligence platform, formerly CTIX), Collaborate, Orchestrate, Respond and Cyware AI.

## What this profile covers

| Surface | Where |
|---|---|
| Intel Exchange (CTIX) v3 Open API — 391 operations | `openapi/cyware-intel-exchange-openapi.yml` |
| Orchestrate (CO) Open API — 47 operations | `openapi/cyware-orchestrate-openapi.yml` |
| Cyware MCP Server — 40 tools, open source (MIT, Go) | `mcp/cyware-mcp.yml` |
| MCP tool -> REST operation crosswalk | `mcp/cyware-tool-crosswalk.yml` |
| Authentication (HMAC-SHA1 signed query string) | `authentication/cyware-authentication.yml` |
| Cross-cutting conventions | `conventions/cyware-conventions.yml` |
| Agent skills for five marquee flows | `skills/` |

Cyware publishes **no OpenAPI document**. The two specifications here were assembled from Cyware's
own published, structured API reference documents at `ctixapiv3.cyware.com` and
`orchestrateapi.cyware.com` — each endpoint page is served as machine-readable markdown carrying a
JSON endpoint model, and every page is indexed from that host's `llms.txt`. Paths, methods,
parameters, descriptions, enumerations and examples are reproduced from those documents.

- https://www.cyware.com/
- https://techdocs.cyware.com/
- https://forgeglobal.com/cyware_stock/ (secondary-market listing that surfaced this company)
