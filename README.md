# Pure Kubernetes Manual Monitoring Pipeline 🚀

A production-grade Prometheus and Grafana monitoring infrastructure built entirely from **raw Kubernetes manifests**, bypassing Helm abstraction layers to maximize architecture clarity, control, and native engineering insight.

---

## 🏗️ Architecture Design

Instead of relying on automated operators, this setup configures individual native components directly to establish a clear telemetry pipeline:

- **Application Container:** A highly instrumented Go microservice simulation (`julius/prometheus-demo-service`) exposing rich metric runtimes at `/metrics`.
- **Prometheus Server:** Deployed natively using `kubernetes_sd_configs` inside its `ConfigMap` to scan the cluster API and dynamically discover target endpoints.
- **Security Pass (RBAC):** Explicit `ClusterRole` and `ClusterRoleBinding` configurations granting the Prometheus `ServiceAccount` authorization to track resources across namespaces.
- **Grafana:** Independent standalone data visualization engine connected manually via internal cluster DNS.

---

## 🚀 Deployment Instructions

Follow these steps to deploy the architecture sequentially from point zero:

### 1. Bring up the Demo Application
Deploy the microservice simulation and its discovery service into your `default` namespace:

`kubectl apply -f 1-app-demo.yaml`

*Verify the match by running `kubectl get endpoints prometheus-example-app -n default`. Ensure it lists a pod cluster IP instead of `<none>`.*

### 2. Configure Scrape Rules & Cluster Permissions
Create a dedicated monitoring namespace, register the Service Discovery configurations, and bind the necessary RBAC tracking permissions so Prometheus can scan across namespaces:

`kubectl create namespace monitoring`
`kubectl apply -f 2-prom-config.yaml`
`kubectl apply -f 3-prom-rbac.yaml`

### 3. Deploy the Monitoring Engine
Launch the standalone core metrics engine and your visualization dashboard layout:

`kubectl apply -f 4-prom-deployment.yaml`
`kubectl apply -f 5-grafana-deployment.yaml`

---

## 📊 Verification & Visualization

### Step 1: Access the Prometheus UI
Verify that your application target state registers as bright green **UP**:

`kubectl port-forward -n monitoring svc/prometheus-service 9090:9090`

Open your browser and navigate to: `http://localhost:9090/targets`

### Step 2: Generate Traffic Load
Since the app metrics initialize when traffic passes through the API paths, fire a few test calls:

`kubectl port-forward -n default svc/prometheus-example-app 8080:8080`
`curl http://localhost:8080/api/foo`
`curl http://localhost:8080/api/bar`

### Step 3: View the Metrics Dashboard in Grafana
`kubectl port-forward -n monitoring svc/grafana-service 3000:3000`

1. Open `http://localhost:3000` (Default Credentials: Username: `admin` / Password: `admin`).
2. Navigate to **Connections** -> **Data Sources** -> **Add data source** -> Select **Prometheus**.
3. Input the internal cluster CoreDNS URL: `http://prometheus-service.monitoring.svc.cluster.local:9090`
4. Scroll to the bottom, click **Save & test**.
5. Head to **Dashboards** -> **New Dashboard** -> **Add Visualization**.
6. Select your manual Prometheus datasource and run a panel query tracking: `demo_api_http_requests_total`
7. *Tip: Change your Grafana view window from `Last 6 hours` to `Last 5 minutes` in the top right corner to isolate your immediate live traffic lines!*
