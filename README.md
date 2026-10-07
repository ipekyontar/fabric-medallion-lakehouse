# Microsoft Fabric Medallion Lakehouse (Bronze / Silver / Gold) with Direct Lake

An end-to-end data engineering and analytics project built on **Microsoft Fabric**. Raw automotive service data is ingested into a **medallion architecture** on OneLake, cleaned with **PySpark**, aggregated into a business-ready Gold table, orchestrated with a **Data Pipeline**, and reported in **Power BI via Direct Lake**.

## Architecture

```
CSV exports (MySQL)  ->  Bronze  ->  Silver  ->  Gold  ->  Direct Lake semantic model  ->  Power BI report
                         (raw)      (clean)    (aggregated)
                                  \_______ orchestrated by a Fabric Data Pipeline _______/
```

| Layer | Lakehouse | What happens |
|---|---|---|
| Bronze | `bronze_lakehouse` | The three source CSV files are uploaded unchanged to `Files/`, then loaded with PySpark as Delta tables (`araclar_bronze`, `musteriler_bronze`, `servis_kayitlari_bronze`). |
| Silver | `silver_lakehouse` | Bronze tables are read with PySpark, duplicate records are dropped on the primary key, rows with a null key are filtered out, and the result is written as Delta tables (`*_silver`). |
| Gold | `gold_lakehouse` | Silver tables are joined and aggregated by fuel type and city (average satisfaction score and total service count) into the `yakit_memnuniyet_gold` table. |
| Reporting | Semantic model + report | A Direct Lake semantic model on the Gold table reads data straight from OneLake, with no data copy, and feeds a Power BI report. |

## Orchestration

A Fabric **Data Pipeline** runs the notebook that performs the Bronze -> Silver -> Gold steps, so the whole flow runs without manual intervention. The pipeline is scheduled to run daily at 12:00, and a full run completed successfully end to end.

## Dataset

Three small, synthetic tables exported from a MySQL database (see [`data/`](data/)):

- `musteriler.csv`: 50 customers (name, city, contact details, registration date)
- `araclar.csv`: vehicles per customer (brand, model, year, fuel type: electric, petrol, diesel, hybrid)
- `servis_kayitlari.csv`: service records per vehicle (service type, cost, satisfaction score)

All names, phone numbers, emails and plates are generated sample data.

## Tech stack

Microsoft Fabric, OneLake, Delta Lake, PySpark (Fabric notebooks), Fabric Data Pipelines, Power BI (Direct Lake), MySQL (source of the CSV exports).

## Results

The report shows total service counts and average satisfaction by city and fuel type, built directly on the Gold table.

## Notes and limitations

- Copilot-assisted report generation was not available in the Fabric trial tier, so the report was built manually in Power BI.
- Data cleaning in the Silver layer is intentionally simple (deduplication and null-key filtering). Extending it with value standardization (for example city names and fuel types) and data quality checks would be the natural next step.

## Repository contents

- `data/`: the three sample CSV files
- `notebooks/`: the PySpark notebook(s) exported from Fabric
- `docs/`: project presentation (PDF)

## What I learned

How a medallion architecture separates raw, cleaned and business-ready data, how Delta tables and OneLake keep every layer in one place, and how Direct Lake removes the need to copy data for reporting.
