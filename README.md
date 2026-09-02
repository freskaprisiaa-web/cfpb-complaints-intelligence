# CFPB Credit Card Complaints Intelligence

A Databricks portfolio project exploring credit card complaints received in January 2025 through data validation, Bronze–Silver–Gold processing, and an analytical dashboard.

This is an independent learning project, not a client engagement or a production deployment.

## Business questions

- How many credit card complaints were received during the selected month?
- How did complaint volume vary by day?
- What company response categories were recorded?
- What proportion of complaints had a response marked timely?

The dashboard provides descriptive evidence for investigating complaint handling. It does not establish the causes of complaint patterns or measure customer satisfaction.

## Project deliverables

- `dashboard/CFPB_Credit_Card_Complaints_January_2025.pdf` — dashboard snapshot for viewing without Databricks access.
- `notebooks/01_data_understanding_public.ipynb` — documented notebook with saved outputs removed and the individual complaint preview omitted.

The original notebook and raw CSV are retained privately and are not part of the intended public release.

## Data source and scope

Source: Consumer Financial Protection Bureau (CFPB), Consumer Complaint Database.

- Input file: `cfpb_credit_card_2025_01_raw.csv`
- Product: Credit card
- Date received: January 1–31, 2025
- Parsed records: 9,073
- Source columns: 16

The downloaded record count has not been reconciled with the result count displayed on the source website. Findings describe this downloaded dataset.

## Tools

Databricks Free Edition, PySpark, Spark SQL, Delta tables, Unity Catalog volumes, and Databricks AI/BI dashboards.

## Data processing

### Bronze: preserve the source

The CSV is read with all source fields initially treated as strings. Column names are standardized, and source-file and ingestion-time metadata are added.

Table: `workspace.cfpb.bronze_complaints`

### Silver: prepare analytical data

Source text is trimmed, empty strings become nulls, date fields are converted, and a forwarding interval is calculated. Consumer complaint narratives are excluded from Silver; complaint IDs and ZIP codes remain strings.

Table: `workspace.cfpb.silver_complaints`

### Gold: aggregate for the dashboard

| Table | Grain |
|---|---|
| `gold_monthly_summary` | Received month × product |
| `gold_monthly_response` | Received month × product × company response category |
| `gold_daily_summary` | Received date × product |

The Gold tables summarize the same complaint population and must not be added together.

## Validation performed in the original run

- All 9,073 complaint IDs were unique after trimming, with no missing or blank IDs.
- Received dates were valid and within January 2025.
- No null or blank values were found in the 10 selected analytical columns.
- Sent dates were valid, with none preceding received dates.
- Saved Silver data matched the transformed data using comparisons in both directions, including duplicate multiplicities.
- Gold aggregates reconciled to 9,073 complaints, 9,047 timely responses, 26 not-timely responses, and zero unknown values.
- The daily summary contained 31 distinct dates, with no inconsistent daily totals.

These checks cover the tested conditions, not every possible data-quality issue.

## Key findings

| Metric | Result |
|---|---:|
| Total complaints | 9,073 |
| Responses marked timely | 9,047 |
| Responses marked not timely | 26 |
| Timely response rate | 99.71% |
| Highest daily volume | 527 on January 16, 2025 |

Company response categories:

| Category | Complaints |
|---|---:|
| Closed with explanation | 5,745 |
| Closed with non-monetary relief | 2,288 |
| Closed with monetary relief | 1,032 |
| Untimely response | 8 |

Timely response rate = Yes ÷ (Yes + No) × 100.

Unknown timeliness values are excluded from the denominator; this dataset has none. The 26 records marked not timely and the 8 records categorized as “Untimely response” refer to different source fields.

## Interpretation limits

- Complaint counts do not represent all credit card customers.
- Timely response and closure categories do not establish customer satisfaction or resolution quality.
- The received-to-sent interval is a forwarding interval, not company response time.
- The reason for the January 16 peak has not been established.
- One month of data is insufficient to establish long-term trends.
- This is a fixed-period analysis; no automated refresh pipeline was configured.

## Reproduction and execution safety

The notebook requires a Databricks environment, the source CSV, and appropriate volume and table permissions. It does not run unchanged in standard local Jupyter.

Use a separate development schema, update all input paths and table references, and execute cells in order while reviewing validation results.

Table creation intentionally fails when a target already exists. Do not drop or overwrite existing project tables merely to rerun the notebook.

The original notebook contains historical execution results. The edited public notebook has been inspected but has not been rerun end-to-end. Dashboard configuration is documented through the PDF rather than recreated automatically by the notebook.