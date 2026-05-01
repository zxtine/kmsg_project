# KMSG Capstone — Nonprofit Funder Discovery Pipeline

Build a **data pipeline for nonprofit funder discovery** using U.S. IRS 990 filings, ProPublica data, and external reference datasets. The pipeline produces a **clean, grant-level dataset** mapping funding relationships (who funds whom, how much, and for what purpose) for analysis, research, and future tool development.

---

## Project structure

```
KMSG_Project/
├── pipeline/
│   └── final_kmsg_data.py            # Full end-to-end pipeline
├── outputs/
│   ├── final_grants_df.csv           # Grant-level dataset (intermediate)
│   ├── funder_org_df.csv             # Enriched funder dataset
│   ├── funder_filings_df.csv         # Funder financial data by year
│   ├── qualified_orgs.csv            # Qualified nonprofit list
│   ├── qualified_with_object_df.csv  # IRS filing mapping
│   └── irs_grants_checkpoint.csv     # Grant extraction checkpoint
├── data/
│   └── pa_nonprofit_data_cleaned.csv # Reference dataset (PA enrichment)
└── README.md
```

---

## Pipeline overview

This pipeline transforms raw public nonprofit data into a structured dataset through a staged process:

* Identify relevant nonprofit organizations
* Filter to high-capacity funders
* Enrich with external metadata
* Extract IRS filing data
* Parse grant relationships (Schedule I)
* Merge, clean, and validate outputs

The result is a dataset that supports:

* Funder discovery
* Funding flow analysis
* Downstream modeling and tools

---

## Script overview

### `final_kmsg_data.py` — End-to-end pipeline

**Purpose:** Build a complete dataset of nonprofit funding relationships by integrating IRS, API, and reference data sources into a single, analysis-ready output.

---

## Pipeline sections

### 1. Setup and configuration

**Purpose:** Define global parameters controlling the pipeline.

* Target state (e.g., Pennsylvania)
* Years of analysis
* Revenue threshold
* API endpoints and file paths

**Why it matters:**
All downstream processing depends on these settings.

---

### 2. Helper functions

**Purpose:** Standardize data and handle repeated operations.

* EIN cleaning and formatting
* Name normalization
* ZIP code cleaning
* API retry handling

**Why it matters:**
Ensures consistency and improves matching accuracy across datasets.

---

### 3. EO BMF data loading and filtering

**Purpose:** Load the base population of nonprofit organizations from IRS data.

* Filters by state and organization type (e.g., 501(c)(3))
* Prepares initial dataset for processing

**Output:**
Initial pool of candidate organizations

---

### 4. Revenue validation and qualification

**Purpose:** Identify high-capacity organizations likely to act as funders.

* Confirms revenue using BMF data
* Recovers missing values via IRS XML filings
* Filters to organizations above threshold

**Output:**
List of qualified funders with verified revenue

---

### 5. Final qualified organization list

**Purpose:** Consolidate confirmed and recovered organizations into a single dataset.

**Why it matters:**
Defines the set of funders used throughout the pipeline.

---

### 6. ProPublica enrichment

**Purpose:** Add structured organization and financial data.

* Organization-level metadata
* Filing-level financial metrics

**Outputs:**

* `funder_org_df`
* `funder_filings_df`

---

### 7. IRS XML data extraction

**Purpose:** Extract detailed information from IRS filings.

* Mission statements
* Website URLs
* Address data

**Why it matters:**
Provides key fields not available in structured APIs.

---

### 8. Data integration and enrichment

**Purpose:** Merge all funder-level data sources.

* Combine ProPublica and IRS XML data
* Fill missing fields using BMF data
* Standardize formatting

**Output:**

* `funder_org_enriched_df`

---

### 9. Grant extraction (Schedule I)

**Purpose:** Extract grant-level relationships from IRS filings.

* Parses Schedule I XML data
* Creates one row per grant

**Output:**

* `irs_grants_df`

---

### 10. Attach funder metadata to grants

**Purpose:** Add organization-level context to each grant.

* Merge funder attributes with grant records

**Output:**

* `final_grants_df`

---

### 11. Reference data enrichment

**Purpose:** Improve completeness using external dataset.

* Matches organizations by EIN (primary)
* Falls back to name matching when needed
* Adds additional attributes (cause area, financials)

**Why it matters:**
Improves accuracy and reduces missing values.

---

### 12. Final cleaning and recovery

**Purpose:** Standardize and finalize dataset.

* Controlled filling of missing values
* Group-based recovery across repeated organizations

**Output:**

* Cleaned dataset versions

---

### 13. Targeted XML backfill

**Purpose:** Recover missing mission and website data.

* Reprocess filings for unresolved cases
* Improves metadata coverage

---

### 14. Final export

**Purpose:** Produce the final dataset for use.

**Output:**

* `final_clean_kmsg_project_df.csv`

---

### 15–16. Exploratory data analysis

**Purpose:** Validate and analyze the dataset.

* Funding distribution
* Top funders and recipients
* Geographic trends
* Data quality checks

---

## How to run the pipeline

1. Open `final_kmsg_data.py` in a notebook environment (Google Colab recommended)
2. Run all sections **from top to bottom**
3. Monitor progress outputs
4. Locate final dataset in:

```
outputs/final_clean_kmsg_project_df.csv
```

---

## Configuration

Key parameters are defined in Section 1:

```python
TARGET_STATE = "PA"
TARGET_YEARS = list(range(2020, 2024))
MIN_REVENUE = 1_000_000
```

---

## Modifying the pipeline

### Change geographic scope

Update:

```python
TARGET_STATE = "PA"
```

Replace IRS dataset:

```
eo_pa.csv → eo_{state}.csv
```

---

### Adjust revenue threshold

```python
MIN_REVENUE = 1_000_000
```

* Lower → broader dataset, more noise
* Higher → narrower dataset, larger funders

---

### Adjust time range

```python
TARGET_YEARS = list(range(2020, 2024))
```

* Expand → more data, longer runtime
* Reduce → faster execution

---

## Data sources

* IRS EO Business Master File (BMF)
* IRS Form 990 XML filings
* ProPublica Nonprofit API
* Pennsylvania nonprofit reference dataset

---

## Limitations

* Includes only grants reported in IRS Schedule I
* Some organizations have incomplete filings
* Mission and website fields are inconsistently available
* Name-based matching may introduce minor inaccuracies
* Current implementation is scoped to Pennsylvania

---

## Summary

This pipeline converts fragmented public nonprofit data into a **structured dataset of funding relationships**.

It is designed to support:

* Funder discovery
* Funding analysis
* Research and policy work
* Future tool and model development

The approach emphasizes **transparency, reproducibility, and practical usability**, while remaining flexible for extension.
