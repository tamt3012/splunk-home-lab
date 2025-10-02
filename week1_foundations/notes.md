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
  
