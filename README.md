# LeadGenius

LeadGenius provides precision B2B contact and account intelligence for go-to-market teams, combining machine learning with a global team of human researchers so every contact and account is human-verified before delivery.

Its developer surface is the **LeadGenius Enrichment API** — a RESTful, API-key authenticated service documented at [docs.leadgenius.com](https://docs.leadgenius.com/) covering company enrichment, contact enrichment and contact append, in two modes:

- **Campaigns (batch)** — create a campaign, upload up to 200 records per request, receive the `record.finalized` webhook, retrieve the enriched results.
- **Rapid enrichment (real time)** — submit a single account, contact or `get_contacts` request and collect the enriched record by id.

## Artifacts

| Artifact | File |
|---|---|
| OpenAPI 3.1 (captured from the docs) | `openapi/leadgenius-enrichment-api-openapi.yml` |
| Overlay | `overlays/leadgenius-enrichment-api-overlay.yaml` |
| Authentication | `authentication/leadgenius-authentication.yml` |
| Conventions | `conventions/leadgenius-conventions.yml` |
| Error catalog | `errors/leadgenius-problem-types.yml` |
| Rate limits | `rate-limits/leadgenius-rate-limits.yml` |
| Lifecycle | `lifecycle/leadgenius-lifecycle.yml` |
| Conformance | `conformance/leadgenius-conformance.yml` |
| Data model | `data-model/leadgenius-data-model.yml` |
| Webhook catalog | `asyncapi/leadgenius-enrichment-webhooks.yml` |
| MCP candidate tools | `mcp/leadgenius-mcp.yml` |
| Agent skills | `skills/_index.yml` |
| Agentic access | `agentic-access/leadgenius-agentic-access.yml` |
| Domain security | `security/leadgenius-domain-security.yml` |
| Well-known probe | `well-known/leadgenius-well-known.yml` |
| llms.txt | `llms/leadgenius-llms.txt` |

LeadGenius does not publish a machine-readable OpenAPI definition; the spec here was transcribed from the published reference by the API Evangelist enrichment pipeline.

Backed by: 500-global, initialized-capital — https://leadgenius.com
