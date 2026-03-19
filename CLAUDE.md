# CC Spring 2026 - Assignment 2: Kubernetes Deployment

## Project Overview
Deploy a Flask + MongoDB TODO web application on Kubernetes using Docker containers.
- Containerize with Docker, push to Docker Hub
- Deploy on Minikube (local) and AWS EKS
- Implement replica sets, rolling updates, health monitoring, and alerting

## Docker Hub
- Image: `krishkuchroo/todo-flask`
- Tags: `v1` (initial), `v2` (for rolling update demo)
- Both tags are multi-platform (linux/amd64 + linux/arm64)

## Architecture
- **Flask app**: Python 3.10-slim, port 5000
- **MongoDB**: mongo:6, port 27017, PVC-backed storage
- **K8s**: 2 Flask replicas, 1 Mongo replica, NodePort service (Minikube) / LoadBalancer (EKS)
- **Monitoring**: kube-prometheus-stack via Helm, Alertmanager → Slack

## Assignment Parts & Status

| Part | Description | Status |
|------|-------------|--------|
| 1 | Flask + MongoDB TODO app | DONE |
| 2 | Dockerfile, docker-compose, push to Docker Hub | DONE |
| 3 | Deploy on Minikube | DONE |
| 4 | Deploy on AWS EKS | DONE |
| 5 | ReplicaSets (OR Part 6) | SKIPPED (chose Part 6) |
| 6 | Rolling update strategy | DONE (tested v1→v2 on Minikube) |
| 7 | Health monitoring (liveness/readiness probes) | DONE (tested pod delete + auto-recovery) |
| 8 | Alerting (Extra Credit - 30pts) | DONE (Prometheus + custom rules on Minikube) |
| -- | Submission document with screenshots | TODO |

## EKS Cluster Details
- **Cluster name**: cc-assignment2
- **Region**: us-east-1
- **Node type**: t3.small (2 nodes)
- **ELB URL**: http://ab6602aa44a93421f9e6ddd750e5fbec-1247538605.us-east-1.elb.amazonaws.com/
- **IMPORTANT**: Delete cluster after screenshots to save credits!
  ```bash
  eksctl delete cluster --name cc-assignment2 --region us-east-1
  ```

## Key Commands Reference
```bash
# Docker (multi-platform build)
docker buildx build --platform linux/amd64,linux/arm64 -t krishkuchroo/todo-flask:v1 --push .

# Minikube
minikube start
kubectl apply -f k8s/
kubectl port-forward service/flask-service 8080:80

# EKS
eksctl create cluster --name cc-assignment2 --region us-east-1 --node-type t3.small --nodes 2
aws eks update-kubeconfig --name cc-assignment2 --region us-east-1
kubectl apply -f k8s/mongo-pvc.yaml -f k8s/mongo-secret.yaml -f k8s/mongo-deployment.yaml -f k8s/mongo-service.yaml -f k8s/flask-deployment.yaml -f k8s/flask-service-eks.yaml

# Monitoring (on Minikube)
helm install monitoring prometheus-community/kube-prometheus-stack \
  -f monitoring/prometheus-values.yaml --namespace monitoring --create-namespace

# Switch kubectl context
kubectl config use-context minikube
kubectl config use-context arn:aws:eks:us-east-1:746140163942:cluster/cc-assignment2
```

## File Structure
```
CC_Spring2026_Assignment2/
├── app/                    # Flask TODO application
├── Dockerfile              # Flask container image
├── docker-compose.yml      # Local dev with Flask + Mongo
├── k8s/                    # Kubernetes manifests
│   ├── flask-deployment.yaml
│   ├── flask-service.yaml      # NodePort (Minikube)
│   ├── flask-service-eks.yaml  # LoadBalancer (EKS)
│   ├── mongo-deployment.yaml
│   ├── mongo-pvc.yaml
│   ├── mongo-secret.yaml
│   └── mongo-service.yaml
├── monitoring/             # Prometheus + Alertmanager configs
│   ├── prometheus-values.yaml
│   └── alertmanager-config.yaml
└── docs/                   # Submission document + screenshots
```

## Notes
- Part 5 and Part 6 are OR — we chose Part 6 (rolling updates)
- mongo-secret exists but is NOT wired into mongo-deployment (app doesn't use auth)
- For EKS: need EBS CSI driver addon + pod identity for PVC to work
- Part 8 (Alerting) is extra credit worth 30 points
- Flask image must be multi-platform (arm64 for local Mac, amd64 for EKS)
