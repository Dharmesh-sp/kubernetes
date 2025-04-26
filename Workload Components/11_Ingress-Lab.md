### To install **MetalLB** for your **NGINX Ingress Controller**, especially in a bare-metal or non-cloud Kubernetes cluster (like Minikube, K3s, or on-prem), follow these steps. MetalLB allows you to expose services of type `LoadBalancer`.

---

### ✅ Pre-requisites:
- A working Kubernetes cluster (e.g., Minikube, K3s, kubeadm)
- `kubectl` configured to point to the cluster
- NGINX Ingress Controller installed **without** a cloud LoadBalancer

---

## 🔧 Step-by-Step Installation

### 1. **Install MetalLB**
```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.10/config/manifests/metallb-native.yaml
```

> This installs MetalLB in **layer2 mode**, which works great for bare-metal environments.

---

### 2. **Create MetalLB Address Pool**
You need to assign a pool of IPs on your local network that MetalLB can use.

```yaml
# metallb-config.yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  namespace: metallb-system
  name: my-ip-pool
spec:
  addresses:
  - 192.168.1.240-192.168.1.250  # Customize to match your LAN
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  namespace: metallb-system
  name: l2-advertisement
```

Apply the configuration:
```bash
kubectl apply -f metallb-config.yaml
```

---

### 3. **Install NGINX Ingress Controller**
Use the `LoadBalancer` type service so that MetalLB can assign it an IP.

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.9.4/deploy/static/provider/baremetal/deploy.yaml
```

---

### 4. **Verify MetalLB Assignment**
After a few seconds, check if the NGINX Ingress Controller received a LoadBalancer IP:

```bash
kubectl get svc -n ingress-nginx
```

You should see something like:

```
NAME                                 TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)
ingress-nginx-controller             LoadBalancer   10.96.232.41    192.168.1.240   80:30890/TCP,443:32429/TCP
```

---

Great! Let's walk through how to:

- Deploy **2 apps** from Docker Hub: one HTTP app (we'll use `hashicorp/http-echo`), and another is `nginx`.
- Expose both using **Ingress** via **NGINX Ingress Controller** + **MetalLB** setup.

---

## ✅ Step-by-Step Setup

### 📁 Directory structure:
We'll use:
```
k8s-ingress-lab/
├── app1-deployment.yaml       # http-echo
├── app2-deployment.yaml       # nginx
├── ingress.yaml               # ingress rules
```

---

### 1. **App 1: `http-echo` Deployment + Service**

```yaml
# app1-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: http-echo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: http-echo
  template:
    metadata:
      labels:
        app: http-echo
    spec:
      containers:
      - name: http-echo
        image: hashicorp/http-echo
        args:
        - "-text=Hello from App 1"
        ports:
        - containerPort: 5678
---
apiVersion: v1
kind: Service
metadata:
  name: http-echo
spec:
  selector:
    app: http-echo
  ports:
  - protocol: TCP
    port: 80
    targetPort: 5678
```

---

### 2. **App 2: `nginx` Deployment + Service**

```yaml
# app2-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-app
  template:
    metadata:
      labels:
        app: nginx-app
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-app
spec:
  selector:
    app: nginx-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

---

### 3. **Ingress Resource for Both Apps**

```yaml
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: myapps.local
    http:
      paths:
      - path: /app1
        pathType: Prefix
        backend:
          service:
            name: http-echo
            port:
              number: 80
      - path: /app2
        pathType: Prefix
        backend:
          service:
            name: nginx-app
            port:
              number: 80
```

---

### 4. **Apply All Resources**

```bash
kubectl apply -f app1-deployment.yaml
kubectl apply -f app2-deployment.yaml
kubectl apply -f ingress.yaml
```

---

### 5. **Access Your Apps**

Get the LoadBalancer IP:
```bash
kubectl get svc -n ingress-nginx
```

Update your **`/etc/hosts`** file (Linux/macOS) or `C:\Windows\System32\drivers\etc\hosts` (Windows):

```bash
<MetalLB_IP>  myapps.local
```

---

### 6. **Test in Browser or Curl**

- `http://myapps.local/app1` → Should return `Hello from App 1`
- `http://myapps.local/app2` → Should serve NGINX default page

---
