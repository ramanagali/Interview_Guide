# CKS Preparation Guide

## 1. Cluster Setup - 10%

### 1.1 Network security policies

- Create default deny all NetworkPolicy
- Create ingress/egress NetPol - ns, pod, port matching rules
- Ref: https://kubernetes.io/docs/concepts/services-networking/network-policies/

### 1.2 Install & Fix using kube-bench

**kube-bench:** Tool to check Kubernetes cluster CIS Kubernetes Benchmarks
- Can Deploy as a Docker Container
- Can Deploy as a POD in a Kubernetes cluster
- Can Install kube-bench binaries
- Can Compile

**Download and Run yaml**

```sh
kubectl create -f https://raw.githubusercontent.com/aquasecurity/kube-bench/main/jobmaster.yaml
kubectl create -f https://raw.githubusercontent.com/aquasecurity/kube-bench/main/job-node.yaml
```

```sh
kubectl logs jobmaster-xxx
kubectl logs job-node-xxx
```

**binary download**

```sh
curl -L https://github.com/aquasecurity/kube-bench/releases/download/v0.4.0/kube-bench_0.4.0_linux_amd64.tar.gz -o kube-bench_0.4.0_linux_amd64.tar.gz
tar -xvf kube-bench_0.4.0_linux_amd64.tar.gz
cd kube-bench_0.4.0_linux_amd64
```

**How to Fix?:**
* perform the the remidiations
* kubelet config located at /var/lib/kubelet/config.yaml
* sudo systemctl restart kubelet

### 1.3 Ingress TLS termination
- Secure an Ingress by specifying a Secret that contains a TLS private key and certificate
- The Ingress resource only supports a single TLS port, 443, and assumes TLS termination at the ingress point

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: testsecret-tls
  namespace: default
data:
  tls.crt: base64 encoded cert
  tls.key: base64 encoded key
type: kubernetes.io/tls
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-example-ingress
spec:
  tls:
  - hosts:
      - https-example.foo.com
    secretName: testsecret-tls
  rules:
  - host: https-example.foo.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: service1
            port:
              number: 80
```
Ref: https://kubernetes.io/docs/concepts/services-networking/ingress/#tls

### 1.4 Protect node metadata and endpoints with NetworkPolicy

Using Kubernetes network policy to restrict pods access to cloud metadata

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-only-cloud-metadata-access
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
      cidr: 0.0.0.0/0
      except:
      - 169.254.169.254/32
```
Ref: https://kubernetes.io/docs/tasks/administer-cluster/securing-a-cluster/#restricting-cloud-metadata-api-access

### 1.5 Minimize use of, and access to, GUI elements

- Access to Kubernetes Dashboard
- Creating a Service Account
- Create ClusterRoleBinding
- Getting a Bearer Token
- Ref: https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/#accessing-the-dashboard-ui

### 1.6 Verify platform binaries before deploying

```sh
kubectl version --short --client

#download checksum for kubectl
curl -LO "https://dl.k8s.io/<kubectl client version>/bin/linux/amd64/kubectl.sha256"

#verify kubectl binary
echo "$(<kubectl.sha256) /usr/bin/kubectl" | sha256sum --check
```

Ref: 

## 2. Cluster Hardening - 15%

## 3. System Hardening - 15%

## 4. Minimize Microservice Vulnerabilities - 20%

## 5. Supply Chain Security - 20%

## 6. Monitoring, Logging and Runtime Security20%
