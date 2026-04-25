# ⚓ Kubernetes Hands-On Practice Labs

This repository contains a collection of practical Kubernetes tasks. These exercises cover deploying pods, managing replica sets, setting resource limits, scheduling cron jobs and troubleshooting basic cluster issues.

---

## Task 1: Deploy Pods in a Kubernetes Cluster

**Goal:** Create a pod named `pod-nginx` using the `nginx:latest` image. Set the `app` label to `nginx_app` and name the container `nginx-container`.

### YAML Manifest (`pod-nginx.yaml`)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-nginx
  labels:
    app: nginx_app
spec:
  containers:
  - name: nginx-container
    image: nginx:latest
```

### Commands

```bash
kubectl apply -f pod-nginx.yaml
```

---

## Task 2: Deploy Applications with Kubernetes Deployments

**Goal:** Create a deployment named `httpd` to deploy the application using the `httpd:latest` image.

### YAML Manifest (`deployment-httpd.yaml`)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd
spec:
  replicas: 1
  selector:
    matchLabels:
      app: httpd
  template:
    metadata:
      labels:
        app: httpd
    spec:
      containers:
      - name: httpd
        image: httpd:latest
```

### Commands

```bash
kubectl apply -f deployment-httpd.yaml
kubectl get deployments
```

---

## Task 3: Setup Kubernetes Namespaces and Pods

**Goal:** Create a namespace named `dev` and deploy a pod within it. Name the pod `dev-nginx-pod` and use the `nginx:latest` image.

### YAML Manifest (`dev-nginx.yaml`)

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
---
apiVersion: v1
kind: Pod
metadata:
  name: dev-nginx-pod
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx:latest
```

### Commands

```bash
kubectl apply -f dev-nginx.yaml
kubectl get namespace
kubectl get pods -n dev
```

---

## Task 4: Set Resource Limits in Kubernetes Pods

**Goal:** Create a pod named `httpd-pod` with a container named `httpd-container` using the `httpd:latest` image. Set specific memory and CPU limits and requests.

### YAML Manifest (`httpd-pod.yaml`)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: httpd-pod
spec:
  containers:
  - name: httpd-container
    image: httpd:latest
    resources:
      requests:
        memory: "15Mi"
        cpu: "100m"
      limits:
        memory: "20Mi"
        cpu: "100m"
```

### Commands

```bash
kubectl apply -f httpd-pod.yaml
kubectl describe pod httpd-pod
```

---

## Task 5: Execute Rolling Updates in Kubernetes

**Goal:** Execute a rolling update for an existing deployment named `nginx-deployment`, integrating the `nginx:1.18` image.

### Commands

```bash
kubectl set image deployment/nginx-deployment nginx-container=nginx:1.18
kubectl describe deployment nginx-deployment
kubectl get pods
```

---

## Task 6: Revert Deployment to Previous Version in Kubernetes

**Goal:** Initiate a rollback to the previous revision for an existing deployment named `nginx-deployment`.

### Commands

```bash
kubectl rollout undo deployment/nginx-deployment
kubectl rollout history deployment/nginx-deployment

# Verify the changes
kubectl describe deployments nginx-deployment
```

---

## Task 7: Deploy a ReplicaSet in a Kubernetes Cluster

**Goal:** Create a ReplicaSet using the `nginx:latest` image and name it `nginx-replicaset`. Apply labels `app: nginx_app` and `type: front-end`. Name the container `nginx-container` and ensure the replica count is `4`.

### YAML Manifest (`nginx-replicaset.yaml`)

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-replicaset
  labels:
    app: nginx_app
    type: front-end
spec:
  replicas: 4
  selector:
    matchLabels:
      app: nginx_app
      type: front-end
  template:
    metadata:
      labels:
        app: nginx_app
        type: front-end
    spec:
      containers:
      - name: nginx-container
        image: nginx:latest
```

### Commands

```bash
kubectl apply -f nginx-replicaset.yaml
kubectl get replicasets
```

---

## Task 8: Schedule CronJobs in Kubernetes

**Goal:** Create a CronJob named `datacenter` that runs every 7 minutes. The container should be named `cron-datacenter` and use the `nginx:latest` image to execute a dummy echo command. Set the restart policy to `OnFailure`.

### YAML Manifest (`datacenter.yaml`)

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: datacenter
spec:
  schedule: "*/7 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: cron-datacenter
            image: nginx:latest
            command: ["echo", "Welcome to xfusioncorp!"]
          restartPolicy: OnFailure
```

### Commands

```bash
kubectl apply -f datacenter.yaml
kubectl get cronjob datacenter
```

---

## Task 9: Create a Countdown Job in Kubernetes

**Goal:** Create a Job named `countdown-nautilus`. The container should use the `ubuntu:latest` image, execute a sleep command, and have the restart policy set to `Never`.

### YAML Manifest (`countdown-nautilus.yaml`)

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: countdown-nautilus
spec:
  template:
    metadata:
      name: countdown-nautilus
    spec:
      containers:
      - name: container-countdown-nautilus
        image: ubuntu:latest
        command: ["sleep", "5"]
      restartPolicy: Never
```

### Commands

```bash
kubectl apply -f countdown-nautilus.yaml
kubectl get jobs
```

---

## Task 10: Set Up a Time Check Pod with a ConfigMap and Volume

**Goal:** Create a pod named `time-check` in the `devops` namespace. Use a ConfigMap to pass an environment variable (`TIME_FREQ=9`) to a busybox container. The container must run a loop to log the date to a mounted volume.

### YAML Manifest (`time-check.yaml`)

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: devops
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: time-config
  namespace: devops
data:
  TIME_FREQ: "9"
---
apiVersion: v1
kind: Pod
metadata:
  name: time-check
  namespace: devops
spec:
  volumes:
  - name: log-volume
    emptyDir: {}
  containers:
  - name: time-check
    image: busybox:latest
    command: ["/bin/sh", "-c"]
    args: ["while true; do date >> /opt/dba/time/time-check.log; sleep $TIME_FREQ; done"]
    env:
    - name: TIME_FREQ
      valueFrom:
        configMapKeyRef:
          name: time-config
          key: TIME_FREQ
    volumeMounts:
    - name: log-volume
      mountPath: /opt/dba/time
```

### Commands

```bash
kubectl apply -f time-check.yaml
kubectl exec -it time-check -n devops -- tail -f /opt/dba/time/time-check.log
```

---

## Task 11: Resolve a Pod Deployment Issue

**Goal:** Troubleshoot and fix a failing pod named `webserver`.

### Issue Identification

Inspected the pod details to find out why it was failing to start:

```bash
kubectl describe pod webserver
```

### Resolution

The logs showed an `ImagePullBackOff` error because the image was incorrectly spelled as `httpd:latests` instead of `httpd:latest`. The image name was corrected on the fly, and the webserver became accessible:

```bash
kubectl set image pod/webserver httpd-container=httpd:latest
```

---

## Task 12: Update a Deployment and Service in Kubernetes

**Goal:** Modify an existing service and deployment with the following changes:

- Change the service NodePort from `30008` to `32165`
- Update the image from `nginx:1.19` to `nginx:latest`
- Scale the replicas count from `1` to `5`

### Commands

First, edit the live service to update the NodePort value:

```bash
kubectl edit service nginx-service
```

Next, update the image and scale the deployment:

```bash
kubectl set image deployment/nginx-deployment nginx-container=nginx:latest
kubectl scale deployment nginx-deployment --replicas=5
```

---

## Task 13: Expose an Application Using a NodePort Service

**Goal:** Create a NodePort Service named `nginx-service` to expose an existing ReplicaSet (`nginx-replicaset`) on port `30080`.

### YAML Manifest (`nginx-service.yaml`)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx_app
    type: front-end
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```

### Commands

```bash
kubectl apply -f nginx-service.yaml

# Confirm the service is successfully routing to the pods
kubectl get endpoints nginx-service
```

---

*This repository documents hands-on Kubernetes practice labs covering core cluster operations, workload management, and troubleshooting.*
