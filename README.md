
# 🚀 Kubernetes Version Upgrades on EKS & AKS: Real-World Walkthrough

## 🌍 Real-World Scenario

> You're a DevOps engineer responsible for managing production clusters on both AWS (EKS) and Azure (AKS). Kubernetes upgrades are critical for security patches, performance improvements, and new features. You must manage version compatibility, ensure smooth upgrades, and automate them where possible using Terraform, CLI tools, and IaC practices.

---

## 📅 Kubernetes Release & Support Overview

| Platform                | Release Cadence          | Supported Versions (as of May 2025)     | End of Support Policy                         |
| ----------------------- | ------------------------ | ---------------------------------------- | --------------------------------------------- |
| **Kubernetes Upstream** | Every 3 months           | Each version supported ~12 months        | Deprecation announcements + upgrade guides    |
| **Amazon EKS**          | ~2 weeks after upstream  | 1.26, 1.27, 1.28, 1.29, 1.30, 1.31, 1.32 | Each version supported for 14–26 months       |
| **Azure AKS**           | Shortly after upstream   | 1.28, 1.29, 1.30                         | Sequential upgrade (e.g., 1.28 → 1.29 → 1.30) |

---

## ⚠️ AWS EKS Kubernetes Upgrade Policy

- **Current Supported Versions**:
  - `1.26` (extended support)
  - `1.27`
  - `1.28`
  - `1.29`
  - `1.30`
  - `1.31`
  - `1.32` (latest)
- **Automatic Upgrade**: Control planes on unsupported versions are auto-upgraded
- **Manual Steps Required**:
  - EC2 Node Groups
  - Add-ons (`aws-node`, `kube-proxy`, `coredns`)
- **Compatibility Skew**: 2 minor versions between control plane and nodes allowed (e.g., 1.30 nodes under 1.32 control plane)

📚 [EKS Version Policy](https://docs.aws.amazon.com/eks/latest/userguide/kubernetes-versions.html)  
📚 [EKS Upgrade Guide](https://docs.aws.amazon.com/eks/latest/userguide/update-cluster.html)

---

## 🔁 AKS Kubernetes Upgrade Policy

- **Current Supported Versions**: `1.28`, `1.29`, `1.30`
- **Upgrade Rules**:
  - Cannot skip minor versions
  - Auto-upgrade is available
- **LTS and Security**: Azure provides patched versions for each supported release

📚 [AKS Upgrade Docs](https://learn.microsoft.com/en-us/azure/aks/upgrade-cluster?tabs=azure-cli)

---

## 🧪 Hands-On Demo: Upgrade Kubernetes on AKS

### ✅ 1. Create Cluster (Terraform or Azure CLI)

```bash
terraform init
terraform plan
terraform apply
```

### ✅ 2. Connect and Verify

```bash
az aks get-credentials --resource-group lili-rg --name lili_cluster --overwrite-existing
kubectl get nodes
kubectl version --short
```

### ✅ 3. Deploy Sample App

```bash
kubectl apply -f sample-app.yml
kubectl get pods
kubectl get svc
```

### ✅ 4. Check Upgrade Availability

```bash
az aks get-upgrades --resource-group lili-rg --name lili_cluster --output table
```

### ✅ 5. Perform Upgrade (example to 1.30.0)

```bash
az aks upgrade --resource-group lili-rg --name lili_cluster --kubernetes-version 1.30.0
```

### ✅ 6. Validate Upgrade

```bash
kubectl get events
kubectl get nodes -o wide
kubectl describe <node-name>
```

---

## 🧪 Hands-On Demo: Upgrade Kubernetes on EKS

### ✅ 1. Create EKS Cluster (eksctl)

```yaml
# cluster.yml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig
metadata:
  name: test-cluster
  region: us-east-1
  version: "1.30"
managedNodeGroups:
  - name: worker-node-1-30
    instanceType: m5.large
    desiredCapacity: 2
    minSize: 1
    maxSize: 3
```

```bash
eksctl create cluster -f cluster.yml
```

### ✅ 2. Deploy App

```bash
kubectl apply -f deployment.yml
kubectl get pods
```

### ✅ 3. Upgrade Control Plane (to latest, e.g., 1.32)

```bash
eksctl upgrade cluster --name test-cluster --version 1.32 --region us-east-1 --approve
```

### ✅ 4. Update Add-ons

```bash
eksctl utils update-kube-proxy --cluster test-cluster --approve
eksctl utils update-aws-node --cluster test-cluster --approve
eksctl utils update-coredns --cluster test-cluster --approve
```

Watch rollout:

```bash
kubectl get pods -n kube-system -w
```

### ✅ 5. Upgrade Nodegroup

```bash
eksctl upgrade nodegroup --name worker-node-1-30 --cluster test-cluster
```

Or recreate it:

```bash
eksctl drain nodegroup --cluster test-cluster --name worker-node-1-30
eksctl delete nodegroup --only-missing -f cluster.yml --approve
```

---

## 📌 Notes on Fargate (EKS Only)

- **Serverless Kubernetes Nodes** — No EC2 to manage
- Ideal for short-lived or burstable workloads
- You still must upgrade control plane and optionally Fargate profiles

📚 [Fargate Overview](https://docs.aws.amazon.com/eks/latest/userguide/fargate.html)

---

## 🧼 Cleanup

### AKS

```bash
az group delete --name lili-rg --yes --no-wait
```

### EKS

```bash
eksctl delete cluster --name test-cluster
```

---

## ✅ Final Recommendations

- Always **test upgrades** in staging before applying to production
- Enable **auto-upgrade** in AKS for simpler version management
- Regularly update EKS **nodegroups** and **add-ons**
- Monitor [Kubernetes Release Notes](https://kubernetes.io/releases/) to stay current
```

