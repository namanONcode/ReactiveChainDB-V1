# ReactiveChainDB — Full Kubernetes + Docker Setup Guide

Modern decentralized microservices must support **secure**, **reactive**, and **high-throughput** transaction flows.    
Blockchain-backed systems like BigchainDB, Tendermint, Redis, and Reactive Java make this possible — but deployment requires a reliable, scalable orchestration layer.

To achieve this, we deploy the entire ReactiveChainDB stack into a **Kubernetes-based distributed environment** for real-time, fault-tolerant transaction processing.

---

## Table of Contents

- [1. Prerequisites \& System Setup](#1-prerequisites--system-setup)
- [2. Build \& Push Docker Images](#2-build--push-docker-images)
- [3. Kubernetes Deployment](#3-kubernetes-deployment)
- [4. Testing Blockchain \& API Connectivity](#4-testing-blockchain--api-connectivity)
- [5.  ReactiveChainDB API Endpoints](#5-reactivechaindb-api-endpoints)
- [6. Real-Time WebSocket Streaming](#6-real-time-websocket-streaming)
- [7. Kubernetes Maintenance Commands](#7-kubernetes-maintenance-commands)
- [8. Log Monitoring](#8-log-monitoring)
- [9. Final Deployment Overview](#9-final-deployment-overview)
- [License](#license)

---

## 1.  Prerequisites & System Setup

### Install Docker

```bash
sudo apt install docker.io -y
```

### Install K3s (Single-node Kubernetes cluster)

```bash
curl -sfL https://get.k3s.io | sh -
```

### Verify Cluster

```bash
kubectl get nodes
```

---

## 2.  Build & Push Docker Images

### Clone the Project

```bash
git clone https://github.com/namanONcode/ReactiveChainDB-V1. git
cd ReactiveChainDB-V1
```

### Build BigchainDB Image

```bash
docker build -t namanoncode/bigchaindb:latest ./bigchaindb
```

### Build ReactiveChainDB API Image

```bash
docker build -t namanoncode/reactivechaindbv1:latest ./reactivechaindb
```

### Push Images to Docker Hub

```bash
docker push namanoncode/bigchaindb:latest
docker push namanoncode/reactivechaindbv1:latest
```

---

## 3. Kubernetes Deployment

### Create Namespace

```bash
kubectl create namespace reactivechaindb
```

### Apply Required Components

> Ensure YAML files exist in your repository.

```bash
kubectl apply -f namespace.yaml
kubectl apply -f mongodb.yaml
kubectl apply -f bigchaindb. yaml
kubectl apply -f redis.yaml
kubectl apply -f reactivechaindb. yaml
```

### Watch Pods Until Ready

```bash
kubectl -n reactivechaindb get pods -w
```

---

## 4.  Testing Blockchain & API Connectivity

### Enter BigchainDB Pod

```bash
kubectl exec -it -n reactivechaindb deploy/bigchaindb -- bash
```

### Check Tendermint RPC

```bash
curl http://localhost:26657/status
```

### Check BigchainDB API

```bash
curl http://localhost:9984/api/v1
```

---

## 5. ReactiveChainDB API Endpoints

### 5. 1 Create Appointment

Creates a new BigchainDB blockchain transaction.

**Endpoint:**

```
POST http://localhost:8080/appointments/create3
```

**Example:**

```bash
curl -X POST "http://localhost:8080/appointments/create3" \
     -H "Content-Type: application/json" \
     -d '{"userId": 648263, "details": "Checkup"}'
```

### 5.2 Update Appointment

Adds a new version transaction to BigchainDB. 

**Endpoint:**

```
PUT http://localhost:8080/appointments/update? userId=31323
```

**Example:**

```bash
curl -X PUT "http://localhost:8080/appointments/update? userId=31323" \
     -H "Content-Type: application/json" \
     -d '{"updated": true}'
```

---

## 6. Real-Time WebSocket Streaming

ReactiveChainDB uses a **Redis Watcher + WebSocket** system to deliver instant appointment updates. 

### Client Subscription

Clients subscribe via:

```
ws://<server-ip>:8080/ws/appointments? userId=<id>
```

**Example:**

```
ws://139.84. 169.197:8080/ws/appointments?userId=648263
```

### Triggered Automatically By

- `/appointments/create3`
- `/appointments/update?userId=<id>`

> **No polling.  Pure real-time updates.**

---

## 7. Kubernetes Maintenance Commands

### Restart Deployment

```bash
kubectl rollout restart deploy/reactivechaindb -n reactivechaindb
```

### Delete All Pods

```bash
kubectl delete pod --all -n reactivechaindb
```

### Delete Namespace

```bash
kubectl delete ns reactivechaindb
```

---

## 8. Log Monitoring

### BigchainDB Logs

```bash
kubectl logs -n reactivechaindb deploy/bigchaindb -f
```

### Tendermint Logs

```bash
kubectl logs -n reactivechaindb deploy/bigchaindb -c tendermint -f
```

### ReactiveChainDB API Logs

```bash
kubectl logs -n reactivechaindb deploy/reactivechaindb -f
```

---

## 9. Final Deployment Overview

Your full decentralized microservice stack is now operational:

| Component                           | Description                        |
| ----------------------------------- | ---------------------------------- |
| **BigchainDB Ledger**               | Immutable blockchain storage       |
| **Tendermint Consensus Engine**     | Byzantine fault-tolerant consensus |
| **MongoDB Document Store**          | Persistent data layer              |
| **Redis Hybrid Cache**              | High-speed caching & pub/sub       |
| **ReactiveChainDB Spring Boot API** | Reactive REST API layer            |
| **Real-Time WebSocket Streaming**   | Live event broadcasting            |

> ✅ All running inside Kubernetes, fully scalable and production-ready. 

---

## License

**Proprietary Freeware License**

This software is provided free of charge for personal, educational, or commercial use. However, the following restrictions apply:

- ❌ No reverse engineering, decompilation, or source code derivation
- ❌ No modification or derivative works
- ❌ No redistribution for a fee
- ✅ Must be used in its original, unmodified form

**© 2025 Naman Jain** — All rights reserved. 

See the full [LICENSE](LICENSE. md) for complete terms. 
