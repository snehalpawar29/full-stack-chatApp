# 🚀 Kubernetes Deployment Instructions

This document explains how to deploy the Full-Stack Chat Application on Kubernetes.

> **Namespace:** `chatapp-ns`
> **Frontend:** React
> **Backend:** Node.js / Express / Socket.io
> **Database:** MongoDB
> **Containerization:** Docker
> **Orchestration:** Kubernetes

---

## 1. 📋 Prerequisites

Make sure the following are installed and configured:

- 🐳 Docker
- ☸️ Kubernetes
- `kubectl`
- Git
- A running Kubernetes cluster

For an AWS EC2-based setup, ensure the EC2 instance can communicate with the Kubernetes nodes and that the required application port is accessible.

---

## 2. 📥 Clone the Repository

```bash
git clone https://github.com/snehalpawar29/full-stack-chatApp.git
cd full-stack-chatApp
```

---

## 3. ☸️ Verify Kubernetes Cluster

Check that the cluster is available:

```bash
kubectl get nodes
```

Expected result:

```text
NAME                     STATUS   ROLES
chatapp-cluster-worker   Ready    <none>
```

Your node name may be different depending on the cluster configuration.

---

## 4. 🏷️ Create the Namespace

The application uses the dedicated namespace:

```bash
kubectl apply -f k8s/namespace.yml
```

Verify:

```bash
kubectl get namespace
```

Check the application namespace:

```bash
kubectl get all -n chatapp-ns
```

---

## 5. 🐳 Build Docker Images

Build the frontend image:

```bash
docker build -t snehalpawar2945/chatapp-frontend:latest ./frontend
```

Build the backend image:

```bash
docker build -t snehalpawar2945/chatapp-backend:latest ./backend
```

Verify:

```bash
docker images
```

---

## 6. 📦 Push Images to Docker Hub

Login to Docker Hub:

```bash
docker login
```

Push the frontend image:

```bash
docker push snehalpawar2945/chatapp-frontend:latest
```

Push the backend image:

```bash
docker push snehalpawar2945/chatapp-backend:latest
```

The Kubernetes Deployments use:

```text
snehalpawar2945/chatapp-frontend:latest
snehalpawar2945/chatapp-backend:latest
mongo:latest
```

---

## 7. ☸️ Deploy Kubernetes Resources

Apply the Kubernetes manifests:

```bash
kubectl apply -f k8s/
```

This creates the required Kubernetes resources, including:

- Namespace
- Frontend Deployment
- Backend Deployment
- MongoDB Deployment
- Services
- PersistentVolume
- PersistentVolumeClaim

---

## 8. 🔍 Verify Pods

Check the application Pods:

```bash
kubectl get pods -n chatapp-ns -o wide
```

Expected state:

```text
frontend   1/1   Running
backend    1/1   Running
mongodb    1/1   Running
```

You can also check everything together:

```bash
kubectl get all -n chatapp-ns -o wide
```

---

## 9. 🌐 Verify Services

Run:

```bash
kubectl get svc -n chatapp-ns
```

The current deployment exposes:

```text
Frontend → NodePort  → 80:30080
Backend  → ClusterIP → 5001
MongoDB  → ClusterIP → 27017
```

### Service roles

**Frontend**

```text
frontend-service
Type: NodePort
Port: 80
NodePort: 30080
```

The frontend is the externally accessible application entry point.

**Backend**

```text
backend-service
Type: ClusterIP
Port: 5001
```

The backend is internally accessible within the Kubernetes cluster.

**MongoDB**

```text
mongodb
Type: ClusterIP
Port: 27017
```

MongoDB is accessible internally by the backend.

---

## 10. 🚀 Access the Application

Find the Kubernetes worker node IP:

```bash
kubectl get nodes -o wide
```

The frontend is exposed through NodePort `30080`.

Use:

```text
http://<NODE-IP>:30080
```

For an AWS EC2 deployment, make sure the relevant inbound security-group rule allows TCP traffic on port `30080`.

> Do not expose MongoDB or the backend ClusterIP directly to the public internet unless your architecture specifically requires it.

---

## 11. 💾 Verify Persistent Storage

Check the PersistentVolume:

```bash
kubectl get pv
```

Check the PersistentVolumeClaim:

```bash
kubectl get pvc -n chatapp-ns
```

The PV/PVC configuration is used to provide persistent storage for MongoDB.

You can inspect the details with:

```bash
kubectl describe pv <PV-NAME>
kubectl describe pvc <PVC-NAME> -n chatapp-ns
```

---

## 12. 📊 Verify Deployments

Run:

```bash
kubectl get deployment -n chatapp-ns
```

Expected result:

```text
backend-deployment    1/1
frontend-deployment   1/1
mongodb-deployment    1/1
```

Check rollout status:

```bash
kubectl rollout status deployment/backend-deployment -n chatapp-ns
kubectl rollout status deployment/frontend-deployment -n chatapp-ns
kubectl rollout status deployment/mongodb-deployment -n chatapp-ns
```

---

## 13. 🔎 Troubleshooting

### Check Pod logs

Backend:

```bash
kubectl logs deployment/backend-deployment -n chatapp-ns
```

Frontend:

```bash
kubectl logs deployment/frontend-deployment -n chatapp-ns
```

MongoDB:

```bash
kubectl logs deployment/mongodb-deployment -n chatapp-ns
```

### Describe a Pod

```bash
kubectl describe pod <POD-NAME> -n chatapp-ns
```

### Check Services

```bash
kubectl get svc -n chatapp-ns
```

### Check Events

```bash
kubectl get events -n chatapp-ns --sort-by=.metadata.creationTimestamp
```

---

## 14. 🔄 Updating the Application

After changing the frontend or backend code:

### Build the updated image

```bash
docker build -t snehalpawar2945/chatapp-frontend:latest ./frontend
```

or:

```bash
docker build -t snehalpawar2945/chatapp-backend:latest ./backend
```

### Push the image

```bash
docker push snehalpawar2945/chatapp-frontend:latest
```

or:

```bash
docker push snehalpawar2945/chatapp-backend:latest
```

### Restart the Deployment

```bash
kubectl rollout restart deployment/frontend-deployment -n chatapp-ns
```

or:

```bash
kubectl rollout restart deployment/backend-deployment -n chatapp-ns
```

Verify:

```bash
kubectl get pods -n chatapp-ns
```

---

## 15. 🧹 Remove the Deployment

To remove the Kubernetes resources created from the manifests:

```bash
kubectl delete -f k8s/
```

Alternatively, remove the entire namespace:

```bash
kubectl delete namespace chatapp-ns
```

> Deleting the namespace removes the resources contained within it. Review your PV configuration before deleting storage if you need to preserve MongoDB data.

---

## 16. 🧾 Useful Commands

| Purpose                  | Command                                                        |
| ------------------------ | -------------------------------------------------------------- |
| Cluster nodes            | `kubectl get nodes`                                            |
| All resources            | `kubectl get all -n chatapp-ns`                                |
| Pods                     | `kubectl get pods -n chatapp-ns`                               |
| Services                 | `kubectl get svc -n chatapp-ns`                                |
| Deployments              | `kubectl get deployment -n chatapp-ns`                         |
| Persistent Volumes       | `kubectl get pv`                                               |
| Persistent Volume Claims | `kubectl get pvc -n chatapp-ns`                                |
| Pod logs                 | `kubectl logs <pod> -n chatapp-ns`                             |
| Pod details              | `kubectl describe pod <pod> -n chatapp-ns`                     |
| Deployment status        | `kubectl rollout status deployment/<deployment> -n chatapp-ns` |

---

## 📌 Current Deployment Summary

```text
Kubernetes Namespace
└── chatapp-ns
    │
    ├── Frontend
    │   ├── Deployment
    │   ├── Pod
    │   └── NodePort : 30080
    │
    ├── Backend
    │   ├── Deployment
    │   ├── Pod
    │   └── ClusterIP : 5001
    │
    └── MongoDB
        ├── Deployment
        ├── Pod
        ├── ClusterIP : 27017
        └── PersistentVolume / PersistentVolumeClaim
```
