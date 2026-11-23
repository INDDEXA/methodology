# Sources and provenance

## Authoritative source

- **Database:** Cloudflare D1, `inddexa-db` (production).
- **Table:** `methodology_versions`.
- **Columns read:** `version`, `published_date`, `last_updated`, `version_summary`, `what_changed`, `previous_version`, `is_current`, `status`, `repo_url`, `body_md`, `created_at`.
- **Read-only queries used:**

  ```sql
  SELECT version, body_md FROM methodology_versions ORDER BY version;
  ```

  ```sql
  SELECT version, published_date, last_updated, version_summary, what_changed,
         previous_version, is_current, status, repo_url, created_at,
         length(body_md) AS body_len
  FROM methodology_versions ORDER BY version;
  ```

- **Rows found:** exactly two — `V1.0` and `V1.1`, both with `status = Published`. `V1.1` is marked `is_current = 1` and `previous_version = V1.0`; `V1.0` has `is_current = 0` and `previous_version = null`. No other methodology rows exist in this table.
- **Extraction method:** read-only HTTPS POST requests to the Cloudflare D1 query endpoint, using the project's existing credential loader (`scripts/cf-cred.mjs`) and the request pattern in `scripts/verify-v1.mjs`. No writes were made to the database and no credential values were printed or stored.

## Live cross-check

Each version was checked against the published site; all returned **HTTP 200**:

- https://inddexa.com/methodology/V1.0/ — page title "V1.0 Research Methodology"
- https://inddexa.com/methodology/V1.1/ — page title "V1.1 Research Methodology"
- https://inddexa.com/methodology/ — canonical page; serves the V1.1 content

Distinctive phrases from each D1 `body_md` were found verbatim in the corresponding live page HTML — for example "Later versions will be published at their own versioned URL" on V1.0, and "A revision differs from an upgrade", "no new canonical URL" and "Prepared by INDDEXA Research Department" on V1.1. This confirms the D1 rows are the published source rather than unpublished drafts.

## Extraction timestamp

- **UTC:** 2026-11-23T07:00:49Z
- **Local date:** 2026-11-23 (UTC+8)

## File mapping

| File in this repository | Source |
| --- | --- |
| `V1.0.md` | `methodology_versions.body_md` where `version = 'V1.0'`, copied byte-for-byte |
| `V1.1.md` | `methodology_versions.body_md` where `version = 'V1.1'`, copied byte-for-byte |
| `CHANGELOG.md` | Comparison of the two published `body_md` values plus the D1 metadata fields quoted above |
| `README.md` | Landing page; version status taken from D1 `is_current`, `published_date` and `version_summary` |
| `SOURCES.md` | This provenance record |

## Director edit after extraction (2026-11-23, 16:12 UTC+8)

The owner instructed that the published documents must carry **full descriptive titles, not bare version numbers**. The command director therefore edited the H1 title line of each file after extraction:

- `V1.0.md`: added a new H1 `# INDDEXA Research Methodology — Version 1.0` above the original content (the D1 V1.0 body had no H1 and began with `## Research method`).
- `V1.1.md`: replaced the original H1 `# V1.1 Research Methodology (Revised)` with `# INDDEXA Research Methodology — Version 1.1 (Revised)`.

Only the H1 title lines were changed; the rest of each body remains byte-for-byte identical to D1 `body_md`. Consequently these two files are **no longer byte-for-byte copies** of `body_md` — they are `body_md` plus the director-approved descriptive title. Recomputed hashes after this edit:

| File | SHA256 after title edit | Lines |
| --- | --- | --- |
| `V1.0.md` | `cda6c05a48fe274d59eef62ef625d9d67b59895d3439b9756b10f1075ff072d1` | 152 |
| `V1.1.md` | `15eb2c4c48ce839005508f322cffe816de8512632b4d37b16b8ac9c3b03e2a7c` | 173 |

The publication runbook (`04-reports/PUB-GITHUB-20261123-018.md` §2.1) was updated to these hashes so the G2 gate checks against the actual publishable bytes.

## Discrepancy note

Local working copies under `06-research/03-public-methodology/` were **not** used as the source of truth. They differ from D1 as follows:

- `网站对外方法论_V1.0_存档.md` wraps the V1.0 text in a Chinese provenance header and title, uses `###` for sections instead of `##`, renders the two policy references as plain text rather than Markdown links, uses straight quotes around "information increment", and appends a Version History block that is not present in the D1 V1.0 body.
- `网站对外方法论_V1.0修正版_英文终稿.md` is a pre-publication English draft titled "V1.0 Research Methodology (Revised)" with a Chinese provenance header. The D1 V1.1 body is titled "V1.1 Research Methodology (Revised)", drops that wrapper and carries the published version summary and version history. The English method text itself matches; the D1 version is the published label.

Per the task rule, D1/live site wins. `V1.0.md` and `V1.1.md` therefore reproduce the D1 `body_md` verbatim, including original punctuation, heading levels and line endings.
