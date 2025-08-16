Event Hubs — SmartGridStream (README)
Overview

This namespace/Hub powers the smart-grid demo. Events are produced from VS Code and consumed in Databricks for streaming validation and analytics.

Namespace: JMEventHubs

Event Hub: smartgridstream

Partitions: 1 (sufficient for demo)

Evidence: Portal metrics show spikes in Incoming Messages and Throughput during sends (see screenshots).

SAS Policies (who can do what)
Policy	Claims	Used by
send-policy	Send	VS Code sender app (producer)
smartgrid_policy	Send, Listen	Databricks notebooks (consumer) and admin tasks

Tip: create separate Send/Listen policies so producers can’t read and consumers can’t write.

Producers (VS Code)

The sender lives in your EventHub_Sender/ folder (Python).

Configure the Send connection string (SAS for send-policy) with EntityPath=smartgridstream.

Run the script from VS Code; the terminal prints each JSON payload and “Event N sent successfully”.

Environment variables (recommended):

EVENTHUB_CONNECTION_STR — SAS for send-policy

EVENTHUB_NAME — smartgridstream

Consumers (Databricks)

Use the Listen connection string (SAS for smartgrid_policy) in your streaming notebook.

Read from Event Hubs (spark-eventhubs), parse JSON, and write Delta to your ADLS Gen2 validated/curated zones.

Checkpointing is enabled so the stream resumes safely.

Secrets: store keys in Databricks Secrets; reference them in notebook configs.

Verify it’s working

Azure Portal → Event Hubs → smartgridstream → Metrics

Look for rises in Incoming Messages and Incoming Bytes while the sender runs.

Databricks

Stream status = active; Delta files and checkpoints updating.

Troubleshooting

Auth errors: confirm you copied the correct policy string (Send vs Listen) and that EntityPath=smartgridstream is present.

No messages: verify the consumer group in Databricks matches (use $Default for the demo), and that partitions=1 is acceptable for your load.

Throughput spikes but no files: check Databricks schema, bad-records path, and permissions on ADLS.

Notes

Keep SAS keys out of source control. Rotate keys if a secret was ever committed.

For production: increase partitions, use Capture to ADLS, Managed Identity/AAD auth, and CI/CD for secrets.
