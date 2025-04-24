## 🔹 What is a StatefulSet in Kubernetes?

A **StatefulSet** manages the **deployment and scaling** of a set of pods, and provides **guarantees about the ordering and uniqueness** of these pods.

### ✅ Key Features:
- **Stable, unique network identifiers (DNS)**: Each pod gets a sticky identity like `pod-0`, `pod-1`, etc.
- **Stable storage**: Each pod gets its own PersistentVolume that stays even if the pod is deleted.
- **Ordered deployment & termination**: Pods are created/deleted in a strict sequence (`pod-0` before `pod-1`).
- **Ordered rolling updates**.

---

## 🔸 Real-time Project Example: **MongoDB Replica Set**

Let’s assume you're working on a project where you need to deploy a MongoDB **Replica Set** in Kubernetes.

> A Replica Set requires that each MongoDB instance (primary & secondary nodes) has a unique identity and persistent data storage.

---

## 🔧 Step-by-Step StatefulSet Demonstration

### 🛠️ Prerequisites:
- Kubernetes Cluster
- `kubectl` installed
- A storage class provisioner like EBS (AWS), Azure Disk, or local-path-provisioner (for Minikube)

---

### 📁 Step 1: Create a Headless Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mongo
spec:
  clusterIP: None  # Important for StatefulSet
  selector:
    app: mongo
  ports:
  - name: mongo
    port: 27017
```

✅ This service provides stable network identity using DNS like `mongo-0.mongo.default.svc.cluster.local`.

---

### 📄 Step 2: Create StatefulSet YAML

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mongo
spec:
  selector:
    matchLabels:
      app: mongo
  serviceName: "mongo"
  replicas: 3
  template:
    metadata:
      labels:
        app: mongo
    spec:
      containers:
      - name: mongod
        image: mongo:5.0
        ports:
        - containerPort: 27017
        volumeMounts:
        - name: mongo-pvc
          mountPath: /data/db
  volumeClaimTemplates:
  - metadata:
      name: mongo-pvc
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 5Gi
```

✅ This will create:
- Pods: `mongo-0`, `mongo-1`, `mongo-2`
- PVCs: `mongo-pvc-mongo-0`, `mongo-pvc-mongo-1`, `mongo-pvc-mongo-2`

---

### 🧪 Step 3: Apply the YAML files

```bash
kubectl apply -f mongo-service.yaml
kubectl apply -f mongo-statefulset.yaml
```

---

### 🔍 Step 4: Verify StatefulSet

```bash
kubectl get statefulsets
kubectl get pods -l app=mongo
kubectl get pvc
```

---

### 🗃️ Step 5: Initialize MongoDB Replica Set (Manual Step)

Exec into the `mongo-0` pod:

```bash
kubectl exec -it mongo-0 -- mongo
```

Initiate replica set:

```js
rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "mongo-0.mongo:27017" },
    { _id: 1, host: "mongo-1.mongo:27017" },
    { _id: 2, host: "mongo-2.mongo:27017" }
  ]
})
```

Check status:

```js
rs.status()
```

---

## 🎯 Final Outcome

✅ You now have a **MongoDB Replica Set** running with:
- Stable pod identities (`mongo-0`, `mongo-1`, etc.)
- Persistent storage for each pod
- DNS resolution between pods
- Easy failover and recovery with data intact

---

## 🧠 When to Use StatefulSet?

Use it for applications that:
- Need **persistent storage** (e.g., DBs like MongoDB, MySQL, PostgreSQL)
- Require **unique network identity** (e.g., Kafka, Elasticsearch)
- Need **ordered startup/shutdown** (e.g., Zookeeper)

