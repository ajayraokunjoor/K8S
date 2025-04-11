Here’s a **step-by-step guide** to install Prometheus on Kubernetes using the **Prometheus Operator** (recommended for production) or **kubectl** (manual deployment).

---

## **Method 1: Install Prometheus using Prometheus Operator (Recommended)**
The **Prometheus Operator** simplifies deployment and management of Prometheus in Kubernetes.

### **Step 1: Install Helm (if not installed)**
Helm is a package manager for Kubernetes.
```sh
# Install Helm (Linux/macOS)
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

### **Step 2: Add the Prometheus Community Helm Repo**
```sh
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

### **Step 3: Install the kube-prometheus-stack**
This includes Prometheus, Grafana, Alertmanager, and necessary CRDs.
```sh
helm install prometheus prometheus-community/kube-prometheus-stack -n monitoring --create-namespace
```

### **Step 4: Verify Installation**
```sh
kubectl get pods -n monitoring
kubectl get svc -n monitoring
```

### **Step 5: Access Prometheus UI**
Port-forward Prometheus service:
```sh
kubectl port-forward svc/prometheus-kube-prometheus-prometheus 9090 -n monitoring
```
Now, access Prometheus at:  
👉 [http://localhost:9090](http://localhost:9090)

---

## **Method 2: Manual Installation using kubectl**
If you prefer a manual setup without Helm:

### **Step 1: Create a Namespace**
```sh
kubectl create namespace monitoring
```

### **Step 2: Deploy Prometheus Server**
Create a `prometheus-config.yaml`:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
  namespace: monitoring
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
    scrape_configs:
      - job_name: 'kubernetes-apiservers'
        kubernetes_sd_configs:
        - role: endpoints
        scheme: https
        tls_config:
          ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
        relabel_configs:
        - source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_service_name, __meta_kubernetes_endpoint_port_name]
          action: keep
          regex: default;kubernetes;https
```

Apply the config:
```sh
kubectl apply -f prometheus-config.yaml
```

### **Step 3: Deploy Prometheus Deployment**
Create `prometheus-deployment.yaml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus
  template:
    metadata:
      labels:
        app: prometheus
    spec:
      containers:
      - name: prometheus
        image: prom/prometheus:latest
        args:
        - "--config.file=/etc/prometheus/prometheus.yml"
        ports:
        - containerPort: 9090
        volumeMounts:
        - name: config-volume
          mountPath: /etc/prometheus
      volumes:
      - name: config-volume
        configMap:
          name: prometheus-config
```

Apply the deployment:
```sh
kubectl apply -f prometheus-deployment.yaml
```

### **Step 4: Expose Prometheus via Service**
Create `prometheus-service.yaml`:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: prometheus
  namespace: monitoring
spec:
  selector:
    app: prometheus
  ports:
    - port: 9090
      targetPort: 9090
  type: NodePort
```

Apply the service:
```sh
kubectl apply -f prometheus-service.yaml
```

### **Step 5: Access Prometheus**
```sh
kubectl port-forward svc/prometheus 9090 -n monitoring
```
Access: [http://localhost:9090](http://localhost:9090)

---

## **Next Steps**
- **Install Grafana** for dashboards:
  ```sh
  helm install grafana grafana/grafana -n monitoring
  ```
- **Configure Persistent Storage** (if needed) for long-term metrics retention.
- **Set up Alertmanager** for alerts.

---

### **Summary**
✅ **Helm (Recommended)** → Fastest & production-ready.  
✅ **Manual (kubectl)** → Good for learning.  

Choose the method that fits your needs! 🚀
