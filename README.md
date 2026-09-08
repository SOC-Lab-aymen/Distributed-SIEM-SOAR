# Distributed SIEM & SOAR Security Platform

## 📌 Overview

This project consists of the design and deployment of a **distributed and centralized Security Information and Event Management (SIEM) platform**, combined with **Security Orchestration, Automation and Response (SOAR)** techniques.

The objective is to centralize security event collection, detect and analyze suspicious activities, automate incident response, and notify security administrators through communication channels such as **Discord and Email**.

The platform integrates:

* **Wazuh** — SIEM, HIDS and security monitoring
* **Traefik** — Reverse proxy and secure entry point
* **Shuffle** — SOAR automation and orchestration
* **TheHive** — Security incident and case management
* **Cortex** — Automated analysis and observables enrichment
* **Discord / Email** — Security alert notification

The entire environment is containerized using **Docker and Docker Compose**.

---

## 🎯 Project Objectives

The main objectives of this project are:

* Centralize security logs and events from monitored systems.
* Detect suspicious and malicious activities.
* Correlate security events using Wazuh detection rules.
* Automate incident response using SOAR techniques.
* Reduce manual intervention during incident investigation.
* Create and manage security incidents automatically.
* Enrich security observables through Cortex analyzers.
* Provide security analysts with a centralized incident-management platform.
* Send real-time notifications through Discord and Email.
* Provide a scalable architecture that can be extended with additional endpoints and security tools.

---

## 🏗️ Architecture

The platform follows a **distributed collection with centralized security management** architecture.

```text
                    ┌─────────────────────────┐
                    │       Monitored         │
                    │        Systems          │
                    │                         │
                    │  Linux/Windows/servers  │
                    └────────────┬────────────┘
                                 │
                                 │ Security Events
                                 ▼
                    ┌─────────────────────────┐
                    │         WAZUH           │
                    │                         │
                    │  • Agents               │
                    │  • Log Collection       │
                    │  • Detection Rules      │
                    │  • Correlation          │
                    │  • Alerts               │
                    └────────────┬────────────┘
                                 │
                                 │ Webhook / API
                                 ▼
                    ┌─────────────────────────┐
                    │        TRAEFIK          │
                    │                         │
                    │  Reverse Proxy          │
                    │  Entry Point            │
                    │  Routing                │
                    └────────────┬────────────┘
                                 │
                                 ▼
                    ┌─────────────────────────┐
                    │        SHUFFLE          │
                    │          SOAR           │
                    │                         │
                    │  • Workflows            │
                    │  • Automation           │
                    │  • Orchestration        │
                    │  • API Integrations      │
                    └───────┬─────────┬────────┘
                            │         │
                 ┌──────────┘         └──────────┐
                 ▼                               ▼
       ┌───────────────────┐           ┌───────────────────┐
       │      THEHIVE      │          │      CORTEX      │
       │                   │           │                   │
       │ Incident Cases    │           │ Observable        │
       │ Investigation     │◄──────────│ Analysis          │
       │ Case Management   │           │ Enrichment        │
       └───────────────────┘           └───────────────────┘
                            │
                            ▼
                 ┌─────────────────────┐
                 │      NOTIFICATION   │
                 │                     │
                 │  Discord / Email    │
                 └─────────────────────┘
```

---

## 🔄 Security Incident Workflow

The automated incident-response process follows these main stages:

### 1. Event Collection

Wazuh agents are deployed on monitored endpoints and collect security-related events such as:

* Authentication events
* Failed login attempts
* Process activity
* File integrity changes
* System events
* Malware-related events
* Configuration changes
* Network-related events

The collected events are transmitted to the centralized Wazuh infrastructure.

---

### 2. Detection

Wazuh analyzes incoming events using its detection and correlation capabilities.

When an event matches a detection rule, Wazuh generates a security alert containing information such as:

* Alert ID
* Rule ID
* Rule description
* Severity level
* Timestamp
* Source IP
* Destination information
* Host information
* User information
* Log data

---

### 3. Alert Forwarding

The Wazuh alert is forwarded to the SOAR platform through an **HTTP webhook/API integration**.

Traefik can act as the controlled entry point for services exposed through HTTP/HTTPS.

```text
Wazuh
   │
   │ Alert
   ▼
Traefik
   │
   │ HTTP/HTTPS
   ▼
Shuffle Webhook
```

---

### 4. SOAR Automation

Shuffle receives the Wazuh alert and starts an automated workflow.

The workflow can:

1. Receive the Wazuh alert.
2. Extract relevant observables.
3. Determine the alert severity.
4. Extract IP addresses, domains, hashes or other indicators.
5. Send observables to Cortex.
6. Execute Cortex analyzers.
7. Enrich the security information.
8. Create an incident/case in TheHive.
9. Add the investigation information to the case.
10. Notify the security administrator.

---

## 🧠 SOAR Workflow

A simplified workflow can be represented as:

```text
                 WAZUH ALERT
                      │
                      ▼
               SHUFFLE WEBHOOK
                      │
                      ▼
              Parse Alert Data
                      │
                      ▼
             Extract Observables
                      │
             ┌────────┴────────┐
             │                 │
             ▼                 ▼
        IP / Domain         File Hash
             │                 │
             └────────┬────────┘
                      ▼
                    CORTEX
                      │
              Observable Analysis
                      │
                      ▼
             Enrichment Results
                      │
                      ▼
                  THEHIVE
                      │
                Create Case
                      │
             ┌────────┴────────┐
             │                 │
             ▼                 ▼
          DISCORD            EMAIL
          Alert              Alert
```

---

# 🧩 Components

## Wazuh

Wazuh is the main SIEM and security monitoring component.

It is responsible for:

* Endpoint monitoring
* Log collection
* Security event analysis
* File Integrity Monitoring
* Vulnerability detection
* Security rule processing
* Alert generation
* Endpoint security monitoring

Wazuh agents are installed on monitored systems and communicate with the centralized Wazuh manager.

---

## Traefik

Traefik is used as the **reverse proxy and entry point** for the platform.

Its responsibilities include:

* HTTP/HTTPS routing
* Reverse proxying
* Service discovery
* Centralized entry point
* Secure service exposure
* Routing requests to internal services

Instead of directly exposing every application port, Traefik can provide controlled access to the required services.

---

## Shuffle

Shuffle is the **SOAR component** of the architecture.

It provides automated security workflows and orchestration between the different security tools.

Shuffle can communicate with:

* Wazuh
* TheHive
* Cortex
* Discord
* Email services
* REST APIs
* Other security tools

Example workflow:

```text
Wazuh Alert
     ↓
Shuffle
     ↓
Extract IOC
     ↓
Cortex Analysis
     ↓
TheHive Case
     ↓
Discord / Email Notification
```

---

## TheHive

TheHive is used as the **Security Incident Response and Case Management platform**.

It allows security analysts to:

* Create security cases
* Track incidents
* Organize investigations
* Store observables
* Assign tasks
* Track investigation status
* Centralize incident information

A Wazuh alert can automatically result in the creation of a TheHive case through Shuffle.

---

## Cortex

Cortex is used for **automated observable analysis and enrichment**.

Depending on the configured analyzers, Cortex can analyze observables such as:

* IP addresses
* Domains
* URLs
* File hashes
* Other indicators of compromise

The analysis results can then be returned to Shuffle and associated with the corresponding TheHive case.

---

## Discord

Discord is used as a real-time notification channel.

When a significant security incident is detected, Shuffle can automatically send a notification containing information such as:

```text
🚨 SECURITY ALERT

Severity: High
Rule: Multiple SSH Authentication Failures
Source IP: x.x.x.x
Host: Ubuntu-Server
Time: 2026-09-01 12:30

A security incident has been automatically created
in TheHive.
```

---

## Email

Email provides an additional notification mechanism for security administrators.

The automated workflow can send:

* Alert severity
* Detection rule
* Affected host
* Source IP
* Observable information
* Investigation status
* TheHive case reference

This provides an alternative notification channel when Discord is unavailable or when email is preferred for incident tracking.

---

# 🐳 Containerization

The platform is deployed using **Docker and Docker Compose**.

Example service architecture:

```text
Docker Host
│
├── Wazuh Manager
├── Wazuh Indexer
├── Wazuh Dashboard
│
├── Traefik
│
├── Shuffle
│   ├── Frontend
│   ├── Backend
│   ├── Database
│   └── Orborus
│
├── TheHive
│
└── Cortex
```

Containerization provides:

* Isolation
* Reproducible deployments
* Simplified service management
* Easier maintenance
* Service scalability
* Consistent environments

---

# 🔐 Security Architecture

The architecture is designed around the following security principles:

### Centralized Monitoring

Security events from multiple endpoints are collected and analyzed by a centralized Wazuh infrastructure.

### Distributed Collection

Wazuh agents can be deployed across multiple systems and environments.

```text
Endpoint 1 ─┐
Endpoint 2 ─┤
Endpoint 3 ─┼──► Wazuh Manager
Endpoint 4 ─┤
Endpoint N ─┘
```

### Automated Response

SOAR workflows reduce the amount of manual work required from security analysts.

### Incident Enrichment

Cortex provides additional information about suspicious observables before or during investigation.

### Centralized Incident Management

TheHive provides a structured location for tracking security incidents.

### Multi-Channel Notification

Security alerts can be delivered through:

* Discord
* Email

---

# 📁 Suggested Project Structure

```text
siem-soar-platform/
│
├── README.md
│
├── wazuh/
│   ├── docker-compose.yml
│   ├── config/
│   └── rules/
│
├── traefik/
│   ├── docker-compose.yml
│   
│  
│
├── shuffle/
│   ├── docker-compose.yml
│   └── workflows/
│
├── thehive/
│   ├── docker-compose.yml
│   └── config/
│
├── cortex/
│   ├── docker-compose.yml
│   └── config/
│
├── workflows/
│   ├── wazuh-to-thehive.json
│   ├── cortex-enrichment.json
│   └── notifications.json
│
├── diagrams/
│   ├── architecture.drawio
│   └── architecture.png
│
└── docs/
    ├── installation.md
    ├── configuration.md
    ├── incident-response.md
    └── troubleshooting.md
```

---

# 🚀 Main Use Case

### Example: Brute-Force SSH Detection

A monitored Linux server receives multiple failed SSH authentication attempts.

```text
Attacker
   │
   │ Multiple SSH attempts
   ▼
Linux Server
   │
   │ Logs
   ▼
Wazuh Agent
   │
   ▼
Wazuh Manager
   │
   │ Detection Rule
   ▼
Security Alert
   │
   ▼
Shuffle
   │
   ├── Extract Source IP
   │
   ├── Send IP to Cortex
   │
   ├── Enrich Observable
   │
   ├── Create TheHive Case
   │
   └── Send Notification
          │
          ├── Discord
          └── Email
```

This transforms a raw security event into a structured and automated incident-response process.

---

# 📊 Benefits

The proposed platform provides several advantages:

| Capability             | Solution                |
| ---------------------- | ----------------------- |
| Security monitoring    | Wazuh                   |
| Log collection         | Wazuh Agents            |
| Threat detection       | Wazuh                   |
| Reverse proxy          | Traefik                 |
| SOAR automation        | Shuffle                 |
| Incident management    | TheHive                 |
| IOC enrichment         | Cortex                  |
| Real-time notification | Discord                 |
| Email notification     | Email                   |
| Deployment             | Docker / Docker Compose |

---

# 🛠️ Technologies Used

```text
Docker
Docker Compose
Linux
Wazuh
Traefik
Shuffle
TheHive
Cortex
Discord
Email
HTTP / REST API
Webhooks
SOAR
SIEM
HIDS
Incident Response
Threat Intelligence
```

---

# 📈 Future Improvements

Possible future improvements include:

* Integration with additional threat-intelligence platforms.
* Automated IP blocking.
* Automated firewall actions.
* Integration with EDR solutions.
* Integration with additional notification platforms.
* Advanced Wazuh correlation rules.
* Automated incident severity classification.
* Automated IOC blocking.
* Integration with MITRE ATT&CK techniques.
* High-availability Wazuh deployment.
* Centralized monitoring of multiple infrastructure environments.
* Backup and disaster-recovery mechanisms.
* TLS encryption for internal and external communications.

---

# 👨‍💻 Project Goal

The ultimate goal of this project is to build a **centralized security operations platform capable of detecting, enriching, managing, and responding to security incidents automatically**.

The combination of **SIEM + SOAR + Incident Response + Automated Enrichment + Multi-Channel Notification** provides a foundation for a small-scale **Security Operations Center (SOC)** environment.

```text
        DETECT
          │
          ▼
       WAZUH
          │
          ▼
      AUTOMATE
          │
          ▼
      SHUFFLE
       │     │
       ▼     ▼
    CORTEX  THEHIVE
       │     │
       └──┬──┘
          ▼
       NOTIFY
       │     │
       ▼     ▼
    DISCORD EMAIL
```

---

## 📜 License

This project is intended for educational, research, and cybersecurity laboratory purposes.
