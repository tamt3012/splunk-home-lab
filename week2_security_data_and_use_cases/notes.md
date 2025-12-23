## Week 2 – Security Data & Use Cases

## Day 1: Add Multiple Data Sources (Linux + Web Logs)

### Objective
Expand the lab beyond basic ingestion by adding multiple security-relevant data sources and preparing the data for detections/dashboards.

---

### What I Added (New Data Sources)
1. **Linux Authentication Logs**
   - Source: `/var/log/auth.log`
   - Index: `os_linux`
   - Sourcetype: `linux_secure`

2. **Linux System Logs**
   - Source: `/var/log/syslog`
   - Index: `os_linux`
   - Sourcetype: `linux_syslog`

3. **Nginx Web Server Logs**
   - Source: `/var/log/nginx/access.log`
     - Index: `web`
     - Sourcetype: `nginx:access`
   - Source: `/var/log/nginx/error.log`
     - Index: `web`
     - Sourcetype: `nginx:error`

---

### Steps Completed

#### 1) Installed & Started Splunk Universal Forwarder (UF) on the VM host
- Downloaded UF `.deb` for `x86_64` and installed via `dpkg`.
- Started UF and accepted license.

#### 2) Resolved UF Port Conflict
- UF initially failed because Splunk Enterprise was already using management port **8089**.
- Changed UF management port to **8090** so UF could run on the same VM.

#### 3) Confirmed Splunk is Ready to Receive Forwarded Data
- Verified the Splunk container exposes **9997/tcp** (receiving port).
- Verified UF forward-server is **active** to `127.0.0.1:9997`.

#### 4) Added Monitors for Linux + Nginx Log Files
- Configured UF to monitor:
  - `/var/log/auth.log`
  - `/var/log/syslog`
  - `/var/log/nginx/access.log`
  - `/var/log/nginx/error.log`
- Restarted UF after adding monitors.

#### 5) Generated Web Traffic to Produce Test Logs
- Used `curl` requests to `http://localhost/` to generate nginx access events.

#### 6) Built Persistent Field Extraction for Nginx Access Logs
- Used Splunk Field Extractor for `sourcetype=nginx:access`.
- Extracted key fields for detections and dashboards:
  - `client_ip`, `method`, `uri`, `status`, `bytes`, `referrer`, `user_agent`
- Verified fields work in SPL (example below).

---

### Validation Searches

#### Linux ingestion check
```spl
index=os_linux earliest=-15m
| stats count by host, sourcetype

### Web ingestion check
```spl
index = web earliest = -15m
| table _time host source sourcetype _raw
| head 20
```

### Web field extraction check
```spl
index=web sourcetype="nginx:access" earliest=-24h
| stats count by status method uri
```

## Results
By the end of Day 1, the lab successfully ingests multiple data sources (Linux + web logs) and has usable parsed fields from nginx access logs

---

## Day 3

Because **Splunk Free** does not support native alerting actions, I implemented “alerts & detections” as **dashboard-based detections** (panels that surface suspicious patterns for investigation). The goal was to build a single SOC-style view that answers:

- **What’s changing over time?** (trend panels)
- **What looks suspicious right now?** (detection panels)
- **What context do I need to triage quickly?** (top talkers / sources)

### Dashboard: `Security Operations Overview`
**Design approach**
- **Top row = trends** (fast situational awareness)
- **Bottom row = detections + context** (triage queue)
- Global time range set to **Last 24 hours** (with detection panels using shorter windows like 15–60 minutes)

---

### Panels & SPL

#### 1) Web Status Codes Over Time (2xx / 3xx / 4xx / 5xx)
Purpose: quick web health + anomaly spotting (4xx/5xx spikes can indicate scanning, broken deployments, or attacks).
```spl
index=web sourcetype=nginx:access earliest=-24h
| eval status_class=case(
  status>=200 AND status<300,"2xx",
  status>=300 AND status<400,"3xx",
  status>=400 AND status<500,"4xx",
  status>=500 AND status<600,"5xx",
  1=1,"other"
)
| timechart span=1h count by status_class
```
### Total Events (Last 24h)
Purpose: single metric accross host + web telemetry
```spl
(index = os_linux OR index = web) earliest = -24h
| timechart span = 1h count
```

### Top Web Client IPs (Context)
Purpose: identify top talkers and quickly pivot into investigation
```spl
index=web sourcetype=nginx:access earliest=-24h
| stats count as requests by client_ip
| sort - requests
| head 15
```

### Detection Panels (Alert Replacement)
These panels are intended to be mostly empty. If results appear, it comes an "investigate now" queue

### DETECTION - Sensitive Path Probing
Purpose: detect common automated probing for high-risk endpoints and misconfigs
```spl
index=web sourcetype=nginx:access earliest=-60m
| where match(uri,"(?i)(/\\.env|wp-login\\.php|xmlrpc\\.php|phpmyadmin|/admin|/cgi-bin/)")
| stats count values(uri) as uris values(user_agent) as user_agents by client_ip status
| sort - count
```
### DETECTION - Web Scan (404 Spike)
Purpose: detect scanning behavior via abnormal 404 volume / URI diversity
```spl
index=web sourcetype=nginx:access earliest=-15m status=404
| stats count dc(uri) as unique_uris values(uri) as sample_uris by client_ip
| where count >= 10 OR unique_uris >= 10
| sort - count
```
### DETECTION - SSh Brute Force (Host Auth)
Purpose: detect repeated SSH auth failures from a single source
```spl
index=os_linux sourcetype=linux_secure earliest=-15m ("Failed password" OR "Invalid user")
| stats count dc(user) as users values(user) as sample_users by src
| where count >= 10
| sort - count
```

## Week 2 Conclusion
Week upgraded the lab from basic Splunk exploration to a SOC-oriented workflow. I expanded telemetry by ingesting both host and web data sources, then used that data to build a Security Operations Overview dashboard that supports basic detection and triage even under Splunk Free limitations. Instead of native alerts, I implemented dashboard-based detections that surface suspicious patterns (web probing, scanning indicators, and SSH brute force behavior). This establishes a foundation for Week 3, where I'll ingest a dedicated attack dataset and begin writing incident-style investigation notes based on the detections.
