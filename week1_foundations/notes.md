## Day 1: Splunk Setup in Google Cloud

###  Objective
Set up a Splunk Enterprise instance in the cloud, accept licensing, and confirm access to the web UI.

---

###  Steps Completed
1. **Provisioned Google Cloud VM**
   - Instance: `splunk-lab`
   - Machine type: `e2-standard-4` (4 vCPUs, 16 GB RAM)
   - Disk: `50 GB` (Ubuntu 22.04 LTS, x86_64)
   - Region: `us-central1`

2. **Installed Docker on VM**
   - Verified with:
     ```bash
     docker ps
     docker run hello-world
     ```

3. **Pulled and Launched Splunk Enterprise Container**
   ```bash
   docker run -d \
     --name splunk \
     -p 8000:8000 -p 8089:8089 \
     -e SPLUNK_START_ARGS="--accept-license" \
     -e SPLUNK_GENERAL_TERMS="--accept-sgt-current-at-splunk-com" \
     -e SPLUNK_PASSWORD="**********" \
     -v splunk-data:/opt/splunk/var \
     splunk/splunk:latest
4. **Configured Google Cloud Firewall Rule**
   - Created firewall rule allow-splunk
   - Allowed inbound traffic on TCP port 8000

5. **Verified Splunk Web Access**
   - Accessed Splunk at http://<External-IP>:8000
   - Logged in with:
   -   username: admin
   -   password: StrongPassw0rd!
   - Confirmed Splunk Enterprise login Screen

  ### Outcome
  - Splunk Enterprise is successfully running in Docker on a Google Cloud VM
  - Web UI is accessible from browser
  - Day 1 objective is completed.
  

# Day 2 – Syslog Ingestion into Splunk

## Goal
Set up a fresh data input (syslog) and confirm logs are being indexed.

---

## Steps

### 1. Expose UDP 1514 on Splunk container
```bash
docker run -d --name splunk \
  -p 8000:8000 \
  -p 8089:8089 \
  -p 9997:9997 \
  -p 1514:1514/udp \
  -e SPLUNK_GENERAL_TERMS=--accept-sgt-current-at-splunk-com \
  -e SPLUNK_START_ARGS=--accept-license \
  -e SPLUNK_PASSWORD="**********" \
  -v splunk-data:/opt/splunk/var \
  -v splunk-etc:/opt/splunk/etc \
  splunk/splunk:latest 
```
### 2. Configure rsyslog forwarding
```bash
sudo tee /etc/rsyslog.d/90-splunk.conf > /dev/null <<'EOF'
auth,authpriv.*    @127.0.0.1:1514
*.info;mail.none;auth.none;authpriv.none    @127.0.0.1:1514
EOF

sudo rsyslogd -N1 && sudo systemctl restart rsyslog
```
### 3. Verify packets leaving host
```bash
sudo tcpdump -n -l -i lo udp port 1514
```
### 4. Send a test log
```bash
logger -p auth.info "SPLUNK TEST: $(hostname) $(date)"
```
### 5. Configure Splunk UDP input
In Splunk Web:
- Navigate to Settings --> Data Inputs --> UDP --> Add New
- Port: 1514
- Source type: syslog
- Index: lab_syslog

### 6. Verify ingestion in Splunk
```spl
index=lab_syslog sourcetype=syslog "SPLUNK TEST:" | table _time host program message
```
### Results
- Test messages successfully ingested from VM --> rsyslog --> Splunk
- Verified in Splunk UI with host = 172.17.0.1
- Screenshot saved in Week1/screenshots

  # Day 3 – SPL Basics & Log Analysis
  ## Goal
  Practice Splunk Search Processing Language (SPL) skills and analyze real Linux authentication logs ingested into Splunk

---
## Steps
### 1. Confirm live data ingestion
```spl
index = lab_syslog
```
### 2. Perform basic SPL searches
Queried the lab_syslog index to explore ingested data
```spl
index = lab_syslog
```
Filtered events by keyword
```spl
index = lab_syslog "SPLUNK LAB"
```
Filtered events by program
```spl
index = lab_syslog program = su
```
### 3. Apply time range filtering
Limit searches to recent activity using time-based filters
```spl
index = lab_syslog ealiest = -15m
```
This limits the results to most recent 15 minutes.

### 4. Summarize logs using stats
Used aggregation to identify which programs generated the most log activity
```
index = lab_syslog
| stats count by program
```
this helped identify authentication-related activity (su, sudo, logger)

### 5. Generate and analyze authentication events
Created real authentication telemetry on the Linux VM
```bash
sudo useradd testuser1
su testuser1
sudo passwd testuser1
su testuser1
exit
```
Analyzed authentication activity in Splunk
```spl
index = lab_syslog testuser 1
```
### Observed: 
- Failed attempts before a password was set
- Successful login after the password was created
- Sessions open and close events

### Results
- Confirmed real-time ingestion of Linux syslog data into Splunk
- Successfully queried logs using SPL (indexing, filtering, and time scoping)
- Summarized log activity using stats
- Analyzed authentication events, including failed and successful su attempts
- Validated end-to-end SIEM pipeline: Linux VM -> rsyslog -> UDP -> Splunk -> SPL analysis.












  
