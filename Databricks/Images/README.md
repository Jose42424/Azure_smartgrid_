Databricks — Smart Grid Streaming (README)

A concise guide to run the Databricks notebooks that ingest from Azure Event Hubs, land validated Delta, and prepare curated CSVs.

What you’ll run (order)

Container Configuration – mounts your ADLS container in DBFS (e.g., /mnt/smartgrid). 

Validated Data Streaming – reads JSON events from Event Hubs and writes a Delta stream to the validated zone with checkpoints and bad-record capture. 

Cleaning raw data – batch-cleans a raw CSV (standardizes columns, dedupes, fills null numeric values) into the clean zone. 

Curating cleaned data – coalesces to one file and renames to a friendly CSV in the curated zone. 

Evidence: Azure portal metrics screenshot (“Incoming Messages” spikes) confirms events arrived in the namespace. Add it to your repo as proof of send/ingest.

Prerequisites

A Databricks cluster (DBR 11+ recommended).

Access to an ADLS Gen2 account & container; a Storage key or SAS.

An Event Hub connection string with a listen policy.

The sender (separate folder) running to publish messages.

Notebook details (what each one does)
1) Container Configuration

Sets your storage account/key and mounts:
dbutils.fs.mount("wasbs://<container>@<account>.blob.core.windows.net", "/mnt/smartgrid", extra_configs={...}).
Output mount point used by the other notebooks: /mnt/smartgrid. 

2) Validated Data Streaming

Encrypts the Event Hubs connection string and configures the spark-eventhubs source.

Reads the stream, parses JSON with schema:
timestamp, meter_id, real_power_watts, voltage, power_factor.

Writes Delta to: abfss://<container>@<account>.dfs.core.windows.net/curated/validated_data
with checkpoint and badRecords paths, and mergeSchema=true. 

Why this matters: Delta + checkpoints let you resume safely; badRecordsPath quarantines malformed events without stopping the stream. 

3) Cleaning raw data (batch)

Loads /mnt/smartgrid/raw/raw_data.csv, standardizes column names, drops duplicates, and fills numeric nulls.

Writes cleaned output to /mnt/smartgrid/clean/…. 

4) Curating cleaned data (batch)

Coalesces to a single file, then renames it to /mnt/smartgrid/clean/smart_grid_cleaned.csv and removes the temp folder—handy for BI tools. 

How to run

Open the notebooks in order and Run All.

For the streaming notebook, leave it running (or switch to a trigger once pattern if you just want a single micro-batch). 

Verify:

Files appear under curated/validated_data/ and checkpoints update. 

Your clean/curated CSVs exist under /mnt/smartgrid/clean/…. 

Event Hubs Incoming Messages metric spikes (screenshot provided).

Ops tips

Secrets: move storage keys/EH strings into Databricks Secrets instead of hardcoding.

Schema drift: mergeSchema=true will evolve columns; combine with data tests if needed. 

Bad records: check the quarantine/bad_records path to investigate parser failures. 

Batch outputs: the curated step renames the part file so downstream (e.g., Power BI) can bind to a stable filename. 

Troubleshooting

Auth errors to storage → verify the account key/SAS and that the mount succeeded. 

No events landing → confirm the Event Hub policy, consumer group, and that the sender is running. 

Multiple small files → keep the streaming writer for raw/validated; use the curation step to coalesce for BI.
