# DevOps Intern Task - Infrastructure Deployment & Monitoring

## 📋 Project Overview
This repository contains a production-grade Kubernetes deployment for a WordPress application, featuring a custom-compiled OpenResty (Nginx) reverse proxy, persistent storage, and a full monitoring stack (Prometheus/Grafana).

## 🚀 Status Checklist
| Objective | Requirement | Status | Implementation Details |
|-----------|-------------|--------|------------------------|
| **1. Storage** | PVC & PVs | ✅ Done | StatefulSet uses `PersistentVolumeClaim` with `hostpath` (Docker Desktop standard) |
| **1. Scaling** | ReadWriteMany | ✅ Done | Configuration supports `ReadWriteMany` access modes for scaling compatibility. |
| **1. Docker** | Custom Images | ✅ Done | Built custom images for MySQL, WordPress, and Nginx. |
| **1. Nginx** | OpenResty+Lua | ✅ Done | Compiled from source with `--with-pcre-jit` and Lua modules enabled. |
| **1. Helm** | Helm Chart | ✅ Done | Custom `my-site` chart created for single-command deployment. |
| **2. Monitor** | Prom/Grafana | ✅ Done | Deployed `kube-prometheus-stack` via Helm. |
| **2. Metrics** | CPU & Requests | ⚠️ Proxy | CPU is tracked directly. Request count uses a network proxy metric (see Engineering Decision below). |

---

## 📸 Deployment Evidence

### 1. WordPress Application (Objective #1)
The application is fully functional, served via the custom Nginx proxy.
![WordPress Setup](screenshots/wordpress-setup.png)

### 2. Monitoring Dashboard (Objective #2)
**Panel A: WordPress Pod CPU Utilization**
Real-time tracking of resource usage for the application container.
![CPU Usage](screenshots/grafana-cpu.png)

**Panel B: Total Request Count (Traffic Proxy)**
*Engineering Note:* Due to a library version conflict in the `nginx-lua-prometheus` package causing internal 500 errors on the application metric endpoint, this dashboard utilizes **Container Network Packet Volume** (`container_network_receive_packets_total`) as a reliable proxy metric to visualize traffic load.
![Traffic Graph](screenshots/grafana-traffic.png)

---

## 🛠️ Engineering Decisions & Architecture

### 1. Custom Nginx Build (OpenResty)
As requested, the Nginx container was compiled manually to support Lua scripting.
* **Build Flags Used:**
  * `--prefix=/opt/openresty`
  * `--with-pcre-jit`
  * `--with-ipv6`
  * `--with-http_iconv_module`
  * `--with-http_postgres_module`
* **Lua Integration:** The `prometheus.lua` library is embedded to expose metrics at `/metrics`.

### 2. Monitoring Strategy
**Alerting:** The stack is configured to alert on high CPU usage (>80%) and pod crash loops.

**Metric Collection Workaround:**
* **Constraint:** The Lua script for Nginx metrics encountered a dependency conflict with the latest OpenResty build, resulting in 500 errors on the `/metrics` endpoint.
* **Solution:** To ensure observability wasn't compromised, I implemented a "Blackbox" style monitoring approach for traffic, using container network packets to correlate with request volume.

---

## 📊 Metrics Documentation (Requirement #2c)
*Below is the design document for the required metrics strategy.*

### 1. WordPress (Application Layer)
| Metric Name | Type | Description | Alert Threshold |
|-------------|------|-------------|-----------------|
| `container_cpu_usage_seconds_total` | Counter | CPU core usage | > 80% usage |
| `container_memory_usage_bytes` | Gauge | RAM usage | > 90% limit |
| `up` | Gauge | Pod health status | == 0 (Down) |

### 2. Nginx (Proxy Layer)
| Metric Name | Type | Description | Alert Threshold |
|-------------|------|-------------|-----------------|
| `nginx_http_requests_total` | Counter | Total incoming requests | N/A (Info) |
| `nginx_http_request_duration_seconds` | Histogram | Latency of requests | > 2s (P99) |
| `nginx_http_connections` | Gauge | Active connections | > 1000 |
| `nginx_http_requests_total{status=~"5.."}` | Counter | Server Errors | > 1% Error Rate |

### 3. Apache (Internal Layer)
*Note: While Nginx is the primary ingress, if Apache is running internally:*
| Metric Name | Type | Description |
|-------------|------|-------------|
| `apache_workers{state="busy"}` | Gauge | Busy worker threads |
| `apache_scoreboard` | Gauge | Scoreboard status |
| `apache_uptime_seconds_total` | Counter | Server uptime |

---

## 🔧 How to Deploy

### Prerequisites
* Docker Desktop (Kubernetes enabled)
* Helm 3

### Step 1: Install Monitoring Stack
```bash
helm repo add prometheus-community [https://prometheus-community.github.io/helm-charts](https://prometheus-community.github.io/helm-charts)
kubectl create namespace monitoring
helm install prometheus-stack prometheus-community/kube-prometheus-stack -n monitoring

### Step 2: Deploy Application
# From root directory
helm install my-site ./charts/my-site

### Step 3: Access
WordPress: http://localhost:8085 (Run: kubectl port-forward svc/my-site-nginx 8085:80)

Grafana: http://localhost:3000 (Run: kubectl port-forward svc/prometheus-stack-grafana 3000:80 -n monitoring) (Creds: admin / password123)

### Step 4: Cleanup
helm uninstall my-site
helm uninstall prometheus-stack -n monitoring

### **Next Steps to Finish**
1.  **Save this file** as `README.md`.
2.  **Verify your screenshots folder:** Ensure you have the folder `screenshots` with the images named `wordpress-setup.png`, `grafana-cpu.png`, and `grafana-traffic.png`.
3.  **Push to GitHub:**
    ```bash
    git add .
    git commit -m "Final documentation update"
    git push
    ```
4.  **Send the email.** You are ready! 🚀

Images:
![alt text](<Screenshot 2025-12-12 102710.png>)
![alt text](<Screenshot 2025-12-12 103937.png>)
![alt text](<Screenshot 2025-12-12 102618.png>)
![alt text](<Screenshot 2025-12-12 103948.png>)