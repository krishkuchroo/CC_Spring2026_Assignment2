# Kubernetes TODO Application Deployment

A full-stack TODO web application deployed on Kubernetes, built with Flask and MongoDB. This project demonstrates containerization, orchestration, health monitoring, rolling updates, and alerting on both local (Minikube) and cloud (AWS EKS) Kubernetes clusters.

## System Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                        Kubernetes Cluster                           │
│                    (Minikube / AWS EKS)                              │
│                                                                     │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │                    Default Namespace                          │   │
│  │                                                              │   │
│  │  ┌─────────────────┐      ┌─────────────────┐               │   │
│  │  │  Flask Service   │      │  Mongo Service   │               │   │
│  │  │  (NodePort/LB)   │      │  (ClusterIP)     │               │   │
│  │  │  Port: 80        │      │  Port: 27017     │               │   │
│  │  └────────┬─────────┘      └────────┬─────────┘               │   │
│  │           │                         │                         │   │
│  │     ┌─────┴──────┐                  │                         │   │
│  │     │  Replicas   │                 │                         │   │
│  │  ┌──┴───┐  ┌──┴───┐         ┌──────┴──────┐                  │   │
│  │  │Flask │  │Flask │────────▶│  MongoDB     │                  │   │
│  │  │Pod 1 │  │Pod 2 │         │  Pod         │                  │   │
│  │  │      │  │      │         │              │                  │   │
│  │  │:5000 │  │:5000 │         │  :27017      │                  │   │
│  │  └──────┘  └──────┘         └──────┬───────┘                  │   │
│  │                                    │                          │   │
│  │  ┌──────────────┐           ┌──────┴───────┐                  │   │
│  │  │ Rolling      │           │  PVC (1Gi)   │                  │   │
│  │  │ Update       │           │  /data/db    │                  │   │
│  │  │ Strategy     │           └──────────────┘                  │   │
│  │  └──────────────┘                                             │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │                 Monitoring Namespace                          │   │
│  │                                                              │   │
│  │  ┌────────────┐  ┌──────────────┐  ┌────────────┐           │   │
│  │  │ Prometheus │─▶│ Alertmanager │─▶│   Slack    │           │   │
│  │  │            │  │              │  │   Alerts   │           │   │
│  │  └────────────┘  └──────────────┘  └────────────┘           │   │
│  │  ┌────────────┐  ┌──────────────┐                            │   │
│  │  │  Grafana   │  │ Node         │                            │   │
│  │  │  Dashboard │  │ Exporter     │                            │   │
│  │  └────────────┘  └──────────────┘                            │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘

         ▲                                          ▲
         │                                          │
    ┌────┴─────┐                              ┌─────┴──────┐
    │  Users   │                              │ Docker Hub │
    │ (Browser)│                              │ Image Repo │
    └──────────┘                              └────────────┘
```

## Tech Stack

| Component | Technology |
|-----------|-----------|
| Backend | Flask 2.1.3 (Python 3.10) |
| Database | MongoDB 6 |
| Container | Docker + Docker Compose |
| Orchestration | Kubernetes (Minikube + AWS EKS) |
| Monitoring | Prometheus + Grafana + Alertmanager |
| CI/CD | Docker Hub (multi-platform builds) |

## Project Structure

```
├── app/                         # Flask application
│   ├── app.py                   # Main application code
│   ├── requirements.txt         # Python dependencies
│   ├── templates/               # HTML templates
│   └── static/                  # CSS, JS, images
├── k8s/                         # Kubernetes manifests
│   ├── flask-deployment.yaml    # Flask deployment (2 replicas, rolling updates, probes)
│   ├── flask-service.yaml       # NodePort service (Minikube)
│   ├── flask-service-eks.yaml   # LoadBalancer service (AWS EKS)
│   ├── mongo-deployment.yaml    # MongoDB deployment with PVC
│   ├── mongo-service.yaml       # ClusterIP service
│   ├── mongo-pvc.yaml           # Persistent Volume Claim (1Gi)
│   └── mongo-secret.yaml        # MongoDB credentials
├── monitoring/                  # Monitoring stack
│   ├── prometheus-values.yaml   # Helm values for kube-prometheus-stack
│   └── alertmanager-config.yaml # AlertmanagerConfig for Slack alerts
├── Dockerfile                   # Multi-stage Flask image
└── docker-compose.yml           # Local development setup
```

## Features

### Containerization (Part 2)
- Multi-platform Docker image (`linux/amd64` + `linux/arm64`)
- Docker Compose for local development with Flask + MongoDB

### Minikube Deployment (Part 3)
- 2 Flask replicas behind a NodePort service
- MongoDB with persistent storage via PVC

### AWS EKS Deployment (Part 4)
- EKS cluster with `t3.small` managed node group
- LoadBalancer service for external access
- EBS CSI driver for persistent volumes

### Rolling Updates (Part 6)
- `RollingUpdate` strategy with `maxUnavailable: 1`, `maxSurge: 1`
- Zero-downtime deployments between image versions (v1 → v2)

### Health Monitoring (Part 7)
- **Liveness Probe**: HTTP GET on `/` every 10s — restarts pod if unhealthy
- **Readiness Probe**: HTTP GET on `/` every 5s — removes pod from service if not ready

### Alerting — Extra Credit (Part 8)
- kube-prometheus-stack (Prometheus + Grafana + Alertmanager)
- Custom alert rules:
  - `PodFrequentlyRestarting` — fires if Flask pods restart >3 times in 1 hour
  - `FlaskPodNotReady` — fires if a Flask pod is unready for >5 minutes
- Slack notification integration via Alertmanager

## Quick Start

### Local (Docker Compose)
```bash
docker-compose up
# App available at http://localhost:5001
```

### Minikube
```bash
minikube start
kubectl apply -f k8s/
kubectl port-forward service/flask-service 8080:80
# App available at http://localhost:8080
```

### AWS EKS
```bash
# Create cluster
eksctl create cluster --name cc-assignment2 --region us-east-1 \
  --node-type t3.small --nodes 2 --managed

# Configure kubectl
aws eks update-kubeconfig --name cc-assignment2 --region us-east-1

# Deploy
kubectl apply -f k8s/mongo-pvc.yaml -f k8s/mongo-secret.yaml \
  -f k8s/mongo-deployment.yaml -f k8s/mongo-service.yaml \
  -f k8s/flask-deployment.yaml -f k8s/flask-service-eks.yaml

# Get external URL
kubectl get svc flask-service -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'

# IMPORTANT: Delete cluster when done to avoid charges
eksctl delete cluster --name cc-assignment2 --region us-east-1
```

## Docker Hub

```bash
# Build and push multi-platform image
docker buildx build --platform linux/amd64,linux/arm64 \
  -t krishkuchroo/todo-flask:v1 --push .
```

Image: [`krishkuchroo/todo-flask`](https://hub.docker.com/r/krishkuchroo/todo-flask)
