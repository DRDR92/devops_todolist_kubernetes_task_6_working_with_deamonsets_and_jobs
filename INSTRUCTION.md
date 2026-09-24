# DaemonSet & CronJob for ToDo app

## Prerequisites
- Kubernetes cluster (kind / Docker Desktop) and `kubectl`
- Docker CLI (to preload the `busyboxplus:curl` image)

## Deploy
All commands are run from the repository root.

1. Deploy the ToDo app and its ClusterIP service:
```bash
kubectl apply -f .infrastructure/namespace.yml
kubectl apply -f .infrastructure/deployment.yml
kubectl apply -f .infrastructure/clusterIp.yml
kubectl get pods -n todoapp
```
Wait until the app pod is `1/1 Running`.

2. Preload the `busyboxplus:curl` image into the cluster node.
`busyboxplus:curl` is not available in the Docker Hub library, so it is taken from `ikulyk404/busyboxplus:curl` and retagged. Manifests use `imagePullPolicy: IfNotPresent`, so the node uses the local image.
```bash
docker pull ikulyk404/busyboxplus:curl
docker tag ikulyk404/busyboxplus:curl busyboxplus:curl
# Docker Desktop (kind) node:
docker save busyboxplus:curl | docker exec -i desktop-control-plane ctr -n k8s.io images import -
# or, for kind with a separate CLI:
# kind load docker-image busyboxplus:curl
```

3. Create the `mateapp` namespace and deploy the DaemonSet and CronJob:
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
Thu Sep 24 00:25:14 UTC 2026