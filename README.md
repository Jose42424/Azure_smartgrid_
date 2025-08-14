# Azure_smartgrid_
This project demonstrates how to simulate and send smart grid energy meter readings into Azure Event Hubs using Python from VS Code. It’s a lightweight implementation designed to test event ingestion pipelines, validate connectivity, and produce sample datasets for further processing in Azure.

Key Features

Python Data Generator
A simple script (send_to_eventhub.py) simulates meter readings — voltage, current, real power, and power factor — with randomised values to mimic realistic data.

Manual Event Sending
Events are sent in batches on a configurable schedule (e.g., every 5 seconds, 20 events per batch). No continuous aggregation or streaming analytics in this version.

Azure Event Hubs Integration
Uses Event Hubs Standard tier to receive and log all incoming messages.

Evidence Data
Includes sample output, screenshots from Azure Portal metrics, and event logs as proof of successful ingestion.

Technologies Used

Azure Event Hubs – Real-time event ingestion endpoint

Python – Event simulation and sending

VS Code – Local development environment

Azure Portal – For monitoring event throughput and verifying message arrival

Example Use Case

Testing Event Hub ingestion before building a full streaming pipeline.

Producing sample IoT/sensor datasets for proof-of-concept projects.

Demonstrating how to connect a Python application to Azure Event Hubs.
