\# Kubernetes HPA Troubleshooting Guide



\# Overview



This document contains the issues encountered during the implementation of Kubernetes Horizontal Pod Autoscaler (HPA) on AWS EKS and the resolutions used to fix them.



The troubleshooting process covered:

\- Metrics Server issues

\- Metrics API availability

\- HPA visibility problems

\- Windows CMD command parsing issues

\- Autoscaling verification challenges



\---



\# 1. Metrics API Not Available



\## Issue



While verifying Kubernetes metrics, the following error occurred:



```bash

error: Metrics API not available

```



\---



\# Root Cause



The Kubernetes Metrics Server was unable to communicate properly with kubelets running on EKS worker nodes due to TLS verification issues.



This is a common issue in many EKS environments.



\---



\# Resolution



The Metrics Server deployment was modified to include:



```yaml

\- --kubelet-insecure-tls

```



The deployment was updated using:



```bash

kubectl edit deployment metrics-server -n kube-system

```



\---



\# Verification



Metrics functionality was verified using:



```bash

kubectl top nodes

```



```bash

kubectl top pods -A

```



Successful output confirmed:

\- Metrics Server functionality

\- Metrics API availability

\- HPA prerequisites readiness



\---



\# 2. HPA TARGETS Showing `<unknown>`



\## Issue



After creating the Horizontal Pod Autoscaler, the TARGETS field displayed:



```text

<unknown>/50%

```



\---



\# Root Cause



The Metrics API pipeline had not fully synchronized immediately after HPA creation.



HPA requires:

\- Metrics Server

\- Metrics API synchronization

\- pod resource metrics availability



before utilization values become visible.



\---



\# Resolution



Waited for:

\- Metrics Server synchronization

\- pod metrics collection

\- Metrics API stabilization



Verification commands:



```bash

kubectl top nodes

```



```bash

kubectl top pods

```



After synchronization:

\- CPU utilization metrics appeared correctly

\- HPA status updated successfully



\---



\# 3. Windows CMD Load Generator Failure



\## Issue



The BusyBox load generation command failed in Windows CMD.



Original command:



```bash

kubectl run load-generator \\

\--image=busybox \\

\--restart=Never \\

\-- /bin/sh -c "while true; do wget -q -O- http://hpa-demo-service; done"

```



\---



\# Root Cause



Windows CMD handled:

\- multiline commands

\- shell quotes

\- Linux shell syntax



differently from Linux terminals.



This caused command parsing issues.



\---



\# Resolution



Used a single-line command format:



```cmd

kubectl run load-generator --image=busybox --restart=Never --command -- sh -c "while true; do wget -q -O- http://hpa-demo-service; done"

```



Alternative approach:

\- created BusyBox pod

\- accessed shell using kubectl exec

\- generated traffic manually



\---



\# 4. HPA Not Scaling Initially



\## Issue



Initially, HPA did not scale immediately after load generation started.



CPU utilization remained low temporarily.



\---



\# Root Cause



Several Kubernetes timing factors influence autoscaling:



\- Metrics collection interval

\- HPA sync period

\- pod startup delay

\- scheduler timing

\- Metrics API refresh interval



HPA scaling is not instantaneous.



\---



\# Resolution



Waited for:

\- sustained CPU utilization increase

\- Metrics API updates

\- HPA reconciliation cycle



Eventually CPU utilization increased significantly:



```text

21% → 73% → 148%

```



This triggered automatic scale-out operations.



\---



\# 5. Autoscaling Verification Challenges



\## Issue



Need to verify whether scaling was actually occurring dynamically.



\---



\# Resolution



Used continuous monitoring commands:



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



These commands provided:

\- live replica visibility

\- real-time CPU utilization

\- dynamic scaling verification



\---



\# 6. Scale-In Delay



\## Observation



After removing load generation:

\- replicas did not immediately reduce

\- scale-in occurred gradually



Observed scale-in lifecycle:



```text

4 → 3 → 2 → 1

```



\---



\# Root Cause



Kubernetes HPA includes:

```text

stabilization windows

```



to prevent:

```text

scaling flapping

```



Flapping refers to:

\- rapid scale-out

\- immediate scale-in

\- unstable scaling oscillations



\---



\# Resolution



Allowed Kubernetes HPA stabilization logic to complete naturally.



Gradual scale-in successfully reduced replicas back to:

```text

1 pod

```



\---



\# Key Troubleshooting Learnings



\- Metrics Server is mandatory for HPA functionality

\- Metrics API synchronization delays are normal

\- HPA scaling is gradual, not immediate

\- Windows CMD requires shell-specific command handling

\- Kubernetes stabilization windows prevent scaling instability

\- Real-time monitoring commands are essential for autoscaling validation

\- CPU requests directly affect HPA calculations



\---



\# Important Operational Commands



\## Verify Metrics



```bash

kubectl top nodes

kubectl top pods

```



\## Verify HPA



```bash

kubectl get hpa

kubectl describe hpa hpa-demo

```



\## Watch Scaling Live



```bash

kubectl get hpa -w

kubectl get pods -w

```



\## Verify Metrics Server



```bash

kubectl get deployment metrics-server -n kube-system

```



\---



\# Conclusion



The troubleshooting process validated several important Kubernetes operational concepts:

\- Metrics pipeline dependency

\- HPA synchronization behavior

\- autoscaling timing considerations

\- stabilization logic

\- real-time monitoring techniques



Resolving these issues provided deeper understanding of Kubernetes autoscaling internals and operational workflows in AWS EKS environments.

