# CKS Preparation Guide

## 1. Cluster Setup - 10%
<details>
<summary></summary>

### 1.1 Network security policies

- Create default deny all NetworkPolicy
- Create ingress/egress NetPol - ns, pod, port matching rules
- Ref: <https://kubernetes.io/docs/concepts/services-networking/network-policies/>

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

**binary download & run**

```sh
curl -L https://github.com/aquasecurity/kube-bench/releases/download/v0.4.0/kube-bench_0.4.0_linux_amd64.tar.gz -o kube-bench_0.4.0_linux_amd64.tar.gz
tar -xvf kube-bench_0.4.0_linux_amd64.tar.gz
cd kube-bench_0.4.0_linux_amd64

kube-bench
./kube-bench --config-dir `pwd`/cfg --config `pwd`/cfg/config.yaml 
```

**docker**

```
docker run --rm -v `pwd`:/host aquasec/kube-bench:latest install
then ./kube-bench
```

**How to Fix?:**

- Read the the remidiation for each finding
- Kubelet config located at /var/lib/kubelet/config.yaml
- ```sudo systemctl restart kubelet```
- Control plane components a ``` /etc/kubernetes/manifests/```

Ref: <https://github.com/aquasecurity/kube-bench/blob/main/docs/installation.md>

### 1.3 Ingress TLS termination

- Secure an Ingress by specifying a Secret that contains a TLS private key and certificate
- The Ingress resource only supports a single TLS port, 443, and assumes TLS termination at the ingress point

Create TLS Certificate & key

```sh
openssl req -nodes -new -x509 -keyout tls-ingress.key -out tls-ingress.crt -subj "/CN=ingress.test
```

Apply this yaml

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: ingress-tls
  namespace: ingresstest
data:
  tls.crt: |
    $(base64-encoded cert data from tls-ingress.crt)
  tls.key: |
    $(base64-encoded key data from tls-ingress.key)
type: kubernetes.io/tls
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-example-ingress
  namespace: ingresstest
spec:
  tls:
  - hosts:
      - ingress.test
    secretName: ingress-tls
  rules:
  - host: ingress.test
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

Ref: <https://kubernetes.io/docs/concepts/services-networking/ingress/#tls>

### 1.4 Protect node metadata and endpoints with NetworkPolicy

- Restrict control plane ports (6443, 2379, 2380, 10250, 10251, 10252)
- Restrict worker node ports(10250, 30000-32767)
- for Cloud, Using Kubernetes network policy to restrict pods access to cloud metadata

Example assumes AWS cloud, and metadata IP address is 169.254.169.254 should be blocked while all other external addresses are not.

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

- <https://kubernetes.io/docs/tasks/administer-cluster/securing-a-cluster/#restricting-cloud-metadata-api-access>

### 1.5 Minimize use of, and access to, GUI elements

- Restrit Access to GUI like Kubernetes Dashboard

**Solution1**

- Creating a Service Account User

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
```

- Create ClusterRoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard

```

- Retrieve Bearer Token & Use

```
kubectl -n kubernetes-dashboard get secret $(kubectl -n kubernetes-dashboard get sa/admin-user -o jsonpath="{.secrets[0].name}") -o go-template="{{.data.token | base64decode}}"
```

**Solution2**

- use `kubectl proxy` to access to the Dashboard <http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/>.

Ref: <https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/#accessing-the-dashboard-ui>

Ref: <https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md>

### 1.6 Verify platform binaries before deploying

- binaries like kubectl, kubeadm and kubelets
- before using binaries compare checksum with its official sha512 hash (cryptographic hash)

Example
```sh
kubectl version --short --client

#download checksum for kubectl
curl -LO "https://dl.k8s.io/v1.20.1/bin/linux/amd64/kubectl.sha256"


#verify kubectl binary
echo "$(<kubectl.sha256) /usr/bin/kubectl" | sha256sum --check
```

Ref: https://github.com/kubernetes/kubernetes/releases
Ref: https://github.com/kubernetes/kubernetes/tree/master/CHANGELOG#changelogs
</details>
<hr /> 

## 2. Cluster Hardening - 15%
<details>
<summary></summary>

### 2.1  Restrict access to Kubernetes API

* Control anonymous requests to Kube-apiserver by using 
```sh --anonymous-auth=false ```
*  non secure access to the kube-apiserver
  1. **localhost** 
      *  port 8080
      *  no TLS
      *  default IP is localhost, change with `--insecure-bind-address`
  2. **secure port**
      *  default is 6443, change with `--secure-port`
      *  set TLS certificate with `--tls-cert-file`
      *  set TLS certificate key with `--tls-private-key-file` flag

Ref: https://kubernetes.io/docs/concepts/security/controlling-access/#api-server-ports-and-ips

Ref: https://kubernetes.io/docs/concepts/security/controlling-access/#api-server-ports-and-ips


### 2.2 Use Role-Based Access Controls to minimize exposure

Roles live in namespace, RoleBinding specific to ns
ClusterRoles live across all namespace, ClusterRoleBidning
ServiceAccount should have only necessary RBAC permissions

**Solution**
* Create virutal users using ServiceAccount for specific namespace
* Create Role in specific namespace 
  * has resources (ex: deployment)
  * has verbs (get, list, create, delete))
* Create RoleBinding n specific namespace & link Role & ServiceAccount
  * can be user, group or service account
* specify service account in deployment/pod level
```yaml
spec:
  serviceAccountName: deployment-viewer-sa
```
Ref: https://kubernetes.io/docs/reference/access-authn-authz/rbac/

### 2.3 Exercise caution in using service accounts e.g. disable defaults, minimize permissions on newly created ones

* Create ServiceAccount to automount to any pod `automountServiceAccountToken: false`
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: build-robot
automountServiceAccountToken: false
```
* Create Pod with serviceAccountName: default, automountServiceAccountToken: false
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  serviceAccountName: build-robot
  automountServiceAccountToken: false
```
Ref: https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#use-the-default-service-account-to-access-the-api-server

### 2.4 Update Kubernetes frequently
* Minor versions(bug fixes) must be patched regularly
* Latest 3 Minor versions receive patch support
* Minor versions receive patches for ~1year


Ref: https://v1-21.docs.kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/


</details>
<hr /> 

## 3. System Hardening - 15%
<details>
<summary></summary>

### 3.1 Minimize host OS footprint (reduce attack surface)
* Limit Node Access
* Remove Obsolete Software
* Limit Access
* Remove Obsolete Services
* Restrict Obsolete Kernal Modules
* Identify and Fix Open Ports

* Create Pod with below when needed.
```yaml
spec: 
  hostIPC: false
  hostNetwork: false
  hostPID: false
```
* dont use  containers[].securityContext.privileged = ture when needed
 *  
### 3.2 Minimize IAM roles

### 3.3. Minimize external access to the network

### 3.4 Appropriately use kernel hardening tools such as AppArmor, seccomp

</details>
<hr /> 

## 4. Minimize Microservice Vulnerabilities - 20%
<details>
<summary></summary>

### 4.1 etup appropriate OS level security domains e.g. using PSP, OPA, security contexts

### 4.2 Manage Kubernetes secrets

### 4.3 Use container runtime sandboxes in multi-tenant environments (e.g. gvisor, kata containers)

### 4.4 Implement pod to pod encryption by use of mTLS

</details>
<hr /> 

## 5. Supply Chain Security - 20%
<details>
<summary></summary>
### 5.1 Minimize base image footprint

### 5.2 Secure your supply chain: whitelist allowed registries, sign and validate images

### 5.3 Use static analysis of user workloads (e.g.Kubernetes resources, Docker files)

### 5.4 Scan images for known vulnerabilities

</details>

## 6. Monitoring, Logging and Runtime Security - 20%

<details>
<summary></summary>

### 6.1 Perform behavioral analytics of syscall process and file activities at the host and container level to detect malicious activities

### 6.2 Detect threats within physical infrastructure, apps, networks, data, users and workloads
### 6.3 Detect all phases of attack regardless where it occurs and how it spreads

### 6.4 Perform deep analytical investigation and identification of bad actors within environment

### 6.5 Ensure immutability of containers at runtime

### 6.6 Use Audit Logs to monitor access

</details>