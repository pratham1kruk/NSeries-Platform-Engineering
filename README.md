# NSeries Platform Engineering

> Production-grade AWS infrastructure, self-managed Kubernetes, Ansible automation, Helm packaging, and Prometheus/Grafana monitoring — deployed around a suite of original Docker-first web scraping tools built and published by Pratham Kumar Uikey.

![AWS](https://img.shields.io/badge/AWS-EC2-orange?style=flat-square&logo=amazonaws)
![Kubernetes](https://img.shields.io/badge/Kubernetes-v1.29-blue?style=flat-square&logo=kubernetes)
![Ansible](https://img.shields.io/badge/Ansible-Automation-red?style=flat-square&logo=ansible)
![Helm](https://img.shields.io/badge/Helm-v3-blueviolet?style=flat-square&logo=helm)
![Prometheus](https://img.shields.io/badge/Prometheus-Monitoring-orange?style=flat-square&logo=prometheus)
![Grafana](https://img.shields.io/badge/Grafana-Dashboards-yellow?style=flat-square&logo=grafana)
![Docker](https://img.shields.io/badge/Docker-Images-0db7ed?style=flat-square&logo=docker)

---

## Architecture

```
DockerHub (pratham1uk/*nscrape-*)       ← Pratham Kumar Uikey's own published images
        ↓
AWS EC2 — 3x m7i-flex.large instances
        ↓
Ansible — automated node preparation
        ↓
kubeadm — Kubernetes cluster bootstrap
        ↓
Kubernetes — 1 control plane + 2 workers
        ↓
Helm — packaged application deployments
        ↓
LetsNScrape + SnapNScrape — frontend(s) + backend(s) + redis
        ↓
Prometheus + Grafana — monitoring & alerting
```

ytNscrape (9-service pipeline) is deployed separately via Docker Compose on a standalone EC2 instance — see [ytNscrape Deployment](#ytnscrape-deployment-docker-compose) below for the reasoning.

---

## Project Objective

This project demonstrates the complete DevOps lifecycle — from raw cloud infrastructure to a monitored, production-style Kubernetes deployment — without using any managed Kubernetes service (no EKS, no GKE). Every component is built and configured manually to showcase deep understanding of each layer, including knowing when *not* to use Kubernetes.

The workloads deployed are not placeholder apps. They are original, independently authored tools from the [NScrape Series](https://github.com/pratham1kruk/Nscrape_Series) — fully containerised web scraping applications designed, built, and published to DockerHub by Pratham Kumar Uikey under the `pratham1uk` namespace. This project uses those real published images as the deployment target, demonstrating both the infrastructure engineering and the underlying applications it runs.

---

## Demo

### Video Walkthroughs

| Video | What it covers |
|---|---|
| `Nseries-platform-complete-demo.mp4` | Full end-to-end walkthrough — AWS console, Ansible runs, cluster formation, Helm deployments, running apps, Grafana/Prometheus |
| `letsNscrape-aws-ec2-kubernetes-deployment.mp4` | LetsNScrape Kubernetes deployment detail |
| `snapNscrape-aws-ec2-kubernetes-deployment.mp4` | SnapNScrape Kubernetes deployment detail |
| `ytNscrape-aws-ec2-kubernetes-deployment.mp4` | ytNscrape Docker Compose deployment detail |
| `grafana-monitoring-dashboard-demo.mp4` | Grafana dashboard walkthrough |
| `prometheus-monitoring-and-alerting-demo.mp4` | Prometheus alerts and rules |

### Demo videos are hosted externally because GitHub's repository size and file size limits are not suitable for high-resolution deployment recordings. Screenshots and reports remain available within the repository. where should i add this note

### Screenshots

Cluster, app UIs, Grafana dashboard, Prometheus alerts page, AWS security group, and EC2 monitoring metrics — all in `assets/images/`.

Key screenshots:

| File | What it shows |
|---|---|
| `Nseries-ec2-deployment.png` | All instances running on AWS |
| `kubernetes-master-node-helm-overview.png` | Helm releases and cluster state |
| `letsNscrape-kubernetes-ec2-deployment.png` | LetsNScrape live on cluster |
| `snapNscrape-kubernetes-ec2-deployment.png` | SnapNScrape live on cluster |
| `ytNscrape-kubernetes-ec2-deployment.png` | ytNscrape on Docker Compose host |
| `grafana_dashboard_01/02/03.png` | Grafana monitoring views |
| `prometheus-alert-rule-triggered.png` | Alert firing in Prometheus |
| `aws-security-group-inbound-rules.png` | Security group configuration |

---

## What is the NScrape Series?

The NScrape Series is a collection of original, purpose-built web scraping tools designed and published by Pratham Kumar Uikey. Each tool ships exclusively as Docker images under the `pratham1uk` namespace on DockerHub — no source distribution, no build step required. Source code for each tool lives in private repositories; the [Nscrape_Series](https://github.com/pratham1kruk/Nscrape_Series) public repo contains usage documentation and Docker Compose files for running the tools locally.

This project uses those published images directly as the workload — demonstrating real deployment of real software, not toy applications.

| Tool | Purpose | Images | Containers |
|---|---|---|---|
| 🕷️ **LetsNScrape** | General-purpose web scraper — any URL, text/images/links/tables/metadata, dual HTTP+Playwright engine | `pratham1uk/letsnscape-backend:v1.0.1` · `pratham1uk/letsnscape-frontend:v1.0.1` | 3 (backend + frontend + Redis) |
| 📸 **SnapNScrape** | Web scraper with OCR failsafe — falls back to Playwright screenshot + Tesseract when HTTP and JS scraping fail | `pratham1uk/snapnscrape-gateway:1.0.0` · `pratham1uk/snapnscrape-scraper:1.0.0` | 3 (gateway + scraper + Redis) |
| ⚡ **ytNscrape** | YouTube lecture scraper — downloads video, deduplicates slide frames by content type (code/UI/slides), syncs captions, outputs study PDF + ZIP | 9 service images (`svc-ingest`, `svc-dedup`, `svc-priority`, `svc-postprocess`, `svc-captions`, `svc-render-zip`, `svc-render-pdf`, `gateway`, `frontend`) | 9+ |

All 13 Docker images (across the three tools) are authored and maintained by Pratham Kumar Uikey at [hub.docker.com/u/pratham1uk](https://hub.docker.com/u/pratham1uk).

---

## Tech Stack

| Layer | Technology |
|---|---|
| Cloud | AWS EC2 |
| OS | Ubuntu 22.04 LTS |
| Automation | Ansible 2.20 |
| Container Runtime | containerd |
| Cluster Bootstrap | kubeadm v1.29 |
| Networking | Calico CNI |
| Package Manager | Helm v3 |
| Applications | LetsNScrape, SnapNScrape (Kubernetes) · ytNscrape (Docker Compose) |
| Monitoring | Prometheus + Grafana (kube-prometheus-stack) |
| Alerting | PrometheusRule CRDs |

---

## Infrastructure Setup

### EC2 Instance Configuration (Kubernetes Cluster)

| Instance | Role | Type | Storage |
|---|---|---|---|
| k8s-master | Control Plane | m7i-flex.large | 20 GB |
| k8s-worker1 | Worker Node | m7i-flex.large | 20 GB |
| k8s-worker2 | Worker Node | m7i-flex.large | 20 GB |

**Why m7i-flex.large?**
kubeadm requires a minimum of 2 vCPU and 2 GB RAM. The m7i-flex.large provides 2 vCPU and 8 GB RAM, giving comfortable headroom for application pods plus the full Prometheus and Grafana monitoring stack.

### AMI Strategy

Rather than setting up all three instances independently, a base instance (`k8s-base`) was prepared first with all Kubernetes dependencies installed via Ansible. An AMI was then created from this base instance, and the two worker nodes were launched from that AMI. This approach:

- Eliminates configuration drift between nodes
- Reduces setup time significantly
- Ensures all nodes have identical base configuration

### Security Group (`k8s-sg`)

| Type | Port | Source | Purpose |
|---|---|---|---|
| SSH | 22 | My IP | Remote access |
| Custom TCP | 6443 | 0.0.0.0/0 | Kubernetes API server |
| Custom TCP | 30000-32767 | 0.0.0.0/0 | NodePort services |
| Custom TCP | 3000 | 0.0.0.0/0 | ytNscrape frontend |
| Custom TCP | 3082 | 0.0.0.0/0 | ytNscrape gateway API |
| All traffic | All | k8s-sg (self) | Inter-node communication |

The self-referencing rule on `k8s-sg` allows all nodes in the security group to communicate freely with each other — required for Kubernetes pod networking and control plane communication. The same security group was reused for the standalone ytNscrape instance to keep management simple.

### Handling Stop/Start Cost Optimization

To avoid idle billing, instances were stopped between work sessions rather than left running. AWS assigns a **new public IP** every time a stopped instance is started. The `inventory.ini` file (and any active SSH session) must be updated with the new IPs before resuming work. This was repeated multiple times throughout the project and is reflected in the troubleshooting log.

---

## Repository Structure

```
NSeries-Platform-Engineering/
├── ansible/
│   ├── inventory.ini
│   ├── inventory-ytnscrape.ini
│   └── playbooks/
│       ├── k8s-base.yml       # Install containerd, kubeadm, kubelet, kubectl, helm
│       ├── master-init.yml    # kubeadm init + Calico CNI + save join command
│       ├── workers-join.yml   # Join worker nodes to cluster
│       └── docker-setup.yml   # Install Docker + Compose plugin (ytNscrape host)
├── helm/
│   ├── letsnscape/
│   │   ├── Chart.yaml
│   │   ├── values.yaml
│   │   └── templates/
│   │       ├── redis.yaml
│   │       ├── backend.yaml
│   │       └── frontend.yaml
│   └── snapnscrape/
│       ├── Chart.yaml
│       ├── values.yaml
│       └── templates/
│           ├── redis.yaml
│           ├── scraper.yaml
│           └── gateway.yaml
├── monitoring/
│   └── alert-rules.yaml       # PrometheusRule for CPU, NodeDown, PodCrashLoop
├── ytnscrape/
│   └── docker-compose.yml     # Standalone deployment, separate EC2
├── assets/
│   ├── demo-links.md
│   ├── images/
│   │   ├── Nseries-ec2-deployment.png
│   │   ├── kubernetes-master-node-helm-overview.png
│   │   ├── letsNscrape-kubernetes-ec2-deployment.png
│   │   ├── letsNscrape-kubernetes-ec2-deployment-results.png
│   │   ├── snapNscrape-kubernetes-ec2-deployment.png
│   │   ├── ytNscrape-kubernetes-ec2-deployment.png
│   │   ├── grafana_overview.png
│   │   ├── grafana_dashboard_01/02/03.png
│   │   ├── prometheus_overview.png
│   │   ├── prometheus-alert-rule-triggered.png
│   │   ├── aws-security-group-inbound-rules.png
│   │   └── aws-ec2-monitoring-metrics_01/02.png
│   └── reports/
│       ├── letsNscrape-summary-report.pdf
│       ├── letsNscrape-summary-report.docx
│       └── ytNscrape-sample-notes.pdf
├── README.md
└── PERSONAL_DOCS.md            (gitignored — not pushed to GitHub)
```

---

## Phase 1 — AWS Infrastructure

1. Launched a single `k8s-base` instance with Ubuntu 22.04 LTS on m7i-flex.large with 20 GB storage
2. Created security group `k8s-sg` with required inbound rules
3. Ran Ansible to install all Kubernetes dependencies on the base instance
4. Created an AMI (`k8s-base-ami`) from the prepared instance
5. Launched two worker instances from the AMI
6. Renamed instances: `k8s-master`, `k8s-worker1`, `k8s-worker2`

---

## Phase 2 — Ansible Automation

Ansible was run from WSL2 on Windows. The inventory defines all three nodes and the playbooks automate the full cluster setup.

**`k8s-base.yml`** — apt update, disables swap, loads `overlay`/`br_netfilter` kernel modules, sets Kubernetes sysctl params, installs and configures containerd (`SystemdCgroup = true`), adds the Kubernetes apt repo, installs `kubeadm`/`kubelet`/`kubectl` at v1.29 (held), installs Helm.

**`master-init.yml`** — runs `kubeadm init` with pod CIDR `192.168.0.0/16`, copies `admin.conf` to the ubuntu user, installs Calico CNI, generates and saves the worker join command locally.

**`workers-join.yml`** — copies the join command to each worker and executes `kubeadm join`.

**`docker-setup.yml`** — installs Docker Engine and the Compose plugin on the standalone ytNscrape host (no Kubernetes involved for this one).

---

## Phase 3 — Kubernetes Cluster

```bash
kubectl get nodes
```

```
NAME               STATUS   ROLES           AGE    VERSION
ip-172-31-35-85    Ready    control-plane   7m     v1.29.15
ip-172-31-33-128   Ready    <none>          65s    v1.29.15
ip-172-31-34-123   Ready    <none>          68s    v1.29.15
```

All three nodes `Ready` on Kubernetes v1.29.15.

---

## Phase 4 — LetsNScrape Deployment (Helm)

LetsNScrape is Pratham Kumar Uikey's general-purpose web scraper — extracting text, images, links, metadata, headings, and tables from any URL via a dual-engine pipeline (fast HTTP scrape, automatic Playwright fallback for JS-heavy sites). Images published at `pratham1uk/letsnscape-*` on DockerHub.

| Component | Image | Internal Port |
|---|---|---|
| Redis | redis:7.2-alpine | 6380 |
| Backend | pratham1uk/letsnscape-backend:v1.0.1 | 3070 |
| Frontend | pratham1uk/letsnscape-frontend:v1.0.1 | 3071 |

Frontend exposed via NodePort `30071`. Backend and Redis are ClusterIP (internal only). The backend Service is named `backend` so the frontend's bundled nginx config can resolve the upstream hostname it expects.

```bash
helm install letsnscape ./helm/letsnscape
helm upgrade letsnscape ./helm/letsnscape   # after any change
kubectl get pods
kubectl get svc
```

Access:
```
http://<ANY_NODE_PUBLIC_IP>:30071
```

---

## Phase 5 — SnapNScrape Deployment (Helm)

SnapNScrape is Pratham Kumar Uikey's web scraper with an OCR failsafe — it attempts HTTP scraping first, falls back to Playwright, and if that fails, screenshots the page and runs Tesseract OCR to extract text. Images published at `pratham1uk/snapnscrape-*` on DockerHub. A second independent application was added to the same cluster to demonstrate multi-app hosting on one Kubernetes cluster.

| Component | Image | Internal Port |
|---|---|---|
| Redis | redis:7.2-alpine | 6379 |
| Scraper | pratham1uk/snapnscrape-scraper:1.0.0 | 8001 |
| Gateway | pratham1uk/snapnscrape-gateway:1.0.0 | 8000 (internally hardcoded; `PORT` env var has no effect in this image) |

Gateway exposed via NodePort `30031` (`30030` was already taken by Grafana). Scraper and Redis are ClusterIP, reached internally via Service names `scraper` and `redis`.

```bash
helm install snapnscrape ./helm/snapnscrape
kubectl get pods
```

Access:
```
http://<ANY_NODE_PUBLIC_IP>:30031
```

---

## Phase 6 — Monitoring

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace \
  --set grafana.service.type=NodePort \
  --set grafana.service.nodePort=30030 \
  --set prometheus.service.type=NodePort \
  --set prometheus.service.nodePort=30090
```

| Service | URL |
|---|---|
| Grafana | http://\<NODE_IP\>:30030 |
| Prometheus | http://\<NODE_IP\>:30090 |

Grafana credentials: `admin` / password retrieved from the Kubernetes secret.

Grafana dashboard ID `15760` (Kubernetes cluster overview) was imported to visualise CPU, memory, pod health, and node health across all three nodes. This dashboard is cluster-wide — it automatically covers every workload deployed afterward with no extra configuration.

---

## Phase 7 — Alerting

| Alert | Condition | Severity |
|---|---|---|
| HighCPUUsage | CPU > 80% for 2 minutes | warning |
| NodeDown | Node exporter unreachable for 1 minute | critical |
| PodCrashLooping | Pod restart rate > 0 for 2 minutes | warning |

```bash
kubectl apply -f monitoring/alert-rules.yaml
```

Visible at `http://<NODE_IP>:30090/alerts`.

---

## ytNscrape Deployment (Docker Compose)

ytNscrape is Pratham Kumar Uikey's YouTube lecture scraper — a 9-service pipeline that downloads a video, extracts and deduplicates slide frames by content type (code / software UI / slides) using OCR and a MiniLM embedding model, syncs captions, and renders output as a study-ready PDF and ZIP archive. It requires 4–8 GB RAM on its own and ships several large images including one that bundles a neural embedding model. Images published at `pratham1uk/ytnscrape-*` on DockerHub (9 service images total).

**Decision: deploy via Docker Compose on a separate, dedicated EC2 instance rather than adding it to the Kubernetes cluster.**

Reasoning:
- The existing cluster nodes had limited headroom left (≈6 GB RAM, ≈9–13 GB disk free per node) after running LetsNScrape, SnapNScrape, and the full Prometheus/Grafana stack
- Porting 9 services plus MinIO into Helm templates would add significant complexity for a tool that is explicitly designed and documented to run via `docker compose up -d`
- Demonstrates the practical engineering judgment of choosing the right orchestration tool for the resource profile and complexity of the workload, rather than defaulting to Kubernetes for everything

### Setup

- Dedicated EC2 instance (`ytnscrape-host`): m7i-flex.large, 30 GB storage, reused `k8s-sg` with two added rules (TCP 3000, TCP 3082)
- Docker Engine and Compose plugin installed via Ansible (`docker-setup.yml`)
- Production `docker-compose.yml` hand-built from the dev version (private repo), replacing each service's `build:` directive with the corresponding published `image:` tag from DockerHub, and removing the GPU reservation block (CPU fallback is automatic)

```bash
docker compose pull
docker compose up -d
docker compose ps
```

Access:
```
http://<YTNSCRAPE_INSTANCE_IP>:3000        # Frontend UI
http://<YTNSCRAPE_INSTANCE_IP>:3082/docs   # Gateway API docs
http://<YTNSCRAPE_INSTANCE_IP>:9001        # MinIO console
```

### Deployment Result

All 12 containers started and reached a running state. Gateway health confirmed:

```bash
curl http://localhost:3082/health
# {"status":"ok","redis":true,"service":"gateway"}
```

**Known limitation — YouTube IP-based rate limiting:** submitting a real YouTube URL produced `429`/`403` responses from YouTube via `yt-dlp` inside `svc-ingest`. This is YouTube's anti-bot defense against requests from cloud provider IP ranges and is unrelated to the deployment itself — the full pipeline (frontend → gateway → Redis → svc-ingest → yt-dlp) executed correctly before being blocked at YouTube's network layer. This is a well-documented constraint of running yt-dlp-based tools from datacenter IPs and is outside the infrastructure scope of this project.

---

## Troubleshooting

### Frontend CrashLoopBackOff on first deploy (LetsNScrape)

nginx referenced `backend` as the upstream hostname, but the Kubernetes Service was named `letsnscape-backend`. Fixed by renaming the Service to `backend` in the Helm template.

### Frontend calling localhost:3070 (LetsNScrape)

The React app was built with `VITE_API_URL=http://localhost:3070/api/v1` baked in at compile time, which fails inside a pod. Fixed by adding `ARG VITE_API_URL=/api/v1` to the frontend Dockerfile so the build uses a relative URL that nginx proxies correctly. Published as `v1.0.1` via GitHub Actions → DockerHub.

### local_action become error in Ansible

`Save join command locally` failed because `local_action` ran on WSL2 where `become: true` prompted for a sudo password. Fixed with `become: false` on that task.

### NodePort already allocated (SnapNScrape)

`30030` was already in use by Grafana's NodePort. Changed SnapNScrape gateway's NodePort to `30031`.

### Helm "cannot re-use a name that is still in use"

A failed `helm install` left a partial release object behind. Resolved with `helm uninstall snapnscrape` before retrying the install.

### ImagePullBackOff — wrong tag (SnapNScrape)

Images were tagged `1.0.0` on DockerHub, not `v1.0.0` as initially assumed. Verified the actual tag via the DockerHub API and corrected `values.yaml`.

### Gateway unreachable despite Running pod (SnapNScrape)

The gateway container hardcodes port `8000` internally and ignores the documented `PORT` environment variable. Verified via `kubectl describe deployment`, corrected `values.yaml`/`gateway.yaml` to target `8000`, and removed the ineffective `PORT` env var.

### docker-compose.yml 404 from GitHub (ytNscrape)

The ytNscrape source repo is private, unlike LetsNScrape/SnapNScrape which expose public compose files. Hand-built a production compose file from the dev/build version by replacing `build:` directives with the equivalent published `image:` tags.

### Gateway shows "(unhealthy)" despite working correctly (ytNscrape)

`docker compose ps` reported the gateway as unhealthy, but a direct curl to `/health` returned a correct response. The container's internal healthcheck probe likely fails because `curl` isn't installed inside that image — a cosmetic packaging quirk with no functional impact.

### YouTube rate-limiting / 403 errors (ytNscrape)

See known limitation above — YouTube's IP-based anti-bot defense against cloud datacenter ranges, not a deployment fault.

### Public IPs change on every instance restart

AWS assigns new public IPs each time a stopped instance restarts. `inventory.ini` and any open SSH commands must be updated with current IPs before running Ansible or connecting.

---

## Skills Demonstrated

- AWS EC2 provisioning and AMI management
- Security group design for multi-node clusters
- Kubernetes cluster deployment using kubeadm (no managed service)
- Infrastructure automation with Ansible across heterogeneous node roles
- Helm chart authoring and multi-application deployment on a shared cluster
- Docker container orchestration — both Kubernetes and Docker Compose, applied by workload profile
- Prometheus monitoring and PrometheusRule alerting
- Grafana dashboard configuration and import
- Multi-node Kubernetes administration
- Publishing and versioning Docker images to DockerHub (via GitHub Actions CI/CD)
- Systematic troubleshooting of real deployment issues: networking, build-time config baking, port conflicts, Helm state, private-repo asset reconstruction, third-party rate limiting

---

## DockerHub Images

All application images deployed in this project are authored and published by Pratham Kumar Uikey at [hub.docker.com/u/pratham1uk](https://hub.docker.com/u/pratham1uk). Source code for each tool is maintained in private repositories; the public [Nscrape_Series](https://github.com/pratham1kruk/Nscrape_Series) repo contains full usage documentation.

| Tool | Image | Tag |
|---|---|---|
| LetsNScrape | `pratham1uk/letsnscape-backend` | `v1.0.1` |
| LetsNScrape | `pratham1uk/letsnscape-frontend` | `v1.0.1` |
| SnapNScrape | `pratham1uk/snapnscrape-gateway` | `1.0.0` |
| SnapNScrape | `pratham1uk/snapnscrape-scraper` | `1.0.0` |
| ytNscrape | `pratham1uk/ytnscrape-gateway` | `v1.0.0` |
| ytNscrape | `pratham1uk/ytnscrape-frontend` | `v1.0.0` |
| ytNscrape | `pratham1uk/ytnscrape-svc-ingest` | `v1.0.0` |
| ytNscrape | `pratham1uk/ytnscrape-svc-dedup` | `v1.0.0` |
| ytNscrape | `pratham1uk/ytnscrape-svc-priority` | `v1.0.0` |
| ytNscrape | `pratham1uk/ytnscrape-svc-postprocess` | `v1.0.0` |
| ytNscrape | `pratham1uk/ytnscrape-svc-captions` | `v1.0.0` |
| ytNscrape | `pratham1uk/ytnscrape-svc-render-zip` | `v1.0.0` |
| ytNscrape | `pratham1uk/ytnscrape-svc-render-pdf` | `v1.0.0` |

---

## Cost

Kubernetes cluster: 3x m7i-flex.large at ~$0.10/hr, run intermittently across multiple sessions (stopped between sessions to avoid idle billing).
ytNscrape: 1x additional m7i-flex.large, run only for its deployment/demo window.

All instances terminated after final documentation and demo recording. Ongoing cost: zero.

---

*Built by Pratham Kumar Uikey — NSeries Platform Engineering · [NScrape Series on DockerHub](https://hub.docker.com/u/pratham1uk) · [Nscrape_Series Docs](https://github.com/pratham1kruk/Nscrape_Series)*