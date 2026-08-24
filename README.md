# Project 7 — Kubernetes Services & Load Balancing

## Project Objective

The objective of Project 7 was to understand how **Kubernetes Services provide networking and load balancing for Pods**.

This project covered:

* ClusterIP Service
* Service-to-Service communication
* Kubernetes DNS
* Endpoints and EndpointSlices
* Load balancing between Pods
* NodePort Service concept
* LoadBalancer Service
* AWS Classic Load Balancer integration with EKS
* External access to Kubernetes applications

---

# 1. Project Architecture

```text
                 Internet / Browser
                        │
                        ▼
              AWS Classic Load Balancer
                        │
                        ▼
           Kubernetes LoadBalancer Service
                        │
                        ▼
                  Kubernetes Pods
              ┌─────────┼─────────┐
              ▼         ▼         ▼
           Pod 1      Pod 2      Pod 3
```

For internal communication:

```text
Pod A
  │
  ▼
ClusterIP Service
  │
  ▼
Pod B
```

---

# 2. ClusterIP Service

A **ClusterIP Service** exposes an application only inside the Kubernetes cluster.

Example:

```text
Pod A
   │
   │ http://pod-b-service
   ▼
ClusterIP Service
   │
   ▼
Pod B
```

The Service provides a stable IP address and DNS name.

Pods can communicate using:

```text
http://pod-b-service
```

instead of directly using Pod IP addresses.

---

# 3. Service DNS

Kubernetes automatically creates DNS records for Services.

For example:

```text
pod-b-service
```

can resolve inside the same Namespace.

The full DNS name follows:

```text
service-name.namespace.svc.cluster.local
```

Example:

```text
pod-b-service.project-7.svc.cluster.local
```

---

# 4. Service Endpoints

When a Service selects Pods using labels, Kubernetes creates Endpoints representing the Pod IP addresses.

Example:

```bash
kubectl get endpoints -n project-7
```

The Endpoint contains something similar to:

```text
192.168.x.x:80
```

This represents the backend Pod where traffic is sent.

---

# 5. EndpointSlices

Modern Kubernetes uses **EndpointSlices** to manage Service endpoints more efficiently.

Command used:

```bash
kubectl get endpointslices -n project-7
```

We verified that multiple Pod IP addresses were registered.

Example concept:

```text
Service
   │
   ├── Pod IP 1
   ├── Pod IP 2
   └── Pod IP 3
```

---

# 6. Kubernetes Service Load Balancing

A Service can distribute traffic between multiple Pods.

The hostname Deployment contained multiple replicas.

Traffic flow:

```text
Client
   │
   ▼
Kubernetes Service
   │
   ├──── Pod 1
   ├──── Pod 2
   └──── Pod 3
```

Testing multiple requests showed responses from different Pods.

Example:

```bash
for i in $(seq 1 10); do
  wget -qO- http://hostname-service
  echo
done
```

Output showed different Pod names, confirming Service-level load balancing.

---

# 7. NodePort Service

A NodePort Service exposes an application through:

```text
Node Public IP + NodePort
```

Concept:

```text
Internet
   │
   ▼
Node Public IP:NodePort
   │
   ▼
Kubernetes Service
   │
   ▼
Pods
```

Example:

```text
http://NODE-PUBLIC-IP:30080
```

We faced external connectivity issues during testing, so we moved forward after understanding the NodePort concept.

---

# 8. LoadBalancer Service

We created a Service with:

```yaml
type: LoadBalancer
```

Kubernetes created an AWS Load Balancer automatically.

Command:

```bash
kubectl get svc -n project-7
```

The Service showed:

```text
TYPE           LoadBalancer
EXTERNAL-IP    AWS ELB DNS Name
```

The traffic flow became:

```text
Browser
   │
   ▼
AWS Load Balancer
   │
   ▼
Kubernetes LoadBalancer Service
   │
   ▼
Service Endpoints
   │
   ▼
Application Pods
```

---

# 9. Load Balancer Verification

We verified the Load Balancer using:

```bash
kubectl describe svc hostname-loadbalancer -n project-7
```

Important information included:

```text
Type: LoadBalancer
LoadBalancer Ingress: <AWS ELB DNS>
Endpoints: Pod-IP:80
```

The events showed:

```text
EnsuringLoadBalancer
EnsuredLoadBalancer
```

This confirmed Kubernetes successfully requested and created the AWS Load Balancer.

---

# 10. AWS Load Balancer Verification

We verified the created AWS Classic Load Balancer using:

```bash
aws elb describe-load-balancers \
  --region ap-south-2
```

The Load Balancer was confirmed as:

```text
Scheme: internet-facing
```

This meant it was intended to accept traffic from outside the AWS network.

---

# 11. DNS Issue and Resolution

Initially, the ELB DNS name returned:

```text
DNS_PROBE_FINISHED_NXDOMAIN
```

and `nslookup` also initially failed.

After waiting for DNS provisioning and propagation:

```bash
nslookup $ELB
```

returned AWS public IP addresses.

Example concept:

```text
ELB DNS Name
       │
       ▼
AWS Public IP Address 1
AWS Public IP Address 2
```

We then successfully accessed the Load Balancer.

---

# 12. Final Application Test

We tested the Load Balancer using:

```bash
curl http://$ELB
```

The response was:

```text
Hello from Pod: hostname-deployment-xxxxx
```

Finally, opening the ELB DNS name in the browser successfully displayed:

```text
Hello from Pod: hostname-deployment-6556bdc69-78gjk
```

This confirmed complete external connectivity.

---

# Final Traffic Flow

```text
User Browser
      │
      ▼
AWS Classic Load Balancer
      │
      ▼
Kubernetes LoadBalancer Service
      │
      ▼
Kubernetes EndpointSlice
      │
      ▼
Application Pods
```

---

# Key Learnings

### ClusterIP

```text
Internal Kubernetes communication
```

### NodePort

```text
External access using:

Node IP + Port
```

### LoadBalancer

```text
External access through a Cloud Provider Load Balancer
```

### Service

A Kubernetes Service provides:

* Stable networking
* DNS
* Pod discovery
* Traffic distribution
* Load balancing

---

# Project 7 Status

## ✅ COMPLETED

**Project 7: Kubernetes Services & Load Balancing**

Main achievement:

```text
Internal Pod Communication
        +
Service Discovery
        +
Kubernetes DNS
        +
Service Load Balancing
        +
AWS External Load Balancer
        =
Complete Kubernetes Service Networking Practice
```
