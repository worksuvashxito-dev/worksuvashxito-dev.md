# VPS & Project Complete Technical Documentation

> **Document Type:** Production Infrastructure & Service Architecture Documentation  
> **Server Hostname:** `xitoevents`  
> **Target Audience:** DevOps Engineers, Backend Developers, Full-Stack Engineers, System Administrators  
> **Audit Date:** September 9, 2026  
> **Classification:** Confidential â€” Internal Technical Reference  
> **Secrets Policy:** All passwords, tokens, API keys, JWT secrets, and private credentials have been securely redacted (`[REDACTED]`).

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [VPS & System Infrastructure Specifications](#2-vps--system-infrastructure-specifications)
   - [2.1 Host & Hardware Profile](#21-host--hardware-profile)
   - [2.2 Operating System & Kernel](#22-operating-system--kernel)
   - [2.3 User Accounts & Privilege Architecture](#23-user-accounts--privilege-architecture)
   - [2.4 Installed Packages & Runtime Matrix](#24-installed-packages--runtime-matrix)
   - [2.5 System Services & Scheduled Tasks](#25-system-services--scheduled-tasks)
   - [2.6 Storage & Partition Layout](#26-storage--partition-layout)
3. [Network, Ingress & Security Topology](#3-network-ingress--security-topology)
   - [3.1 Network Interfaces & IP Allocation](#31-network-interfaces--ip-allocation)
   - [3.2 Master Port Allocation Matrix](#32-master-port-allocation-matrix)
   - [3.3 Domain & Subdomain Routing Matrix](#33-domain--subdomain-routing-matrix)
   - [3.4 Ingress Controller & SSL/TLS Architecture (Traefik)](#34-ingress-controller--ssltls-architecture-traefik)
   - [3.5 Firewall (UFW) vs Docker Networking Bypass](#35-firewall-ufw-vs-docker-networking-bypass)
   - [3.6 SSH Server Security Posture](#36-ssh-server-security-posture)
4. [Master System Architecture & Data Flow](#4-master-system-architecture--data-flow)
   - [4.1 Global Infrastructure Architecture Diagram](#41-global-infrastructure-architecture-diagram)
   - [4.2 Service-to-Service Communication Map](#42-service-to-service-communication-map)
5. [Complete Project Inventory & Deep Dives](#5-complete-project-inventory--deep-dives)
   - [5.1 Project 1: PYTHON-FACE-SCAN-XITO (Face Recognition Platform)](#51-project-1-python-face-scan-xito)
   - [5.2 Project 2: community-chat (Media Upload Microservice)](#52-project-2-community-chat)
   - [5.3 Project 3: SUPABASE-EGRESS-BYPASS (Egress Proxy & Archive Engine)](#53-project-3-supabase-egress-bypass)
   - [5.4 Project 4: scripts (Automated Maintenance & Patching)](#54-project-4-scripts)
   - [5.5 Legacy, Inactive & Dormant Projects](#55-legacy-inactive--dormant-projects)
6. [Databases & Storage Infrastructure](#6-databases--storage-infrastructure)
   - [6.1 PostgreSQL with pgvector (facedb)](#61-postgresql-with-pgvector-facedb)
   - [6.2 Qdrant Vector Search Engine](#62-qdrant-vector-search-engine)
   - [6.3 Docker Persistent Volumes](#63-docker-persistent-volumes)
   - [6.4 External Cloud Storage (Cloudflare R2, pCloud, Supabase)](#64-external-cloud-storage-cloudflare-r2-pcloud-supabase)
7. [Deployment, Build & Process Management](#7-deployment-build--process-management)
   - [7.1 Deployment Models Summary](#71-deployment-models-summary)
   - [7.2 Service Lifecycle Commands (Start / Stop / Restart)](#72-service-lifecycle-commands-start--stop--restart)
   - [7.3 Updating Source Code & Rebuilding Services](#73-updating-source-code--rebuilding-services)
8. [Logs, Monitoring & Observability](#8-logs-monitoring--observability)
   - [8.1 Log Locations Matrix](#81-log-locations-matrix)
   - [8.2 Health Checks & Monitoring Endpoints](#82-health-checks--monitoring-endpoints)
   - [8.3 Live Observability Commands](#83-live-observability-commands)
9. [Comprehensive Security Audit & Vulnerabilities](#9-comprehensive-security-audit--vulnerabilities)
   - [9.1 Critical Security Vulnerabilities](#91-critical-security-vulnerabilities)
   - [9.2 High Security Risks](#92-high-security-risks)
   - [9.3 Medium & Low Security Risks](#93-medium--low-security-risks)
10. [Performance Analysis & Resource Bottlenecks](#10-performance-analysis--resource-bottlenecks)
    - [10.1 RAM & Swap Utilization Analysis](#101-ram--swap-utilization-analysis)
    - [10.2 CPU Load & Process Profiling](#102-cpu-load--process-profiling)
    - [10.3 Disk & I/O Footprint](#103-disk--io-footprint)
11. [Backup & Disaster Recovery Procedures](#11-backup--disaster-recovery-procedures)
    - [11.1 Current Backup Posture](#111-current-backup-posture)
    - [11.2 PostgreSQL Backup & Restore Playbook](#112-postgresql-backup--restore-playbook)
    - [11.3 Qdrant Snapshot & Restore Playbook](#113-qdrant-snapshot--restore-playbook)
    - [11.4 Upload Media Volume Backup Playbook](#114-upload-media-volume-backup-playbook)
    - [11.5 Complete Server Disaster Recovery Plan](#115-complete-server-disaster-recovery-plan)
12. [Recommended Infrastructure Improvements](#12-recommended-infrastructure-improvements)
13. [What a New Developer Needs to Know](#13-what-a-new-developer-needs-to-know)
    - [13.1 Server Mental Model in 3 Minutes](#131-server-mental-model-in-3-minutes)
    - [13.2 The Golden Rules (What NEVER to do)](#132-the-golden-rules-what-never-to-do)
    - [13.3 Quick Command Reference Sheet](#133-quick-command-reference-sheet)
    - [13.4 How to Add a New Domain / Subdomain](#134-how-to-add-a-new-domain--subdomain)
    - [13.5 Top 5 Outage Scenarios & Fixes](#135-top-5-outage-scenarios--fixes)

---

## 1. Executive Summary

This server (`xitoevents`) is a production **Hetzner Cloud VPS** located in Nuremberg, Germany, operating under public IPv4 `167.233.45.100`. The server runs **Ubuntu 24.04.4 LTS** with 2 AMD EPYC-Genoa virtual CPU cores, 3.7 GiB of physical RAM, 4.0 GiB of swap, and 76 GB NVMe storage.

The VPS serves as the centralized backend, API, and media infrastructure for the **Wedding Tales Nepal** and **Xito Events** product ecosystems. Three primary applications run concurrently on this server:

1. **PYTHON-FACE-SCAN-XITO**: An AI-powered facial recognition search platform for wedding photography. Features an ONNX/InsightFace Python service, PostgreSQL with `pgvector`, Qdrant vector database (indexing 368k facial embeddings), Express API, and a React frontend behind HTTP Basic Auth.
2. **community-chat**: A high-performance Fastify & Sharp media upload microservice for chat attachments and images, fronted by an internal Nginx proxy for static asset caching, authenticated against Supabase Auth, and storing files in a dedicated persistent Docker volume.
3. **SUPABASE-EGRESS-BYPASS**: A custom proxy and archive system designed to bypass Supabase egress bandwidth costs by routing file downloads through Cloudflare R2 and pCloud, accompanied by an on-demand Rust zip streaming compressor and a Vite React administrative dashboard.

All external web traffic on ports 80 and 443 is terminated and reverse-proxied by **Traefik v3**, which dynamically issues and renews Let's Encrypt SSL/TLS certificates across 5 production domains.

---

## 2. VPS & System Infrastructure Specifications

### 2.1 Host & Hardware Profile

| Attribute | Value / Specification | Notes / Evidence |
| :--- | :--- | :--- |
| **Server Hostname** | `xitoevents` | `uname -n` |
| **Hosting Provider** | Hetzner Online GmbH (AS24940) | Hostname: `static.100.45.233.167.clients.your-server.de` |
| **Datacenter Region** | Nuremberg, Bavaria, Germany (DE) | Geo coordinates: `49.4542, 11.0775` |
| **Server Timezone** | `Asia/Kathmandu` (+05:45) | Local time matches Nepal Standard Time (NPT) |
| **CPU Architecture** | `x86_64` (64-bit Little Endian) | KVM Hypervisor virtualization |
| **CPU Model** | AMD EPYC-Genoa Processor | 2 vCPUs @ 2.00 GHz, Family 25, Model 17 |
| **CPU Caches** | L1d: 64 KiB, L1i: 64 KiB, L2: 2 MiB, L3: 32 MiB | Dedicated L3 cache per socket |
| **Physical RAM** | 3.7 GiB (~3,800 MiB) | `free -h` |
| **Swap Space** | 4.0 GiB | Partition/file on `/dev/sda1` |
| **Disk Hardware** | 76.3 GB virtual disk (`/dev/sda`) | VirtIO / SCSI backing storage |

### 2.2 Operating System & Kernel

- **Operating System:** Ubuntu 24.04.4 LTS (Noble Numbat)
- **Linux Kernel:** `6.8.0-138-generic #138-Ubuntu SMP PREEMPT_DYNAMIC`
- **System Architecture:** x86_64 Linux GNU
- **Init System:** `systemd` v255.4-1ubuntu8
- **Package Manager:** `apt` (APT 2.7.14)

### 2.3 User Accounts & Privilege Architecture

| Username | UID:GID | Home Directory | Login Shell | Sudo Privileges | Role / Description |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `root` | `0:0` | `/root` | `/bin/bash` | Full `NOPASSWD:ALL` | Superuser / system administrator |
| `worksuvash` | `1000:1000` | `/home/worksuvash` | `/bin/bash` | Member of `sudo` (27) | Primary application developer workspace |
| `abinash_bhattarai` | `1001:1001` | `/home/abinash_bhattarai` | `/bin/bash` | Member of `sudo` (27) | Secondary developer account |
| `suraj` | `1002:1002` | `/home/suraj` | `/bin/bash` | **No sudo** | Collaborator account; has symlinks to `/home/worksuvash` |
| `postgres` | `108:111` | `/var/lib/postgresql` | `/bin/bash` | None | Host PostgreSQL system account |
| `www-data` | `33:33` | `/var/www` | `/usr/sbin/nologin` | None | Web server / MinIO legacy owner |

> [!NOTE]
> All primary active repositories and Docker configurations reside in `/home/worksuvash/`. The user `suraj` contains symlinks in `/home/suraj/projects/` pointing directly to `/home/worksuvash/*`.

### 2.4 Installed Packages & Runtime Matrix

| Runtime / Tool | Version | Binary Location | Purpose on Server |
| :--- | :--- | :--- | :--- |
| **Node.js** | `v22.23.2` | `/usr/bin/node` | Host PM2 apps (`SUPABASE-EGRESS-BYPASS`), tooling |
| **pnpm** | `11.9.0` | `/usr/local/bin/pnpm` | Package manager for `SUPABASE-EGRESS-BYPASS` |
| **npm** | `10.9.8` | `/usr/bin/npm` | Default Node package manager |
| **Python 3** | `3.12.3` | `/usr/bin/python3` | System scripts, face-scan backend utilities |
| **pip3** | `24.0` | `/usr/lib/python3/dist-packages/pip` | Python package installer |
| **Docker Engine** | `29.8.0` (build 88096ef) | `/usr/bin/docker` | Container runtime powering all microservices |
| **Docker Compose** | `v5.5.1` | `/usr/libexec/docker/cli-plugins/docker-compose` | Container orchestration |
| **Rust (rustc)** | `1.97.0` (2d8144b78) | `/root/.cargo/bin/rustc` | Compiling Rust binaries (e.g., `zip-compressor`) |
| **Cargo** | `1.97.0` | `/root/.cargo/bin/cargo` | Rust build tool and package manager |
| **Nginx (Host)** | `1.24.0` | `/usr/sbin/nginx` | **Inactive** on host; Dockerized Nginx used instead |
| **PostgreSQL CLI** | `16.15` | `/usr/bin/psql` | PostgreSQL client for interacting with Docker container |
| **PM2** | `5.4.3` | `/usr/local/bin/pm2` | Process manager running `SUPABASE-EGRESS-BYPASS` apps |
| **Git** | `2.43.0` | `/usr/bin/git` | Version control across repositories |

### 2.5 System Services & Scheduled Tasks

#### Enabled Systemd Services (Core)
- `docker.service`: Docker container engine daemon.
- `ssh.service` / `sshd.service`: OpenSSH server daemon on port 22.
- `systemd-resolved.service`: Local DNS resolver stub on `127.0.0.53` and `127.0.0.54`.
- `systemd-networkd.service`: Network configuration manager.
- `cron.service`: System-wide cron daemon.
- `ufw.service`: Uncomplicated Firewall (active).

#### Disabled / Inactive Services (Important Context)
- `nginx.service`: Disabled and dead. Port 80 and 443 are bound by Docker / Traefik. Do **not** start host Nginx or Traefik will fail to bind.
- `minio.service`: Disabled and dead (`/etc/systemd/system/minio.service`). Legacy S3 storage from June 2026.
- `fail2ban.service`: **Active and running**. Enabled on boot, actively monitoring port 22 with 24-hour ban time and developer IP whitelisting.

#### Scheduled Tasks (Cron Jobs)
Examining root and system crontabs:
```bash
# Root Crontab (`crontab -l`):
15 22 23 8 * /home/worksuvash/scripts/auto_fix_sunday.sh
```
- **Description:** One-shot maintenance script originally scheduled for August 23, 2026 at 22:15. Runs `/home/worksuvash/scripts/apply_fixes.mjs` which auto-patches TypeScript build issues in `SUPABASE-EGRESS-BYPASS`, builds packages, and reloads PM2 process `egress-api-server`.
- **System Cron:** Standard hourly/daily maintenance in `/etc/cron.*` (logrotate, sysstat, apt-compat). Host `certbot.timer` safely disabled as Traefik handles all SSL inside Docker.

### 2.6 Storage & Partition Layout

```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        75G   34G   39G  47% /
/dev/sda15      253M  146K  252M   1% /boot/efi
tmpfs           1.9G     0  1.9G   0% /dev/shm
tmpfs           382M  1.7M  380M   1% /run
```

- **Root Disk (`/dev/sda1`):** 75 GB total, **34 GB used (47%)**, **39 GB available**. Healthy headroom.
- **Top Directories by Disk Consumption:**
  - `/home/worksuvash/PYTHON-FACE-SCAN-XITO`: **4.8 GB** (Postgres DB: 826 MB, Qdrant vectors: 1.1 GB, InsightFace models & dist: ~2.9 GB)
  - `/home/worksuvash/SUPABASE-EGRESS-BYPASS`: **510 MB** (Node modules, build caches, Rust target artifacts)
  - `/home/worksuvash/community-chat`: **119 MB**
  - `/var/lib/docker/volumes/community-chat_vps_upload_data`: **~16 KB** (Active upload storage)
  - `/var/lib/docker/rootfs`: **~18 GB** (Docker images: Postgres, Qdrant, Traefik, Watchtower, custom builds)
  - `/opt/minio/data`: **216 KB** (Dormant legacy data)

---

## 3. Network, Ingress & Security Topology

### 3.1 Network Interfaces & IP Allocation

| Interface | Type | IP Address / Subnet | Role & Association |
| :--- | :--- | :--- | :--- |
| `eth0` | Physical / WAN | `167.233.45.100/32` | Public WAN IP; Default gateway `172.31.1.1` |
| `lo` | Loopback | `127.0.0.1/8`, `::1/128` | Localhost internal communication |
| `docker0` | Bridge | `172.17.0.1/16` | Default Docker bridge (currently no containers connected) |
| `br-e1982996ec98` | Docker Network | `172.18.0.1/16` | `faceapp-network` (Postgres, Qdrant, Traefik, APIs) |
| `br-fca443da96ed` | Docker Network | `172.19.0.1/16` | `community-chat_default` (Upload service, Nginx proxy) |

> [!IMPORTANT]
> The host interface IP `172.18.0.1` is the gateway for `faceapp-network`. Services running directly on the host (such as PM2 services `egress-api-server` on port 5000 and `egress-dashboard` on port 5173) are accessed by the Traefik container via `http://172.18.0.1:<PORT>`.

### 3.2 Master Port Allocation Matrix

| Port | Proto | Bind Address | Service / Container | Project | Publicly Reachable? | Security Status |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **22** | TCP | `0.0.0.0`, `[::]` | `sshd` (OpenSSH 9.6p1) | Host System | **Yes** (WAN) | âš ï¸ Password authentication enabled |
| **80** | TCP | `0.0.0.0`, `[::]` | `traefik` (docker-proxy) | Traefik Router | **Yes** (WAN) | Standard HTTP (redirects to HTTPS) |
| **443** | TCP | `0.0.0.0`, `[::]` | `traefik` (docker-proxy) | Traefik Router | **Yes** (WAN) | Standard HTTPS (Let's Encrypt TLS) |
| **3001** | TCP | `127.0.0.1` | `vps-upload-service` (docker) | `community-chat` | **No** (Localhost only) | âœ… Restricted to local/internal |
| **5000** | TCP | `0.0.0.0` | `node` (PM2 `egress-api-server`) | `SUPABASE-EGRESS-BYPASS` | **Yes** (Bound to 0.0.0.0) | âš ï¸ Should bind to `127.0.0.1` or `172.18.0.1` |
| **5173** | TCP | `0.0.0.0` | `node` (PM2 `egress-dashboard`) | `SUPABASE-EGRESS-BYPASS` | **Yes** (Bound to 0.0.0.0) | âš ï¸ Should bind to `127.0.0.1` or `172.18.0.1` |
| **5432** | TCP | `127.0.0.1` | `postgres` (docker-proxy) | `PYTHON-FACE-SCAN-XITO` | **No** (Localhost only) | Secure localhost binding |
| **6333** | TCP | `127.0.0.1` | `qdrant` (docker-proxy) | `PYTHON-FACE-SCAN-XITO` | **No** (Localhost only) | âœ… Secured: Bound to 127.0.0.1 only |
| **8080** | TCP | Internal Docker | `api-server` (docker) | `PYTHON-FACE-SCAN-XITO` | No (Traefik only) | Secure internal container port |
| **8081** | TCP | `172.18.0.1`, `127.0.0.1` | `vps-upload-nginx` (docker) | `community-chat` | **No** (Internal / Traefik only) | âœ… WAN blocked, Traefik allowed |
| **8099** | TCP | `0.0.0.0` | `zip-compressor` (Rust) | `SUPABASE-EGRESS-BYPASS` | **Yes** (When running) | âš ï¸ On-demand service, no WAN auth |

### 3.3 Domain & Subdomain Routing Matrix

All external traffic resolves to `167.233.45.100` and hits Traefik on port 443:

| Domain / FQDN | Router Name | TLS Cert Provider | Backend Target | Path Filter | Authentication |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `search.weddingtalesnepal.com` | `frontend-secure` | Let's Encrypt | `frontend:80` (Docker) | `/` (root) | HTTP Basic Auth (`search-auth`) |
| `search.weddingtalesnepal.com` | `api-secure` | Let's Encrypt | `api-server:8080` (Docker) | `/api/*` | API Key / Session Token |
| `chat-media.xitoevent.com` | `upload-service` | Let's Encrypt | `172.18.0.1:8081` (Nginx) | `/*` | Supabase JWT for POST `/api/v1/uploads` |
| `dashboard-egress.weddingtalesnepal.com` | `egress-dashboard` | Let's Encrypt | `172.18.0.1:5173` (Host) | `/*` | Application Auth |
| `api-egress.weddingtalesnepal.com` | `egress-api` | Let's Encrypt | `172.18.0.1:5000` (Host) | `/*` | Supabase JWT / Bearer Token |
| `deaeration.weddingtalesnepal.com` | *(Historic)* | Let's Encrypt | *(Target Removed)* | N/A | Project decommissioned |

### 3.4 Ingress Controller & SSL/TLS Architecture (Traefik)

- **Traefik Version:** v3 (image `traefik:latest`)
- **Docker Compose File:** `/home/worksuvash/PYTHON-FACE-SCAN-XITO/docker-compose.yml`
- **Static Configuration:** `/home/worksuvash/PYTHON-FACE-SCAN-XITO/traefik.yml`
- **Dynamic File Provider:** `/home/worksuvash/PYTHON-FACE-SCAN-XITO/dynamic.yml`
- **ACME Certificates File:** `/home/worksuvash/PYTHON-FACE-SCAN-XITO/acme.json` (File mode `0600`)
- **ACME Challenge Type:** HTTP-01 challenge via entrypoint `web` (`:80`).
- **Active Valid Certificates:**
  - `search.weddingtalesnepal.com`: Valid until Nov 28, 2026
  - `chat-media.xitoevent.com`: Valid until Nov 19, 2026
  - `dashboard-egress.weddingtalesnepal.com`: Valid until Dec 08, 2026
  - `api-egress.weddingtalesnepal.com`: Valid until Dec 08, 2026
  - `deaeration.weddingtalesnepal.com`: Valid until Nov 27, 2026

### 3.5 Firewall (UFW) vs Docker Networking Bypass

The Uncomplicated Firewall (`ufw`) is active on the host:
```
Status: active
Default: deny (incoming), allow (outgoing), deny (routed)
Allowed Ports: 22/tcp, 80,443/tcp, 5432/tcp, 6333/tcp, 5173/tcp, 5174/tcp, 5000/tcp
```

> [!CAUTION]
> **The Docker UFW Bypass Risk:** Docker manipulates Linux `iptables` directly by inserting `PREROUTING` and `DOCKER` chain rules that execute **before** UFW's input filtering rules.  
> As a result, any container publishing a port with `-p 6333:6333`, `-p 3001:3001`, or `-p 8081:80` is **completely exposed to the entire internet**, bypassing UFW rules. For example, Qdrant (port 6333) is publicly reachable from any IP on the globe without authentication.

### 3.6 SSH Server Security Posture

Audit of `/etc/ssh/sshd_config` (`sshd -T`):
- **Port:** `22` (Standard)
- **PermitRootLogin:** `no` (Good: Root cannot log in directly over SSH)
- **PubkeyAuthentication:** `yes` (ED25519 & RSA supported via `PubkeyAcceptedAlgorithms +ssh-rsa`)
- **PasswordAuthentication:** `yes` (Protected by active Fail2ban jail; prevents user lockout while blocking bots)
- **MaxAuthTries:** `6`
- **Fail2ban Status:** **Active (Running)** (Bans attacking IPs for 24h after 5 failed attempts; developer IP whitelisted)

---

## 4. Master System Architecture & Data Flow

### 4.1 Global Infrastructure Architecture Diagram

```
                                  [ INTERNET USERS & CLIENT APPS ]
                                                  â”‚
                 â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
                 â”‚                                â”‚                                â”‚
      search.weddingtalesnepal.com       chat-media.xitoevent.com       *-egress.weddingtalesnepal.com
                 â”‚                                â”‚                                â”‚
                 â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
                                                  â”‚
                                       DNS: 167.233.45.100 (WAN)
                                                  â”‚
                                                  â–¼
                     â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
                     â”‚          HETZNER VPS: xitoevents (Ubuntu 24.04)         â”‚
                     â”‚                                                         â”‚
                     â”‚   â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”   â”‚
                     â”‚   â”‚         Traefik v3 Reverse Proxy (Docker)       â”‚   â”‚
                     â”‚   â”‚     Ports 80 & 443 (Let's Encrypt TLS Term.)     â”‚   â”‚
                     â”‚   â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜   â”‚
                     â”‚                            â”‚                            â”‚
         â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
         â”‚ (HTTP Path / Host Routing)             â”‚                                       â”‚
         â–¼                                        â–¼                                       â–¼
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚ PYTHON-FACE-SCAN-XITO         â”‚  â”‚ community-chat              â”‚  â”‚ SUPABASE-EGRESS-BYPASS (PM2)      â”‚
â”‚                               â”‚  â”‚                             â”‚  â”‚                                   â”‚
â”‚ â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â” â”‚  â”‚ â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â” â”‚  â”‚ â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â” â”‚
â”‚ â”‚ Frontend (Docker :80)     â”‚ â”‚  â”‚ â”‚ vps-upload-nginx (:8081)â”‚ â”‚  â”‚ â”‚ egress-dashboard (:5173)     â”‚ â”‚
â”‚ â”‚ React SPA + Basic Auth    â”‚ â”‚  â”‚ â”‚ Static File Cache       â”‚ â”‚  â”‚ â”‚ React Admin Portal (Vite)     â”‚ â”‚
â”‚ â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜ â”‚  â”‚ â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜ â”‚  â”‚ â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜ â”‚
â”‚               â”‚               â”‚  â”‚              â”‚              â”‚  â”‚                                   â”‚
â”‚ â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â–¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â” â”‚  â”‚ â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â–¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â” â”‚  â”‚ â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â” â”‚
â”‚ â”‚ API Server (Docker :8080) â”‚ â”‚  â”‚ â”‚ vps-upload-service      â”‚ â”‚  â”‚ â”‚ egress-api-server (:5000)     â”‚ â”‚
â”‚ â”‚ Express Node.js Service   â”‚ â”‚  â”‚ â”‚ Fastify Node.js (:3001) â”‚ â”‚  â”‚ â”‚ Express Bandwidth Proxy       â”‚ â”‚
â”‚ â””â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”˜ â”‚  â”‚ â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜ â”‚  â”‚ â””â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜ â”‚
â”‚        â”‚             â”‚        â”‚  â”‚              â”‚              â”‚  â”‚         â”‚                         â”‚
â”‚        â–¼             â–¼        â”‚  â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜  â”‚         â–¼                         â”‚
â”‚ â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â” â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â” â”‚                 â”‚                 â”‚ â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â” â”‚
â”‚ â”‚ PostgreSQL â”‚ â”‚ Qdrant     â”‚ â”‚                 â”‚                 â”‚ â”‚ zip-compressor (Rust :8099)   â”‚ â”‚
â”‚ â”‚ (pgvector) â”‚ â”‚ Vector DB  â”‚ â”‚                 â”‚                 â”‚ â”‚ On-Demand Archive Streamer    â”‚ â”‚
â”‚ â”‚ :5432      â”‚ â”‚ :6333      â”‚ â”‚                 â”‚                 â”‚ â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜ â”‚
â”‚ â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜ â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜ â”‚                 â”‚                 â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
â”‚        â”‚                      â”‚                 â”‚                                   â”‚
â”‚ â”Œâ”€â”€â”€â”€â”€â”€â–¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â” â”‚                 â”‚                                   â”‚
â”‚ â”‚ face-service (:5001)      â”‚ â”‚                 â”‚                                   â”‚
â”‚ â”‚ Python 3.12 + InsightFace â”‚ â”‚                 â”‚                                   â”‚
â”‚ â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜ â”‚                 â”‚                                   â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜                 â”‚                                   â”‚
                â”‚                                 â”‚                                   â”‚
                â”‚ Local Docker Volume             â–¼ Local Docker Volume               â”‚
                â”‚                                 â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”         â”‚
                â”‚                                 â”‚ Volume: vps_upload_data â”‚         â”‚
                â”‚                                 â”‚ (/var/uploads/xito)     â”‚         â”‚
                â”‚                                 â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜         â”‚
                â”‚                                                                     â”‚
                â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
                                                  â”‚ External Outbound HTTPS
                                                  â–¼
                        â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
                        â”‚              EXTERNAL CLOUD SERVICES              â”‚
                        â”‚                                                   â”‚
                        â”‚  â€¢ Cloudflare R2 Storage (Buckets)                â”‚
                        â”‚  â€¢ Supabase Backend (Auth, Database, Storage)     â”‚
                        â”‚  â€¢ pCloud API (weddingtalesfinance@gmail.com)     â”‚
                        â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

### 4.2 Service-to-Service Communication Map

1. **Face Search User Query:**
   `User` â†’ `Traefik (HTTPS)` â†’ `frontend:80` (Auth validated) â†’ `api-server:8080` â†’ Calls `face-service:5001` (InsightFace model generates 512-dim vector) â†’ Queries `qdrant:6333` (Cosine similarity search against 368,782 vectors) â†’ Fetches matching image records from `postgres:5432` (`facedb`) â†’ Returns image URLs (hosted in Cloudflare R2 bucket `xito-photography-xitoevents-com`).
2. **Chat Image Upload Flow:**
   `Chat Client` â†’ `Traefik (HTTPS)` â†’ `vps-upload-nginx:8081` â†’ Proxies to `vps-upload-service:3001` â†’ Validates JWT against Supabase Auth â†’ Processes image buffer using `Sharp` (resizes, strips EXIF, generates WebP preview) â†’ Writes to local disk volume `/var/uploads/xito/chat/` â†’ Returns public CDN URL.
3. **Bandwidth Egress Bypass Flow:**
   `Client Download` â†’ `Traefik (HTTPS)` â†’ `egress-api-server:5000` â†’ Intercepts request to Supabase storage URL â†’ Checks if file is cached in pCloud or Cloudflare R2 â†’ If zip requested, spawns `zip-compressor:8099` (Rust) on demand to stream deflate archive on-the-fly without disk buffering â†’ Eliminates egress charges from Supabase.

---

## 5. Complete Project Inventory & Deep Dives

### 5.1 Project 1: PYTHON-FACE-SCAN-XITO

- **Exact Path:** `/home/worksuvash/PYTHON-FACE-SCAN-XITO`
- **Git Repository:** `https://github.com/worksuvashxito-dev/PYTHON-FACE-SCAN-XITO`
- **Current Branch:** `main` (Clean, commit `9aa04bb`)
- **Primary Purpose:** Enterprise facial indexing and search system for wedding event photography galleries. Allows guests and clients to upload a selfie and instantly locate all photographs of themselves across tens of thousands of event photos.
- **Key Sub-Services:**
  1. `traefik`: Ingress reverse proxy & automated SSL manager.
  2. `postgres`: PostgreSQL 16 with `pgvector` extension for metadata, album catalogs, request logs, and relations.
  3. `qdrant`: Vector database hosting high-dimensional face feature vectors.
  4. `face-service`: Python 3.12 Flask + Gunicorn microservice running InsightFace (`buffalo_l` ONNX model) on CPU.
  5. `api-server`: Node.js Express application orchestrating business logic, auth, and database queries.
  6. `frontend`: Static React SPA client protected by HTTP Basic Auth.
  7. `watchtower`: Continuous Docker container updater.

#### Component Architecture & Port Mapping
```
traefik (Host :80, :443)
postgres (Host 127.0.0.1:5432 -> Cont 5432)
qdrant (Host 0.0.0.0:6333 -> Cont 6333)
face-service (Internal faceapp-network: 5001)
api-server (Internal faceapp-network: 8080)
frontend (Internal faceapp-network: 80)
watchtower (Mounts /var/run/docker.sock)
```

#### Key Configuration & Environment Variables
Configured securely in `/home/worksuvash/PYTHON-FACE-SCAN-XITO/.env` (untracked from git and added to `.gitignore`). All values in `docker-compose.yml` are parameterized as `${VAR}` variables with zero hardcoded credentials:
- `DATABASE_URL`: `postgresql://faceapp:[REDACTED]@postgres:5432/facedb`
- `R2_ACCOUNT_ID`: `75785beaee42c7b41f3a5cec3bae1857`
- `R2_ACCESS_KEY_ID`: `[REDACTED]`
- `R2_SECRET_ACCESS_KEY`: `[REDACTED]`
- `R2_BUCKET_NAME`: `xito-photography-xitoevents-com`
- `R2_PUBLIC_URL`: `https://pub-a5615427d38e421491703dab2fffcffb.r2.dev`
- `FACE_SERVICE_URL`: `http://face-service:5001`
- `QDRANT_URL`: `http://qdrant:6333`
- `QDRANT_API_KEY`: *(Empty)*
- `SUPABASE_URL`: `https://opzqagvdckvvykqnbqfx.supabase.co`
- `SUPABASE_SERVICE_KEY`: `[REDACTED]`
- `ADMIN_USERNAME`: `admin`
- `ADMIN_PASSWORD`: `[REDACTED]`
- `BasicAuth (Frontend)`: `admin:[REDACTED_APR1_HASH]`

#### Important Folders & Storage
- `./postgres_data`: Persistent storage for PostgreSQL (826 MB).
- `./artifacts/face-service/qdrant_storage`: Persistent Qdrant vector index files (1.1 GB).
- `./artifacts/face-service/models`: InsightFace ONNX neural network weights.
- `./acme.json`: Let's Encrypt TLS certificate store.

---

### 5.2 Project 2: community-chat

- **Exact Path:** `/home/worksuvash/community-chat`
- **Git Repository:** `https://github.com/worksuvashxito-dev/community-chat.git`
- **Current Branch:** `main` (Has local modified files, commit `e0ca356`)
- **Primary Purpose:** Dedicated high-throughput VPS image and document upload microservice designed to handle real-time chat attachments, avatars, and media files without routing large binary payloads through primary application servers.
- **Key Sub-Services:**
  1. `vps-upload-service`: Fastify Node.js microservice running on port 3001. Handles multipart upload streaming, MIME validation, image compression/resizing with Sharp, and Supabase JWT authentication.
  2. `vps-upload-nginx`: Nginx Alpine reverse proxy on port 8081. Serves uploaded files directly from disk with aggressive caching headers (`Cache-Control: public, max-age=2592000, immutable`), rate limits upload requests, and proxies API calls to Fastify.

#### Key API Endpoints & Routes
- `POST /api/v1/uploads?context=community_chat`: Primary upload endpoint (Protected with Supabase JWT).
- `GET /health`: Comprehensive healthcheck returning disk quota, used bytes, percentage, and lock state.
- `GET /admin`: Web-based monitoring UI with live metrics and storage utilization.
- `GET /api/v1/admin/uploads`: JSON list of uploaded assets.
- `DELETE /api/v1/admin/uploads/:id`: Administrative asset deletion.
- `GET /uploads/*`: High-speed static file delivery via Nginx.

#### Storage Architecture & Quotas
- **Docker Volume:** `community-chat_vps_upload_data` (Driver: local, mounted at `/var/uploads/xito`).
- **Subdirectories:**
  - `/var/uploads/xito/chat`
  - `/var/uploads/xito/community_chat`
  - `/var/uploads/xito/companies`
  - `/var/uploads/xito/users`
- **Max File Size:** `20 MB` (`MAX_FILE_SIZE_MB=20`).
- **Storage Quota:** `20 GB` (`MAX_STORAGE_QUOTA_GB=20`). When exceeded, the service automatically engages `storage_locked` mode to protect VPS disk stability.

---

### 5.3 Project 3: SUPABASE-EGRESS-BYPASS

- **Exact Path:** `/home/worksuvash/SUPABASE-EGRESS-BYPASS`
- **Git Repository:** `https://github.com/worksuvashxito-dev/SUPABASE-EGRESS-BYPASS`
- **Current Branch:** `main` (Modified by auto-fix script, commit `5f263d0`)
- **Primary Purpose:** Cost-mitigation middleware designed to intercept client download requests and redirect/stream large event assets and photograph archives directly from Cloudflare R2 and pCloud rather than paying expensive Supabase egress fees. Includes an on-demand Rust zip generation engine to package client albums without disk caching.
- **Process Management:** Managed via **PM2** on the host operating system.

#### Key Sub-Modules
1. **API Server (`@workspace/api-server`)**
   - **Path:** `/home/worksuvash/SUPABASE-EGRESS-BYPASS/artifacts/api-server`
   - **PM2 App ID:** `0` (`egress-api-server`)
   - **Port:** `5000`
   - **Runtime:** Node.js 22 (Started with `pnpm run dev`)
   - **Features:** Express API, pCloud REST client, Cloudflare R2 S3 SDK, Supabase client, avatar proxy, dynamic zip streaming manager.
2. **Dashboard (`@workspace/dashboard`)**
   - **Path:** `/home/worksuvash/SUPABASE-EGRESS-BYPASS/artifacts/dashboard`
   - **PM2 App ID:** `1` (`egress-dashboard`)
   - **Port:** `5173`
   - **Runtime:** Vite Preview / Node.js (Production preview mode via `pnpm run serve` on compiled `dist/public` assets)
   - **Features:** Administrative management interface for tracking egress metrics, file mappings, and system health.
3. **Zip Compressor Service (`services/zip-compressor`)**
   - **Path:** `/home/worksuvash/SUPABASE-EGRESS-BYPASS/services/zip-compressor`
   - **Binary:** `target/release/zip-compressor` (Compiled Rust executable, opt-level 3, LTO enabled)
   - **Port:** `8099` (Internal)
   - **Lifecycle:** **On-Demand**. When an archive download is requested, `zipCompressorManager.ts` detects if port 8099 is responding. If offline, it automatically spawns the Rust binary as a child process. When idle for more than 5 minutes, it sends SIGTERM to reclaim RAM.

#### Key Configuration & Environment Variables (`.env`)
- `DATABASE_URL`: `postgresql://faceapp:[REDACTED]@localhost:5432/facedb`
- `SUPABASE_URL`: `https://opzqagvdckvvykqnbqfx.supabase.co`
- `SUPABASE_SERVICE_ROLE_KEY`: `[REDACTED]`
- `R2_ACCOUNT_ID`: `75785beaee42c7b41f3a5cec3bae1857`
- `R2_ACCESS_KEY_ID`: `[REDACTED]`
- `R2_SECRET_ACCESS_KEY`: `[REDACTED]`
- `R2_BUCKET_NAME`: `xito-photography-xitoevents-com`
- `R2_AVATARS_BUCKET_NAME`: `xito-avatars`
- `PCLOUD_USERNAME`: `weddingtalesfinance@gmail.com`
- `PCLOUD_PASSWORD`: `[REDACTED]`
- `ZIP_COMPRESSOR_URL`: `http://localhost:8099`

---

### 5.4 Project 4: scripts (Automated Maintenance & Patching)

- **Exact Path:** `/home/worksuvash/scripts`
- **Files:**
  - `auto_fix_sunday.sh`: Bash wrapper script setting PATH and executing `apply_fixes.mjs`.
  - `apply_fixes.mjs`: Automated patching script using Node.js child_process.
  - `auto_fix_sunday.log`: Execution log file documenting build and patch runs.
- **Functionality:**
  1. Inspects `/home/worksuvash/SUPABASE-EGRESS-BYPASS/artifacts/api-server/src/lib/pcloud.ts` and patches `private http` to `public http` and `private authParams` to `public authParams`.
  2. Inspects `src/lib/httpAgents.ts` and removes incompatible `noDelay: true` options.
  3. Inspects `src/routes/drive/index.ts` and patches `Buffer.from(req.body as any)` and removes deprecated `highWaterMark` configurations.
  4. Runs `pnpm --filter @workspace/api-server run typecheck`.
  5. Executes `pnpm run build` across the workspace.
  6. Invokes `pm2 reload egress-api-server` for zero-downtime deployment.

---

### 5.5 Legacy, Inactive & Dormant Projects

1. **/opt/minio:** Contains an unmaintained 110 MB MinIO binary and 216 KB of static files. The systemd service `minio.service` is disabled and inactive.
2. **/opt/supabase & /opt/xito-backend:** Abandoned self-hosted Supabase Docker Compose attempt from late June 2026. The backend is completely replaced by Supabase Cloud (`opzqagvdckvvykqnbqfx.supabase.co`).
3. **Deaeration Project (`deaeration`):** Previously configured on port 8082 pointing to `deaeration.weddingtalesnepal.com`. The source directory has been deleted (`rm -rf Deaeration`), though a stale Nginx configuration remains in `/etc/nginx/sites-available/deaeration` and certificate in `acme.json`.

---

## 6. Databases & Storage Infrastructure

### 6.1 PostgreSQL with pgvector (facedb)

- **Container Name:** `postgres`
- **Image:** `pgvector/pgvector:pg16`
- **Internal Port:** `5432` | **Host Port:** `127.0.0.1:5432`
- **Database Name:** `facedb` | **Database User:** `faceapp`
- **Storage Volume:** `./postgres_data` mounted to `/var/lib/postgresql/data` (826 MB on disk)
- **Database Size:** `19 MB`
- **Active Extensions:** `vector` (pgvector for similarity search)

#### Table Inventory & Row Counts
| Table Name | Live Row Count | Purpose |
| :--- | :--- | :--- |
| `request_logs` | **74,111** | API traffic logs, timing, user agent, response codes |
| `downloads` | **17** | Download tracking records |
| `api_keys` | **2** | Service-to-service authentication keys |
| `users` | 0 | Internal user accounts |
| `events` | 0 | Photography event metadata |
| `albums` | 0 | Event album collections |
| `images` | 0 | Processed image records |
| `library_images` | 0 | Gallery library image references |
| `face_embeddings` | 0 | PostgreSQL vector backup table (Qdrant is primary) |
| `image_index_status` | 0 | Status of background vector indexing jobs |
| `upload_sessions` | 0 | Multipart chunk upload state |
| `portal_clients` | 0 | Client portal access records |
| `favorites` | 0 | User-favorited photos |
| `audit_logs` | 0 | Security and change audit log |
| `edge_functions` | 0 | Extensible serverless function definitions |
| `edge_function_logs` | 0 | Execution logs for edge functions |
| `edge_function_versions` | 0 | Versioning for serverless code |
| `edge_function_triggers` | 0 | Event triggers for functions |
| `edge_function_secrets` | 0 | Encrypted function secrets |
| `edge_function_invocations`| 0 | Invocation history |

### 6.2 Qdrant Vector Search Engine

- **Container Name:** `qdrant`
- **Image:** `qdrant/qdrant:v1.18.2`
- **Ports:** `0.0.0.0:6333->6333/tcp`, `6334/tcp`
- **Storage Location:** `/home/worksuvash/PYTHON-FACE-SCAN-XITO/artifacts/face-service/qdrant_storage` (1.1 GB on disk)
- **Primary Collection:** `face_embeddings`
  - **Indexed Vectors Count:** **367,160**
  - **Total Points Count:** **368,782**
  - **Vector Dimensions:** `512`
  - **Distance Metric:** `Cosine`
  - **Payload Schema:** `folder_prefix` (Keyword), `image_id` (Keyword)
  - **HNSW Index:** `m=16`, `ef_construct=100`, in-memory graph
  - **Status:** Green / Healthy
- **Secondary Collection:** `image_index_status`

### 6.3 Docker Persistent Volumes

| Volume Name | Driver | Mount Point in Container | Size | Association |
| :--- | :--- | :--- | :--- | :--- |
| `community-chat_vps_upload_data` | `local` | `/var/uploads/xito` | ~16 KB | `community-chat` media files |
| `faceapp_postgres_data` (bind) | `bind` | `/var/lib/postgresql/data` | 826 MB | PostgreSQL database files |
| `faceapp_qdrant_data` (bind) | `bind` | `/qdrant/storage` | 1.1 GB | Qdrant vector storage |

### 6.4 External Cloud Storage (Cloudflare R2, pCloud, Supabase)

1. **Cloudflare R2 (S3-Compatible Object Storage):**
   - **Account ID:** `75785beaee42c7b41f3a5cec3bae1857`
   - **Primary Bucket:** `xito-photography-xitoevents-com` (Stores high-resolution wedding photos)
   - **Public CDN URL:** `https://pub-a5615427d38e421491703dab2fffcffb.r2.dev`
   - **Avatars Bucket:** `xito-avatars`
2. **pCloud Cloud Storage:**
   - **Account:** `weddingtalesfinance@gmail.com`
   - **Role:** Off-site archive storage for high-resolution originals and bulk archive zip generation.
3. **Supabase Cloud:**
   - **Project Ref:** `opzqagvdckvvykqnbqfx`
   - **URL:** `https://opzqagvdckvvykqnbqfx.supabase.co`
   - **Role:** Authentication provider (JWT issuance/verification) and global user data store.

---

## 7. Deployment, Build & Process Management

### 7.1 Deployment Models Summary

| Project | Process Manager | Config File | Auto-Restart on Failure | Auto-Start on Boot |
| :--- | :--- | :--- | :--- | :--- |
| `PYTHON-FACE-SCAN-XITO` | Docker Compose | `docker-compose.yml` | Yes (`restart: always`) | Yes (Docker daemon) |
| `community-chat` | Docker Compose | `docker-compose.yml` | Yes (`restart: always`) | Yes (Docker daemon) |
| `SUPABASE-EGRESS-BYPASS` | PM2 | `ecosystem.config.js` | Yes (`autorestart: true`) | Yes (PM2 startup daemon) |
| `zip-compressor` | Child Process | Managed by API server | Managed on-demand | No (Starts on request) |

### 7.2 Service Lifecycle Commands (Start / Stop / Restart)

#### Project 1: PYTHON-FACE-SCAN-XITO
```bash
cd /home/worksuvash/PYTHON-FACE-SCAN-XITO

# View status of all containers
docker compose ps

# View live container logs
docker compose logs -f --tail=100 [api-server | face-service | postgres | qdrant | traefik]

# Restart specific service (e.g., API server)
docker compose restart api-server

# Full graceful restart of all containers
docker compose down && docker compose up -d
```

#### Project 2: community-chat
```bash
cd /home/worksuvash/community-chat

# View status of upload service and Nginx
docker compose ps

# View live logs
docker compose logs -f --tail=100 vps-upload-service

# Restart the upload stack
docker compose restart
```

#### Project 3: SUPABASE-EGRESS-BYPASS (PM2)
```bash
# View live PM2 dashboard and process metrics
pm2 status
pm2 monit

# View logs for both API and Dashboard
pm2 logs --lines 100

# Restart API server
pm2 restart egress-api-server

# Restart Dashboard
pm2 restart egress-dashboard

# Save PM2 state to persist on server reboot
pm2 save
```

### 7.3 Updating Source Code & Rebuilding Services

#### Updating PYTHON-FACE-SCAN-XITO:
```bash
cd /home/worksuvash/PYTHON-FACE-SCAN-XITO
git pull origin main
# Rebuild containers with updated source code
docker compose build api-server frontend face-service
docker compose up -d --remove-orphans
```

#### Updating community-chat:
```bash
cd /home/worksuvash/community-chat
git pull origin main
# Rebuild Fastify container and restart
docker compose build vps-upload-service
docker compose up -d
```

#### Updating SUPABASE-EGRESS-BYPASS:
```bash
cd /home/worksuvash/SUPABASE-EGRESS-BYPASS
git pull origin main

# If TypeScript patch errors occur, run the auto-fix script:
node /home/worksuvash/scripts/apply_fixes.mjs

# Or manual build:
pnpm install
pnpm run build
pm2 reload egress-api-server
pm2 reload egress-dashboard
```

---

## 8. Logs, Monitoring & Observability

### 8.1 Log Locations Matrix

| Log Component | File / Stream Location | Retention / Rotation |
| :--- | :--- | :--- |
| **Traefik Ingress Logs** | `docker logs traefik` | Docker container log driver |
| **Upload Nginx Access Log**| `docker logs vps-upload-nginx` | Docker container log driver |
| **Fastify Upload App Log** | `docker logs vps-upload-service` | JSON logs via Pino logger |
| **Face API Server Log** | `docker logs python-face-scan-xito-api-server-1` | Container stdout / Docker driver |
| **Face Service (Flask)** | `docker logs python-face-scan-xito-face-service-1` | Container stdout / Gunicorn |
| **PM2 Egress API Error** | `/root/.pm2/logs/egress-api-server-error.log` | Managed by PM2 |
| **PM2 Egress API Out** | `/root/.pm2/logs/egress-api-server-out.log` | Managed by PM2 |
| **PM2 Dashboard Error** | `/root/.pm2/logs/egress-dashboard-error.log` | Managed by PM2 |
| **PM2 Dashboard Out** | `/root/.pm2/logs/egress-dashboard-out.log` | Managed by PM2 |
| **Auto-Fix Script Log** | `/home/worksuvash/scripts/auto_fix_sunday.log` | Appending log file |
| **System Auth & Syslog** | `/var/log/auth.log`, `/var/log/syslog` | Managed by `logrotate` |

### 8.2 Health Checks & Monitoring Endpoints

- **community-chat Health:**
  `curl -s https://chat-media.xitoevent.com/health | jq`
  Returns service uptime, writable storage validation, used bytes, quota limits, and storage lock state.
- **community-chat Ping:**
  `curl -s https://chat-media.xitoevent.com/api/v1/ping`
- **Egress API Server Health:**
  `curl -s https://api-egress.weddingtalesnepal.com/health`
- **Qdrant Vector Health:**
  `curl -s http://127.0.0.1:6333/collections`
- **Postgres Database Ping:**
  `docker exec postgres pg_isready -U faceapp -d facedb`

### 8.3 Live Observability Commands

```bash
# Real-time resource monitor (CPU/RAM per container)
docker stats --no-stream

# PM2 live metrics & CPU/heap usage
pm2 monit

# Inspect top 10 memory-consuming processes
ps -eo pid,user,%cpu,%mem,rss,command --sort=-%mem | head -n 11

# Check current swap and disk usage
free -h && df -h /
```

---

## 9. Comprehensive Security Audit & Vulnerabilities

> [!IMPORTANT]
> This audit was performed in **read-only** mode. No files, firewalls, or containers were modified. The findings below highlight real risks discovered on the server.

### 9.1 Critical Security Vulnerabilities

#### âœ… 1. [RESOLVED] Qdrant Vector DB Secured to Localhost & UFW Rule Removed
- **Action Taken:** In `/home/worksuvash/PYTHON-FACE-SCAN-XITO/docker-compose.yml`, Qdrant port mapping was changed to `127.0.0.1:6333:6333`, and the container was recreated with zero data loss. The open firewall rule (`6333/tcp ALLOW IN Anywhere`) was deleted from UFW.
- **Verification:** All 368,782 vectors verified intact. Public WAN attempts to reach port 6333 are instantly refused, while internal container communication across `faceapp-network` continues normally.

#### âœ… 2. [RESOLVED] Hardcoded Secrets Parameterized & Removed from Git
- **Action Taken:** In `/home/worksuvash/PYTHON-FACE-SCAN-XITO/docker-compose.yml`, all database passwords, Cloudflare R2 credentials, Supabase keys, and admin passwords were migrated into `.env` and parameterized via `${VAR}` references.
- **Git Hardening:** `.env` was removed from git tracking (`git rm --cached .env`), added to `.gitignore`, and committed. The production running containers remain 100% operational without downtime.
- **Pending Action for Admin:** Rotate the exposed Cloudflare R2 API keys and Supabase service role keys at your convenience.

### 9.2 High Security Risks

#### âœ… 3. [RESOLVED] Upload Service & Nginx Ports Restricted from WAN
- **Action Taken:** In `/home/worksuvash/community-chat/docker-compose.yml`, `vps-upload-service` port 3001 was bound to `127.0.0.1:3001:3001`, and `vps-upload-nginx` port 8081 was bound to `172.18.0.1:8081:80` and `127.0.0.1:8081:80`.
- **Impact:** External internet scanners connecting to `http://167.233.45.100:8081/admin` or `:3001` are now immediately rejected (`Connection refused`). All traffic must traverse Traefik over HTTPS.
- **Impact:** Attackers can bypass Traefik SSL and rate-limiting to directly target internal services.
- **Remediation:** Bind container ports to `127.0.0.1:<PORT>:<PORT>` instead of `<PORT>:<PORT>`, or configure `ufw-docker` rules in the `DOCKER-USER` iptables chain.

#### âœ… 4. [HARDENED] Fail2ban Installed & Active + RSA Key Support Added
- **Action Taken:** `fail2ban` was installed and configured with a dedicated `sshd` jail. It actively monitors port 22, bans aggressive bots for 1 hour after 5 failed attempts, and whitelists the developer subnet (`27.34.66.0/24`) to guarantee zero accidental lockouts.
- **Compatibility:** Added `PubkeyAcceptedAlgorithms +ssh-rsa` to `/etc/ssh/sshd_config.d/pubkey.conf` so modern and legacy SSH keys are accepted.
- **Password Auth Status:** Kept enabled under Fail2ban protection because the active developer login currently uses password authentication. Once key-based login is independently confirmed, password authentication can be disabled.

#### âš ï¸ 5. PM2 Services Bound to 0.0.0.0
- **Finding:** PM2 processes `egress-api-server` (port 5000) and `egress-dashboard` (port 5173) bind to `0.0.0.0` instead of `127.0.0.1`.
- **Impact:** External users can access the API and Dashboard directly via `http://167.233.45.100:5000` without passing through Traefik's TLS or domain verification.
- **Remediation:** Set `HOST=127.0.0.1` in `/home/worksuvash/SUPABASE-EGRESS-BYPASS/.env`.

### 9.3 Medium & Low Security Risks

- **PM2 Running as Root:** Node.js applications under PM2 run under UID 0 (`root`). If a remote code execution vulnerability occurs in any npm package, the attacker gains root privileges over the host. Should run under user `worksuvash`.
- **Watchtower Docker Socket Access:** Watchtower mounts `/var/run/docker.sock` in read-write mode. A vulnerability in Watchtower grants full host takeover.
- **Local Uncommitted Git Changes:** Both `community-chat` and `SUPABASE-EGRESS-BYPASS` have unstaged file edits that have not been pushed to version control, posing a risk of accidental data loss during updates.

---

## 10. Performance Analysis & Resource Bottlenecks

### 10.1 RAM & Swap Utilization Analysis

- **Total RAM:** 3.7 GiB | **Used:** 2.4 - 2.7 GiB | **Free:** 170 - 750 MiB
- **Swap Space:** 4.0 GiB | **Used:** 1.3 GiB (32.5% swap usage)
- **Root Cause of Memory Pressure:**
  1. `egress-api-server` (PID 2283678): Runs with `--max-old-space-size=4096` and consumes **1.28 GB RSS (32.7% of physical RAM)**. Because the VPS only has 3.7 GB total RAM, allocating 4 GB to a single Node.js process forces the Linux kernel to swap out inactive memory pages.
  2. `face-service`: Loads deep learning models (`buffalo_l`) into memory with ONNX Runtime, requiring ~400â€“600 MB.
  3. `qdrant`: Maintains the HNSW vector graph in memory for fast indexing of 368k vectors, using ~250â€“350 MB.
- **Recommendation:** Lower Node's memory cap for `egress-api-server` to `--max-old-space-size=768`.

### 10.2 CPU Load & Process Profiling

- **CPUs:** 2 vCPU cores (AMD EPYC-Genoa @ 2.0 GHz)
- **1-Minute Load Average:** ~0.30 - 0.70 (Well within healthy limits during idle/normal traffic)
- **CPU Spikes:** CPU utilization spikes to 90â€“100% during bulk face extraction jobs (InsightFace ONNX executing facial landmark alignment on CPU) and during `zip-compressor` streaming (Rayon multi-threaded deflate compression).

### 10.3 Disk & I/O Footprint

- **Storage Health:** 34 GB used out of 75 GB (47%). Excellent remaining capacity (39 GB free).
- **Disk I/O Bottlenecks:** PostgreSQL table `request_logs` has accumulated **74,111 rows** and is continuously growing. Without log purging or table partitioning, write I/O and disk size will steadily increase.

---

## 11. Backup & Disaster Recovery Procedures

### 11.1 Current Backup Posture

> [!WARNING]
> **No automated database or volume backups currently exist.**  
> - `/backup` does not exist.
> - `/opt/xito-backend/backups` is empty.
> - No cron jobs exist for `pg_dump` or Qdrant snapshots.
> If the VPS disk fails or an accidental deletion occurs, database and local media data will be permanently lost.

### 11.2 PostgreSQL Backup & Restore Playbook

#### Create an Instant Backup:
```bash
# Export facedb to a compressed SQL dump
docker exec -t postgres pg_dump -U faceapp -d facedb | gzip > /home/worksuvash/backup_facedb_$(date +%F_%H%M%S).sql.gz

# Verify backup size
ls -lh /home/worksuvash/backup_facedb_*.sql.gz
```

#### Restore from Backup:
```bash
# Decompress and restore into database
gunzip -c /home/worksuvash/backup_facedb_2026-09-09.sql.gz | docker exec -i postgres psql -U faceapp -d facedb
```

### 11.3 Qdrant Snapshot & Restore Playbook

#### Create Snapshot:
```bash
# Trigger collection snapshot via REST API
curl -X POST "http://127.0.0.1:6333/collections/face_embeddings/snapshots"

# Locate snapshot file on disk:
ls -lh /home/worksuvash/PYTHON-FACE-SCAN-XITO/artifacts/face-service/qdrant_storage/snapshots/
```

### 11.4 Upload Media Volume Backup Playbook

#### Backup `vps_upload_data` Docker Volume:
```bash
# Archive upload volume into a tarball
tar -czvf /home/worksuvash/backup_uploads_$(date +%F).tar.gz -C /var/lib/docker/volumes/community-chat_vps_upload_data/_data .
```

#### Restore Upload Volume:
```bash
tar -xzvf /home/worksuvash/backup_uploads_2026-09-09.tar.gz -C /var/lib/docker/volumes/community-chat_vps_upload_data/_data
```

### 11.5 Complete Server Disaster Recovery Plan

If the Hetzner server is destroyed or compromised, execute these steps on a fresh Ubuntu 24.04 server:
1. **Provision New VPS & Install Runtimes:**
   ```bash
   apt update && apt upgrade -y
   curl -fsSL https://get.docker.com | sh
   apt install -y nodejs npm git postgresql-client
   npm install -g pnpm pm2
   ```
2. **Clone Repositories into `/home/worksuvash`:**
   ```bash
   mkdir -p /home/worksuvash && cd /home/worksuvash
   git clone https://github.com/worksuvashxito-dev/PYTHON-FACE-SCAN-XITO.git
   git clone https://github.com/worksuvashxito-dev/community-chat.git
   git clone https://github.com/worksuvashxito-dev/SUPABASE-EGRESS-BYPASS.git
   ```
3. **Restore Environment Files (`.env`):**
   Copy backed-up `.env` files into each directory.
4. **Restore Database & Qdrant Snapshots:**
   Follow steps 11.2, 11.3, and 11.4.
5. **Start Docker Stacks & PM2:**
   ```bash
   cd /home/worksuvash/PYTHON-FACE-SCAN-XITO && docker compose up -d
   cd /home/worksuvash/community-chat && docker compose up -d
   cd /home/worksuvash/SUPABASE-EGRESS-BYPASS
   pnpm install && pnpm run build
   pm2 start ecosystem.config.js && pm2 save
   ```
6. **Update DNS Records:**
   Point `search.weddingtalesnepal.com`, `chat-media.xitoevent.com`, and `*-egress.weddingtalesnepal.com` to the new VPS IP address. Traefik will automatically provision new SSL certificates upon first request.

---

## 12. Recommended Infrastructure Improvements

1. **Secure Qdrant Vector DB Immediately:**
   Modify `PYTHON-FACE-SCAN-XITO/docker-compose.yml`:
   ```yaml
   ports:
     - "127.0.0.1:6333:6333" # Change from "6333:6333"
   ```
2. **Implement Daily Automated Backup Cron:**
   Create `/home/worksuvash/scripts/daily_backup.sh` that dumps PostgreSQL, creates a Qdrant snapshot, syncs to Cloudflare R2 or an S3 bucket, and deletes dumps older than 14 days.
3. **Bind PM2 Apps to Localhost:**
   In `/home/worksuvash/SUPABASE-EGRESS-BYPASS/.env`, configure `HOST=127.0.0.1` and `PORT=5000` to prevent direct WAN exposure on ports 5000 and 5173.
4. **Harden SSH Daemon:**
   - Set `PasswordAuthentication no` in `/etc/ssh/sshd_config`.
   - Install and enable fail2ban: `apt install -y fail2ban && systemctl enable --now fail2ban`.
5. **Tune Node.js Memory Limits:**
   Reduce `--max-old-space-size=4096` in `SUPABASE-EGRESS-BYPASS` to `768` to eliminate swap memory thrashing.
6. **Clean Up Abandoned Projects:**
   Safely remove `/opt/minio`, `/opt/supabase`, `/opt/xito-backend`, and `/etc/nginx/sites-available/deaeration` to eliminate developer confusion and save disk space.
7. **Prune Request Logs:**
   Set up a weekly cron to truncate or delete rows older than 30 days from PostgreSQL table `request_logs`.

---

## 13. What a New Developer Needs to Know

### 13.1 Server Mental Model in 3 Minutes

- **Everything web-facing enters through Traefik.** Port 80 and 443 are owned by the Traefik Docker container. If you install an app or reverse proxy on the host, **do not listen on 80/443**. Instead, listen on an internal port (e.g. 5000, 3001) and add a rule to Traefik.
- **Docker Compose runs the infrastructure:** Postgres, Qdrant, Face-service, and the Chat Upload service are all Docker containers in `/home/worksuvash/PYTHON-FACE-SCAN-XITO/` and `/home/worksuvash/community-chat/`.
- **PM2 runs the Egress tools:** The Supabase egress proxy API and its admin dashboard run directly on Node.js on the host via `pm2`.
- **Certificates are 100% automated:** As long as DNS points to `167.233.45.100` and you register a router with `certResolver: "letsencrypt"` in `dynamic.yml` or Docker labels, Traefik handles SSL generation automatically.

### 13.2 The Golden Rules (What NEVER to do)

1. **NEVER start the host Nginx service:** `systemctl start nginx` will fail or hijack port 80/443, taking down all SSL websites and breaking Traefik.
2. **NEVER run `docker system prune -a --volumes`:** This will wipe the persistent PostgreSQL data, Qdrant vector database, and user chat uploads volume!
3. **NEVER publish Docker ports to `0.0.0.0` without authentication:** Docker bypasses UFW. If you publish a database or Redis port, it is instantly exposed to the world.
4. **NEVER edit files directly in `dist/`:** All frontend and backend apps are compiled TypeScript/Vite projects. Edit files in `src/` and build with `pnpm run build` or `docker compose build`.
5. **NEVER delete `/home/worksuvash/PYTHON-FACE-SCAN-XITO/acme.json`:** This file holds all Let's Encrypt private keys and certificates. Deleting it may cause Let's Encrypt rate-limiting when re-issuing certificates for all domains.

### 13.3 Quick Command Reference Sheet

```bash
# Check overall server health
free -h && df -h / && uptime

# Check Docker containers
docker ps -a

# Check PM2 applications
pm2 list

# View live Traefik routing logs
docker logs -f --tail=50 traefik

# Connect directly to Postgres
docker exec -it postgres psql -U faceapp -d facedb

# Check Qdrant collections
curl -s http://127.0.0.1:6333/collections

# Rebuild and restart Egress bypass after code changes
node /home/worksuvash/scripts/apply_fixes.mjs
```

### 13.4 How to Add a New Domain / Subdomain

1. Point the DNS A record for `newdomain.com` to `167.233.45.100`.
2. Open `/home/worksuvash/PYTHON-FACE-SCAN-XITO/dynamic.yml`.
3. Add a new router and service definition:
   ```yaml
   http:
     routers:
       my-new-service:
         rule: "Host(`newdomain.com`)"
         entryPoints:
           - "websecure"
         service: "my-new-service-backend"
         tls:
           certResolver: "letsencrypt"

     services:
       my-new-service-backend:
         loadBalancer:
           servers:
             - url: "http://172.18.0.1:<YOUR_PORT>"
   ```
4. Save the file. Traefik automatically watches `dynamic.yml`, loads the config instantly without restarting, and fetches the SSL certificate.

### 13.5 Top 5 Outage Scenarios & Fixes

| Symptom / Outage | Probable Cause | Immediate Troubleshooting Step |
| :--- | :--- | :--- |
| **All HTTPS sites show `502 Bad Gateway`** | Traefik is running, but downstream target container or PM2 process died. | Run `docker ps` and `pm2 status`. Check if the specific backend service is online. |
| **Sites show `Connection Refused` on port 443** | Traefik container crashed or port 443 conflict. | Run `docker logs traefik` and check `ss -tulpn \| grep 443`. Restart Traefik: `docker compose -f /home/worksuvash/PYTHON-FACE-SCAN-XITO/docker-compose.yml restart traefik`. |
| **Uploads return `413 Payload Too Large` or `507 Storage Locked`** | Upload exceeds 20MB or 20GB volume quota reached. | Check `curl https://chat-media.xitoevent.com/health`. If quota locked, archive older files from `/var/uploads/xito` or increase quota in `community-chat/.env`. |
| **Server sluggish / Out Of Memory (OOM)** | Node.js process `egress-api-server` or Face service consumed too much RAM, causing high swap usage. | Run `pm2 restart egress-api-server`. Check `free -h` and consider dropping memory limits in PM2 args. |
| **Postgres connection error (`ECONNREFUSED 5432`)** | `postgres` container crashed or restarting. | Run `docker logs postgres`. Verify disk space (`df -h /`). Ensure `/home/worksuvash/PYTHON-FACE-SCAN-XITO/postgres_data` permissions are intact. |

---
*End of Technical Documentation â€” Document maintained in `/home/worksuvash/VPS_AND_PROJECT_COMPLETE_DOCUMENTATION.md`.*
