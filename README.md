Azure_smartgrid_ — End-to-End Demo

Simulates smart-grid meter readings, streams them to Azure Event Hubs, lands them in Databricks as Delta, curates a BI-friendly dataset, and visualizes insights in Power BI.

Architecture (high level)

VS Code (Python Sender) → Event Hubs (smartgridstream) → Databricks Structured Streaming → ADLS Gen2

raw/ → validated Delta (+ checkpoints & bad-records) → clean/ → curated/ smart_grid_curated.csv
→ Power BI (connected to curated CSV; can swap to ADLS Gen2 folder/Parquet)

Repo structure
/Event Hub/        – Mini README (SAS Send/Listen policies, portal metrics, producers/consumers)
/VS Code/          – Sender overview (how to run; no secrets committed)
/Databricks/       – Notebooks: mount storage, stream from EH, clean, curate + README
/Power BI/         – Visuals README + source swap notes (CSV ↔ ADLS Gen2)
/adls_gen2_structure/ – Example lake layout & conventions
/README.md         – This file (quickstart, architecture, links)

Quickstart

Clone

git clone <repo-url>
cd Azure_smartgrid_


Create Event Hub + policies

Event Hub: smartgridstream

SAS policies:

send-policy → Send (for the VS Code producer)

smartgrid_policy → Send, Listen (for Databricks)

Run the producer (VS Code)

See /VS Code/README.md to set:

EVENTHUB_CONNECTION_STR (SAS for send-policy, includes EntityPath=smartgridstream)

EVENTHUB_NAME=smartgridstream

Start the sender; terminal prints each JSON payload and “Event N sent successfully”.

Start the pipeline (Databricks)

Open /Databricks/ notebooks and run in order:

Container Configuration (mount ADLS Gen2)

Validated Data Streaming (reads from Event Hubs → Delta + checkpoints + bad-records)

Cleaning raw data (standardize/dedupe/null handling)

Curating cleaned data (coalesce & export smart_grid_curated.csv)

Outputs:

curated/validated_data/ (Delta)

clean/...

curated/smart_grid_curated.csv (for BI)

Visualize (Power BI)

Open /Power BI/README.md and connect to the curated CSV.

You can swap the source to ADLS Gen2 Parquet (instructions in the same README).

Evidence (screenshots to include in repo)

Event Hubs metrics showing spikes in:

Incoming Messages / Requests / Bytes during sends.

Shared Access Policies view showing:

send-policy (Send) and smartgrid_policy (Send, Listen).

Validated Delta files in ADLS Gen2 (validated/curated paths).

Power BI visuals (the 4 you’re using):

Global Watts Trend

Meter Energy by Daypart

Daily kWh per Meter

Target Daily kWh (gauge with min/max/rolling target)

Data model (essentials)

Event schema (from sender):
timestamp, meter_id, real_power_watts, voltage, power_factor

Curated BI table (columns may extend):
timestamp, meter_id, energy_kwh, real_power_watts, voltage_v, power_factor, high_demand_flag, anomaly_flag, time_of_day, …

Security & costs

Do not commit secrets. Use environment variables locally and Databricks Secrets in notebooks.

For demos/dev, the Power BI report points to a CSV artifact for portability and cost control; the same model can point back to ADLS Gen2 when needed.

Troubleshooting

Producer auth errors → confirm SAS is for send-policy and includes EntityPath=smartgridstream.

Consumer not landing files → check Listen SAS (smartgrid_policy), consumer group ($Default for demo), and ADLS permissions; inspect badRecords path.

Many small files → keep streaming for raw/validated; use the curation step to coalesce for BI.

Power BI source → see /Power BI/README.md for CSV ↔ ADLS swap snippets.

Requirements

Python 3.9+ with azure-eventhub (see /VS Code/).

Databricks Runtime 11+ recommended.

ADLS Gen2 account + container.

Power BI Desktop (latest).
