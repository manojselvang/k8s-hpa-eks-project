\# Kubernetes HPA Scaling Analysis



\# Overview



This document analyzes the autoscaling behavior observed during Kubernetes Horizontal Pod Autoscaler (HPA) testing on Amazon EKS.



The analysis focuses on:

\- CPU utilization behavior

\- HPA scaling decisions

\- replica scaling progression

\- workload distribution

\- stabilization behavior

\- scale-out and scale-in lifecycle



\---



\# Initial Deployment State



The Kubernetes deployment initially started with:



| Parameter | Value |

|---|---|

| Initial Replicas | 1 |

| Minimum Replicas | 1 |

| Maximum Replicas | 10 |

| HPA Target CPU Utilization | 50% |



The application pod was deployed with the following CPU configuration:



```yaml

resources:

&#x20; requests:

&#x20;   cpu: "100m"



&#x20; limits:

&#x20;   cpu: "500m"

```



\---



\# Why CPU Requests Matter



Horizontal Pod Autoscaler calculates utilization percentages based on:



```text

Current CPU Usage / Requested CPU

```



NOT:

```text

Current CPU Usage / CPU Limit

```



Example:



If:

\- CPU request = 100m

\- current usage = 50m



Then:



```text

50m / 100m = 50%

```



This directly affects HPA scaling decisions.



\---



\# Load Generation Phase



Traffic was generated using BusyBox inside the Kubernetes cluster.



Command used:



```bash

kubectl run load-generator --image=busybox --restart=Never --command -- sh -c "while true; do wget -q -O- http://hpa-demo-service; done"

```



The load generator continuously sent HTTP requests to the ClusterIP service.



\---



\# CPU Utilization Growth



During testing, CPU utilization increased progressively.



Observed utilization:



```text

21% → 73% → 148%

```



The configured HPA threshold was:



```text

50%

```



Once utilization exceeded the threshold:

\- HPA detected resource pressure

\- autoscaling logic triggered

\- additional replicas were created



\---



\# HPA Scaling Formula



Kubernetes HPA calculates desired replicas using:



DesiredReplicas = CurrentReplicas × (CurrentMetricValue / DesiredMetricValue)



Observed scaling example:



```text

DesiredReplicas = 1 × (148 / 50) ≈ 2.96

```



Since CPU utilization remained above the configured threshold:

\- HPA progressively scaled the deployment

\- additional scaling cycles occurred



\---



\# Replica Scaling Progression



Observed scaling lifecycle:



```text

1 → 2 → 3 → 4

```



This demonstrated:

\- sustained CPU pressure

\- repeated HPA reconciliation cycles

\- dynamic autoscaling behavior



Maximum replicas observed:



```text

4 replicas

```



\---



\# Workload Distribution Effect



After additional replicas were created:

\- traffic distribution improved

\- workload spread across multiple pods

\- CPU utilization reduced gradually



Observed CPU reduction:



```text

58% → 44% → 31% → 15% → 0%

```



This validated:

\- successful load balancing

\- effective horizontal scaling

\- reduced pressure per pod



\---



\# Scale-Out Behavior Analysis



The scale-out process was not instantaneous.



Several Kubernetes components contributed to scaling delay:



\- Metrics collection interval

\- Metrics API synchronization

\- HPA reconciliation timing

\- Kubernetes scheduler operations

\- pod startup initialization

\- container startup time



This behavior is normal in Kubernetes autoscaling environments.



\---



\# Scale-In Behavior



After removing the load generator:

\- CPU utilization reduced

\- workload pressure disappeared

\- HPA eventually reduced replica count



Observed scale-in lifecycle:



```text

4 → 3 → 2 → 1

```



Final stable state:



```text

1 replica

```



\---



\# Stabilization Window Behavior



Kubernetes HPA intentionally delays immediate scale-in operations.



Purpose:

```text

Prevent scaling flapping

```



Flapping refers to:

\- rapid scale-out

\- immediate scale-in

\- unstable scaling oscillation



The stabilization window improves:

\- workload stability

\- cluster consistency

\- application reliability



\---



\# Metrics Pipeline Analysis



The autoscaling workflow depended on the following metrics pipeline:



```text

Pods

&#x20; ↓

kubelet

&#x20; ↓

Metrics Server

&#x20; ↓

Metrics API

&#x20; ↓

HPA Controller

```



Without Metrics Server:

\- HPA cannot retrieve utilization metrics

\- scaling decisions cannot occur



Metrics availability was verified using:



```bash

kubectl top nodes

kubectl top pods

```



\---



\# Operational Observations



\## Observation 1 — Sustained CPU Pressure



CPU utilization reached:

```text

148%

```



This demonstrated:

\- successful load generation

\- real CPU pressure

\- valid autoscaling conditions



\---



\# Observation 2 — Progressive Scaling



HPA did not scale directly from:

```text

1 → 4

```



Instead:

```text

1 → 2 → 3 → 4

```



This demonstrated:

\- gradual reconciliation logic

\- continuous metrics evaluation

\- controlled scaling behavior



\---



\# Observation 3 — Automatic Load Distribution



After scaling:

\- CPU utilization reduced automatically

\- traffic spread across replicas

\- workload stabilized



This validated effective horizontal scaling.



\---



\# Observation 4 — Controlled Scale-In



Scale-in occurred gradually after load removal.



This demonstrated:

\- Kubernetes stabilization logic

\- flapping prevention

\- controlled workload normalization



\---



\# Key Learnings



\- HPA depends entirely on Metrics API availability

\- CPU requests directly influence scaling calculations

\- Kubernetes scaling is gradual, not immediate

\- Sustained CPU pressure triggers progressive scaling

\- Traffic distribution reduces CPU utilization after scale-out

\- Stabilization windows prevent unstable scaling behavior

\- Real-time monitoring is critical during autoscaling validation



\---



\# Commands Used During Analysis



\## Monitor HPA



```bash

kubectl get hpa -w

```



\## Monitor Pods



```bash

kubectl get pods -w

```



\## Monitor Metrics



```bash

kubectl top pods --watch

```



\---



\# Conclusion



The scaling analysis successfully validated:

\- Kubernetes Horizontal Pod Autoscaler functionality

\- CPU-based scaling decisions

\- Metrics-driven autoscaling

\- workload distribution effectiveness

\- scale-out and scale-in lifecycle

\- Kubernetes stabilization behavior



The implementation demonstrated practical Kubernetes autoscaling operations in an AWS EKS environment under sustained CPU load conditions.

