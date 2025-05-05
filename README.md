# Splunk Home Lab

Setting up Splunk Enterprise in Docker for SOC practice: log inspection, search, detection, and dashboarding.

---

## Project Objective
Build a hands-on home lab using Splunk to simulate a basic Security Operations Center (SOC) environment. Practice ingesting logs, writing SPL queries, and creating detections. 

---

## Environment Setup
**Platform**: MacOS (14" M1 Macbook Pro)

**Tools Used**:
- Docker Desktop
- Splunk Enterprise

---

## Getting Started

### 1. Install Docker
Install Docker Desktop from [https://www.docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop) 
Sign up and Log into Docker Hub account.

### 2. Run Splunk in Docker using terminal
```bash
docker run -d \
  --platform linux/amd64 \
  -p 8000:8000 \
  -e SPLUNK_START_ARGS="--accept-license" \
  -e SPLUNK_PASSWORD=My5tr0ngP@ssw0rd \
  --name splunk \
  splunk/splunk:9.1.2
```

### 3. Access Splunk
Go to [http://localhost:8000](http://localhost:8000)

Login with:
Username: admin
Password: My5tr0ngP@ssw0rd
