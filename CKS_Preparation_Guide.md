# KS Preparation Guide

## 1. Cluster Setup - 10%

### 1.1 Network security policies

- create default deny all NetPol
- create ingress/egress NetPol - ns, pod, port matching rules

### 1.2 Install & Fix using kube-bench

**kube-bench:** tool to check the cluster of CIS Benchmarks
- Can Deploy as a Docker Container
- Can Deploy as a POD in a Kubernetes cluster
- Can Install kube-bench binaries
- Can Compile

**Download and Run yaml**
```
kubectl create -f https://raw.githubusercontent.com/aquasecurity/kube-bench/main/jobmaster.yaml
kubectl create -f https://raw.githubusercontent.com/aquasecurity/kube-bench/main/job-node.yaml
```

- `kubectl logs jobmaster-xxx`
- `kubectl logs job-node-xxx`

**binary download**

- `curl -L https://github.com/aquasecurity/kube-bench/releases/download/v0.4.0/kube-bench_0.4.0_linux_amd64.tar.gz -o kube-bench_0.4.0_linux_amd64.tar.gz`
- `tar -xvf kube-bench_0.4.0_linux_amd64.tar.gz`
- `cd kube-bench_0.4.0_linux_amd64`

**How to Fix?:**
perform the the remidiations
kubelet config is at /var/lib/kubelet/config.yaml
sudo systemctl restart kubelet

### 1.3 Ingress TLS termination
- Secure an Ingress by specifying a Secret that contains a TLS private key and certificate
- The Ingress resource only supports a single TLS port, 443, and assumes TLS termination at the ingress point

```
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
refer: https://kubernetes.io/docs/concepts/services-networking/ingress/#tls

### 1.4 Protect node metadata and endpoints with NP

Using Kubernetes network policy to restrict pods access to cloud metadata

```
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

### 1.5 Minimize use of, and access to, GUI elements

- Access to Kubernetes Dashboard
- Creating a Service Account
- Create ClusterRoleBinding
- Getting a Bearer Token

### 1.6 Verify platform binaries before deploying

```
kubectl version --short --client

#download checksum for kubectl
curl -LO "https://dl.k8s.io/<kubectl client version>/bin/linux/amd64/kubectl.sha256"

#verify kubectl binary
echo "$(<kubectl.sha256) /usr/bin/kubectl" | sha256sum --check
```

## 2. Cluster Hardening - 15%

## 3. System Hardening - 15%

## 4. Minimize Microservice Vulnerabilities - 20%

## 5. Supply Chain Security - 20%

## 6. Monitoring, Logging and Runtime Security20%
