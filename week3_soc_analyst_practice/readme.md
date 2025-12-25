# Week 3 SOC Analyst Practice

## Day 1 – Ingested Splunk BOTSv2 Dataset
### Purpose
Bring a realistic attack dataset into Splunk to practice SOC-style investigation, detection building, and incident notes using real security telemetry (network, endpoint, IDS, etc.)

### 1. Selected dataset
I chose Splunk Boss of the SOC (BOTSv2) as the sample attack dataset for Week 3

### 2. Loaded data into Splunk
I ingested the dataset into Splunk under: 
Index: botsv2

### 3. Validated ingestio
I ran searches to confirm that events were successfully indexed and that multiple data sources were present
```spl
eventcount summarize = false index = botsv2
```
confirm the index contains events

```spl
index = botsv2 earliest = 0
|stats count by sourcetype
|sort - count
```
inventory sourcetypes in the dataset

## Results
- Splunk returned 68,870,348 events in index = botsv2
- Splunk identified 103 unique sourcetypes
- High-volume sourcetypes included:
  - mysql:server:stats, mysql:transaction:details
  - wineventlog:security
  - suricata
  - stream:tcp, stream:ip, stream:smtp
  - plus host telemetry and system metrics sourcetypes
 This confirmed that the dataset is fully ingested and contains diverse telemetry suitable for SOC practice.

