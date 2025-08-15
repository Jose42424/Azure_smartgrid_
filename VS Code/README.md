Smart Grid — Event Hub Sender (overview only)

The sender lives in a separate folder and is referenced here for documentation only. No script code is included in this README.

Where it is

Folder: EventHub_Sender/

Files inside (for reference):

send_to_eventhub_continuous.py – sends a steady trickle of simulated events

(optional) send_raw_smartgrid.py – alternate/simple sender

What it does

Generates lightweight smart-grid meter events and publishes them to Azure Event Hubs.

Each event contains: timestamp, meter_id, real_power_watts, voltage, power_factor.

Prints each payload and a success message after send, which serves as evidence.

Prerequisites

Python 3.9+

Package: azure-eventhub (install via pip install azure-eventhub==5.*)

An Event Hub with a SAS policy that has Send permission.

Configuration (no secrets in code)

Set credentials via environment variables (recommended):

Windows PowerShell

$env:EVENTHUB_CONNECTION_STR = "Endpoint=sb://<namespace>.servicebus.windows.net/...;EntityPath=<hub>"
$env:EVENTHUB_NAME = "<hub>"


macOS/Linux

export EVENTHUB_CONNECTION_STR="Endpoint=sb://<namespace>.servicebus.windows.net/...;EntityPath=<hub>"
export EVENTHUB_NAME="<hub>"


Keep secrets out of the repo; if you use a .env, add it to .gitignore.

How to run (from the folder)
python send_to_eventhub_continuous.py

Evidence of success

Terminal shows lines like:

Sending: {"timestamp":"...","meter_id":"MTR-1234", ... }

Event N sent successfully.

Optional portal checks:

Azure Portal → Event Hubs → Metrics → Incoming Messages increases.

If Capture is enabled, files appear in ADLS Gen2.

Tuning knobs

Volume/Cadence: edit the loop count and sleep interval in the continuous sender.

Value ranges: adjust random ranges for watts/voltage/power factor inside the generator.

Troubleshooting

Auth/403: confirm SAS policy includes Send and the connection string’s EntityPath matches the hub.

Throttling/Timeouts: increase sleep or batch multiple events per send.

Schema drift: keep field names consistent with downstream consumers.

Notes

This sender is for dev/demo; for production testing, prefer replayable datasets (e.g., Event Hubs Capture to ADLS Gen2) and CI/CD secrets management (Key Vault, GitHub Actions secrets, etc.).
