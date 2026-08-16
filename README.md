# Splunk & SOC Upskilling Guide

My working reference notes for SOC operations, threat monitoring, and Splunk usage.

---

## Table of Contents
- [Splunk \& SOC Upskilling Guide](#splunk--soc-upskilling-guide)
  - [Table of Contents](#table-of-contents)
  - [1. Security Operations Center (SOC) Overview](#1-security-operations-center-soc-overview)
    - [What is a SOC \& Key Functions](#what-is-a-soc--key-functions)
    - [Common Technologies](#common-technologies)
    - [SOC Processes](#soc-processes)
    - [Challenges \& Best Practices](#challenges--best-practices)
      - [Common SOC Challenges](#common-soc-challenges)
      - [SOC Best Practices](#soc-best-practices)
    - [SOC Roles \& Responsibilities](#soc-roles--responsibilities)
    - [Threat Hunting \& Key Event Types](#threat-hunting--key-event-types)
      - [Threat Hunting Overview](#threat-hunting-overview)
      - [Common Event Types to Monitor](#common-event-types-to-monitor)
  - [2. Security Information and Event Management (SIEM)](#2-security-information-and-event-management-siem)
    - [What is a SIEM?](#what-is-a-siem)
    - [SIEM Analogy](#siem-analogy)
    - [Modern SIEM Solutions](#modern-siem-solutions)
  - [3. Splunk Fundamentals](#3-splunk-fundamentals)
    - [What is Splunk \& Core Use Cases](#what-is-splunk--core-use-cases)
    - [The Splunk / SOC Analyst Role](#the-splunk--soc-analyst-role)
    - [Splunk Deployment Editions](#splunk-deployment-editions)
    - [Splunk Core Architecture](#splunk-core-architecture)
      - [1. Core Architectural Components](#1-core-architectural-components)
      - [2. Supporting Architecture Nodes](#2-supporting-architecture-nodes)
      - [3. Data Pipeline Stages](#3-data-pipeline-stages)
  - [References \& Image Credits](#references--image-credits)

---

## 1. Security Operations Center (SOC) Overview

<div align="center">
  <img src="https://i.vgy.me/s1JmKb.png" alt="SOC Overview">
  <br>
  <i>Source: AT&T Cybersecurity</i>
</div>

<br>

<details>
<summary><b>📖 Glossary: Section 1 Key Terms</b></summary>
<br>

| Term | Category | Definition |
| :--- | :--- | :--- |
| **C2 (Command and Control)** | Cybersecurity | External servers infrastructure operated by attackers to send commands to malware on compromised internal networks. |
| **EDR (Endpoint Detection and Response)** | Security Tools | Endpoint agents that log process trees, file writes, and registry changes while allowing remote host isolation. |
| **MITRE ATT&CK** | Cybersecurity | Matrix mapping real-world adversary behavior into specific tactics (goals) and techniques (methods). |
| **MTTD (Mean Time to Detect)** | SOC Metrics | How long it takes from initial compromise/event to an alert being opened. |
| **MTTR (Mean Time to Respond)** | SOC Metrics | How long it takes from initial alert creation to containment and resolution. |
| **NDR (Network Detection & Response)** | Security Tools | Inline or tap devices analyzing raw network traffic and protocol behavior for anomalies. |
| **SOAR (Security Orchestration, Automation, and Response)** | Security Tools | Middleware used to run automated playbooks (e.g., auto-blocking an IP or querying VirusTotal). |
| **Sysmon (System Monitor)** | Windows | Free Sysinternals tool providing granular Windows event logging (especially Process Creation - Event ID 1). |
| **TTPs (Tactics, Techniques, and Procedures)** | Cybersecurity | The specific patterns, tools, and operational methods used by threat actors. |

</details>

### What is a SOC & Key Functions

Modern IT environments are messy, highly targeted, and constantly changing. A SOC is simply the dedicated team responsible for keeping an eye on the enterprise network and responding when things go sideways.

Core responsibilities break down to:
* **Monitoring:** Eye-on-glass tracking of system alerts and unusual telemetry.
* **Detection:** Parsing logs and building alerts to catch suspicious activity before it causes real damage.
* **Response:** Containing infected hosts, killing malicious processes, and revoking compromised credentials.
* **Compliance:** Keeping log retention aligned with frameworks like PCI-DSS, ISO 27001, or local laws.
* **Collaboration:** Working with SysAdmins, DevOps, and NOC teams during major outages or security incidents.

### Common Technologies
| Technology / Tool Category | Primary Purpose | Commercial Example | Open Source Example |
| :--- | :--- | :--- | :--- |
| **SIEM** | Log centralization & correlation | Splunk | Wazuh |
| **EDR / XDR** | Host-level process/file telemetry | CrowdStrike | OpenEDR |
| **SOAR** | Automated playbooks & API actions | Splunk SOAR | Shuffle |
| **NDR** | Network packet & flow analysis | Darktrace | Zeek |
| **Ticketing / ITSM** | Case management & incident records | ServiceNow | Zammad |

### SOC Processes
* **Monitoring & Triage:** Review incoming alerts, check basic context (user, IP, host), and filter out obvious false positives.
* **Investigation:** Deep dive into EDR logs, network captures, and user history to confirm true positive status.
* **Containment:** Isolate the endpoint from the network, block C2 IPs at the firewall, and lock user accounts.
* **Eradication & Recovery:** Clean malware off the host (or re-image it) and restore operations from clean backups.
* **Post-Incident Review:** Figure out what broke, why the detection didn't trigger earlier, and tune rules.

> 💡 **Analyst Note:** Re-imaging a machine is almost always faster and safer than manually trying to clean deep malware persistence.

### Challenges & Best Practices

#### Common SOC Challenges
* **Alert Fatigue:** SOCs get bombarded with thousands of low-fidelity alerts daily. Analysts burn out, and real incidents get missed in the noise.
* **Noisy Default Rules:** Out-of-the-box SIEM rules are rarely tuned for specific environments, leading to constant false alarms on routine admin work.
* **Visibility Blind Spots:** Unmonitored cloud environments, legacy servers without EDR agents, and unmanaged BYOD devices.

#### SOC Best Practices
* **Filter at Ingestion:** Don't waste SIEM licensing costs or storage indexing debug logs or useless health checks.
* **Tie Alerts to Frameworks:** Map detections directly to MITRE ATT&CK techniques so analysts know *what* stage of an attack they are looking at.
* **Automate Routine Enrichment:** Use SOAR or scripts to automatically run IP lookups on AbuseIPDB or VirusTotal before a human even opens the ticket.

### SOC Roles & Responsibilities
| Role | Core Responsibilities |
| :--- | :--- |
| **Tier 1 Analyst** | Front-line triage. Filters out noise, performs initial context checks, and escalates legit alerts. |
| **Tier 2 Analyst** | Handles escalations. Conducts deep host/network forensics, executes containment, and writes root-cause reports. |
| **Tier 3 Analyst / Threat Hunter** | Proactively searches for silent threats that bypassed alerts. Does reverse engineering and advanced hunting. |
| **Detection Engineer** | Writes, tests, and tunes SIEM rules/SPL queries to catch new attacker techniques without spamming Tier 1. |
| **SOC Manager** | Handles operations, staffing schedules, metric tracking (MTTD/MTTR), and reports status to management. |

### Threat Hunting & Key Event Types

#### Threat Hunting Overview
Threat hunting isn't waiting for a dashboard alert to go off. It's starting with a hypothesis - like *"What if an attacker is abusing PowerShell on our domain controllers?"* - and manually digging through baseline dataset logs to find stealthy activity that bypassed automated signatures.

#### Common Event Types to Monitor
* **Authentication Events:** Logins, failed attempts, impossible travel, and password resets.
  * *Windows:* Event ID `4624` (Successful Logon), `4625` (Failed Logon: watch for brute force spikes).
* **Process Creation Events:** Spawning command-line tools or unusual parent-child process chains.
  * *Examples:* `cmd.exe` spawning from `word.exe`, or `powershell.exe` running encoded commands (`-enc`).
  * *Windows/Sysmon:* Event ID `4688` or Sysmon Event ID `1`.
* **Network Connection Logs:** High-volume outbound transfers, odd port usage, or direct connection attempts to known bad external IPs.
* **DNS Query Logs:** Massive spikes in failed lookups, long random subdomain strings (DNS tunneling/exfiltration), or connections to freshly registered domains.
* **Privilege Changes:** Account additions to privileged groups.
  * *Windows:* Event ID `4728` or `4732` (Member added to security group).

---

## 2. Security Information and Event Management (SIEM)

<details>
<summary><b>📖 Glossary: Section 2 Key Terms</b></summary>
<br>

| Term | Category | Definition |
| :--- | :--- | :--- |
| **Log Normalization** | SIEM Concepts | Mapping different log fields across platforms to standard names (e.g., making sure Windows, Linux, and Firewall logs all use `src_ip`). |
| **Real-Time Correlation** | SIEM Concepts | Combining multiple log sources through logic rules to flag suspicious combinations (e.g., Failed VPN Login + Success 2 seconds later from another country). |
| **SIEM** | Security Tools | Central system for ingesting, parsing, indexing, and alerting on log telemetry across the organization. |

</details>

### What is a SIEM?
A SIEM is the central bucket where logs from endpoints, firewalls, domain controllers, and cloud environments get dumped, parsed, and searched. Its main job is to turn massive, unreadable streams of raw text logs into actionable alerts and searchable data during an investigation.

### SIEM Analogy
> 💡 **Analogy:** Think of a SIEM like a city-wide CCTV control room. A camera at a bank, a door sensor at a warehouse, and an emergency call from a street corner are all separate feeds. The SIEM is the software linking them together - flagging an alert when the same license plate triggers an alarm at the warehouse and shows up outside the bank 3 minutes later.

### Modern SIEM Solutions
| Platform | Key Features / Notes |
| :--- | :--- |
| **Splunk Enterprise Security (ES)** | Industry standard. Uses SPL. Extremely flexible for large enterprise environments, but can get expensive on high daily log volumes. |
| **Microsoft Sentinel** | Cloud-native SIEM built into Azure. Easy integration with M365 logs, pay-as-you-go pricing model. |
| **Elastic SIEM** | Built on ELK stack. Fast search performance, popular in tech-heavy teams wanting open/flexible architectures. |
| **Google SecOps (Chronicle)** | Handles massive historical data volumes fast. Good for searching years of back logs without waiting for searches to time out. |
| **IBM QRadar** | Legacy enterprise favorite. Strong built-in network flow analysis, though setup and tuning can be heavy. |

---

## 3. Splunk Fundamentals

<details>
<summary><b>📖 Glossary: Section 3 Key Terms</b></summary>
<br>

| Term | Category | Definition |
| :--- | :--- | :--- |
| **Bucket** | Splunk | Directories on Indexers containing indexed data files (`Hot`, `Warm`, `Cold`, `Frozen` based on age). |
| **CIM (Common Information Model)** | Splunk | Splunk's field-normalization standard to ensure queries work across multiple vendor formats. |
| **Forwarder** | Splunk Architecture | Lightweight agent installed on endpoints to ship logs to Indexers (UF = raw shipping, HF = parses before shipping). |
| **Indexer** | Splunk Architecture | The worker node that receives, parses, indexes, and stores raw data on disk. |
| **Notable Event** | Splunk | A flagged security incident generated inside Splunk Enterprise Security that requires analyst review. |
| **Search Head** | Splunk Architecture | The web interface server where users run SPL queries, view dashboards, and dispatch jobs to Indexers. |
| **SPL (Search Processing Language)** | Splunk | Splunk's query language used to search, filter, parse, and transform log data. |

</details>

### What is Splunk & Core Use Cases
At its core, Splunk is a massive search engine for text files and system logs. It ingests unstructured data, breaks it into events, and lets you query it in real time using SPL.

Common practical uses:
* **Security / SIEM:** Correlation rules, threat investigations, tracking attacker movements across endpoints.
* **IT Operations:** Monitoring host performance, tracking application error logs, keeping uptime dashboards.
* **Compliance:** Keeping logs searchable for required retention windows (1–7 years depending on regulations).

### The Splunk / SOC Analyst Role
As a SOC Analyst, you'll spend a huge chunk of your day inside Splunk. Typical tasks include:

* Writing SPL queries to investigate host activity or narrow down timeframes.
* Checking the **Incident Review** dashboard in Splunk ES for new Notable Events.
* Building simple dashboard panels for management or SOC display screens.
* Writing hunting queries to look for suspicious process executions or rare network destinations.

### Splunk Deployment Editions
| Edition | Deployment Model | Key Characteristics |
| :--- | :--- | :--- |
| **Splunk Enterprise** | On-Prem / Private Cloud | Full control over indexer placement and storage retention. Requires manual OS/app maintenance. |
| **Splunk Cloud Platform** | SaaS | Managed infrastructure. No hardware management required, scales automatically. |
| **Splunk Free** | Local install | 500 MB/day limit. Great for local testing/home labs, but lacks user authentication, alerts, or cluster features. |

### Splunk Core Architecture

Splunk breaks down into three core roles:

#### 1. Core Architectural Components
* **Forwarders:** Installed on endpoints to grab raw logs and ship them out.
  * *Universal Forwarder (UF):* Very lightweight. Just collects data and ships it without processing.
  * *Heavy Forwarder (HF):* Full Splunk instance used to parse, filter, or route data before hitting the indexers.
* **Indexers:** The workhorses. They receive logs from forwarders, parse raw text into fields, write data to disk (`buckets`), and execute searches dispatched by Search Heads.
* **Search Heads:** The UI tier. Where analysts log in, write SPL queries, view dashboards, and manage alerts.

#### 2. Supporting Architecture Nodes
* **Deployment Server (DS):** Manages configuration files (`inputs.conf`, `outputs.conf`) and pushes updates to fleets of forwarders.
* **Cluster Manager:** Controls indexer clusters to ensure data replication and high availability.
* **License Manager:** Tracks total daily ingestion volume across all indexers.

#### 3. Data Pipeline Stages

<div align="center">
  <img src="https://i.vgy.me/sjPDJR.jpg" alt="Splunk Data Pipeline Stages">
  <br>
  <i>Image generated via Google Gemini</i>
</div>

---

## References & Image Credits

* **SOC Overview Diagram:** Image sourced from [AT&T Cybersecurity](https://cybersecurity.att.com/).
* **Splunk Data Pipeline Stages Diagram:** AI-generated via Google Gemini.