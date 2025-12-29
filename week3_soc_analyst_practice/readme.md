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

## Day 2 - Identify Suspicious Patterns

### Objective
Identify suspicious patterns in the BOTS v2 dataset inside Splunk by going through high-signal sourcetypes. The goal was to quickly surface activity consistent with recon/scanning, brute force authentication attempts, and anonymized/TOR-related network traffic.

### Data Source / Scope
- Index: botsv2
- Primary sourcetypes used:
- suricata (Network IDS alert)
- wineventlog:security (Windows auth/security events)

### Approach
1. Validated ingestion volume and enumerated sourcetypes
2. Prioritized security useful sources
3. Ran targeted SPL searches to detect:
   - Port scanning/recon behavior
   - Brute force or spray-like authentication failures
   - TOR/TLS IDS indicators
4. Summarized findings with counts, top talkers, and time trends

### Finding 1 (primary) - Port 135 scanning / RPC recon (Suricata)
What I found:
- Suricata flagged high-volume unusual port 135 traffic consistent with scanning/recon
- Most common activity was
  - src_ip 10.0.1.1 -> dest_ip 10.0.1.100
  - Signature: ET SCAN Behavioral Unusual Port 135 traffic Potential Scan or Infection
  - Count: 1547
- A minor additional scan indicator appeared for Port 445 (count 1), which can be related to SMB probing

#### Why it matters 
- Port 135 is commonly associated with Windows RPC and often appears in early-stage recon prior to lateral movement attempts. The high concentration (single source targeting a single destination at scale) is strongly suspicious.

#### How it's validated
- First query identified the top src/dest/signature combinations
- Second query confirmed the activity overwhelmingly targeted 10.0.1.100

### Finding 2 (Secondary) - Failed logon bursts (Windows Security EventCode 4625)
What I saw:
- Windows Security logs show a high number of failed logons (4625), consistent with brute force, password spray, or misconfigured authentication atttemps.
- Top failure sources include:
  - src_ip 10.0.1.100 with 3162 fails
  - src_ip 10.0.1.220 with 109 fails
- Timechart visualization shows the failures occur in bursts/spikes, not a flat baseline

#### Why it matters
- high failure volume from specific sources, especially when burst, is a common sign of credential attacks or scripted authentication attempts.

### Finding 3 (secondary) - TOR/TLS-related IDS alerts targeting 10.0.2.101 (Suricata)
What I saw:
- Suricata alerts indicate TOR-related or TOR-like TLS traffic involving dest_ip 10.0.2.101
- Example top entries:
  - 37.187.30.78 -> 10.0.2.101
  - 138.68.174.81 -> 10.0.2.101
- Time trend shows these are low volume compared to the Port 135 scanning, but still notable as an indicator category

#### Why it matters
- TOR indicators can suggest anonymized communication paths. In real environments, this can be suspicious depending on policy and asset criticality (however, IDS signatures can also produce false positives).

