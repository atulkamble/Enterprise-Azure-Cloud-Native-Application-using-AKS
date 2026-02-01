# Enterprise-Azure-Cloud-Native-Application-using-AKS

Below is a **fully updated, copy-paste ready `README.md`** combining **ACR + AKS**, with **required fields**, **production-aligned flags**, and **clean structure**.
You can drop this directly into your GitHub repo as `README.md`.

---

```md
<div align="center">

<h1>☁️ Azure Container Registry (ACR) & AKS Deployment</h1>

<p><strong>Built with ❤️ by <a href="https://github.com/atulkamble">Atul Kamble</a></strong></p>

<p>
<a href="https://codespaces.new/atulkamble/template.git">
<img src="https://github.com/codespaces/badge.svg" alt="Open in GitHub Codespaces" />
</a>
<a href="https://vscode.dev/github/atulkamble/template">
<img src="https://img.shields.io/badge/Open%20with-VS%20Code-007ACC?logo=visualstudiocode&style=for-the-badge" />
</a>
<a href="https://desktop.github.com/">
<img src="https://img.shields.io/badge/GitHub-Desktop-6f42c1?logo=github&style=for-the-badge" />
</a>
</p>

<p>
<a href="https://github.com/atulkamble">
<img src="https://img.shields.io/badge/GitHub-atulkamble-181717?logo=github&style=flat-square" />
</a>
<a href="https://www.linkedin.com/in/atuljkamble/">
<img src="https://img.shields.io/badge/LinkedIn-atuljkamble-0A66C2?logo=linkedin&style=flat-square" />
</a>
<a href="https://x.com/atul_kamble">
<img src="https://img.shields.io/badge/X-@atul_kamble-000000?logo=x&style=flat-square" />
</a>
</p>

<strong>Version:</strong> 1.0.0  
<strong>Last Updated:</strong> January 2026

</div>

---

## 📌 Overview

This repository provides a **production-aligned, step-by-step guide** to:

- Create **Azure Container Registry (ACR)**
- Build & push Docker images to ACR
- Create **Azure Kubernetes Service (AKS)**
- Attach AKS to ACR using **Managed Identity**
- Deploy an application from ACR to AKS

✅ Suitable for **labs, interviews, training, and real-world foundations**

---

## 🏗️ Architecture (High Level)

```

Developer / VM
│
├── Docker Build
│
└── Push Image
│
▼
Azure Container Registry (ACR)
│
▼
Azure Kubernetes Service (AKS)
│
▼
Kubernetes Deployment & Service
│
▼
Application (LoadBalancer / Ingress)

````

---

## 🔧 Prerequisites (Required)

| Requirement | Mandatory |
|------------|----------|
| Azure Subscription | ✅ |
| Azure CLI (`az`) | ✅ |
| Docker | ✅ |
| kubectl | ✅ |
| Git | ✅ |
| Linux VM (Ubuntu recommended) | ⚠️ Recommended |

---

## 🏗️ Step 1 — Create Resource Group

```bash
az group create \
  --name devops \
  --location eastus
````

---

## ☁️ Step 2 — Create Azure Container Registry (ACR)

### Required Fields

| Field          | Example      |
| -------------- | ------------ |
| Registry Name  | `atulkamble` |
| Resource Group | `devops`     |
| Region         | `eastus`     |
| SKU            | `Basic`      |

### Create ACR (Portal or CLI)

```bash
az acr create \
  --resource-group devops \
  --name atulkamble \
  --sku Basic \
  --admin-enabled true
```

> 🔐 **Note:**
> Admin user is enabled for labs.
> In production, AKS uses **Managed Identity** instead.

---

## 🖥️ Step 3 — Prepare Build VM

```bash
sudo apt update -y
sudo apt install docker.io git azure-cli -y
sudo systemctl start docker
sudo systemctl enable docker
```

Verify:

```bash
docker --version
az version
```

Login:

```bash
az login
```

---

## 🔑 Step 4 — Login to ACR

```bash
docker login atulkamble.azurecr.io
```

✔ Required to push images
✔ Use ACR **Access Keys** (Portal → ACR → Access keys)

---

## 📦 Step 5 — Clone Docker Project

```bash
git clone https://github.com/atulkamble/docker-hello-world.git
cd docker-hello-world
```

---

## 🧱 Step 6 — Build & Tag Docker Image (Required Format)

**ACR Image Naming Convention**

```
<registry>.azurecr.io/<repo>/<image>:<tag>
```

```bash
docker build -t atulkamble.azurecr.io/cloudnautic/hello-world:latest .
```

Verify:

```bash
docker images
```

---

## ☁️ Step 7 — Push Image to ACR

```bash
docker push atulkamble.azurecr.io/cloudnautic/hello-world:latest
```

Verify in Azure Portal:

```
ACR → Repositories → cloudnautic/hello-world
```

---

## 🚀 Step 8 — Create AKS Cluster (Production-Aligned)

```bash
az aks create \
  --resource-group devops \
  --name mycluster \
  --node-count 1 \
  --node-vm-size Standard_DS2_v2 \
  --enable-managed-identity \
  --enable-addons monitoring \
  --ssh-access disabled \
  --generate-ssh-keys
```

⏱️ Provisioning Time: ~5–10 minutes

---

## ⚙️ Step 9 — Install kubectl

```bash
sudo snap install kubectl --classic
```

Verify:

```bash
kubectl version --client
```

---

## 🔗 Step 10 — Connect to AKS

```bash
az aks get-credentials \
  --resource-group devops \
  --name mycluster \
  --overwrite-existing
```

Verify:

```bash
kubectl get nodes
```

---

## 🧩 Step 11 — Attach ACR to AKS (Required)

```bash
az aks update \
  --resource-group devops \
  --name mycluster \
  --attach-acr atulkamble
```

✔ Enables private image pulls
✔ No Docker secrets required

---

## 🗂️ Step 12 — Create Namespace

```bash
kubectl create namespace cloudnautic
```

---

## 📦 Step 13 — Deploy Application from ACR

### deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-web
  namespace: cloudnautic
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-web
  template:
    metadata:
      labels:
        app: hello-web
    spec:
      containers:
      - name: hello-web
        image: atulkamble.azurecr.io/cloudnautic/hello-world:latest
        ports:
        - containerPort: 80
```

### service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: hello-web-svc
  namespace: cloudnautic
spec:
  type: LoadBalancer
  selector:
    app: hello-web
  ports:
  - port: 80
    targetPort: 80
```

Apply:

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

---

## ✅ Step 14 — Verify Deployment

```bash
kubectl get pods -n cloudnautic
kubectl get svc -n cloudnautic
```

Access:

```
http://<EXTERNAL-IP>
```

---

## 🔐 Security Best Practices

* ✅ Managed Identity (AKS ↔ ACR)
* ✅ SSH disabled on nodes
* ✅ Private container registry
* ✅ Namespace isolation
* ✅ Azure Monitor enabled

---

## 🧹 Cleanup (Avoid Cost)

```bash
az group delete \
  --name devops \
  --yes --no-wait
```

---

## 🎯 Interview One-Liner

> “I built and pushed Docker images to Azure Container Registry, securely attached ACR to AKS using managed identity, and deployed workloads without storing credentials in Kubernetes.”

---

## 👨‍💻 Author

**Atul Kamble**
Cloud & DevOps Architect | Trainer | Founder

🔗 GitHub: [https://github.com/atulkamble](https://github.com/atulkamble)
🔗 LinkedIn: [https://www.linkedin.com/in/atuljkamble/](https://www.linkedin.com/in/atuljkamble/)

---

## 📄 License

This project is licensed under the **MIT License**.

```

---

If you want next, I can:
- 🔹 Convert this into **Terraform (ACR + AKS)**
- 🔹 Add **Private AKS + Private ACR**
- 🔹 Add **Ingress (NGINX / App Gateway)**
- 🔹 Create **architecture diagrams (PNG / Draw.io)**

Just tell me 👍
```
