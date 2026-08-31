Standalone installation guide

This document contains two independent Docker installations:


1.Wazuh standalone installation with the Wazuh Manager, Wazuh Indexer, and Wazuh Dashboard.


2.Traefik standalone installation as an independent reverse-proxy project.

There is no integration between the two installations in this guide. They use separate directories, separate Compose projects, and separate Docker resources.


Important: Choose one Wazuh release and use the matching certificate tool, configuration template, and container image tags. Do not mix files from different Wazuh releases.




# Part 1 — Install Wazuh on Docker

Wazuh officially supports a single-node Docker stack consisting of one Wazuh Manager, one Wazuh Indexer, and one Wazuh Dashboard container .

1. Wazuh prerequisites

Install Docker Engine and the Docker Compose plugin by following the official . The host should also have Git, curl, OpenSSL, and a modern web browser.

The Wazuh Docker requirements documentation recommends at least 2 CPU cores, 4 GB RAM for a small test deployment, and 50 GB of available disk space. A more comfortable small environment should use additional memory and SSD-backed storage .

On Linux, configure the kernel parameter required by the Wazuh Indexer:

Bash


sudo sysctl -w vm.max_map_count=262144



To make the setting persistent, add this line to /etc/sysctl.conf:

Plain Text


vm.max_map_count=262144



Then reload the configuration:

Bash


sudo sysctl -p



# 2. Download the official Wazuh Docker repository

Create a dedicated directory and clone the official Wazuh Docker repository:

Bash


mkdir -p ~/wazuh-docker
cd ~/wazuh-docker
git clone https://github.com/wazuh/wazuh-docker.git .



Select the Wazuh release directory that matches the version you intend to deploy. The official repository contains the Docker Compose definitions and supporting deployment documentation .

For a single-node deployment, the relevant Compose definition is the official single-node stack. If you are using this project’s Compose file instead, run it as a separate Compose project and do not combine it with the standalone Traefik project described later in this document.

# 3. Generate Wazuh certificates

Wazuh uses certificates for secure communication between the Manager, Indexer, and Dashboard. From the directory containing the single-node Compose file, download the certificate tool and matching configuration template.

The following example uses Wazuh 5.1.0. Replace the version in both URLs when using another release:

Bash


cd ~/wazuh-docker/single-node

curl -fL -o wazuh-certs-tool.sh \\
  https://packages.wazuh.com/5.0/wazuh-certs-tool-5.1.0-1.sh

curl -fL -o config.yml \\
  https://packages.wazuh.com/5.0/config-5.1.0-1.yml

chmod 700 wazuh-certs-tool.sh



Edit config.yml with the single-node names used by the official Compose file:

YAML


nodes:
  indexer:
    - name: wazuh.indexer
      dns:
        - wazuh.indexer

  manager:
    - name: wazuh.manager
      dns:
        - wazuh.manager

  dashboard:
    - name: wazuh.dashboard
      dns:
        - wazuh.dashboard



Generate the certificates using the certificate procedure documented by Wazuh for the selected release . In the official Wazuh Docker repository, the deployment helper can be used as follows:

Bash


sudo bash ../tools/utils/deployment/certificates-conf.sh --cert --copy --priv



The generated certificates must be present at the paths expected by the selected Wazuh Compose file. Do not commit config.yml, the certificate tool, or generated private keys to Git.

# 4. Start the Wazuh single-node stack

From the directory containing the official single-node Compose file, start Wazuh:

Bash


cd ~/wazuh-docker/single-node
docker compose up -d



Check the service status:

Bash


docker compose ps



The expected Wazuh services are:

Plain Text


wazuh.manager
wazuh.indexer
wazuh.dashboard



The first startup may take several minutes while the Indexer initializes its data directories and security configuration .

# 5. Verify Wazuh services

Inspect the logs for each Wazuh component:

Bash


docker compose logs --tail=100 wazuh.indexer
docker compose logs --tail=100 wazuh.manager
docker compose logs --tail=100 wazuh.dashboard



The Wazuh Dashboard is normally available on the port defined by the official Compose file. Open that address in a browser and sign in using the credentials defined by the official Wazuh deployment. Change default credentials immediately after the first login .

# 6. Wazuh ports

Port
Protocol
Purpose
1514
TCP/UDP
Wazuh agent event communication.
1515
TCP
Wazuh agent enrollment.
514
UDP
Optional syslog input.
55000
TCP
Wazuh Manager API.
9200
TCP
Wazuh Indexer API. Keep restricted to trusted administration networks.
5601
TCP
Wazuh Dashboard web interface in the standalone Wazuh installation.




Use host firewall rules to restrict access. The Indexer API should not be exposed to the public internet.

# 7. Stop the standalone Wazuh installation

Stop the Wazuh containers while keeping their named volumes:

Bash


docker compose down



Avoid docker compose down -v unless you intentionally want to delete the persistent Wazuh data and dashboard configuration.




# Part 2 — Install Traefik on Docker

This section installs Traefik as a standalone Docker project. It does not connect Traefik to Wazuh and does not contain Wazuh routing labels.

Traefik’s Docker provider watches Docker resources and can discover containers through Docker labels. The provider can be enabled with providers.docker=true, while exposedByDefault=false prevents containers from being routed unless explicitly enabled .

# 1. Traefik prerequisites

Install Docker Engine and Docker Compose using the official Docker instructions . Also install curl and OpenSSL if you want to test the standalone installation with a local TLS certificate.

Ensure ports 80 and 443 are available on the host. If another application is already using either port, stop it or select different host ports in the Traefik Compose file.

# 2. Create an isolated Traefik project

Create a separate directory that is independent from the Wazuh directory:

Bash


mkdir -p ~/traefik-standalone/{dynamic,certs}
cd ~/traefik-standalone



Create a local certificate for testing:

Bash


openssl req -x509 -nodes -days 365 -newkey rsa:2048 \\
  -keyout certs/local.key \\
  -out certs/local.crt \\
  -subj "/CN=traefik.localhost"

chmod 600 certs/local.key



For production, use a certificate issued for a real DNS name by a trusted certificate authority. Traefik supports user-defined certificates in dynamic configuration .

# 3. Create Traefik dynamic TLS configuration

Create dynamic/tls.yml:

YAML


tls:
  certificates:
    - certFile: /certs/local.crt
      keyFile: /certs/local.key



This is a standalone Traefik TLS configuration. It does not reference Wazuh or any other application.

# 4. Create the standalone Traefik Compose file

Create docker-compose.yml in ~/traefik-standalone:

YAML


services:
  traefik:
    image: traefik:v3.7
    container_name: traefik-standalone
    restart: unless-stopped
    security_opt:
      - no-new-privileges:true
    command:
      - --api.dashboard=true
      - --api.insecure=false
      - --providers.docker=true
      - --providers.docker.exposedbydefault=false
      - --providers.file.directory=/etc/traefik/dynamic
      - --entrypoints.web.address=:80
      - --entrypoints.websecure.address=:443
      - --entrypoints.web.http.redirections.entrypoint.to=websecure
      - --entrypoints.web.http.redirections.entrypoint.scheme=https
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./dynamic:/etc/traefik/dynamic:ro
      - ./certs:/certs:ro



This Compose file installs Traefik only. It has no Wazuh service, no Wazuh network, and no Wazuh-specific routing rule.

The read-only Docker socket is used for Docker provider discovery. Because Docker API access is powerful, protect the host and consider a restricted Docker socket proxy for production deployments .

# 5. Start Traefik

Start the standalone Traefik container:

Bash


cd ~/traefik-standalone
docker compose up -d



Check the container status:

Bash


docker compose ps



Inspect the logs:

Bash


docker compose logs --tail=100 traefik



# 6. Verify the standalone Traefik container

Confirm that Traefik is listening on the published ports:

Bash


curl -kI https://localhost



A response from Traefik confirms that the HTTPS entry point is reachable. A self-signed certificate produces a browser warning during local testing; use a trusted certificate for production.

The Traefik dashboard is disabled as an externally reachable application in this standalone example because no dashboard router is configured. If you later enable a dashboard router, protect it with HTTPS and authentication according to the official Traefik guidance  .

7. Stop the standalone Traefik installation

Stop the Traefik container without deleting the project files:

Bash


cd ~/traefik-standalone
docker compose down






Separation summary

The two installations are intentionally independent:

Area
Wazuh standalone
Traefik standalone
Project directory
~/wazuh-docker
~/traefik-standalone
Main services
Manager, Indexer, Dashboard
Traefik only
Compose project
Wazuh Compose project
Traefik Compose project
Certificates
Wazuh node certificates
Traefik TLS certificate
Routing
Wazuh Dashboard’s own port
Traefik entry points only
Integration
None in this guide
None in this guide




References

[1] Wazuh Docker deployment
[2] Changing the default password of Wazuh users
[3] Official Wazuh Docker repository
[4] Install Docker Engine
[5] Wazuh Docker requirements
[6] Traefik Docker provider
[7] Traefik TLS certificates
