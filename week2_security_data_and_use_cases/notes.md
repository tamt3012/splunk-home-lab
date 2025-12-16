## Week 2 – Security Data & Use Cases

### Day 1: Add Multiple Data Sources (Linux + Web Logs)

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

