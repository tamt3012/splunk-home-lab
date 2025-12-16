## Day 1: Splunk Setup in Google Cloud

###  Objective
Set up a Splunk Enterprise instance in the cloud, accept licensing, and confirm access to the web UI.


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
  
---

# Day 2 – Syslog Ingestion into Splunk

## Goal
Set up a fresh data input (syslog) and confirm logs are being indexed.


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

---

  # Day 3 – SPL Basics & Log Analysis
  ## Goal
  Practice Splunk Search Processing Language (SPL) skills and analyze real Linux authentication logs ingested into Splunk


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

___


# Day 4 – Field Exploration, Tables, and Basic Statistics in Splunk

## Goal
Practice exploring existing fields, creating simple tables, and generating basic statistics and time-based visualizations using SPL

## Steps
### 1. Verify data exists in index
```spl
index = lab_syslog
```
this confirms events are still actively ingested

### 2. Explore available fields
fields observed: process, host, sourcetype, source, _time

### 3. Count events by process
```spl
index = lab_syslog
|stats count by process
|sort - count
```
identified high-volume processes such as systemmd, rsyslogd, and su
learned to aggregate events using stats

### 4. Filter events by a specific process
```spl
index = lab_syslog process = su
```
```spl
index = lab_syslog process = su
| table _time host process message
```

### 5. Create time-based statistics
```spl
index = lab_syslog process = su
|timechart count
```
visualized event frequency over time
introduced time-based analysis 

### 6. Perform a basic field extraction with rex
extracted usernames from su messages:
```spl
index=lab_syslog process=su
| rex "user (?<user>\w+)"
| stats count by user
```
created a new field dynamically
aggregated activity by extracted user

## Results 
- Successfully identified valid fields through exploration
- Built tables and statistics using SPL
- Created time-based visualizations
- Extracted custom fields from raw log messages

___
# Day 5 – Security Dashboards in Splunk
## Goal
Create multiple security-focused dashboards in Splunk to visualize authentication activity, process behavior, and overall syslog health using real ingested data.

These dashboards show that raw logs can be transformed into actionable security insights.

## Dashboard 1: Syslog Health Overview

### Purpose
Monitor overall log ingestion health to ensure data is flowing consistently and identify spikes or drop-offs in event volume.

### Panel 1 – Syslog Events Over Time
```spl
index=lab_syslog
| timechart span=1h count
```
- column chart
- shows hourly syslog ingestion volume

### Panel 2 – Event by Sourcetype
```spl
index = lab_syslog
|stats count by sourcetype
```
- bar chart
- confirms all ingested events are classified under the syslog source type.

### Panel 3 – Eevnts by Host
```spl
index = lab_syslog
|stats count by host
```
- bar chart
- validates log volume coming from the lab VM host

## Dashboard 2: Authentication Monitoring

### Purpose
Track privilege escalation activity using su logs to identify potential misuse or abnormal authentication behavior.

### Panel 1 – su Attempts Over Time
```spl
index = lab_syslog process = su
|timechart count
```
- column chart
- displays frequency of su usage over time

### Panel 2 - su Usage by User
```spl
index = lab_syslog process = su
| rex "user (?<user>\w+)"
|stats count by user
```
- bar chart
- shows which users are attempting privilege escalation

## Dashboard 3: Process Activity Monitoring

### Purpose
Establish a baseline of process behavior to identify high-volumn or abnormal process activity.

### Panel 1 – Process Activity Over Time
```spl
index = lab_syslog
| timechart count by process limit=5
```
- stacked column chart
- displays activity trends for the most active processes

### Panel 2 – Total Process Event Volume
```spl
index = lab_syslog
|timechart span=1h count
```
- column chart
- shows overall process-related event volume over time

## Results
- Built 3 functional security dashboards
- Added multiple panels per dashboard
- Used real ingested syslog data
- Applied SPL commands such as stats, timechart, and rex
- Gained hands-on experience with SOC-style monitoring workflows.

___

# Day 6 – Splunk Knowledge Object (Saved Searches & Macros)

## Goal
Understand and implement Splunk knowledge objects to make searches reusable and scalable, focusing on **saved searches** and **macros** (alerts skipped due to Splunk Free limitations)

## Saved Searches
```spl
index=lab_syslog process=su
| rex "user (?<user>\w+)"
| stats count by user
| sort - count
```

## Search Macros
### Base Syslog Search
create a resuable base search for all syslog analysis
```spl
`syslog_base`
```

### Process Filter Macro
filter syslog event by process name dynamically
```spl
`proc_filter(1)`
```

## Results
- saved searches improve analyst efficiency
- macros enforce consistent detection logic
- parameterized macros enable flexible filtering
- knowledge objects form the backbone of scalable SOC workflows

# Week 1 Reflection
This week focused on setting up Splunk and understanding how real log data flows from a system into a SIEM.

I successfully deployed Splunk Enterprise on a Google Cloud VM using Docker and configured syslog ingestion from the same VM. By forwarding Linux syslog data into Splunk, I worked with real authentication and system activity instead of sample data.

Throughout the week, I practiced core SPL skills such as filtering, time-based searches, aggregations with `stats`, and visualizations with `timechart`. I analyzed authentication events (`su`), observed failed and successful login attempts, and built dashboards to monitor syslog health, authentication activity, and process behavior.

I also learned how Splunk knowledge objects like saved searches and macros help make searches reusable and consistent, even though alerting was limited in the Splunk Free version.

Overall, this week gave me a solid foundation in SIEM basics, log ingestion, and security-focused analysis. I’m more comfortable navigating Splunk, asking the right questions of log data, and understanding how these skills apply to real SOC workflows.






  
