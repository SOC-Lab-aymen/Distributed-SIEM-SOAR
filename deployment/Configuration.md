# Wazuh + Traefik Configuration

## Overview

This document explains how Traefik was integrated into the existing Wazuh single-node Docker Compose deployment and configured as a reverse proxy for the Wazuh Dashboard.

The objective is to make Traefik the entry point for accessing the Wazuh Dashboard instead of accessing the Dashboard directly.

### Architecture

```text
Browser
   |
   | http://wazuh.localhost
   v
Traefik :80
   |
   | Docker network: single-node_default
   v
Wazuh Dashboard :5601
```

---

## 1. Existing Wazuh Deployment

Wazuh was already deployed using the single-node Docker Compose configuration.

The main Wazuh services are:

- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard

The Wazuh Dashboard listens on port `5601` inside the Docker network.

Initially, the Dashboard was exposed directly to the host using:

```yaml
ports:
  - 443:5601
```

This means:

```text
Host :443
    |
    v
Wazuh Dashboard :5601
```

Therefore, the Dashboard could initially be accessed directly using:

```text
https://localhost
```

---

## 2. Adding Traefik

Traefik was added as another service inside the same `docker-compose.yml`.

The Traefik service is configured as follows:

```yaml
traefik:
  image: traefik:v3.7
  command:
    - "--api.insecure=true"
    - "--providers.docker=true"
    - "--providers.docker.exposedbydefault=false"
    - "--entrypoints.web.address=:80"
  ports:
    - "80:80"
    - "8080:8080"
  volumes:
    - /var/run/docker.sock:/var/run/docker.sock:ro
  networks:
    - default
```

### Configuration explanation

#### Docker provider

```yaml
--providers.docker=true
```

This allows Traefik to automatically discover Docker containers and read their Traefik labels.

#### Disable automatic exposure

```yaml
--providers.docker.exposedbydefault=false
```

Containers are not automatically exposed through Traefik.

A container must explicitly enable Traefik using:

```yaml
traefik.enable=true
```

This is safer because only explicitly configured services become accessible through Traefik.

#### HTTP entry point

```yaml
--entrypoints.web.address=:80
```

This creates an entry point called `web` listening on port `80`.

Therefore:

```text
Host :80
   |
   v
Traefik web entry point
```

#### Traefik dashboard

```yaml
- "8080:8080"
```

Port `8080` exposes the Traefik dashboard for local administration and testing.

It can be accessed using:

```text
http://localhost:8080/dashboard/
```

---

## 3. Docker Network

The Wazuh Dashboard was inspected and found to be connected to:

```text
single-node_default
```

This was verified using:

```bash
docker inspect single-node-wazuh.dashboard-1 \
  --format '{{json .NetworkSettings.Networks}}'
```

The Dashboard was using:

```text
single-node_default
```

Traefik was therefore attached to the same Compose default network:

```yaml
networks:
  - default
```

Because the Compose project is named `single-node`, the default network is:

```text
single-node_default
```

This allows Traefik to communicate directly with the Wazuh Dashboard through Docker's internal network.

The communication path is:

```text
Traefik
   |
   | single-node_default
   v
wazuh.dashboard:5601
```

Traefik does not need to access the Dashboard through the host's `localhost`.

---

## 4. Configuring the Wazuh Dashboard

Traefik labels were added to the `wazuh.dashboard` service.

```yaml
labels:
  - "traefik.enable=true"
  - "traefik.http.routers.wazuh.rule=Host(`wazuh.localhost`)"
  - "traefik.http.routers.wazuh.entrypoints=web"
  - "traefik.http.services.wazuh.loadbalancer.server.port=5601"
```

These labels tell Traefik how to route traffic to Wazuh.

---

## 5. Enabling Traefik for Wazuh

The following label enables Traefik:

```yaml
- "traefik.enable=true"
```

Because the Docker provider was configured with:

```yaml
--providers.docker.exposedbydefault=false
```

the Wazuh Dashboard must explicitly enable Traefik.

---

## 6. Creating the Wazuh Router

The following label defines the routing rule:

```yaml
- "traefik.http.routers.wazuh.rule=Host(`wazuh.localhost`)"
```

This tells Traefik:

> When a request arrives for `wazuh.localhost`, send the request to the Wazuh router.

Therefore:

```text
http://wazuh.localhost
```

is recognized by Traefik as a request for Wazuh.

`wazuh.localhost` is a local hostname and does not require a public domain.

---

## 7. Connecting the Router to the Entry Point

The following label connects the Wazuh router to the Traefik `web` entry point:

```yaml
- "traefik.http.routers.wazuh.entrypoints=web"
```

The `web` entry point was previously defined as:

```yaml
--entrypoints.web.address=:80
```

The resulting flow is:

```text
Browser
   |
   | http://wazuh.localhost
   v
Traefik :80
   |
   v
web entry point
   |
   v
Wazuh router
```

---

## 8. Connecting Traefik to Wazuh Dashboard

The following label specifies the Wazuh backend port:

```yaml
- "traefik.http.services.wazuh.loadbalancer.server.port=5601"
```

This tells Traefik that the Wazuh Dashboard is listening on port:

```text
5601
```

inside the Docker network.

The complete internal path is:

```text
Traefik
   |
   | single-node_default
   |
   v
wazuh.dashboard:5601
```

---

## 9. Complete Request Flow

When the user opens:

```text
http://wazuh.localhost
```

the request follows this path:

```text
                         Browser
                            |
                            |
                    http://wazuh.localhost
                            |
                            v
                    +---------------+
                    |    Traefik    |
                    |      :80      |
                    +-------+-------+
                            |
                            | Host rule
                            | wazuh.localhost
                            v
                    +---------------+
                    | Wazuh Router  |
                    +-------+-------+
                            |
                            | Port 5601
                            v
                  single-node_default
                            |
                            v
                    +---------------+
                    | Wazuh Dashboard|
                    |      :5601     |
                    +---------------+
```

Therefore, **Traefik is the entry point**, while the Wazuh Dashboard remains an internal Docker service from Traefik's perspective.

---

## 10. Port Configuration

The current configuration uses three important host ports:

| Host Port | Service | Purpose |
|---|---|---|
| `80` | Traefik | HTTP entry point |
| `443` | Wazuh Dashboard | Existing direct HTTPS access |
| `8080` | Traefik | Traefik dashboard |

The current routing is therefore:

```text
http://wazuh.localhost
        |
        v
     Traefik :80
        |
        v
Wazuh Dashboard :5601
```

The original direct Wazuh access is still available through:

```text
https://localhost
```

because the following mapping has not yet been removed:

```yaml
ports:
  - 443:5601
```

---

## 11. Verifying Traefik

The Compose configuration should first be validated:

```bash
docker compose config
```

If the configuration is valid, start the stack:

```bash
docker compose up -d
```

Check the running containers:

```bash
docker compose ps
```

Traefik should appear alongside the Wazuh services.

---

## 12. Checking the Traefik Dashboard

The Traefik dashboard is available at:

```text
http://localhost:8080/dashboard/
```

The dashboard can be used to verify that Traefik discovered the Wazuh Dashboard.

The Wazuh router should use a rule similar to:

```text
Host(`wazuh.localhost`)
```

and the backend should point to:

```text
wazuh.dashboard:5601
```

---

## 13. Testing the Wazuh Route

The route can be tested from the terminal:

```bash
curl -I http://wazuh.localhost
```

Traefik logs can also be inspected:

```bash
docker logs single-node-traefik-1 --tail 100
```

The Docker network can be inspected using:

```bash
docker network inspect single-node_default
```

Both Traefik and the Wazuh Dashboard should be connected to this network.

---

## 14. Current Architecture

The current configuration is:

```text
                         Browser
                            |
              +-------------+-------------+
              |                           |
              | http://wazuh.localhost    | https://localhost
              v                           v
       +-------------+             +-------------+
       |   Traefik   |             |    Wazuh   |
       |    :80      |             |  Dashboard |
       +------+------+             |    :5601   |
              |                    +-------------+
              |
              | single-node_default
              |
              v
       +-------------+
       |    Wazuh    |
       |  Dashboard  |
       |    :5601    |
       +-------------+
```

The first path passes through Traefik.

The second path is the original direct Wazuh access.

---

## 15. Recommended Final Architecture

The final objective is to make Traefik the **single entry point** to the Wazuh Dashboard.

The desired architecture is:

```text
                         Browser
                            |
                            |
                       HTTP / HTTPS
                            |
                            v
                    +---------------+
                    |    Traefik    |
                    |   :80 / :443  |
                    +-------+-------+
                            |
                            |
                    single-node_default
                            |
                            v
                    +---------------+
                    | Wazuh Dashboard|
                    |      :5601     |
                    +---------------+
```

In the final configuration, the Wazuh Dashboard should no longer expose:

```yaml
ports:
  - 443:5601
```

Instead, Traefik should handle the external HTTP/HTTPS connection and forward traffic internally to:

```text
wazuh.dashboard:5601
```

This prevents users from bypassing the reverse proxy.

---

## 16. HTTPS/TLS

The current test configuration uses the HTTP entry point:

```yaml
--entrypoints.web.address=:80
```

For the final deployment, HTTPS should be configured on Traefik.

The desired final request flow is:

```text
Browser
   |
   | https://wazuh.localhost
   v
Traefik :443
   |
   | TLS termination / reverse proxy
   v
Wazuh Dashboard :5601
```

Traefik should then become responsible for the external TLS connection.

---

## 17. Security Considerations

The current configuration contains:

```yaml
--api.insecure=true
```

This is useful for local development and testing but should not be used as the final production configuration.

For a production deployment:

- Disable the insecure Traefik API/dashboard.
- Configure HTTPS/TLS.
- Protect the Traefik dashboard with authentication or restrict access to it.
- Avoid exposing internal Wazuh services directly to the host.
- Keep the Docker socket mounted read-only where possible.
- Use explicit Docker networks.
- Expose only the ports that are actually required.

---

## 18. Summary

The Wazuh and Traefik integration is based on four main components:

### 1. Traefik Docker provider

```yaml
--providers.docker=true
```

Traefik discovers Docker services and their labels.

### 2. Explicit Wazuh exposure

```yaml
traefik.enable=true
```

Only the Wazuh Dashboard is explicitly exposed because automatic exposure is disabled.

### 3. Host-based routing

```yaml
traefik.http.routers.wazuh.rule=Host(`wazuh.localhost`)
```

Requests for `wazuh.localhost` are sent to the Wazuh router.

### 4. Backend service

```yaml
traefik.http.services.wazuh.loadbalancer.server.port=5601
```

Traefik forwards the request to the Wazuh Dashboard on port `5601`.

The complete routing chain is:

```text
http://wazuh.localhost
        |
        v
   Traefik :80
        |
        v
  Wazuh Router
        |
        v
single-node_default
        |
        v
wazuh.dashboard:5601
```

This establishes Traefik as the reverse-proxy entry point for the Wazuh Dashboard.