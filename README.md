# Splunk & SOC Upskilling Guide

A knowledge repository for Security Operations Center (SOC) fundamentals and Security Information and Event Management (SIEM) concepts.

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
      - [2. Supporting Management Components](#2-supporting-management-components)
      - [3. Data Pipeline Stages](#3-data-pipeline-stages)
  - [4. Glossary of Terms](#4-glossary-of-terms)
  - [References \& Image Credits](#references--image-credits)

---

## 1. Security Operations Center (SOC) Overview

![SOC Overview](https://i.vgy.me/s1JmKb.png)

*Source: AT&T Cybersecurity*

### What is a SOC & Key Functions

In the modern age, businesses are highly dependent on digital infrastructure, which enhances business operations but also exposes them to a wide range of cyber threats. With cyberattacks becoming more sophisticated, organizations need a dedicated team to monitor, detect, and respond to security incidents. This is where a Security Operations Center (SOC) comes into play.

A Security Operations Center (SOC) is a centralized unit that deals with security issues on an organizational and technical level. The primary functions of a SOC include:
* **Monitoring:** Continuously observing events in the organization's IT environment for anomalies.
* **Detection:** Identifying potential security incidents through the analysis of logs, alerts, and other data sources.
* **Response:** Taking immediate action to mitigate and contain security incidents, minimizing their impact.
* **Compliance:** Ensuring that the organization adheres to relevant security policies, standards, and regulations.
* **Collaboration:** Working with other departments such as devs, NOC (Network Operations Center), and stakeholders to improve overall security posture and awareness.

### Common Technologies
| Technology / Tool Category | Primary Purpose | Commercial Example | Open Source Example |
| :--- | :--- | :--- | :--- |
| **SIEM** | Centralized security information and event management | Splunk | Wazuh |
| **EDR / XDR** | Endpoint detection and response | CrowdStrike | OpenEDR |
| **SOAR** | Security orchestration, automation, and response | Splunk SOAR | Shuffle |
| **NDR** | Network detection and response | Darktrace | Zeek |
| **Ticketing / ITSM** | Incident management and service desk functionality | ServiceNow SecOps | Zammad |

### SOC Processes
* **Monitoring & Detection:** Observe event logs, alerts, and other data sources to identify potential security incidents.
* **Triage & Analysis:** Evaluate and prioritize incidents based on severity, impact, and urgency.
* **Incident Response & Containment:** Take immediate action to mitigate and contain security incidents, minimizing their impact.
* **Eradication & Recovery:** Remove the root cause of the incident and restore affected systems to normal operation.
* **Post-Incident Review:** Conduct a thorough analysis of the incident to identify lessons learned, improve processes, and prevent future occurrences.

### Challenges & Best Practices

#### Common SOC Challenges
* **Alert Fatigue:** High volume of false positives overwhelms analysts, leading to burn-out and potentially overlooked critical alerts.
* **Skill Shortage & High Turnover:** A global deficit in cybersecurity talent makes recruiting and retaining qualified SOC analysts difficult.
* **Data Overload & Visibility Gaps:** Rapidly expanding hybrid, multi-cloud, and IoT infrastructure makes collecting, ingesting, and making sense of massive log volumes challenging.

#### SOC Best Practices
* **Implement SOAR & Automation:** Automate repetitive Tier 1 triaging tasks, evidence collection, and routine containment playbooks to reduce analyst manual workload.
* **Continuous Rule Tuning & Threat Modeling:** Regularly update, test, and tune SIEM correlation searches and detection logic against frameworks like MITRE ATT&CK to lower false positives.
* **Standardized Playbooks & SOPs:** Maintain clear, up-to-date Incident Response playbooks to ensure quick, consistent, and repeatable responses across the team.

### SOC Roles & Responsibilities
| Role | Core Responsibilities |
| :--- | :--- |
| **Tier 1 Analyst (Triage)** | Monitors incoming security alerts, filters out false positives, performs initial triage/enrichment, and escalates true positives to Tier 2. |
| **Tier 2 Analyst (Incident Response)** | Performs deep-dive investigations on escalated alerts, executes containment and eradication actions, root-cause analysis, and writes post-incident reports. |
| **Tier 3 Analyst (Threat Hunter / Specialist)** | Proactively searches for hidden threats that bypass automated controls, conducts advanced forensics, malware analysis, and advises on defense enhancements. |
| **SOC Manager** | Oversees daily SOC operations, manages budgets and personnel, tracks operational metrics (e.g., MTTD, MTTR), and reports security posture to leadership. |
| **Detection Engineer** | Designs, builds, tests, and tunes SIEM search queries, correlation rules, and automated detections tailored to organization-specific threat models. |

### Threat Hunting & Key Event Types

#### Threat Hunting Overview
Threat hunting is the proactive, human-driven practice of searching through networks, endpoints, and security datasets to detect malicious activity or lingering adversaries that have successfully bypassed automated security controls. Rather than reactively waiting for an alert to trigger, threat hunters formulate hypotheses based on current threat intelligence, recent adversary TTPs (Tactics, Techniques, and Procedures), or anomalous system behavior to catch threats early in the cyber kill chain.

#### Common Event Types to Monitor
* **Authentication Events:** Failed and successful logins, brute-force patterns, impossible travel anomalies, password resets, account lockouts, and MFA bypass attempts (e.g., Windows Event IDs 4624, 4625).
* **Process Creation Events:** Launching command-line utilities (cmd, PowerShell, bash), unexpected parent-child process relationships, and execution from temporary or suspicious directories (e.g., Windows Event ID 4688, Sysmon Event ID 1).
* **Network Connection Logs:** Inbound and outbound connections, non-standard port usage, connections to known malicious C2 (Command and Control) IPs/domains, and unusually large outbound data transfers.
* **DNS Query Logs:** High volumes of failed DNS requests, Domain Generation Algorithm (DGA) patterns, DNS tunneling signatures used for exfiltration, and newly registered domain lookups.
* **Privilege Escalation Activity:** Changes to local/domain administrative groups, privilege assignment modifications, exploitation of vulnerability primitives, and unauthorized `sudo` or run-as execution (e.g., Windows Event IDs 4672, 4720, 4728).

---

## 2. Security Information and Event Management (SIEM)

### What is a SIEM?
A Security Information and Event Management (SIEM) platform is a centralized security solution that collects, aggregates, normalizes, and analyzes log data and security events across an organization's entire IT ecosystem (servers, network appliances, endpoints, applications, and cloud environments). SIEM systems perform real-time correlation to detect potential security incidents, provide centralized dashboard visibility, generate alerts for security teams, and store historical data required for incident investigation and regulatory compliance.

### SIEM Analogy
> 💡 **Analogy:** Think of a SIEM as the master security control room of a busy international airport. Security cameras, metal detectors, baggage scanners, and door access cards continuously send security feeds back to the control room. The SIEM acts as the central intelligence engine that connects these isolated signals—flagging an alert when a person scanned at an outer gate without a verified ticket suddenly triggers a restricted-access door sensor in the hangar five minutes later.

### Modern SIEM Solutions
| Platform | Key Features / Notes |
| :--- | :--- |
| **Splunk Enterprise Security (ES)** | Industry-leading SIEM powered by the Search Processing Language (SPL), high scalability, extensive app ecosystem, and advanced threat detection frameworks. |
| **Microsoft Sentinel** | Cloud-native SIEM and SOAR solution integrated natively into Azure and Microsoft 365, featuring pay-as-you-go pricing, built-in AI, and seamless cloud log ingestion. |
| **Elastic SIEM** | Built on top of the open-source ELK (Elasticsearch, Logstash, Kibana) stack, offering high-speed search performance, flexible deployment, and strong threat hunting capabilities. |
| **Google SecOps (Chronicle)** | Hyper-scalable, cloud-native telemetry platform leveraging Google infrastructure to perform instant log analysis and threat matching across years of historical data. |
| **IBM QRadar** | Enterprise-grade SIEM known for robust network flow analysis, automated risk prioritization, integrated vulnerability management, and compliance auditing tools. |

---

## 3. Splunk Fundamentals

### What is Splunk & Core Use Cases
Splunk is a horizontal data platform designed to collect, index, search, analyze, and visualize unstructured, machine-generated data (logs, metrics, traces, and events) in real time. By converting raw telemetry into searchable indexed data, Splunk provides visibility across enterprise IT and security environments.

* **Security Information & Event Management (SIEM):** Centralizing security telemetry to detect threats, correlate anomalous behavior, perform incident investigations, and manage compliance reporting (via Splunk Enterprise Security).
* **IT Operations & Observability:** Monitoring system health, troubleshooting application performance, tracking uptime, and accelerating Mean Time to Resolution (MTTR).
* **Compliance & Auditing:** Retaining long-term searchable log history required to fulfill regulatory frameworks such as PCI-DSS, HIPAA, GDPR, and ISO 27001.
* **Business Analytics & Operational Intelligence:** Extracting actionable business metrics from raw log streams, such as user journey tracking, e-commerce transaction volumes, and application usage trends.

### The Splunk / SOC Analyst Role
A SOC Analyst uses Splunk as their primary tool for daily threat monitoring, alert investigation, and incident response. Core responsibilities using Splunk include:

* **Querying Data via SPL:** Writing efficiently scoped Search Processing Language (SPL) queries to investigate security events, filter logs, parse field values, and trace attacker activity across datasets.
* **Triage & Notable Event Analysis:** Analyzing triggered alerts within Splunk Enterprise Security (ES) to determine false vs. true positives, evaluate risk scores, and gather context.
* **Building Dashboards & Visualizations:** Creating and customizing real-time dashboards to track ongoing security trends, monitor system health, and surface threat intelligence metrics.
* **Threat Hunting:** Crafting statistical searches and baseline queries to discover anomalous network connections, unusual process launches, or credential abuse that bypassed static detection rules.

### Splunk Deployment Editions
| Edition | Deployment Model | Key Characteristics |
| :--- | :--- | :--- |
| **Splunk Enterprise** | On-Premises / Self-Managed Cloud (AWS, Azure, GCP) | Full organizational control over infrastructure, data location, and retention policies. Requires self-management of hardware, cluster scale, OS, and software updates. |
| **Splunk Cloud Platform** | Fully Managed SaaS | SaaS solution hosted and maintained entirely by Splunk. Eliminates infrastructure management overhead while scaling indexing and search capacity dynamically. |
| **Splunk Free** | On-Premises (Single-instance local install) | Local development/lab environment capped at 500 MB/day indexing volume. Excludes native alerting, user authentication/RBAC, cluster distributed searching, and Splunk ES. |

### Splunk Core Architecture

Splunk processes raw machine data through three main functional tiers and data pipeline stages:

#### 1. Core Architectural Components
* **Forwarders (Data Collection Tier):** Agents installed on remote endpoints or servers to collect raw log data and transmit it to the indexing layer.
  * **Universal Forwarder (UF):** Lightweight agent with minimal resource footprint; handles basic log gathering and forwarding without local data parsing.
  * **Heavy Forwarder (HF):** Full Splunk instance dedicated to pre-parsing, filtering, routing, or transforming data before sending it to indexers.
* **Indexers (Storage & Processing Tier):** Receives incoming raw data, parses and breaks logs into discrete events, extracts timestamp/default fields, writes data to searchable index files on disk (`buckets`), and executes search requests from Search Heads.
* **Search Heads (User Interface & Search Management Tier):** Serves the Splunk Web UI where analysts author SPL queries, run searches, view dashboards, manage knowledge objects, and trigger alerts. Coordinates distributed search queries across multiple Indexers and aggregates their results.

#### 2. Supporting Management Components
* **Deployment Server (DS):** Centralized management server used to push configuration files and app updates to fleets of Universal and Heavy Forwarders.
* **Cluster Manager / Master Node:** Coordinates indexer clustering, manages data replication/search factor compliance, and handles failovers.
* **License Manager (LM):** Enforces overall daily indexing quota across all indexers within a deployment framework.

#### 3. Data Pipeline Stages
![Splunk Data Pipeline Stages](https://i.vgy.me/sjPDJR.jpg)
*Image generated via Google Gemini*

---

## 4. Glossary of Terms

| Term | Category | Definition |
| :--- | :--- | :--- |
| **C2 (Command and Control)** | Cybersecurity | Infrastructure utilized by threat actors to maintain communications with compromised systems inside a target network. |
| **CIM (Common Information Model)** | Splunk | A standardized field-naming schema in Splunk that normalizes data from disparate sources into unified fields (e.g., `src_ip`, `user`). |
| **EDR (Endpoint Detection and Response)** | Security Tools | Software installed on endpoints to monitor process execution, file changes, and network activity to detect and contain threats. |
| **Index** | Splunk | The logical storage container in Splunk where parsed log data and associated search keys reside on disk (`buckets`). |
| **MITRE ATT&CK** | Cybersecurity | A globally accessible knowledge base of adversary tactics, techniques, and procedures (TTPs) based on real-world observations. |
| **MTTD (Mean Time to Detect)** | SOC Metrics | The average time taken by a security team to discover a security incident or breach after it occurs. |
| **MTTR (Mean Time to Respond)** | SOC Metrics | The average time taken by a security team to contain, mitigate, and resolve a detected security incident. |
| **NDR (Network Detection & Response)** | Security Tools | Security technology that monitors network traffic flows and protocols to detect malicious activity or lateral movement. |
| **Notable Event** | Splunk | A high-priority alert generated by correlation rules within Splunk Enterprise Security requiring analyst investigation. |
| **SOAR (Security Orchestration, Automation, and Response)** | Security Tools | Platforms that automate incident triage, evidence collection, and containment workflows across integrated security stacks. |
| **SPL (Search Processing Language)** | Splunk | Splunk's proprietary search language used to query, parse, calculate, and visualize indexed telemetry. |
| **Sysmon (System Monitor)** | Windows | A Windows system service and device driver that logs detailed system activity (e.g., process creation, network connections) to the Windows Event Log. |
| **TTPs (Tactics, Techniques, and Procedures)** | Cybersecurity | The specific patterns of behavior, technical methods, and operational strategies used by cyber adversaries. |

---

## References & Image Credits

* **SOC Overview Diagram:** Image sourced from [AT&T Cybersecurity](https://cybersecurity.att.com/).
* **Splunk Data Pipeline Stages Diagram:** AI-generated via Google Gemini.