### Ingress in Kubernetes?

---

Ingress in Kubernetes is a powerful API object that **manages external access to services** within a Kubernetes cluster, typically **HTTP and HTTPS traffic**. It acts as a **smart router**, allowing you to route requests to different services based on the **URL path**, **hostname**, or other rules.

---

## 🔍 What is Ingress in Kubernetes?

- In Kubernetes, services expose your pods internally or externally.
- If you want to **route traffic intelligently** (e.g., `example.com/app1`, `example.com/app2`), instead of exposing multiple LoadBalancers or NodePorts, you use an **Ingress** resource.
- An **Ingress Controller** (like NGINX Ingress Controller) must be installed to interpret Ingress rules and **handle the routing**.

---

## 🛠 Real-Time Project Scenario

> You’re working on a **microservices-based eCommerce application** with multiple services like `frontend`, `cart`, `product`, `orders`. Each runs as a separate deployment in the same cluster.

- Without Ingress: You’d expose each service with a separate LoadBalancer/IP.
- With Ingress: One single LoadBalancer IP handles all routes and domains.

---

## 📦 Key Components:

1. **Ingress Resource** – defines rules for routing.
2. **Ingress Controller** – actual implementation that listens and routes traffic.

We'll use **NGINX Ingress Controller** here.

---

## ✅ Step-by-Step: Deploying Multiple Apps using NGINX Ingress Controller

### 🔧 Prerequisites:

- Kubernetes cluster (minikube, EKS, GKE, or AKS).
- kubectl configured.
- Helm installed.

---

### 🌀 Step 1: Install NGINX Ingress Controller

Using Helm (recommended):

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

helm install nginx-ingress ingress-nginx/ingress-nginx \
  --set controller.publishService.enabled=true
```

> This creates a LoadBalancer Service that handles traffic to the cluster.

---

### 🌐 Step 2: Deploy Sample Applications

#### App 1: `app1`

```yaml
# app1-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app1
spec:
  replicas: 2
  selector:
    matchLabels:
      app: app1
  template:
    metadata:
      labels:
        app: app1
    spec:
      containers:
        - name: app1
          image: hashicorp/http-echo
          args:
            - "-text=Welcome to App1"
---
apiVersion: v1
kind: Service
metadata:
  name: app1
spec:
  selector:
    app: app1
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5678
```

#### App 2: `app2`

```yaml
# app2-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app2
spec:
  replicas: 2
  selector:
    matchLabels:
      app: app2
  template:
    metadata:
      labels:
        app: app2
    spec:
      containers:
        - name: app2
          image: hashicorp/http-echo
          args:
            - "-text=Welcome to App2"
---
apiVersion: v1
kind: Service
metadata:
  name: app2
spec:
  selector:
    app: app2
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5678
```

Apply them:

```bash
kubectl apply -f app1-deployment.yaml
kubectl apply -f app2-deployment.yaml
```

---

### 🌉 Step 3: Create Ingress Resource

```yaml
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: demo-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
    - host: demo.local
      http:
        paths:
          - path: /app1
            pathType: Prefix
            backend:
              service:
                name: app1
                port:
                  number: 80
          - path: /app2
            pathType: Prefix
            backend:
              service:
                name: app2
                port:
                  number: 80
```

Apply:

```bash
kubectl apply -f ingress.yaml
```

---

### 🧪 Step 4: Test the Setup

> Add the following entry to your `/etc/hosts` file (Linux/Mac) or `C:\Windows\System32\drivers\etc\hosts` (Windows):

```
<External-IP>  demo.local
```

Where `<External-IP>` is the LoadBalancer IP of the Ingress controller:

```bash
kubectl get svc -n ingress-nginx
```

Now test:

```bash
curl http://demo.local/app1
# Output: Welcome to App1

curl http://demo.local/app2
# Output: Welcome to App2
```

---

## 🎯 Outcome

✅ With a single **Ingress controller and LoadBalancer**, we:

- Hosted multiple applications.
- Routed them based on **URL paths**.
- Kept infrastructure lightweight and maintainable.

---

## 🧠 Real-Time Use Case Benefits

- **Microservices Architecture**: Each service has its own endpoint (via path).
- **Cost Optimization**: Only one LoadBalancer IP.
- **SSL Termination**: Easily add TLS certs via Ingress.
- **Traffic Control**: Rate-limiting, whitelisting, and more via annotations.
- **Centralized Routing Logic**: All HTTP traffic managed at one place.

---

If you'd like, I can help you extend this with:
- TLS/HTTPS via Let's Encrypt.
- Rewrite/redirect rules.
- Host-based routing instead of path-based.
- CI/CD pipeline integration.

Want to continue with that?

