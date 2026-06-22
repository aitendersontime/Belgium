<div align="center">

# Belgium Public Procurement Dataset

**A structured, versioned collection of Belgian tender notices for research, analysis, and opportunity discovery.**

21,155 records · 14,276 unique tender IDs · 14 sectors · JSONL

[Explore the data](#repository-layout) · [Understand the schema](#record-structure) · [Start analysing](#quick-start)

</div>

## Why this repository exists

Public procurement data is valuable, but it is often scattered across portals, formats, and languages. This repository makes a substantial collection of Belgian tender notices easier to inspect and process by presenting each notice as a consistent JSON record.

The collection is useful for:

- suppliers researching public-sector demand in Belgium;
- analysts studying purchasers, sectors, CPV codes, and procurement activity;
- data teams building search, classification, or alerting prototypes;
- researchers experimenting with information extraction from tender text.

This is a historical dataset, not a live tender feed. Always verify deadlines and official requirements with the original procurement authority before acting on a notice.

## Dataset at a glance

| Attribute | Value |
|---|---:|
| Country | Belgium |
| Format | JSON Lines (`.jsonl`) |
| Dataset files | 27 |
| Total records | 21,155 |
| Unique `TOT_ID` values | 14,276 |
| Sector labels | 14 |
| Published-date coverage | 14 June 2020 to 27 December 2025 |
| Latest file snapshot | 27 December 2025 |
| JSON validation | All rows parsed successfully during the June 2026 review |

The difference between total records and unique IDs is expected: the repository contains multiple snapshots, so some tenders appear more than once.

## Repository layout

All datasets are stored on the default branch. Files follow this convention:

```text
dataset_BE_<sector-index>_<YYYYMMDD_HHMMSS>.jsonl
```

For example:

```text
dataset_BE_1_20251227_110333.jsonl
```

The timestamp identifies when that snapshot was generated. Use the `Sector` field inside each record as the authoritative category label; the numeric sector index is an internal file grouping.

The collection covers areas including agriculture and food, IT, construction, defence and security, education, environment, finance, materials, mining, power and energy, printing and publishing, research, transport, and other services.

## Record structure

Each line is an independent JSON object with three top-level properties:

```json
{
  "instruction": "Extraction and classification task",
  "input": "Original tender text",
  "output": {
    "TOT_ID": "129564426",
    "Tender_Notice_No": "PPP0D1-6381/5675/2025-6381_001",
    "Country": "Belgium",
    "Purchaser_Name": "HASSELT",
    "Summary": "Framework agreement for construction materials",
    "Description": "Tender description",
    "Published_Date": "2025-11-06",
    "Closing_Date": "2025-11-28",
    "Competition": "ICB",
    "Financier_Name": "Self Financed",
    "CPV": "14210000",
    "Sector": "Mining and Ores",
    "Sub_Sector": "Mining and Basic Metal",
    "More_Details": "https://tendersontime.com/register/"
  }
}
```

Common optional fields include `Tender_Value`, `Currency`, and `USD_Tender_Value`. Their absence means the value was not available in the normalized record; it should not be interpreted as zero.

## Quick start

The following Python example loads every snapshot and keeps the latest occurrence of each `TOT_ID`:

```python
from pathlib import Path
import json


def snapshot_time(path: Path) -> tuple[str, str]:
    return tuple(path.stem.rsplit("_", 2)[1:])


latest_by_id = {}

for path in sorted(Path(".").glob("dataset_BE_*.jsonl"), key=snapshot_time):
    with path.open(encoding="utf-8") as source:
        for line in source:
            row = json.loads(line)
            tender = row["output"]
            tender["_source_file"] = path.name
            latest_by_id[tender["TOT_ID"]] = tender

print(f"{len(latest_by_id):,} unique tenders loaded")
```

To inspect normalized records with `jq`:

```bash
jq -c '.output' dataset_BE_*.jsonl | head
```

## Data quality notes

- All 21,155 non-empty rows are valid JSON.
- Snapshot overlap creates 6,879 repeated `TOT_ID` occurrences.
- `TOT_ID` is the recommended deduplication key.
- Tender values and currencies are optional and have lower coverage than core descriptive fields.
- Descriptions may contain HTML entities inherited from source notices.
- Dates are stored as `YYYY-MM-DD` strings.
- A `More_Details` link may lead to a registration or access page rather than directly to the contracting authority.

For production use, normalize HTML entities, parse dates explicitly, retain the source filename, and verify important records against an official source.

## About TendersOnTime

[TendersOnTime](https://www.tendersontime.com) helps organizations discover public procurement opportunities across markets and sectors. This repository offers a structured sample of that work for Belgium.

- [Browse current Belgium tenders](https://www.tendersontime.com/belgium-tenders/)
- [Explore tenders by country](https://www.tendersontime.com/tendersby/country/)
- [Subscription options](https://www.tendersontime.com/subscribe/)
- [Contact the team](https://www.tendersontime.com/contact/)

## Usage and responsibility

Before redistributing or using this dataset commercially, review the [TendersOnTime Terms & Conditions](https://www.tendersontime.com/terms/) and [Privacy Policy](https://www.tendersontime.com/privacy/). This repository does not grant rights beyond those terms.

Tender information changes. Confirm eligibility, deadlines, values, documents, and submission instructions with the relevant official authority.

## Contributing

If you find malformed JSON, an incorrect classification, a duplicate that cannot be explained by snapshot overlap, or a documentation issue, open a GitHub issue with the filename and `TOT_ID`. Avoid posting confidential or account-specific information.

---

Maintained by [TendersOnTime](https://www.tendersontime.com) · Documentation refreshed 22 June 2026
