# 🗳️ Kubernetes Voting App

A microservices-based **voting application** deployed on a Kubernetes cluster.
The cluster was created using **Kind** on an **AWS EC2** instance, the application
was deployed and managed using **Argo CD** and the **Kubernetes Dashboard**, and
the cluster was monitored using **Prometheus** and **Grafana**.

<p align="left">
  <img alt="Kubernetes" src="https://img.shields.io/badge/Kubernetes-326CE5?logo=kubernetes&logoColor=white">
  <img alt="Docker" src="https://img.shields.io/badge/Docker-2496ED?logo=docker&logoColor=white">
  <img alt="AWS" src="https://img.shields.io/badge/AWS%20EC2-FF9900?logo=amazonaws&logoColor=white">
  <img alt="Argo CD" src="https://img.shields.io/badge/Argo%20CD-EF7B4D?logo=argo&logoColor=white">
  <img alt="Prometheus" src="https://img.shields.io/badge/Prometheus-E6522C?logo=prometheus&logoColor=white">
  <img alt="Grafana" src="https://img.shields.io/badge/Grafana-F46800?logo=grafana&logoColor=white">
</p>

---

## What this project does

- Runs a **3-node Kubernetes cluster** (1 control-plane + 2 workers) created with **Kind** on **AWS EC2**.
- Deploys a **5-service voting application** to the cluster.
- Uses **Argo CD** to deploy and view the application.
- Uses the **Kubernetes Dashboard** to view workloads (pods, deployments, services).
- Uses **Prometheus & Grafana** to monitor the cluster.

---

## About the application

A user votes between two options, the vote is stored, and the live results
are shown on a separate page.

| Service | Built with | What it does |
|---------|-----------|--------------|
| **vote** | Python | Web page to cast a vote |
| **redis** | Redis | Holds incoming votes |
| **worker** | .NET | Moves votes from Redis into the database |
| **db** | PostgreSQL | Stores the votes |
| **result** | Node.js | Shows live voting results |

---

## Architecture

```mermaid
flowchart LR
    user(["User"])

    subgraph ec2["AWS EC2 Instance"]
        subgraph kind["Kind Kubernetes Cluster — 1 control-plane + 2 workers"]
            subgraph app["Voting Application"]
                direction LR
                vote["vote<br/>(Python)"]
                redis[("redis<br/>queue")]
                worker["worker<br/>(.NET)"]
                db[("db<br/>PostgreSQL")]
                result["result<br/>(Node.js)"]
            end
            argocd["Argo CD<br/>deploy &amp; manage"]
            dash["Kubernetes<br/>Dashboard"]
            mon["Prometheus<br/>+ Grafana"]
        end
    end

    user -->|cast vote| vote
    vote -->|store vote| redis
    redis -->|read vote| worker
    worker -->|save| db
    db -->|read results| result
    result -->|live results| user

    argocd -.deploys.-> app
    dash -.views.-> app
    mon -.monitors.-> kind

    classDef store fill:#ffe9c7,stroke:#d18f00,color:#5c3d00;
    classDef svc fill:#d6e8ff,stroke:#2b6cb0,color:#10243e;
    classDef ops fill:#e3dcff,stroke:#6b46c1,color:#241b4d;
    class vote,worker,result svc;
    class redis,db store;
    class argocd,dash,mon ops;
```

---

## Screenshots

### 1. Kubernetes cluster created with Kind (3 nodes, all Ready)

<img src="assets/screenshots/kind-cluster-setup.png" width="800" alt="Kind cluster setup">

<br>

### 2. Kubernetes Dashboard — running workloads

<img src="assets/screenshots/kubernetes-dashboard.png" width="800" alt="Kubernetes Dashboard">

<br>

### 3. Argo CD — application deployed (Healthy & Synced)

<img src="assets/screenshots/argocd-application-tree.png" width="800" alt="Argo CD application">

<br>

### 4. Argo CD — services and pods view

<img src="assets/screenshots/argocd-network-view.png" width="800" alt="Argo CD network view">

<br>

### 5. Prometheus — cluster metrics

<img src="prometheus.png" width="800" alt="Prometheus">

<br>

### 6. Grafana — monitoring dashboards

<img src="grafana.png" width="800" alt="Grafana">

---

## Tools & technologies used

- **Cloud:** AWS EC2
- **Containers:** Docker
- **Kubernetes:** Kind, kubectl, Kubernetes Dashboard
- **Deployment:** Argo CD
- **Monitoring:** Prometheus, Grafana (installed with Helm)

---

## How to set it up

> You need an AWS EC2 instance (or any Linux machine) with Docker installed.

**1. Install Kind and kubectl**
```bash
cd kind-cluster
./install_kind.sh
./install_kubectl.sh
```

**2. Create the cluster**
```bash
kind create cluster --config=config.yml
kubectl get nodes
```

**3. Install Argo CD and deploy the app**
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
Then open the Argo CD UI and add an application pointing to the `k8s-specifications/` folder.

**4. Install the Kubernetes Dashboard**
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
```

**5. Install Prometheus & Grafana (with Helm)**
```bash
helm install kind-prometheus prometheus-community/kube-prometheus-stack --namespace monitoring --create-namespace
```

> All the exact commands are in [`kind-cluster/commands.md`](kind-cluster/commands.md).
