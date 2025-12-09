# SQL Server Big Brother – Telemetry & Alerting Pipeline

---
<img width="1238" height="543" alt="image" src="https://github.com/user-attachments/assets/80795665-d509-4a90-8bd5-de561087c37d" />



## Overview
SQL Server Big Brother is a custom telemetry, monitoring, and alerting pipeline designed to provide at-a-glance overall visibility across our entire SQL Server environment.  
This project was built to **complement our team's long-standing, reliable email alerting system** by adding centralized, visual, and actionable monitoring.

As a Jr. DBA at the time, I wanted to deepen my SQL Server, scripting, and automation skills.  
Building this system became both a learning journey and an opportunity to contribute something meaningful to the team.

---

## Why This Was Built

Our existing internally built email alerts served the team well for many years — and were the backbone of our operational awareness.  
But as our SQL Server environment expanded, email-based alerting naturally became harder to manage:

- Buried across hundreds of unrelated emails  
- Hard to correlate    
- Required manual ad-hoc queries  
- No single place to view overall SQL Server health  

As a Jr. DBA learning the environment, I saw an opportunity:  
**Not to replace the email system, but to enhance it** with a centralized, visual telemetry layer that could surface trends and issues faster.

To solve this, I designed and implemented a unified monitoring ecosystem that:

- **Collects telemetry every 5 minutes**  
- **Stores it in a unified SQL monitoring database**  
- **Visualizes insights in Power BI**  
- **Generates automated tickets when issues arise**  

This became the foundation of a complete in-house monitoring platform.

---

## Solution Architecture

### 1. PowerShell & Python Monitoring Scripts (~30+ scripts)
- Runs every 5 minutes  
- Collects telemetry from DEV / TEST / PROD  
- Writes results into central SQL monitoring tables  

### 2. Power BI Dashboard
- Auto-refreshing visuals
- Top layer of conditional format driven cards are clickable 
- 15+ drill-down pages  
- Heat maps, filters, sliders, bar and pie charts 
- Designed for fast triage and pattern detection  

### 3. Automated Ticketing (Freshservice API)
- Reads telemetry tables  
- Creates alert tickets when thresholds are met  
- Covers blocking, backups, disk pressure, job failures, and more  

---

## A Real Issue This System Prevented

One major production issue was avoided thanks to this system:

A disk heat-map visualization showed a production drive slowly shifting from green → yellow — something subtle enough that traditional alerts never triggered.

Investigation revealed unexpected shadow copy growth from a Commvault backup component.  
These files were invisible through normal SQL workflows and would eventually have caused a **critical production outage**.

Because the dashboard surfaced the anomaly early:

- Production downtime was prevented  
- Data loss was avoided  
- Emergency after-hours remediation was avoided  
- The root cause was fixed proactively  

This became one of several early-warning wins delivered by the platform.

---

## What the System Monitors

- SQL Agent job failures  
- Down or unreachable servers  
- Last server restart  
- Noteworthy login failures  
- Lower environment refresh status  
- Blocking, waits, CPU/memory load  
- Azure backup status  
- Logshipping health  
- Missing/failed SQL backups  
- Unusual database states  
- Terminated AD users with active SQL logins  
- Encryption status  
- TempDB health  
- Disk space & low-disk alerting  
- Memory dump detection  
- And more…  

---

## Impact

This project significantly improved operational visibility and reliability by providing:

- Centralized telemetry  
- Real-time dashboards  
- Faster triage and response  
- Automated, auditable alerting  
- Trend analysis across the SQL estate  
- Earlier detection of evolving issues  

What began as a personal learning initiative became a key monitoring tool for the DBA team and broader IT organization.

---

## Tech Stack
- PowerShell  
- Python  
- T-SQL  
- SQL Server  
- Power BI (Desktop and Report Server)  
- Freshservice API  
- Windows Task Scheduler

---
<img width="918" height="1308" alt="image" src="https://github.com/user-attachments/assets/027319b2-36ae-4c15-8446-43f506e03051" />

