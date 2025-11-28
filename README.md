# **ReactiveChainDB-V1**

[![Setup Guide](https://img.shields.io/badge/📘-Setup_Guide-blue)](#reactivechaindb--full-kubernetes--docker-setup-guide)
![License](https://img.shields.io/badge/License-Proprietary_Freeware-orange)

Modern distributed systems require increasingly secure, low-latency, and high-throughput transaction handling—particularly in healthcare, finance, and real-time scheduling.   
Conventional blockchain architectures often lack the performance efficiency needed for such reactive environments. 

To address this, a **high-throughput reactive microservice framework** extending **BigchainDB** is proposed for decentralized and scalable transaction processing. 

---

## Table of Contents

### Project Overview
- [Problem 1: CPU and Queue Congestion Under Load](#problem-1-cpu-and-queue-congestion-under-load)
- [Problem 2: Complexity of Blockchain Integration](#problem-2-complexity-of-blockchain-integration)
- [Problem 3: Handling Real-Time Data Updates](#problem-3-handling-real-time-data-updates)

### Setup Guide
- [1. Prerequisites & System Setup](#1-prerequisites--system-setup)
- [2. Build & Push Docker Images](#2-build--push-docker-images)
- [3.  Kubernetes Deployment](#3-kubernetes-deployment)
- [4.  Testing Blockchain & API Connectivity](#4-testing-blockchain--api-connectivity)
- [5. ReactiveChainDB API Endpoints](#5-reactivechaindb-api-endpoints)
- [6. Real-Time WebSocket Streaming](#6-real-time-websocket-streaming)
- [7. Kubernetes Maintenance Commands](#7-kubernetes-maintenance-commands)
- [8. Log Monitoring](#8-log-monitoring)
- [9. Final Deployment Overview](#9-final-deployment-overview)
- [License](#license)

---

# Project Overview

## Problem 1: CPU and Queue Congestion Under Load

### The Hurdle
During high-traffic simulations (1,000 requests in batches), the system hit a bottleneck in **Batch 5**, where a temporary slowdown caused the **maximum response time to spike to 4. 37 seconds** due to CPU and queue congestion. 

### Solution
We implemented a **Parallel Backpressure-Handling Algorithm** using `Sinks. Many`. 

- The reactive buffer was dynamically reconfigured and resized at runtime to adapt to fluctuating workloads. 
- Leveraging **Non-Blocking I/O** combined with reactive backpressure allowed the system to recover quickly.
- After the temporary surge, response times stabilized back to the **30–70 ms** range.

![Backpressure Solution](https://assets.devfolio.co/content/9285a1bb63944d76aee21933e63c35bb/17dea845-dc84-4653-bffa-0d47ae0dd6d0.png)

---

## Problem 2: Complexity of Blockchain Integration

### The Hurdle
Hyperledger Fabric was initially explored due to its modular design.   
However, its complexity conflicted with the goal of creating a **lightweight, reactive integration**. 

### Solution
We pivoted to **BigchainDB**, which offered a lighter and more reactive-friendly framework.

To further mitigate slowness from direct blockchain queries, we developed a **hybrid caching layer using Redis**:

- Instead of querying BigchainDB on each read,
- We cached transaction URLs in Redis mapped to User IDs
- Allowing *instant* lookups and minimal blockchain load.

![Redis Caching Layer](https://assets.devfolio.co/content/9285a1bb63944d76aee21933e63c35bb/a8a4485d-da13-4eb0-bb34-2a4e10fb9c21.png)

---

## Problem 3: Handling Real-Time Data Updates

### The Hurdle
Traditional REST API polling was too slow and inefficient for low-latency environments such as **healthcare**, **finance**, and **real-time systems**.  
This caused delays in propagating fresh data to clients.

### Solution
We built a **Redis Watcher Service + WebSockets** architecture. 

- The service watches for specific User ID updates in the Redis cache
- On change, it triggers an immediate request to BigchainDB
- The updated data is **pushed instantly** to the client via WebSocket
- Completely eliminating the need for repeated polling

![WebSocket Architecture](https://assets.devfolio.co/content/9285a1bb63944d76aee21933e63c35bb/b7d8ad3e-245d-413c-a615-cdc49434c61c.png)

---

# ReactiveChainDB — Full Kubernetes + Docker Setup Guide

Modern decentralized microservices must support **secure**, **reactive**, and **high-throughput** transaction flows.   
Blockchain-backed systems like BigchainDB, Tendermint, Redis, and Reactive Java make this possible — but deployment requires a reliable, scalable orchestration layer.

To achieve this, we deploy the entire ReactiveChainDB stack into a **Kubernetes-based distributed environment** for real-time, fault-tolerant transaction processing.

---

## 1. Prerequisites & System Setup

### Install Docker

```bash
sudo apt install docker.io -y
```

### Install K3s (Single-node Kubernetes cluster)

```bash
curl -sfL https://get. k3s.io | sh -
```

### Verify Cluster

```bash
kubectl get nodes
```

---

## 2.  Pull Docker Images (Optional) for custom deployments
### skip the step (if continue for kubernetes deployment)


```

### Pull Images from Docker Hub


docker pull namanoncode/bigchaindb:latest
docker pull namanoncode/reactivechaindbv1:latest
docker pull tendermint/tendermint:v0.31.5
docker pull redis:7
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
kubectl apply -f mongodb. yaml
kubectl apply -f bigchaindb.yaml
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

## 5.  ReactiveChainDB API Endpoints

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
ws://139.84. 169.197:8080/ws/appointments? userId=648263
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
