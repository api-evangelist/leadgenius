---
name: Enrich a company in real time with LeadGenius
description: >-
  Submit a single company (domain, name or LinkedIn URL) to LeadGenius rapid enrichment and collect
  the returned firmographics. Use when a CRM account record is missing revenue, employee count,
  industry or address.
api: openapi/leadgenius-enrichment-api-openapi.yml
operations:
  - createRapidEnrichmentRequest
  - listEnrichmentRequests
  - retrieveEnrichedRecord
---

# Enrich a company in real time

Base URL `https://leadgenius.com`. Every request carries `Authorization: Token {apikey}` and
`Content-Type: application/json`. See `authentication/leadgenius-authentication.yml`.

## Steps

1. **Submit the account** — `createRapidEnrichmentRequest`
   `POST /api/v1/enrichment/rapid/` with `type: "account"` and a `data` object. At least one of
   `org_website`, `org_linkedin_url` or `org_company_name` is required; supply all three when you
   have them, because match rates rise with more identifiers.

   ```json
   {"type": "account", "data": {"org_company_name": "microsoft", "org_website": "microsoft.com", "org_linkedin_url": "linkedin.com/in/microsoft"}}
   ```

   Keep the `id` from the response — it is the enrichment request id.

2. **Wait for the request to complete** — `listEnrichmentRequests`
   `GET /api/v1/enrichment/rapid/?id={id}` returns the request with a `status` of `scheduled`,
   `in_progress`, `erred`, `cancelled` or `ok`. Poll until `ok`. Enrichment is asynchronous — do not
   assume the record is ready immediately after submission.

3. **Collect the enriched record** — `retrieveEnrichedRecord`
   `GET /api/v1/enrichment/rapid/{id}/` returns the `org_*` output keys: `org_annual_revenue`,
   `org_city`, `org_company_name`, `org_country`, `org_linkedin_url`, `org_num_employees`,
   `org_state`, `org_street`, `org_website`, `org_zip_code`.

## Rules

- **Rate limit**: 50 requests per minute. Exceeding it returns `429`. Batch your polling rather than
  tight-looping — see `rate-limits/leadgenius-rate-limits.yml`.
- **No idempotency key.** LeadGenius documents no idempotency header, so a retried submission creates
  a second enrichment request. Before retrying a submission that may have succeeded, call
  `listEnrichmentRequests` with `created_from` set to the submission time and check for a duplicate.
  Duplicate records are de-duplicated on the billing side (`deduplicated` in the usage stats) but
  duplicate *requests* are still requests.
- **Errors** are a field-keyed validation envelope, not problem+json — e.g.
  `{"type": ["\"foo\" is not a valid choice."]}`. `401` means the API key is invalid. See
  `errors/leadgenius-problem-types.yml`.
- **Status `erred` is terminal.** Do not re-poll an `erred` or `cancelled` request; resubmit with
  better identifiers instead.
