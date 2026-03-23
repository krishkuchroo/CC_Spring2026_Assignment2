# Screenshot Checklist - Assignment 2

Track screenshots as you go. Mark [x] when captured.
Save screenshots to this `docs/` folder with the naming convention shown.

---

## Part 2: Docker Containerization

- [x] `docker-compose up --build` output showing both containers starting — `p2-docker-compose-up.png` : done  
- [x] App running in browser at http://localhost:5001 — `p2-app-browser.png` : done 
- [x] `docker images` showing the built Flask image — `p2-docker-images.png` : done 
- [x] Docker Hub page showing `krishkuchroo/todo-flask` with v1 and v2 tags — `p2-dockerhub.png` : done 
- [x] `docker manifest inspect` output showing multi-platform (amd64 + arm64) — `p2-manifest-inspect.png` : done 

---

## Part 3: Minikube Deployment

- [x] `minikube start` output — `p3-minikube-start.png` : Done 
- [x] `kubectl apply -f k8s/...` output showing resources created — `p3-kubectl-apply.png` : done
- [x] `kubectl get all` showing pods (2 flask + 1 mongo), services, deployments, replicasets — `p3-kubectl-get-all.png` : done 
- [x] `kubectl get pvc` showing mongo-pvc in Bound state — `p3-pvc-bound.png` : done
- [x] App running in browser via Minikube (NodePort or port-forward) — `p3-app-minikube.png` : done 
- [x] A TODO item added/visible in the app — `p3-todo-added.png` : done 

---

## Part 4: AWS EKS Deployment

- [x] `eksctl create cluster` output or AWS Console showing cluster active — `p4-eks-cluster-created.png` : done
- [x] `kubectl get nodes` showing 2 t3.small nodes — `p4-eks-nodes.png` : done 
- [x] `kubectl apply` output for EKS resources — `p4-kubectl-apply-eks.png` : done 
- [x] `kubectl get all` on EKS showing pods, services (LoadBalancer), deployments — `p4-kubectl-get-all-eks.png` : done 
- [x] `kubectl get svc flask-service` showing EXTERNAL-IP (ELB URL) — `p4-elb-url.png` : done 
- [x] `kubectl get pvc` showing Bound PVC on EKS — `p4-pvc-eks.png` : done 
- [x] App running in browser via the ELB URL — `p4-app-eks-browser.png` : done 
- [x] A TODO item added/visible in the app on EKS — `p4-todo-eks.png` : done 

---

## Part 6: Rolling Update Strategy

- [x] `kubectl describe deployment flask-deployment` showing RollingUpdate strategy (maxUnavailable:1, maxSurge:1) — `p6-rolling-strategy.png` : 
- [x] `kubectl set image deployment/flask-deployment flask=krishkuchroo/todo-flask:v2` output — `p6-set-image.png`
- [x] `kubectl rollout status deployment/flask-deployment` showing successful rollout — `p6-rollout-status.png`
- [x] `kubectl get pods` during/after rollout showing new pods — `p6-pods-after-rollout.png`
- [x] `kubectl describe deployment flask-deployment | grep Image` showing v2 — `p6-image-v2.png`
- [x] `kubectl rollout history deployment/flask-deployment` showing revision history — `p6-rollout-history.png`
- [x] App in browser after update — `p6-app-after-update.png`

---

## Part 7: Health Monitoring

- [x] `kubectl describe pod <flask-pod>` showing livenessProbe and readinessProbe config — `p7-probe-config.png` : done 
- [x] `kubectl delete pod <flask-pod>` + `kubectl get pods -w` showing auto-recovery — `p7-pod-recovery.png` : done 
- [x] `kubectl get pods` after recovery showing replacement pod with new name and young AGE — `p7-pods-after-recovery.png` : done 

---

## Part 8: Alerting (Extra Credit - 30 pts)

- [ ] `kubectl get pods -n monitoring` showing Prometheus stack running — `p8-monitoring-pods.png` : done 
- [ ] Prometheus UI (port-forward :9090) > Alerts page showing custom rules (PodFrequentlyRestarting, FlaskPodNotReady) — `p8-prometheus-alerts.png` : done 
- [ ] Alertmanager UI (port-forward :9093) showing Slack receiver config — `p8-alertmanager-config.png` : done 
- [ ] Grafana dashboard (port-forward :3000) showing cluster metrics — `p8-grafana-dashboard.png` : done 
- [ ] Trigger failure + alert firing in Prometheus/Alertmanager — `p8-alert-firing.png` : done 
- [ ] Slack channel showing received alert (if real webhook configured) — `p8-slack-alert.png` : done 

---

## Recommended Order

1. **Minikube first** (free) — Parts 3 → 6 → 7 → 8
2. **EKS last** (costs money) — Part 4, then immediately `eksctl delete cluster`
3. **Part 2** — Can be done anytime locally

## Tips

- Use `Cmd+Shift+4` (Mac) for selective screenshots
- Save all screenshots into this `docs/` folder
- Name files exactly as listed above for easy reference in the submission document
