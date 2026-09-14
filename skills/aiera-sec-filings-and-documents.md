---
name: aiera-sec-filings-and-documents
description: Retrieve SEC filings by EDGAR form number and company-published documents from Aiera, including the text and PDF renditions and the category/keyword vocabulary.
api: Aiera REST API
generated: '2026-09-14'
method: generated
source: openapi/aiera-rest-api-openapi.json, https://rest.aiera.com/docs/find-filings, https://rest.aiera.com/docs/find-company-docs
operations:
  - get_get_filings
  - get_get_filing
  - get_get_filing_text
  - get_get_filing_json
  - get_get_filing_datafile
  - get_get_docs
  - get_get_doc
  - get_get_doc_text
  - get_doc_categories
  - get_doc_keywords
---

# Pull SEC filings and company documents

Base URL `https://premium.aiera.com/api`, header `X-API-Key: <your key>`.

Aiera keeps these in two separate surfaces, and which one you want depends on who published the
document.

## SEC filings — `filings-v1`

`GET /filings-v1/` (`get_get_filings`) takes `start_date`, `end_date`, a symbology identifier, and
`form_number` for the EDGAR form type — `10-K`, `10-Q`, `8-K` and so on. Aiera addresses filings by
their real SEC form number, not by an internal document class, so a form list you already have works
unchanged.

From a `filing_id`:

- `GET /filings-v1/{filing_id}` (`get_get_filing`) — metadata, with `equity` and `datafiles`.
- `GET /filings-v1/{filing_id}/text` (`get_get_filing_text`) — plain text, the usual choice for
  model input.
- `GET /filings-v1/{filing_id}/json` (`get_get_filing_json`) — structured form.
- `GET /filings-v1/{filing_id}/pdf` (`get_get_filing_pdf`) — the rendered document.
- `GET /filings-v1/{filing_id}/datafile/{datafile_name}` (`get_get_filing_datafile`) — the individual
  data files listed in `datafiles`.

## Company-published documents — `company-docs-v1`

These are what the company itself puts out: press releases, annual reports, earnings releases, slide
presentations.

`GET /company-docs-v1/` (`get_get_docs`) takes date range, symbology, and `categories` / `keywords`
filters, with `exclude_categories` and `exclude_keywords` to cut noise.

**Read the vocabulary first.** `GET /company-docs-v1/categories` (`get_doc_categories`) and
`GET /company-docs-v1/keywords` (`get_doc_keywords`) return the valid values. Passing a guessed
category is how you get an empty result set and conclude, wrongly, that Aiera has no coverage.

`GET /company-docs-v1/coverage` (`get_doc_coverage`) tells you what is actually covered before you
build a query around it.

From a `doc_id`: `/company-docs-v1/{doc_id}` for metadata, `/text` for the text,
`/pdf` for the rendered file.

## Things that will bite you

- **Empty is not the same as unavailable.** Aiera is entitlement-gated. An empty list can mean no
  documents exist, or that your account is not entitled to that content set.
- **Pagination is page-number, capped at 25.** `page` and `page_size`; read the `pagination` block.
- **No 429 is declared.** No published rate limit, no `Retry-After`. Back off on your own schedule.
- **Semantic search is not here.** `search_filings` and `search_company_docs` exist only as MCP tools
  (see `mcp/aiera-tool-crosswalk.yml`); this REST surface does filtered retrieval, not embedding search.
