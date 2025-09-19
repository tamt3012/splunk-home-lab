# splunk-home-lab

## Overview
This repository documents my **Splunk Home Lab project**, where I simulate a mini Security Operation Center (SOC).
The goal is to gain **hands-on experience with SIEM, log analysis, detection engineering, and threat hunting**, while building a portfolio of real-world skills.

I'll be following a 6-week roadmap that covers everything from Splunk basics to a red team vs. blue team capstone.

---

## Objectives
- Rebuild Splunk fundamentals (searching, field extraction, dashboards).  
- Ingest multiple log sources (Windows, Linux, firewall, simulated attacks).  
- Map detections to **MITRE ATT&CK techniques**.  
- Practice SOC workflows: alerting, incident notes, dashboards.  
- Explore intermediate Splunk admin topics (indexes, lookups, RBAC, performance tuning).  
- Perform hands-on **threat hunting** with custom queries.  
- Simulate attacker activity and detect it.  
- Package everything into a **capstone dashboard + incident report** for my portfolio.  

---

## Roadmap

### Week 1 – Foundations
- Install/set up Splunk
- Ingest logs (Windows, syslog, etc.)
- Learn SPL basics
- Build first dashboards

### Week 2 – Security Data & Use Cases
- Add multiple data sources
- Research MITRE ATT&CK
- Create alerts & detections
- Build Security Operations Overview dashboard

### Week 3 – SOC Analyst Practice
- Ingest sample attack dataset (e.g., Splunk BOTS)
- Identify suspicious patterns
- Build detections
- Write incident notes

### Week 4 – Intermediate Engineering
- Work with indexes, sourcetypes, and retention
- Field extractions (`props.conf`, `transforms.conf`)
- Lookups (e.g., malicious IPs)
- Search performance tuning
- Scheduled reports

### Week 5 – Threat Hunting & Advanced Dashboards
- Hypothesis-driven hunts
- Abnormal behavior queries
- Build hunting dashboards
- Simulate attacks with Kali Linux
- Detect simulated attacks

### Week 6 – Capstone & Wrap-Up
- Red team vs. Blue team scenario
- Final SOC dashboard
- Incident report
- Portfolio documentation
