# DaemonSet & CronJob for ToDo app

## Prerequisites
- Kubernetes cluster (kind / minikube / Docker Desktop) and `kubectl`

## Deploy
All commands are run from the repository root
1. Deploy the ToDo app and its ClusterIP service:
```bash
kubectl apply -f .infrastructure/namespace.yml
kubectl apply -f .infrastructure/deployment.yml
kubectl apply -f .infrastructure/clusterIp.yml
kubectl get pods -n todoapp
```
Wait until the app pod is `1/1 Running`.

2. Create the `mateapp` namespace and deploy the DaemonSet and CronJob:
```bash
kubectl apply -f .infrastructure/mateapp-namespace.yml
kubectl apply -f .infrastructure/daemonset.yml
kubectl apply -f .infrastructure/cronjob.yml
```

## Validate

1. Check resources:
```bash
kubectl get ds,cronjob,jobs,pods -n mateapp
```

2. DaemonSet logs (curl to the ClusterIP service every 5 seconds):
```bash
kubectl logs -n mateapp -l app=todoapp-curl --tail=5
```
Expected output:
```
Thu Sep 24 00:25:14 UTC 2026 200
Thu Sep 24 00:25:19 UTC 2026 200
Thu Sep 24 00:25:24 UTC 2026 200
```

3. CronJob logs (calls `/api/health` every 4 minutes):
```bash
kubectl get jobs -n mateapp
kubectl logs -n mateapp job/<job-name>
```
Expected output:
```
Thu Sep 24 00:20:02 UTC 2026 checking health
Health OK
```

To trigger the CronJob immediately without waiting:
```bash
kubectl create job --from=cronjob/todoapp-health health-manual -n mateapp
kubectl logs -n mateapp job/health-manual
```