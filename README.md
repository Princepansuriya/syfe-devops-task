# Infrastructure & DevOps Intern Task - Syfe

## 📋 Project Summary
This repository contains a production-grade deployment of a WordPress application on Kubernetes. The project demonstrates a full GitOps workflow, including custom Docker image compilation (OpenResty), Helm chart orchestration, and a persistent storage strategy.

Additionally, a full monitoring stack (Prometheus & Grafana) is integrated to provide observability into application performance and resource usage.

## 🚀 Key Features
| Component | Implementation Details |
|-----------|------------------------|
| **Reverse Proxy** | Custom-built OpenResty (Nginx) image compiled from source with Lua support. |
| **Application** | Stateless WordPress container connecting to a decoupled MySQL backend. |
| **Storage** | StatefulSet database with `PersistentVolumeClaim` (PVC) for data durability. |
| **Monitoring** | Prometheus Operator stack with custom `ServiceMonitor` configurations. |

---

## 📸 Deployment Proof

### 1. Application Deployment
The WordPress application is fully functional, successfully connecting to the MySQL database via the Nginx reverse proxy.
![WordPress Setup](screenshots/wordpress-setup.png)

### 2. Observability & Monitoring
Grafana dashboards are configured to visualize container performance.

**CPU Utilization:**
Real-time tracking of WordPress container resource consumption.
![CPU Usage](screenshots/grafana-cpu.png)

**Traffic Volume (Traffic Proxy):**
*Technical Note:* Due to a library version conflict in the upstream `nginx-lua-prometheus` package causing internal 500 errors on the application metric endpoint, this dashboard utilizes **Container Network Packet Volume** (`container_network_receive_packets_total`) as a reliable proxy metric to visualize request load.
![Traffic Graph](screenshots/grafana-traffic.png)

---

## 🛠️ Architecture & Decisions

### Infrastructure Code
The project is packaged as a unified Helm Chart (`charts/my-site`) containing:
* **Deployments:** For Nginx and WordPress (stateless).
* **StatefulSets:** For MySQL (stateful).
* **Services:** ClusterIP for internal comms, LoadBalancer for external access.

### The OpenResty Build
The Nginx image was built from source to include specific modules required for high-performance routing:
* `--with-pcre-jit`: Just-In-Time compilation for regex speed.
* `--with-http_stub_status_module`: For basic Nginx metrics.
* `--with-http_realip_module`: To correctly identify client IPs behind the load balancer.

### Monitoring Strategy
The `kube-prometheus-stack` is used for scraping. A custom `ServiceMonitor` resource was created to auto-discover the Nginx pods. 
* **Constraint:** The custom Lua metric script faced compatibility issues with the current OpenResty build.
* **Solution:** Implemented a fallback monitoring strategy using **Container Network Metrics** to ensure traffic visibility was not lost despite the application-layer metric issue.

---

## 🔧 How to Run

### Prerequisites
* Docker Desktop or Minikube
* Helm 3+
* Kubectl

### Installation Steps

1.  **Clone the Repository**
    ```bash
    git clone [https://github.com/YOUR_USERNAME/syfe-devops-task.git](https://github.com/Princepansuriya/syfe-devops-task.git)
    cd syfe-devops-task
    ```

2.  **Deploy Monitoring Stack**
    ```bash
    helm repo add prometheus-community [https://prometheus-community.github.io/helm-charts](https://prometheus-community.github.io/helm-charts)
    helm install prometheus-stack prometheus-community/kube-prometheus-stack -n monitoring --create-namespace
    ```

3.  **Deploy Application**
    ```bash
    helm install my-site ./charts/my-site
    ```

4.  **Access Services**
    * **WordPress:** `http://localhost:8085` (via `kubectl port-forward svc/my-site-nginx 8085:80`)
    * **Grafana:** `http://localhost:3000` (User: `admin` / Pass: `password123`)

Images:
![alt text](<Screenshot 2025-12-12 102710.png>)
![alt text](<Screenshot 2025-12-12 103937.png>)
![alt text](<Screenshot 2025-12-12 102618.png>)
![alt text](<Screenshot 2025-12-12 103948.png>)