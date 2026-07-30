---
name: Run a batch enrichment campaign with LeadGenius
description: >-
  Create a LeadGenius campaign, upload a batch of CRM records, receive the record-finalized webhook
  and download the enriched results. Use for CRM enrichment, data audits and large ABM refreshes.
api: openapi/leadgenius-enrichment-api-openapi.yml
operations:
  - createCampaign
  - getUploadRequestSample
  - uploadRecordsToCampaign
  - updateCampaign
  - retrieveCampaignRecords
  - getApiUsageStats
---

# Run a batch enrichment campaign

Base URL `https://leadgenius.com`, header `Authorization: Token {apikey}`.

## Steps

1. **Check your budget first** — `getApiUsageStats`
   `GET /api/v1/enrichment/status/` returns `remaining` (records available for upload this
   subscription period) plus all-time and current-period `uploaded` / `enriched` / `deduplicated`
   counts and the period `start_date` / `end_date`. These figures can be up to 10 minutes stale.
   Do not start a batch larger than `remaining`.

2. **Create the campaign** — `createCampaign`
   `POST /api/v1/enrichment/campaigns/` with `name` and `columns` (both required). `columns` lists
   the fields to return. Set `precision: false` to skip human review, or `true` to have researchers
   verify every record. Set `webhook_url` now so completion is pushed rather than polled. Add
   `contacts[]` and `contacts_per_company` to append contacts as well as enrich accounts.

   ```json
   {"name": "net new org contact precision", "precision": true,
    "webhook_url": "https://example.com/webhook/",
    "columns": ["org_website", "org_company_name", "contact_email", "contact_linkedin_url"],
    "contacts_per_company": 2,
    "contacts": [{"priority": 1, "seniority": "C-level", "department": "Executive"}]}
   ```

   Keep the campaign `slug`.

3. **Confirm the payload shape** — `getUploadRequestSample`
   `GET /api/v1/enrichment/upload/{slug}/` returns a sample upload body with the exact field names
   the campaign expects. Campaigns created in Dashboard can carry non-standard fields whose object
   names only appear here — always read the sample before your first upload.

4. **Upload the records** — `uploadRecordsToCampaign`
   `POST /api/v1/enrichment/upload/{slug}/` with `slug`, `fields[]` and `records[]`. **Maximum 200
   records per request** — chunk larger batches. Carry your CRM ids through on `org_record_id` and
   `contact_record_id`; they do not need to appear in `columns` and they come back on the enriched
   record so you can write results home without a fuzzy match.

5. **Receive completion** — the `record.finalized` webhook
   LeadGenius POSTs `{slug, records[], uploaded, enriched, complete}` to your `webhook_url` each time
   records are finalized. `records[]` holds the finalized record ids; `complete: true` means every
   uploaded record was verified. No signature header is documented — treat the callback URL as a
   secret and always re-fetch from the API rather than trusting the payload body. To change the URL
   later use `updateCampaign` (`PATCH /api/v1/enrichment/update/{slug}/`).

6. **Download the results** — `retrieveCampaignRecords`
   `GET /api/v1/enrichment/retrieve/{slug}/` returns the job status (`Completed` or `In-Progress`)
   and, when complete, the records. Paginated at 100 records. Fetch just the ids from the webhook
   with `?id=1&id=2`, or sweep a window with `finalized_from` / `finalized_to`
   (`2022-01-02` or `2022-01-03T18:03:11Z`).

## Rules

- **Read the record status, not just the payload.** Each record carries one of `original`,
  `finalized_complete`, `finalized_incomplete`, `finalized_unenriched`, `finalized_cancelled`.
  `finalized_incomplete` means some fields could not be filled; `finalized_unenriched` means the
  company or contact could not be verified at all (out of business, contact no longer valid). Never
  write an unenriched record over good CRM data.
- **Rate limit** 50 requests/minute, 200 records/request, `429` on breach.
- **No idempotency key** — a retried upload is a second upload. Reconcile against
  `retrieveCampaignRecords` before re-sending a chunk. See `conventions/leadgenius-conventions.yml`.
- **Errors** use a field-keyed envelope; see `errors/leadgenius-problem-types.yml`.
