# 🛡️ Distributed SIEM & SOAR Security Platform

<p align="center">
  <strong>Centralized Security Monitoring • Automated Incident Response • SOAR</strong>
</p>

<p align="center">
  A containerized cybersecurity platform combining SIEM, SOAR, incident management, IOC enrichment and real-time notifications.
</p>

<p align="center">

![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge\&logo=docker\&logoColor=white)
![Wazuh](https://img.shields.io/badge/Wazuh-SIEM-4C9AFF?style=for-the-badge)
![Traefik](https://img.shields.io/badge/Traefik-Reverse%20Proxy-24A1C1?style=for-the-badge\&logo=traefik\&logoColor=white)
![Shuffle](https://img.shields.io/badge/Shuffle-SOAR-6C5CE7?style=for-the-badge)
![TheHive](https://img.shields.io/badge/TheHive-Incident%20Response-F5A623?style=for-the-badge)
![Cortex](https://img.shields.io/badge/Cortex-Analyzer-E74C3C?style=for-the-badge)

</p>

---

## 🔎 Overview

This project implements a **distributed and centralized SIEM/SOAR platform** designed to monitor infrastructure, detect security threats, automate incident-response workflows, enrich indicators of compromise, and notify security teams in real time.

The platform combines several open-source cybersecurity technologies into a unified security workflow:

```text
┌──────────────────────────────────────────────────────────────┐
│                    SECURITY OPERATIONS                       │
└──────────────────────────────┬───────────────────────────────┘
                               │
                               ▼
                     ┌─────────────────┐
                     │     WAZUH       │
                     │      SIEM       │
                     └────────┬────────┘
                              │
                         Security Alert
                              │
                              ▼
                     ┌─────────────────┐
                     │     TRAEFIK     │
                     │ Reverse Proxy   │
                     └────────┬────────┘
                              │
                              ▼
                     ┌─────────────────┐
                     │     SHUFFLE     │
                     │      SOAR       │
                     └───────┬─┬───────┘
                             │ │
              ┌──────────────┘ └──────────────┐
              ▼                               ▼
       ┌──────────────┐                ┌──────────────┐
       │    CORTEX    │                │   THEHIVE    │
       │ IOC Analysis │                │ Case Mgmt.   │
       └──────┬───────┘                └──────┬───────┘
              │                               │
              └──────────────┬────────────────┘
                             ▼
                  ┌─────────────────────┐
                  │     NOTIFICATION    │
                  │                     │
                  │   Discord / Email   │
                  └─────────────────────┘
```

---

# 🎯 Objectives

The platform was designed to achieve the following objectives:

| Objective               | Description                                                 |
| ----------------------- | ----------------------------------------------------------- |
| 🔍 **Monitoring**       | Collect and monitor security events from multiple endpoints |
| 🚨 **Detection**        | Detect suspicious and malicious activities                  |
| 🧠 **Correlation**      | Analyze events using Wazuh rules and detection logic        |
| ⚙️ **Automation**       | Automate repetitive incident-response tasks                 |
| 🔬 **Enrichment**       | Analyze suspicious indicators using Cortex                  |
| 📋 **Investigation**    | Manage incidents through TheHive                            |
| 📢 **Notification**     | Deliver alerts through Discord and Email                    |
| 🐳 **Containerization** | Deploy the infrastructure using Docker                      |
| 🔐 **Secure Access**    | Use Traefik as a controlled reverse-proxy entry point       |

---

# 🏗️ Architecture

The architecture follows a **distributed collection + centralized analysis + automated response** model.

### Infrastructure Layer

```text
                         MONITORED INFRASTRUCTURE

       ┌────────────┐     ┌────────────┐     ┌────────────┐
       │ Windows PC │     │ Linux Host │     │   Server   │
       │  Wazuh     │     │   Wazuh    │     │   Wazuh    │
       │   Agent    │     │   Agent    │     │   Agent    │
       └─────┬──────┘     └─────┬──────┘     └─────┬──────┘
             │                  │                  │
             └──────────────────┼──────────────────┘
                                │
                                ▼
                     ┌─────────────────────┐
                     │   CENTRAL WAZUH     │
                     │      MANAGER        │
                     └─────────────────────┘
```

### SOC / Automation Layer

```text
                         ┌──────────────────┐
                         │      WAZUH       │
                         │       SIEM       │
                         └────────┬─────────┘
                                  │
                                  │ Alert
                                  ▼
                         ┌──────────────────┐
                         │     TRAEFIK      │
                         │  Secure Gateway  │
                         └────────┬─────────┘
                                  │
                                  ▼
                         ┌──────────────────┐
                         │     SHUFFLE      │
                         │       SOAR       │
                         └───────┬───┬──────┘
                                 │   │
                    ┌────────────┘   └────────────┐
                    ▼                             ▼
             ┌──────────────┐              ┌──────────────┐
             │    CORTEX    │              │   THEHIVE    │
             │   Analysis   │              │   Incident   │
             │  & Enrich.   │              │   Management │
             └──────────────┘              └───────┬──────┘
                                                   │
                                                   ▼
                                      ┌──────────────────────┐
                                      │     NOTIFICATION     │
                                      │                      │
                                      │  Discord  •  Email  │
                                      └──────────────────────┘
```

---

# 🔄 Automated Incident Response

The main workflow transforms a raw security event into an automated incident-response process.

```text
  ┌─────────────┐
  │   EVENT     │
  │  Detected   │
  └──────┬──────┘
         │
         ▼
  ┌─────────────┐
  │    WAZUH    │
  │   Detect    │
  └──────┬──────┘
         │
         │ Alert
         ▼
  ┌─────────────┐
  │   SHUFFLE   │
  │  Automate   │
  └──────┬──────┘
         │
         ├─────────────────────┐
         │                     │
         ▼                     ▼
  ┌─────────────┐       ┌─────────────┐
  │   CORTEX    │       │   THEHIVE   │
  │ IOC Enrich. │       │ Create Case │
  └──────┬──────┘       └──────┬──────┘
         │                     │
         └──────────┬──────────┘
                    │
                    ▼
            ┌───────────────┐
            │   NOTIFY SOC  │
            └───────┬───────┘
                    │
             ┌──────┴──────┐
             ▼             ▼
        ┌─────────┐   ┌─────────┐
        │ Discord │   │  Email  │
        └─────────┘   └─────────┘
```

---

# 🧩 Technology Stack

## 🛡️ Wazuh — SIEM

Wazuh is the core security monitoring and detection component.

### Responsibilities

* Endpoint monitoring
* Log collection
* Security event analysis
* File Integrity Monitoring
* Vulnerability detection
* Security rule processing
* Alert generation
* Agent management

```text
Endpoints
    │
    ▼
Wazuh Agents
    │
    ▼
Wazuh Manager
    │
    ▼
Security Alerts
```

---

## 🚦 Traefik — Reverse Proxy

Traefik provides the controlled network entry point for services.

### Responsibilities

* Reverse proxy
* HTTP/HTTPS routing
* Service discovery
* Centralized access point
* Secure service exposure
* Internal service routing

```text
                    Internet / LAN
                          │
                          ▼
                   ┌────────────┐
                   │  TRAEFIK   │
                   └─────┬──────┘
                         │
              ┌──────────┼──────────┐
              ▼          ▼          ▼
           Wazuh      Shuffle    TheHive
```

---

## ⚙️ Shuffle — SOAR

Shuffle is responsible for **Security Orchestration, Automation and Response**.

It connects the different security components and executes automated workflows.

### Example

```text
Wazuh Alert
     │
     ▼
Shuffle Webhook
     │
     ▼
Parse Alert
     │
     ▼
Extract IOC
     │
     ▼
Cortex
     │
     ▼
Enrichment
     │
     ▼
TheHive
     │
     ▼
Notification
```

---

## 🐝 TheHive — Incident Management

TheHive provides centralized security case management.

It allows analysts to:

* Create cases
* Track incidents
* Store observables
* Assign investigation tasks
* Document findings
* Track incident status
* Centralize investigation information

---

## 🔬 Cortex — Observable Analysis

Cortex is used to analyze and enrich suspicious observables.

Potential observables include:

* IP addresses
* Domains
* URLs
* File hashes
* Other indicators of compromise

```text
             Observable
                  │
                  ▼
             ┌─────────┐
             │ CORTEX  │
             └────┬────┘
                  │
          ┌───────┼───────┐
          ▼       ▼       ▼
        IP       Hash    Domain
          │       │       │
          └───────┼───────┘
                  ▼
             Enrichment
                  │
                  ▼
              TheHive
```

---

# 📢 Notifications

The platform supports multiple notification channels.

### Discord

Real-time security notifications can be sent to a dedicated security channel.

Example:

```text
🚨 SECURITY ALERT

Severity: HIGH
Rule: Multiple SSH Authentication Failures
Host: Ubuntu-Server
Source IP: 192.168.X.X

Status: Investigation Created
TheHive: Case #12345
```

### Email

Email notifications can provide a more formal incident notification containing:

* Alert severity
* Detection rule
* Affected host
* Source information
* Observable details
* Investigation status
* TheHive case reference

---

# 🐳 Docker Architecture

The platform is containerized using **Docker / Docker Compose**.

```text
                    Docker Host
                         │
       ┌─────────────────┼──────────────────┐
       │                 │                  │
       ▼                 ▼                  ▼
   ┌────────┐       ┌──────────┐      ┌──────────┐
   │ Wazuh  │       │ Traefik  │      │ Shuffle  │
   │ Stack  │       │          │      │  SOAR    │
   └────────┘       └──────────┘      └──────────┘
                                            │
                                    ┌───────┴───────┐
                                    ▼               ▼
                               ┌─────────┐     ┌─────────┐
                               │ TheHive │     │ Cortex  │
                               └─────────┘     └─────────┘
```

---

# 📂 Project Structure

```text
siem-soar-platform/
│
├── 📄 README.md
│
├── 📁 wazuh/
│   ├── docker-compose.yml
│   ├── config/
│   └── rules/
│
├── 📁 traefik/
│   ├── docker-compose.yml
│   ├── dynamic/
│   └── certs/
│
├── 📁 shuffle/
│   ├── docker-compose.yml
│   └── workflows/
│
├── 📁 thehive/
│   ├── docker-compose.yml
│   └── config/
│
├── 📁 cortex/
│   ├── docker-compose.yml
│   └── config/
│
├── 📁 workflows/
│   ├── wazuh-to-shuffle.json
│   ├── cortex-enrichment.json
│   └── notification.json
│
├── 📁 diagrams/
│   ├── architecture.drawio
│   └── architecture.png
│
└── 📁 docs/
    ├── installation.md
    ├── configuration.md
    ├── workflows.md
    └── troubleshooting.md
```

---

# 🧪 Example Use Case

## SSH Brute-Force Detection

Consider a Linux server receiving multiple failed SSH authentication attempts.

### Detection

```text
Attacker
   │
   │ SSH Attempts
   ▼
Linux Server
   │
   │ Authentication Logs
   ▼
Wazuh Agent
   │
   ▼
Wazuh Manager
   │
   │ Detection Rule
   ▼
Security Alert
```

### Automated Response

```text
Security Alert
      │
      ▼
   Shuffle
      │
      ├──────────────► Extract Source IP
      │
      ▼
    Cortex
      │
      └──────────────► Analyze IP
                            │
                            ▼
                       Enrichment
                            │
                            ▼
                         TheHive
                            │
                            ├──────► Create Case
                            │
                            ▼
                       Notification
                       ┌────┴────┐
                       ▼         ▼
                    Discord    Email
```

### Result

A single suspicious event can automatically trigger:

**Detection → Analysis → Enrichment → Case Creation → Notification**

with minimal manual intervention.

---

# 📊 Platform Capabilities

```text
┌────────────────────────────────────────────────────┐
│                 SECURITY PLATFORM                  │
├────────────────────────────────────────────────────┤
│                                                    │
│  🔍 Monitoring              Wazuh                  │
│  🚨 Detection               Wazuh                  │
│  🔗 Integration             Traefik / APIs        │
│  ⚙️ Automation              Shuffle                │
│  🔬 IOC Enrichment          Cortex                 │
│  🐝 Incident Management     TheHive                │
│  📢 Notifications           Discord / Email        │
│  🐳 Deployment              Docker                 │
│                                                    │
└────────────────────────────────────────────────────┘
```

---

# 🔐 Security Design Principles

The platform is based on several security principles:

### Centralized Security Monitoring

Security events from multiple endpoints are collected and analyzed centrally.

### Distributed Endpoint Collection

Wazuh agents can be deployed across multiple operating systems and infrastructure components.

### Automated Response

Shuffle minimizes repetitive manual operations by orchestrating security workflows.

### Indicator Enrichment

Cortex provides additional context about suspicious observables.

### Centralized Case Management

TheHive provides a structured environment for incident investigation and tracking.

### Controlled Service Exposure

Traefik acts as a centralized reverse-proxy layer rather than exposing every application directly.

### Multi-Channel Alerting

Security teams can receive notifications through multiple communication channels.

---

# 📈 Future Improvements

The platform can be extended with:

* 🔥 Automated firewall blocking
* 🚫 Automated IP blocking
* 🧠 Additional threat-intelligence integrations
* 🛡️ EDR integration
* 🔎 Advanced Wazuh detection rules
* 🤖 More advanced SOAR workflows
* 🗺️ MITRE ATT&CK mapping
* 📊 SOC dashboards
* 🔐 TLS for internal services
* 💾 Automated backups
* ♻️ Disaster recovery
* 📈 High-availability architecture
* 📡 Additional notification channels

---

# 🧠 Security Operations Workflow

The complete concept can be summarized as:

```text
                  ┌─────────────┐
                  │   MONITOR   │
                  └──────┬──────┘
                         ▼
                  ┌─────────────┐
                  │   DETECT    │
                  │    WAZUH    │
                  └──────┬──────┘
                         ▼
                  ┌─────────────┐
                  │   ROUTE     │
                  │   TRAEFIK   │
                  └──────┬──────┘
                         ▼
                  ┌─────────────┐
                  │  AUTOMATE   │
                  │   SHUFFLE   │
                  └──────┬──────┘
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
       ┌─────────────┐       ┌─────────────┐
       │   ANALYZE   │       │  INVESTIGATE│
       │   CORTEX    │       │   THEHIVE   │
       └──────┬──────┘       └──────┬──────┘
              │                     │
              └──────────┬──────────┘
                         ▼
                  ┌─────────────┐
                  │   NOTIFY    │
                  └──────┬──────┘
                         │
                    ┌────┴────┐
                    ▼         ▼
                 Discord     Email
```

---

# 🏁 Project Goal

The goal of this project is to build a **centralized, automated and scalable security operations platform** capable of transforming raw security events into actionable incidents.

The combination of:

> **SIEM + SOAR + IOC Enrichment + Incident Management + Automated Notifications**

creates a complete security monitoring and incident-response workflow suitable for a **SOC laboratory, academic project, cybersecurity research environment, or small-scale production deployment**.

---

# 🛠️ Technologies

| Category            | Technology          |
| ------------------- | ------------------- |
| 🛡️ SIEM            | Wazuh               |
| 🚦 Reverse Proxy    | Traefik             |
| ⚙️ SOAR             | Shuffle             |
| 🐝 Case Management  | TheHive             |
| 🔬 Analysis         | Cortex              |
| 📢 Notification     | Discord / Email     |
| 🐳 Containerization | Docker              |
| 📦 Orchestration    | Docker Compose      |
| 🔗 Integration      | REST API / Webhooks |
| 🐧 Infrastructure   | Linux               |

---

<p align="center">

### 🛡️ Detect • Automate • Investigate • Respond

**Distributed SIEM & SOAR Security Platform**

</p>
