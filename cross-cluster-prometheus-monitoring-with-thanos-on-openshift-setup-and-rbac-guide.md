Cross-Cluster Prometheus Monitoring with Thanos on OpenShift: Setup and RBAC Guide

Here is a step-by-step guide and the necessary YAML for deploying Thanos to monitor Prometheus metrics across two OpenShift clusters: a **source cluster** sending metrics and a **destination cluster** running Thanos listeners.

### Step-by-Step Guide

---

## **Step 1: Setup Prometheus on the Source OpenShift Cluster**

1. **Install Prometheus Operator:**
   If Prometheus is not already installed, deploy Prometheus using the Prometheus Operator in the source OpenShift cluster.

2. **Configure Remote Write:**
   Configure Prometheus in the source cluster to send metrics to the Thanos Receiver in the destination cluster by adding the `remote_write` section in the Prometheus configuration.

```yaml
remote_write:
  - url: "http://thanos-receiver.destination-cluster.svc:19291/api/v1/receive"
```

---

## **Step 2: Install Thanos on the Destination OpenShift Cluster**

1. **Deploy Thanos Components**:
   Install the following components in the destination OpenShift cluster:
   
   - Thanos Receiver
   - Thanos Query
   - Thanos Store
   - Thanos Compactor (optional)
   - Thanos Sidecar (optional for HA setups)
   
   Use the following Thanos Receiver YAML to start:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: thanos-receiver
  namespace: thanos
spec:
  replicas: 1
  selector:
    matchLabels:
      app: thanos-receiver
  template:
    metadata:
      labels:
        app: thanos-receiver
    spec:
      containers:
      - name: thanos-receiver
        image: thanosio/thanos:v0.22.0
        args:
        - receive
        - --grpc-address=0.0.0.0:10901
        - --http-address=0.0.0.0:10902
        - --receive.hashrings-file=/etc/thanos/hashrings.json
        ports:
        - containerPort: 10901
        - containerPort: 10902
        volumeMounts:
        - mountPath: /etc/thanos
          name: thanos-hashrings
      volumes:
      - name: thanos-hashrings
        configMap:
          name: thanos-hashrings
---
apiVersion: v1
kind: Service
metadata:
  name: thanos-receiver
  namespace: thanos
spec:
  ports:
  - name: grpc
    port: 10901
  - name: http
    port: 10902
  selector:
    app: thanos-receiver
```

---

## **Step 3: Set Up RBAC for Source and Destination Clusters**

You need to create appropriate `ClusterRole`, `RoleBinding`, and `ServiceAccount` for Prometheus to access the necessary metrics on the source cluster and for Thanos to access them on the destination cluster.

### **Source Cluster RBAC**

This RBAC will allow Prometheus to collect the required metrics.

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: prometheus-sa
  namespace: monitoring
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: prometheus-role
rules:
- apiGroups: [""]
  resources: ["nodes", "pods", "services", "endpoints", "events"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["monitoring.coreos.com"]
  resources: ["prometheuses", "servicemonitors", "alertmanagers"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: prometheus-role-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: prometheus-role
subjects:
- kind: ServiceAccount
  name: prometheus-sa
  namespace: monitoring
```

### **Destination Cluster RBAC**

On the destination cluster, allow Thanos to receive and process the Prometheus data.

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: thanos-sa
  namespace: thanos
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: thanos-role
rules:
- apiGroups: [""]
  resources: ["pods", "services", "endpoints"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["monitoring.coreos.com"]
  resources: ["prometheuses", "thanosrulers"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: thanos-role-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: thanos-role
subjects:
- kind: ServiceAccount
  name: thanos-sa
  namespace: thanos
```

---

## **Step 4: Configure Thanos Query**

Thanos Query allows you to query metrics across different Prometheus instances. Deploy it on the destination cluster.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: thanos-query
  namespace: thanos
spec:
  replicas: 1
  selector:
    matchLabels:
      app: thanos-query
  template:
    metadata:
      labels:
        app: thanos-query
    spec:
      containers:
      - name: thanos-query
        image: thanosio/thanos:v0.22.0
        args:
        - query
        - --grpc-address=0.0.0.0:10901
        - --http-address=0.0.0.0:10902
        - --store=thanos-receiver.thanos.svc.cluster.local:10901
        ports:
        - containerPort: 10901
        - containerPort: 10902
---
apiVersion: v1
kind: Service
metadata:
  name: thanos-query
  namespace: thanos
spec:
  ports:
  - name: grpc
    port: 10901
  - name: http
    port: 10902
  selector:
    app: thanos-query
```

---

## **Step 5: Configure ServiceMonitor (Optional)**

If you're using Prometheus to scrape metrics for Thanos components, you can define a `ServiceMonitor`.

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: thanos-query-monitor
  namespace: thanos
spec:
  selector:
    matchLabels:
      app: thanos-query
  endpoints:
  - port: http
    interval: 30s
```

---

## **Step 6: Verify the Setup**

1. **Check the Metrics Flow**: 
   Ensure that the Prometheus in the source cluster is sending metrics to the Thanos receiver in the destination cluster.
   
2. **Access Thanos Query**:
   You can access Thanos Query by port-forwarding the Thanos Query service or exposing it through a route.

```bash
oc port-forward svc/thanos-query 9090:10902 -n thanos
```

---

### README.md

```markdown
# Thanos Listener Architecture for OpenShift

This repository contains the setup and RBAC configuration to deploy a Thanos architecture across two OpenShift clusters. The **source cluster** sends Prometheus metrics to the **destination cluster**, where Thanos listeners collect and query the metrics.

## Prerequisites

- OpenShift 4.x
- Prometheus Operator installed on the source cluster
- Thanos installed on the destination cluster

## Architecture Overview

- **Source Cluster**: Prometheus is configured to use `remote_write` to send metrics to the Thanos Receiver in the destination cluster.
- **Destination Cluster**: Thanos components (Receiver, Query, Store, etc.) are installed to aggregate, store, and query metrics.

## Installation Steps

1. Install Prometheus Operator on the source cluster if it's not already installed.
2. Configure Prometheus on the source cluster to `remote_write` to the Thanos Receiver.
3. Deploy Thanos components on the destination cluster using the provided YAML files.
4. Set up RBAC roles for Prometheus on the source cluster and Thanos on the destination cluster.
5. Verify metrics are flowing and accessible through the Thanos Query service.

## RBAC Configuration

Refer to the YAML files for setting up service accounts, roles, and role bindings for both the source and destination clusters.
```

This setup provides a working solution for cross-cluster Prometheus metrics aggregation using Thanos. Let me know if you need further refinements or customizations!
