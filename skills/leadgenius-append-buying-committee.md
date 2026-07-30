---
name: Append a buying committee to an account with LeadGenius
description: >-
  Given a target company, ask LeadGenius for matched contacts filtered by seniority and department,
  then retrieve their full profiles. Use for ABM buying-committee coverage and SDR/BDR list building.
api: openapi/leadgenius-enrichment-api-openapi.yml
operations:
  - createRapidEnrichmentRequest
  - listEnrichmentRequests
  - retrieveEnrichedRecord
  - uploadExclusionRecords
---

# Append a buying committee to an account

Base URL `https://leadgenius.com`, header `Authorization: Token {apikey}`.

## Steps

1. **Request contacts for the company** — `createRapidEnrichmentRequest`
   `POST /api/v1/enrichment/rapid/` with `type: "get_contacts"`. The `data` object identifies the
   company (`org_company_name`, `org_website`, `org_linkedin_url`) and the `contacts` object defines
   the committee you want.

   ```json
   {"type": "get_contacts",
    "data": {"org_company_name": "microsoft", "org_website": "microsoft.com", "org_linkedin_url": "linkedin.com/in/microsoft"},
    "contacts": {"contacts_per_company": 1, "seniority": "C-level", "title": "Consulting"}}
   ```

   `contacts_per_company` accepts 1–10. `seniority` is one of `VP+`, `Director+`, `Manager+`,
   `C-level`, `VP`, `Director`, `Manager`, `Senior`, `Entry level` — the `+` forms are inclusive
   (`Director+` covers C-level, VP and Director). `department` is one of the 16 published values
   (Business Development, Consulting, Design, Education, Engineering, Executive, Finance,
   Health Services, Human Resources, Information Technology, Legal, Marketing,
   Media and Communications, Operations, Product, Sales).

2. **Poll for completion** — `listEnrichmentRequests`
   `GET /api/v1/enrichment/rapid/?id={id}&status=ok`, or filter the whole queue with
   `enrichment_type=get_contacts` and `created_from`/`created_to`.

3. **Retrieve the profiles** — `retrieveEnrichedRecord`
   `GET /api/v1/enrichment/rapid/{id}/` returns the `contact_*` keys: `contact_first_name`,
   `contact_last_name`, `contact_email`, `contact_job_title`, `contact_linkedin_url`,
   `contact_department`, `contact_seniority`, `contact_phone_number`, `contact_city`,
   `contact_country`, plus `contact_previous_company` and the LeadGenius `_lg` signal fields.

4. **Suppress anyone you must not contact** — `uploadExclusionRecords`
   `POST /api/v1/exclusion/upload/` with `records[]`. Contact entries use `entity: "contact"` and one
   of `contact_email` or `contact_linkedin_url`; account entries use `entity: "org"` and one of
   `org_website` or `org_linkedin_url`. Run this **before** the next append so opted-out people and
   existing customers are never returned again.

## Rules

- **Choose seniority deliberately.** `contacts_per_company` caps the result set, so a broad
  `Manager+` with `contacts_per_company: 1` will not reliably return the C-level contact. Request
  narrow seniority bands separately when committee coverage matters.
- **Privacy first.** LeadGenius markets a privacy-compliance posture (GDPR, CCPA — see
  `conformance/leadgenius-conformance.yml`). Feed suppression/opt-out lists into
  `uploadExclusionRecords` rather than filtering downstream.
- **Rate limit** 50 requests/minute, `429` on breach.
- **Errors**: `{"type": ["This field is required."]}` means you omitted `type`; a `fields` error means
  the field count does not match the campaign type. See `errors/leadgenius-problem-types.yml`.
