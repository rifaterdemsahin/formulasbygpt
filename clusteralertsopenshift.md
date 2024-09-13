Here are 10 sample OpenShift alerts for a ChatOps implementation aimed at Site Reliability Engineering (SRE) teams:

1. **Pod CPU Usage Alert**
   - **Alert:** `High CPU Usage`
   - **Description:** CPU usage on pod `{{pod_name}}` has exceeded the threshold of `{{threshold}}%`.
   - **Severity:** High
   - **Query:** `sum(rate(container_cpu_usage_seconds_total{pod="{{pod_name}}"})[5m]) > {{threshold}}`

2. **Pod Memory Usage Alert**
   - **Alert:** `High Memory Usage`
   - **Description:** Memory usage on pod `{{pod_name}}` has exceeded `{{threshold}} MiB`.
   - **Severity:** High
   - **Query:** `container_memory_usage_bytes{pod="{{pod_name}}"} > {{threshold}} * 1024 * 1024`

3. **Pod Restart Alert**
   - **Alert:** `Pod Restarted`
   - **Description:** Pod `{{pod_name}}` has restarted more than `{{threshold}}` times in the last `{{time_period}}`.
   - **Severity:** Medium
   - **Query:** `changes(kube_pod_container_status_restarts_total{pod="{{pod_name}}"}[{{time_period}}]) > {{threshold}}`

4. **Node Disk Pressure Alert**
   - **Alert:** `Node Disk Pressure`
   - **Description:** Node `{{node_name}}` is experiencing disk pressure.
   - **Severity:** High
   - **Query:** `kube_node_status_condition{condition="DiskPressure",status="true"} == 1`

5. **Service Unavailable Alert**
   - **Alert:** `Service Unavailable`
   - **Description:** Service `{{service_name}}` is not reachable or has downtime.
   - **Severity:** Critical
   - **Query:** `probe_success{job="kubernetes-service-endpoints",service="{{service_name}}"} == 0`

6. **Deployment Rollout Failure Alert**
   - **Alert:** `Deployment Rollout Failure`
   - **Description:** Deployment `{{deployment_name}}` has failed to rollout successfully.
   - **Severity:** Critical
   - **Query:** `kube_deployment_status_replicas_unavailable{deployment="{{deployment_name}}"} > 0`

7. **Ingress Controller Error Rate Alert**
   - **Alert:** `High Error Rate`
   - **Description:** Ingress controller `{{controller_name}}` is experiencing high error rates.
   - **Severity:** Medium
   - **Query:** `rate(ingress_controller_errors_total{controller="{{controller_name}}"}) > {{threshold}}`

8. **Network Latency Alert**
   - **Alert:** `High Network Latency`
   - **Description:** Network latency for service `{{service_name}}` has exceeded `{{threshold}} ms`.
   - **Severity:** Medium
   - **Query:** `avg(rate(network_latency_seconds{service="{{service_name}}"})[5m]) > {{threshold}} / 1000`

9. **Etcd Cluster Health Alert**
   - **Alert:** `Etcd Cluster Unhealthy`
   - **Description:** Etcd cluster health is degraded.
   - **Severity:** Critical
   - **Query:** `etcd_server_has_leader{job="etcd"} == 0`

10. **Custom Metric Threshold Alert**
    - **Alert:** `Custom Metric Threshold Exceeded`
    - **Description:** Custom metric `{{metric_name}}` has exceeded the threshold of `{{threshold}}`.
    - **Severity:** Medium
    - **Query:** `custom_metric{job="{{job_name}}"}` > {{threshold}}

These alerts can be customized based on your specific OpenShift environment and monitoring requirements.
