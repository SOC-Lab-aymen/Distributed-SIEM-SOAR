
# Distributed-SIEM-Wazuh-using-Treafik

A distributed Security Information and Event Management (SIEM) infrastructure built with Wazuh, Docker, and Traefik.

The project focuses on centralized security monitoring, log collection, event analysis, threat detection, and secure service exposure through a reverse-proxy architecture.

📌 Project Overview

Modern infrastructures generate security events across multiple systems, services, and endpoints. A centralized SIEM allows these events to be collected, analyzed, correlated, and visualized from a single security platform.

This project implements a distributed SIEM architecture using Wazuh as the security monitoring platform and Traefik as the reverse proxy and traffic-management layer.

The infrastructure is designed to provide:

- Centralized security monitoring
- Distributed endpoint monitoring
- Security event collection and analysis
- Log aggregation
- Threat detection
- Custom detection rules
- Centralized visualization
- Secure service exposure
- Containerized deployment
- Infrastructure monitoring


🎯 Objectives

The main objectives of this project are:

- Deploy a centralized Wazuh SIEM platform.
- Monitor multiple Linux and Windows endpoints.
- Collect and analyze security events.
- Create custom detection rules.
- Deploy the infrastructure using Docker.
- Use Traefik as a reverse proxy.
- Secure access to exposed services.
- Monitor the health and availability of the infrastructure.
- Build a scalable architecture that can be extended to SOAR and incident response platforms.

# 🏗️ Architecture
                         ┌───────────────────────┐
                         │       Endpoints       │
                         │                       │
                         │  Windows / Linux      │
                         │  Wazuh Agents         │
                         └───────────┬───────────┘
                                     │
                                     │ Security Events
                                     ▼
                         ┌───────────────────────┐
                         │       Traefik         │
                         │                       │
                         │ Reverse Proxy         │
                         │ Routing               │
                         │ TLS                   │
                         └───────────┬───────────┘
                                     │
                                     ▼
                         ┌───────────────────────┐
                         │    Wazuh Manager      │
                         │                       │
                         │ • Log Collection      │
                         │ • Decoders            │
                         │ • Detection Rules     │
                         │ • Correlation         │
                         │ • Active Response     │
                         └───────────┬───────────┘
                                     │
                                     ▼
                         ┌───────────────────────┐
                         │    Wazuh Indexer      │
                         │                       │
                         │ • Event Storage       │
                         │ • Search              │
                         │ • Indexing            │
                         └───────────┬───────────┘
                                     │
                                     ▼
                         ┌───────────────────────┐
                         │   Wazuh Dashboard     │
                         │                       │
                         │ • Security Events     │
                         │ • Alerts              │
                         │ • Visualization       │
                         │ • Investigation       │
                         └───────────────────────┘

# 🔐 Security Monitoring Workflow

<img width="1983" height="793" alt="ChatGPT Image Aug 31, 2026, 05_56_23 PM" src="https://github.com/user-attachments/assets/70aa2c99-a087-4893-a1d7-37a1ecd85e96" />

# 🧩 Components

# Wazuh

Wazuh is the core security monitoring platform.

It provides:

- Security event monitoring
- Log analysis
- File Integrity Monitoring (FIM)
- Vulnerability detection
- Security configuration assessment
- Threat detection
- Active response
- Endpoint monitoring

# Wazuh Manager

The Wazuh Manager is responsible for processing events received from Wazuh agents.

Main responsibilities include:

- Agent management
- Log analysis
- Decoding
- Rule evaluation
- Alert generation
- Event correlation
- Active response

# Wazuh Indexer

The Wazuh Indexer provides storage and search capabilities for security events and alerts.

It allows security data to be:

- Indexed
- Stored
- Searched
- Analyzed
- Visualized

# Wazuh Dashboard

The Wazuh Dashboard provides the graphical interface used by analysts and administrators.

It can be used for:

- Security monitoring
- Alert investigation
- Endpoint monitoring
- Threat hunting
- Compliance monitoring
- Security visualization
  
# Traefik

Traefik is deployed as the reverse proxy and traffic-management layer.

It provides:

Reverse proxy functionality
HTTP routing
Service discovery
TLS termination
Middleware
Container-aware routing
Centralized service exposure

Example architecture:
<img width="2875" height="1501" alt="traefik-architecture" src="https://github.com/user-attachments/assets/bada0108-8eb5-4e3b-a57c-823f748451dc" />

# 🐳 Containerization

The infrastructure is deployed using Docker.

Example container architecture:

<img width="3120" height="1200" alt="docker-container-architecture" src="https://github.com/user-attachments/assets/767a1888-e720-4164-9435-a19536e51a24" />

Docker provides:

- Isolation
- Reproducible deployment
- Simplified management
- Service portability
- Easy infrastructure recovery

# 📁 Repository Structure
```
distributed-siem-wazuh-traefik/
│
├── README.md
│
├── docs/
│   ├── architecture/
│   │   ├── architecture.drawio
│   │   ├── architecture.png
│   │  
│   │
│   ├── deployment/
│   │   ├── installation.md
│   │   ├── configuration.md
│   │
│   └── screenshots/
│
├── docker/
│   ├── docker-compose.yml
│   │
│   ├── traefik/
│   │   ├── traefik.yml
│   │   └── dynamic/
│   │       ├── routers.yml
│   │       └── middlewares.yml
│   │
│   └── wazuh/
│       ├── manager/
│       ├── indexer/
│       └── dashboard
│
└── examples/
    ├── failed-login.md
    ├── ssh-bruteforce.md
    └── malware-detection.md








                         
