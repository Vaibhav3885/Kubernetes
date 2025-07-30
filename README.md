# Kubernetes

## 🔷 Part 1: What is Kubernetes?
Kubernetes (K8s) is an open-source system used to automate deployment, scaling, and management of containerized applications.

### 🔹 Why Kubernetes?
Docker runs 1 container well, but managing 100s or 1000s? That’s hard.

Kubernetes helps you manage containers across many servers, restart failed apps, scale easily, and more.

### 🔹 Basic Kubernetes Components (Simple Overview)
Component                                    Description
Pod	                          Smallest unit in K8s, holds 1 or more containers
Node	                        A machine (VM or physical) that runs Pods
Cluster	                      Group of nodes managed by Kubernetes
Deployment                   	Manages multiple replicas of a Pod
Service	                      xposes Pods to internal or external traffic
Namespace	                    Separates different environments (like dev, prod)
ConfigMap & Secret	          Used to inject config values & secrets into Pods

### 🔹 Kubernetes Objects with Real Examples
1️⃣ Pod
```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: nginx
      image: nginx
```
🔹 Use: Run a basic container.

2️⃣ Deployment
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:latest
```
🔹 Use: Maintain 3 replicas, restart if one fails.

3️⃣ Service
```
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: NodePort
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30001
```
🔹 Use: Expose the deployment on port 30001 of the node.

4️⃣ ConfigMap
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: "prod"
  DB_URL: "mysql-service"
```
🔹 Use: Inject environment variables.

5️⃣ Secret
```
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  password: cGFzc3dvcmQ=  # base64 encoded
```
🔹 Use: Store sensitive data like passwords.

## 🔸 Real-World Kubernetes Interview Questions & Answers
### Q1. What is a Pod in Kubernetes?
Answer:
A pod is the smallest unit in Kubernetes. It contains one or more containers that share storage, network, and a spec. If a pod dies, a new one is created by the Deployment or ReplicaSet.

### Q2. What is the difference between a Pod and a Deployment?
Pod	                           Deployment
Manages 1 unit	           Manages multiple pods
No auto-restart	           Handles auto-restart
Not ideal for production	Suitable for production workloads

### Q3. How do you expose your application to the internet?
Answer:
We create a Service of type LoadBalancer or NodePort. In production, we use Ingress controllers for routing and TLS termination.

### Q4. How do you perform rolling updates?
Answer:
Kubernetes Deployments support rolling updates by default. When you change the image version, it gradually updates pods one by one without downtime.

```
kubectl set image deployment/web-deployment nginx=nginx:1.21
```
### Q5. What is a ConfigMap and Secret?
Answer:

ConfigMap stores non-sensitive key-value data.

Secret stores encrypted sensitive data (like DB passwords).

Both are injected into Pods as environment variables or volumes.

### Q6. How do you check the logs of a pod?
```
kubectl logs pod-name
```
🔹 If multiple containers:

```
kubectl logs pod-name -c container-name
```
### Q7. How do you scale your application?
```
kubectl scale deployment web-deployment --replicas=5
```
### Q8. How do you monitor Kubernetes?
Answer:

Use Prometheus + Grafana for metrics

Use tools like Datadog, NewRelic, or ELK stack for logging

kubectl top shows resource usage

### Q9. How does Kubernetes handle self-healing?
Answer:
If a Pod crashes, the Deployment or ReplicaSet automatically starts a new one. If a Node goes down, pods are rescheduled on another node.

### Q10. What challenges have you faced with Kubernetes?
Answer:

Config drift → Fixed using GitOps

Service discovery issues → Used DNS and labels properly

Scaling problems → Implemented Horizontal Pod Autoscaler

Debugging → Used kubectl describe, logs, and probes (readiness/liveness)

## 🔷 Part 2: Intermediate to Advanced Kubernetes Concepts
### ✅ 1. Horizontal Pod Autoscaler (HPA)
What it does:
It automatically increases or decreases the number of pods based on CPU/memory usage or custom metrics.

Example:

```
kubectl autoscale deployment web-deployment --cpu-percent=50 --min=2 --max=5
```
⏩ Means: If CPU goes above 50%, Kubernetes can scale pods between 2 and 5.

Why it matters in interviews:
Shows you understand how to save costs and handle traffic spikes.

### ✅ 2. Readiness and Liveness Probes
These check if your app is healthy and ready to accept traffic.

Readiness Probe = "Can the app take traffic?"

Liveness Probe = "Is the app still alive or stuck?"

YAML Example:

```
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5

readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 3
```
### ✅ 3. Helm Charts (Package Manager for Kubernetes)
Helm is like YUM or APT for Kubernetes. It helps you install apps using templates.

Use case:
Instead of writing multiple YAML files, use Helm charts to manage apps like NGINX, Prometheus, Jenkins.

Install example:

```
helm install my-nginx bitnami/nginx
```
Interview tip:
Say: "We used Helm for reusable templates and version control of Kubernetes resources."

### ✅ 4. StatefulSet vs Deployment
Feature	                      Deployment	               StatefulSet
Pod names	                     Random	                     Fixed (web-0, web-1)
Storage                   	Shared/ephemeral	         Persistent, stable
Use case	                  Stateless apps	           Databases (MySQL, etc)

🧠 Interview answer:
If your app needs stable storage or ordered start (like DBs), use StatefulSet.

### ✅ 5. DaemonSet
Runs one pod per node (or more if needed).

Use case: Logging agent, monitoring agent, etc.

```
kind: DaemonSet
metadata:
  name: log-agent
spec:
  selector:
    matchLabels:
      app: log-agent
  template:
    metadata:
      labels:
        app: log-agent
    spec:
      containers:
      - name: filebeat
        image: elastic/filebeat
```
### ✅ 6. RBAC (Role-Based Access Control)
Controls who can do what in a cluster.

Example:

```
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: dev
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```
Then bind with a user/service account using RoleBinding.

🧠 Interview line:
"We use RBAC to separate access between developers and admins in dev/staging/prod clusters."

### ✅ 7. NetworkPolicy
Controls which pods can talk to each other. By default, all pods can communicate.

Example:

```
kind: NetworkPolicy
metadata:
  name: allow-app
spec:
  podSelector:
    matchLabels:
      app: frontend
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: backend
```
🧠 Use case: "Only frontend should access backend, others denied."

### ✅ 8. Namespaces
Used to separate environments (like dev, test, prod).

```
kubectl create namespace dev
kubectl get pods -n dev
```
🧠 Interview answer:
"We used namespaces to isolate teams and workloads across the same cluster."

## 🧠 Real-World Kubernetes Interview Questions (Advanced)
### Q11. What is the difference between ConfigMap and Secret?
Answer:
ConfigMap = plain text
Secret = base64 encoded (for sensitive info)
Both can be injected into Pods.

### Q12. How do you manage app upgrades in Kubernetes?
Answer:

Use rolling updates in Deployments.

Use Helm to version control and rollback if needed.

Example: ``` helm upgrade app-name new-version ```

### Q13. What are the ways to monitor a Kubernetes cluster?
Answer:

``` kubectl top ``` – for basic CPU/Memory

Prometheus + Grafana – full metrics

ELK Stack – logs

Datadog, New Relic, or Dynatrace – cloud monitoring

### Q14. How do you debug a Kubernetes issue?
Steps:

``` kubectl describe pod pod-name ```

``` kubectl logs pod-name ```

Check events section

Look into probes, image pull errors, or service mapping

### Q15. How do you roll back to a previous version?
```
kubectl rollout undo deployment my-deployment
```
Or with Helm:

```
helm rollback release-name version
```

## 🔷 Kubernetes Part 3 – Deployment Strategies, Troubleshooting & Scenarios
### ✅ 1. Rolling Update
Default strategy in Kubernetes.
It updates pods gradually with zero downtime.

How it works:

Takes old pods down one by one

Brings up new pods step by step

```
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
    maxSurge: 1
```
🧠 Used when: No downtime is acceptable, and app supports gradual upgrade.

### ✅ 2. Blue-Green Deployment
You have two environments: Blue (old) and Green (new).
Traffic is switched to Green after successful deployment.

Kubernetes trick:
Use two services or label selectors, and change service selector to new pods.

🧠 Used when: You need fast rollback and testing the new version before exposing.

### ✅ 3. Canary Deployment
Release the new version to a small percentage of users first.
If all goes well, increase rollout to 100%.

How it’s done:

Use multiple Deployments

Adjust weights using Ingress or Service mesh like Istio/Linkerd

🧠 Good for: Testing new features on real traffic with low risk.

### ✅ 4. Kubernetes Troubleshooting Steps
🔹 a. Pod CrashLoopBackOff
```
kubectl logs <pod-name>
kubectl describe pod <pod-name>
```
Check app logs

Check health probes

Check ConfigMap or Secret issues

🔹 b. ImagePullBackOff
Wrong image name or registry permissions

Fix image name or use secret for private registry

🔹 c. Pending Pods
No resources available on nodes

Use ``` kubectl describe ``` to see scheduler errors

### ✅ 5. Service Types in Kubernetes
Type	                         Purpose
ClusterIP	              Default, internal communication only
NodePort	              Expose service on a static port
LoadBalancer	          For external traffic (like AWS ELB)
ExternalName          	Map service to external DNS

🧠 Tip: "For internal communication, we used ClusterIP. For user access, we used LoadBalancer on AWS."

### ✅ 6. Ingress Controller
Manages external access (HTTP/HTTPS) to services.

YAML Example:

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: demo-ingress
spec:
  rules:
  - host: demo.myapp.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: demo-service
            port:
              number: 80
```
🧠 Used for: Exposing multiple apps using a single LoadBalancer.

### ✅ 7. Kubernetes YAML Breakdown (Basic)
Pod YAML

```
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: app
    image: nginx
    ports:
    - containerPort: 80
Deployment YAML

yaml
Copy
Edit
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
```
## 🧠 Real Interview Scenario Questions
### Q16. What happens if a node dies in Kubernetes?
Answer:
Kubernetes automatically reschedules the pods to another available node using the scheduler.

### Q17. How do you ensure a pod always runs on a specific node?
Answer: Use nodeSelector, nodeAffinity, or taints and tolerations.

### Q18. How do you handle secrets in Kubernetes?
Answer:

Use Secret object (base64 encoded)

Use external tools like HashiCorp Vault or AWS Secrets Manager for more security

### Q19. How do you upgrade Kubernetes in production?
Answer:

Upgrade control plane first

Then upgrade worker nodes one by one (cordon → drain → upgrade)

### Q20. How do you reduce cost in Kubernetes?
Answer:

Use HPA to auto-scale pods

Use cluster auto-scaler to scale nodes

Use Spot Instances/Node Pools

Clean up unused resources

✅ Summary of Part 3 Topics
✅ Rolling, Blue-Green, and Canary deployments

✅ Pod & Node troubleshooting

✅ YAML templates for pod, deployment, ingress

✅ Interview scenarios and real-world examples

