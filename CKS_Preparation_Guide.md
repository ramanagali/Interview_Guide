# Certified Kubernetes Security Specialist (CKS) Simplified Preparation Guide

Those who know about Kubernetes Administration, for the next level
Certified Kubernetes Security Specialist (CKS) exam point of view, below complete one pager, simplified/shortened/finetuned study material

pull requests are welcome
<hr />

# Table of Contents
   * [1. Cluster Setup - 10%](#1-cluster-setup---10)
      * [1.1 Network security policies](#11-network-security-policies)
      * [1.2 Install &amp; Fix using kube-bench](#12-install--fix-using-kube-bench)
      * [1.3 Ingress TLS termination](#13-ingress-tls-termination)
      * [1.4 Protect node metadata and endpoints with NetworkPolicy](#14-protect-node-metadata-and-endpoints-with-networkpolicy)
      * [1.5 Minimize use of, and access to, GUI elements](#15-minimize-use-of-and-access-to-gui-elements)
      * [1.6 Verify platform binaries before deploying](#16-verify-platform-binaries-before-deploying)
   * [2. Cluster Hardening - 15%](#2-cluster-hardening---15)
      * [2.1  Restrict access to Kubernetes API](#21--restrict-access-to-kubernetes-api)
      * [2.2 Use Role-Based Access Controls to minimize exposure](#22-use-role-based-access-controls-to-minimize-exposure)
      * [2.3 Exercise caution in using service accounts e.g. disable defaults, minimize permissions on newly created ones](#23-exercise-caution-in-using-service-accounts-eg-disable-defaults-minimize-permissions-on-newly-created-ones)
      * [2.4 Update Kubernetes frequently](#24-update-kubernetes-frequently)
   * [3. System Hardening - 15%](#3-system-hardening---15)
      * [3.1 Minimize host OS footprint (reduce attack surface)](#31-minimize-host-os-footprint-reduce-attack-surface)
      * [3.2 Minimize IAM roles](#32-minimize-iam-roles)
      * [3.3. Minimize external access to the network](#33-minimize-external-access-to-the-network)
      * [3.4 Appropriately use kernel hardening tools such as AppArmor, seccomp](#34-appropriately-use-kernel-hardening-tools-such-as-apparmor-seccomp)
         * [3.4.1 SECCOMP PROFILES](#341-seccomp-profiles)
         * [3.4.2 APPARMOR ](#342-apparmor)
   * [4. Minimize Microservice Vulnerabilities - 20%](#4-minimize-microservice-vulnerabilities---20)
      * [4.1 Setup appropriate OS level security domains e.g. using PSP, OPA, security contexts](#41-setup-appropriate-os-level-security-domains-eg-using-psp-opa-security-contexts)
         * [Admission Controller](#admission-controller)
         * [4.1.1 Pod Security Policies (PSP)](#411-pod-security-policies-psp)
         * [4.1.2 Open Policy Agent (OPA)](#412-open-policy-agent-opa)
         * [4.1.3 Security Contexts](#413-security-contexts)
      * [4.2 Manage Kubernetes secrets](#42-manage-kubernetes-secrets)
      * [4.3 Use container runtime sandboxes in multi-tenant environments (e.g. gvisor, kata containers)](#43-use-container-runtime-sandboxes-in-multi-tenant-environments-eg-gvisor-kata-containers)
         * [4.3.1 gvisor](#431-gvisor)
         * [4.3.4 kata containers](#434-kata-containers)
         * [Install gVisor](#install-gvisor)
         * [Container Runtime](#container-runtime)
      * [4.4 Implement pod to pod encryption by use of mTLS](#44-implement-pod-to-pod-encryption-by-use-of-mtls)
   * [5. Supply Chain Security - 20%](#5-supply-chain-security---20)
      * [5.1 Minimize base image footprint](#51-minimize-base-image-footprint)
      * [5.2 Secure your supply chain: whitelist allowed registries, sign and validate images](#52-secure-your-supply-chain-whitelist-allowed-registries-sign-and-validate-images)
      * [5.3 Use static analysis of user workloads (e.g.Kubernetes resources, Docker files)](#53-use-static-analysis-of-user-workloads-egkubernetes-resources-docker-files)
      * [5.4 Scan images for known vulnerabilities](#54-scan-images-for-known-vulnerabilities)
   * [6. Monitoring, Logging and Runtime Security - 20%](#6-monitoring-logging-and-runtime-security---20)
      * [6.1 Perform behavioral analytics of syscall process and file activities at the host and container level to detect malicious activities](#61-perform-behavioral-analytics-of-syscall-process-and-file-activities-at-the-host-and-container-level-to-detect-malicious-activities)
      * [6.2 Detect threats within physical infrastructure, apps, networks, data, users and workloads](#62-detect-threats-within-physical-infrastructure-apps-networks-data-users-and-workloads)
      * [6.3 Detect all phases of attack regardless where it occurs and how it spreads](#63-detect-all-phases-of-attack-regardless-where-it-occurs-and-how-it-spreads)
      * [6.4 Perform deep analytical investigation and identification of bad actors within environment](#64-perform-deep-analytical-investigation-and-identification-of-bad-actors-within-environment)
      * [6.5 Ensure immutability of containers at runtime](#65-ensure-immutability-of-containers-at-runtime)
      * [6.6 Use Audit Logs to monitor access](#66-use-audit-logs-to-monitor-access)
      * [debugging api-server/kubelet failures](#debugging-api-serverkubelet-failures)
   * [IMP Notes Tips for CKSExam](#imp-notes-tips-for-cksexam)
      * [CKS Exam Catergories](#cks-exam-catergories)
      * [CKS Exam Tips](#cks-exam-tips)
  * [CKS Youtube Video full tutorials - Lean with GVR ](#cks---youtube-playlist----my-youtube-channel---learn-with-gvr)
         
<hr />


## 1. Cluster Setup - 10%
[🔼 Back to top](#table-of-contents)


### 1.1 Network security policies                                                    
[🔼 Back to top](#table-of-contents)
- Create default deny all NetworkPolicy & allow required traffic
  ```yaml
  apiVersion: networking.k8s.io/v1
  kind: NetworkPolicy
  metadata:
    name: default-deny-ingress
    namespace: default
  spec:
    podSelector: {}
    policyTypes:
    - Ingress
    ```
- Create ingress/egress NetPol - for ns, pod, port matching rules
  ```yaml
  apiVersion: networking.k8s.io/v1
  kind: NetworkPolicy
  metadata:
    name: test-network-policy
    namespace: default  #target namespace
  spec:
    podSelector:  #target pod
      matchLabels:
        role: db
    policyTypes:
    - Ingress
    - Egress
    ingress:
    - from:
      - ipBlock:
          cidr: 172.17.0.0/16
          except:
          - 172.17.1.0/24
      - namespaceSelector:
          matchLabels:
            project: myproject
      - podSelector:
          matchLabels:
            role: frontend
      ports:
      - protocol: TCP
        port: 6379
    egress:
    - to:
      - ipBlock:
          cidr: 10.0.0.0/24
      ports:
      - protocol: TCP
        port: 5978
        endPort: 6000
  ```
- Ref: <https://kubernetes.io/docs/concepts/services-networking/network-policies/>
- Ref: https://editor.cilium.io/
- 
### 1.2 Install & Fix using kube-bench
[🔼 Back to top](#table-of-contents)

**kube-bench:** Tool to check Kubernetes cluster CIS Kubernetes Benchmarks

- Can Deploy as a Docker Container
  - `docker run --rm -v `pwd`:/host aquasec/kube-bench:latest install`
- Can Deploy as a POD in a Kubernetes cluster
- Can Install kube-bench binaries
- Can Compile

**Download and Run yaml**

```sh
kubectl create -f https://raw.githubusercontent.com/aquasecurity/kube-bench/main/job-master.yaml
kubectl create -f https://raw.githubusercontent.com/aquasecurity/kube-bench/main/job-node.yaml
```

```sh
kubectl logs jobmaster-xxx
kubectl logs job-node-xxx
```

**binary download & run**

```sh
curl -L https://github.com/aquasecurity/kube-bench/releases/download/v0.6.5/kube-bench_0.6.5_linux_amd64.tar.gz -o kube-bench_0.6.5_linux_amd64.tar.gz
tar -xvf kube-bench_0.6.5_linux_amd64.tar.gz
# run all tests
./kube-bench

# run kube bench with specific target
kube-bench run --targets=master/node/etcd

./kube-bench --config-dir `pwd`/cfg --config `pwd`/cfg/config.yaml master
./kube-bench --config-dir `pwd`/cfg --config `pwd`/cfg/config.yaml node
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
- Control plane components a ```/etc/kubernetes/manifests/```

Ref: <https://github.com/aquasecurity/kube-bench/blob/main/docs/installation.md>
### 1.3 Ingress TLS termination
[🔼 Back to top](#table-of-contents)

- Secure an Ingress by specifying a Secret that contains a TLS private key and certificate
- The Ingress resource only supports a single TLS port, 443, and assumes TLS termination at the ingress point

Create TLS Certificate & key

```sh
openssl req -x509 -newkey rsa:4096 -sha256 -nodes -keyout tls.key -out tls.crt -subj "/CN=learnwithgvr.com" -days 365
```
Create a Secret
```sh
kubectl create secret -n ingresstest tls learnwithgvr-sec --cert=tls.crt --key=tls.key
```

Alternatively Apply this yaml

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: learnwithgvr-sec
  namespace: ingresstest
data:
  tls.crt: |
    $(base64-encoded cert data from tls.crt)
  tls.key: |
    $(base64-encoded key data from tls.key)
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
      - learnwithgvr.com
    secretName: learnwithgvr-sec
  rules:
  - host: learnwithgvr.com
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
```sh
# minikube on mac - add to hosts
echo $(minkube ip) learnwithgvr.com > | sudo tee -a /etc/hosts
```

Ref: <https://kubernetes.io/docs/concepts/services-networking/ingress/#tls>

### 1.4 Protect node metadata and endpoints with NetworkPolicy
[🔼 Back to top](#table-of-contents)

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
  - Ingress
  ingress:
  - from:
    - ipBlock:
      cidr: 0.0.0.0/0
      except:
      - 169.254.169.254/32
```

- <https://kubernetes.io/docs/tasks/administer-cluster/securing-a-cluster/#restricting-cloud-metadata-api-access>

### 1.5 Minimize use of, and access to, GUI elements
[🔼 Back to top](#table-of-contents)

- Restrict Access to GUI like Kubernetes Dashboard

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

```sh
kubectl -n kubernetes-dashboard get secret $(kubectl -n kubernetes-dashboard get sa/admin-user -o jsonpath="{.secrets[0].name}") -o go-template="{{.data.token | base64decode}}"
```

```sh
#steps to create serviceaccount & use
kubectl create serviceaccount simple-user -n kube-system
kubectl create clusterrole simple-reader --verb=get,list,watch --resource=pods --resource=pods,deployments,services,configmaps
kubectl create clusterrolebinding cluster-simple-reader --clusterrole=simple-reader --serviceaccount=kube-system:simple-user

SEC_NAME=$(kubectl get serviceAccount simple-user -o jsonpath='{.secrets[0].name}')
USER_TOKEN=$(kubectl get secret $SEC_NAME -o json | jq -r '.data["token"]' | base64 -d)

cluster_name=$(kubectl config get-contexts $(kubectl config current-context) | awk '{print $3}' | tail -n 1)
kubectl config set-credentials simple-user --token="${USER_TOKEN}"
kubectl config set-context simple-reader --cluster=$cluster_name --user simple-user
kubectl config set-context simple-reader
```

**How to Access Dashboard?**

- use `kubectl proxy` to access to the Dashboard <http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/>.

Ref: <https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/#accessing-the-dashboard-ui>

Ref: <https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md>

### 1.6 Verify platform binaries before deploying
[🔼 Back to top](#table-of-contents)

- binaries like kubectl, kubeadm and kubelets
- before using binaries compare checksum with its official sha256/sha512 cryptographic hash value

Example 1
```sh
curl -LO https://dl.k8s.io/v1.23.1/kubernetes-client-darwin-arm64.tar.gz -o kubernetes-client-darwin-arm64.tar.gz

# Print SHA Checksums - mac
shasum -a 512 kubernetes-client-darwin-arm64.tar.gz
# Print SHA Checksums - linux
sha512sum kubernetes-client-darwin-arm64.tar.gz
sha512sum kube-apiserver
sha512sum kube-proxy
```

Example 2

```sh
kubectl version --short --client

#download checksum for kubectl for linux - (change version)
curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"

#download old version
curl -LO "https://dl.k8s.io/v1.22.1/bin/linux/amd64/kubectl.sha256"

#download checksum for kubectl for mac - (change version)
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/amd64/kubectl.sha256"

#insall coreutils (for mac)
brew install coreutils

#verify kubectl binary (for linux)
echo "$(<kubectl.sha256) /usr/bin/kubectl" | sha256sum --check
#verify kubectl binary (for mac)
echo "$(<kubectl.sha256) /usr/local/bin/kubectl" | sha256sum -c
```

* Ref: https://github.com/kubernetes/kubernetes/releases
* Ref: https://github.com/kubernetes/kubernetes/tree/master/CHANGELOG#changelogs
* Ref: https://kubernetes.io/docs/tasks/tools/
</details>
<hr />

## 2. Cluster Hardening - 15%
[🔼 Back to top](#table-of-contents)


### 2.1  Restrict access to Kubernetes API
[🔼 Back to top](#table-of-contents)

- secure access to the kube-apiserver

  1. **localhost**
      - port 8080
      - no TLS
      - default IP is localhost, change with `--insecure-bind-address`
  2. **secure port**
      - default is 6443, change with `--secure-port`
      - set TLS certificate with `--tls-cert-file`
      - set TLS certificate key with `--tls-private-key-file` flag

- Control anonymous requests to Kube-apiserver by using `--anonymous-auth=false`
  - example for adding anonymous access
    ```yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRole
    metadata:
      name: anonymous-review-access
    rules:
    - apiGroups:
      - authorization.k8s.io
      resources:
      - selfsubjectaccessreviews
      - selfsubjectrulesreviews
      verbs:
      - create
    ---
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
      name: anonymous-review-access
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: anonymous-review-access
    subjects:
    - kind: User
      name: system:anonymous
      namespace: default
    ```
    ```sh
    #check using anonymous
    kubectl auth can-i --list --as=system:anonymous -n default
    #check using yourown account
    kubectl auth can-i --list
    ```

* Ref: https://kubernetes.io/docs/concepts/security/controlling-access/#api-server-ports-and-ips
* Ref: https://kubernetes.io/docs/reference/access-authn-authz/authentication/#anonymous-requests
* Ref: https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/#without-kubectl-proxy
* Ref: https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet-authentication-authorization/#kubelet-authentication

### 2.2 Use Role-Based Access Controls to minimize exposure
[🔼 Back to top](#table-of-contents)

Roles live in namespace, RoleBinding specific to ns
ClusterRoles live across all namespace, ClusterRoleBidning
ServiceAccount should have only necessary RBAC permissions

**Solution**

- Create virtual users using ServiceAccount for specific namespace
- Create Role in specific namespace
  - has resources (ex: deployment)
  - has verbs (get, list, create, delete))
- Create RoleBinding in specific namespace & link Role & ServiceAccount
  - can be user, group or service account
- specify service account in deployment/pod level

```yaml
spec:
  serviceAccountName: deployment-viewer-sa
```

Ref: <https://kubernetes.io/docs/reference/access-authn-authz/rbac/>

### 2.3 Exercise caution in using service accounts e.g. disable defaults, minimize permissions on newly created ones
[🔼 Back to top](#table-of-contents)

- Create ServiceAccount to not to automount secret to any pod `automountServiceAccountToken: false`

  ```yaml
  apiVersion: v1
  kind: ServiceAccount
  metadata:
    name: test-svc
    namespace: dev
  automountServiceAccountToken: false
  ```
- Create Role for service account 
  `kubectl create role test-role -n dev --verb=get,list,watch,create --resource=pods --resource=pods,deployments,services,configmaps`
- Create RoleBinding for role & service account
  `kubectl create rolebinding cluster-test-binding -n dev --role=test-role --serviceaccount=dev:test-svc`
- Create Pod with serviceAccountName: test-svc. (note: pod level can be overridden to automountServiceAccountToken: true)

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: web
    namespace: dev
  spec:
    containers:
    - image: web
      name: web
    serviceAccountName: test-svc
    automountServiceAccountToken: false
  ```

  ```sh
  # hack
  curl -k https://kubernetes.default/api/v1/namespaces/kube-system/secrets 
    -H "Authorization: Bearer $(cat /run/secrets/kubernetes.io/serviceaccount/token)" 
  ```

Ref: <https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#use-the-default-service-account-to-access-the-api-server>

### 2.4 Update Kubernetes frequently
[🔼 Back to top](#table-of-contents)

- Minor versions(bug fixes) must be patched regularly

- Latest 3 Minor versions receive patch support
- Minor versions receive patches for ~1year

Ref: <https://v1-21.docs.kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/>

</details>
<hr />

## 3. System Hardening - 15%
[🔼 Back to top](#table-of-contents)


### 3.1 Minimize host OS footprint (reduce attack surface)
[🔼 Back to top](#table-of-contents)

- Containers will use host namespace(not k8s ns)

1. **Create Pod to use host namespace only if necessary.**

```yaml
spec:
  # container will use host IPC namespace (Default is false)
  hostIPC: true
  # containers will use host network namespace (Default is false)
  hostNetwork: true
  # containers will use host pid namespace (Default is false)
  hostPID: true
  containers:
  - name: nginx
    image: nginx:latest
```

2. **Don't run containers in privileged mode (privileged = false)**

```yaml
spec:
  securityContext:
    runAsUser: RunAsAny
    runAsGroup: RunAsAny
    fsGroup: RunAsAny
  # container will use host IPC namespace (Default is false)
  hostIPC: true
  # containers will use host network namespace (Default is false)
  hostNetwork: true
  # containers will use host pid namespace (Default is false)
  hostPID: true
  containers:
    - image: nginx:latsts
      name: web
      resources: {}
      securityContext:
        # container will run as root (Default is false)
        privileged: true

```

- **Limit Node Access**

```sh
# delete user
userdel user1
# delete group
groupdel group1
#suspend user
usermod -s /usr/sbin/nologin user2
#create user sam, home dir is /opt/sam, uid 2328 & login shell bash
useradd -d /opt/sam -s /bin/bash -G admin -u 2328 sam
```

- **Remove Obsolete/unnecessary Software**

```sh
# list all services
systemctl list-units --type service
# stop services
systemctl stop squid
# disable services
systemctl disable squid
# uninstall
apt remove squid
```

- **SSH hardening**

```sh
#generate keys
ssh-keygen –t rsa

#view auth key
cat /home/mark/.ssh/authorized_keys

# harden ssh config /etc/ssh/sshd_config
PermitRootLogin no
PasswordAuthentication no

systemctl restart sshd
```

- **Restrict Obsolete Kernel Modules**

```sh
# list all kernel modules
lsmod
# blocklist module sctp, dccp
vi /etc/modprobe.d/blacklist.conf
blacklist sctp
blacklist dccp
#reboot
shutdown –r now
```

- **UFW**

```
Install ufw   apt-get intall ufw
    systemctl enable ufw
    systemctl start ufw

check ufw firewall status  ufw status/ufw status numbered
    ufw default allow outgoing
    ufw default deny incoming

Allow specific (80)   ufw allow from 172.1.2.5 to any port 22 proto tcp
    ufw allow 1000:2000/tcp
    ufw allow from 172.1.3.0/25 to any port 80 proto tcp
    ufw allow 22
default deny 8080  ufw deny 80
activate ufw firewall  ufw enable
    ufw delete deny 80
    ufw delete 5
reset ufw   ufw reset
activate ufw firewall  ufw disable
```

- **Restrict allowed hostpaths with PodSecurityPolicy**
  - using PodSecurityPolicy can restrict AllowedHostPaths (used by hostPath volumes)
  - Ref: <https://kubernetes.io/docs/concepts/policy/pod-security-policy/#volumes-and-file-systems>

- **Identify and Fix Open Ports, Remove Packages**

```sh
#Identify Open Ports, Remove Packages
list all installed packages         apt list --installed
list active services                systemctl list-units --type service
list the kernel modules             lsmod
search for service                  systemctl list-units --all | grep -i nginx
stop remove nginx services          systemctl stop nginx
remove nginx service packages       rm /lib/systemd/system/nginx.service
remove packages from controlplane   apt remove nginx -y
check service listing on 9090       netstat -atnlp | grep -i 9090 | grep -w -i listen
check port to service mapping       cat /etc/services | grep -i ssh
check port listing on 22            netstat -an | grep 22  | grep -w -i listen
check lighthttpd service port       netstat -natulp | grep -i light
```

### 3.2 Minimize IAM roles
[🔼 Back to top](#table-of-contents)

- **Least Privilege:** make sure IAM roles of EC2 permissions are limited,
- **Block Access:** If not IAM; block CIDR, Firewall or NetPol. Example block EC2 169.254.169.254
- Use AWS Trusted Advisor/GCP Security Command Center/Adviser
Ref: <https://kubernetes.io/docs/reference/access-authn-authz/authentication/>

### 3.3. Minimize external access to the network
[🔼 Back to top](#table-of-contents)

- by default anyone has access cluster n/w can comminicate all pods and services
- by defualt limit access to cluster n/w from outside

- All pods can talk to all pods in all namespaces

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-external-egress
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
    to:
    - namespaceSelector: {}

```

### 3.4 Appropriately use kernel hardening tools such as AppArmor, seccomp
[🔼 Back to top](#table-of-contents)

#### 3.4.1 **SECCOMP PROFILES**
[🔼 Back to top](#table-of-contents)

- restricting the system calls it is able to make from userspace into the kernel
- SECCOMP can operate with 3 modes
  - Mode **0 - disabled**
  - Mode **1 - strict mode**
  - Mode **2 - filtered mode**
- SECCOMP profile default directory `/var/lib/kubelet/seccomp/profiles`
- SECCOMP Profiles are 3 types
  - **Default Profile**
  - **Audit** - audit.json
  - **Violation**- violation.json
  - **Custom** - fine-grained.json
- SECCOMP profile action can be
  - `"action": "SCMP_ACT_ALLOW"`
  - `"action": "SCMP_ACT_ERRNO"`
  - `"defaultAction": "SCMP_ACT_LOG"`
  - Create POD with Default Profile

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: audit-pod
    labels:
      app: audit-pod
  spec:
    securityContext:
      seccompProfile:
        # default seccomp
        type: RuntimeDefault
    containers:
    - name: test-container
      image: hashicorp/http-echo:0.2.3
      args:
      - "-text=just made some syscalls!"
      securityContext:
        allowPrivilegeEscalation: false
  ```

- Create POD with Specific Profile

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: audit-pod
    labels:
      app: audit-pod
  spec:
    securityContext:
      seccompProfile:
        type: Localhost
        # specfy violation.json or fine-grained.json
        localhostProfile: profiles/audit.json
    containers:
    - name: test-container
      image: hashicorp/http-echo:0.2.3
      args:
      - "-text=just made some syscalls!"
      securityContext:
        allowPrivilegeEscalation: false
    ```

  ```
  #trace system calls
  strace -c touch /tmp/test.log

  # check SECCOMP is supported by kernel
  grep -i seccomp /boot/config-$(uname -r)

  #Run Kubbernetes POD (amicontained - open source inspection tool)
  kubectl run amicontained --image r.j3ss.co/amicontained amicontained -- amicontained
  kubectl logs amicontained

  #create dir in default location /var/lib/kubelet/seccomp
  mkdir -p /var/lib/kubelet/seccomp/profiles

  #use in pod
  localhostProfile: profiles/audit.json
  ```

  - Ref: https://kubernetes.io/docs/tutorials/clusters/seccomp
  - Ref: https://man7.org/linux/man-pages/man2/syscalls.2.html

#### 3.4.2 **APPARMOR**
[🔼 Back to top](#table-of-contents)

- Kernel Security Module to granular access control for programs on Host OS
- **AppArmor Profile** - Set of Rules, to be enabled in nodes
- AppArmor Profile loaded in 2 modes
  - **Complain Mode** - Discover the program
  - **Enforce Mode** - prevent the program
- **create AppArmor Profile**

  ```sh
  sudo vi /etc/apparmor.d/deny-write

  #include <tunables/global>
  profile k8s-apparmor-example-deny-write flags=(attach_disconnected) {
    #include <abstractions/base>
    file,
    # Deny all file writes.
    deny /** w,
  }
  ```
- copy deny-write profile to worker node01
  `scp deny-write node01:/tmp`

- load the profile on all our nodes default directory /etc/apparmor.d
  `sudo apparmor_parser /etc/apparmor.d/deny-write`
- apply to pod

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: hello-apparmor
    annotations:
      container.apparmor.security.beta.kubernetes.io/hello: localhost/k8s-apparmor-example-deny-write
  spec:
    containers:
    - name: hello
      image: busybox
      command: [ "sh", "-c", "echo 'Hello AppArmor!' && sleep 1h" ]
  ```

- useful commands

  ```
  #check status            
  systemctl status apparmor

  #check enabled in nodes  
  cat /sys/module/apparmor/parameters/enabled
  
  #check profiles          
  cat /sys/kernel/security/apparmor/profiles

  #install                 
  apt-get install apparmor-utils
  
  #default Profile file directory  is /etc/apparmor.d/

  #create apparmor profile   
  aa-genprof /root/add_data.sh
  
  #apparmor module status    
  aa-status
  
  #load profile file           
  apparmor_parser -q /etc/apparmor.d/usr.sbin.nginx

  #load profile                
  apparmor_parser /etc/apparmor.d/root.add_data.sh
  aa-complain /etc/apparmor.d/profile1
  aa-enforce  /etc/apparmor.d/profile2

  #disable profile             
  apparmor_parser -R /etc/apparmor.d/root.add_data.sh
  ```

- Ref: https://kubernetes.io/docs/tutorials/clusters/apparmor
- Ref: https://gitlab.com/apparmor/apparmor/-/wikis/Documentation
- Ref: https://man7.org/linux/man-pages/man2/syscalls.2.html
- 
</details>
<hr />

## 4. Minimize Microservice Vulnerabilities - 20%
[🔼 Back to top](#table-of-contents)


### 4.1 Setup appropriate OS level security domains e.g. using PSP, OPA, security contexts
[🔼 Back to top](#table-of-contents)

#### **Admission Controller**

- Implement security measures to enforce. Triggers before creating a pod
- Enable a Controller in kubeadm cluster `/etc/kubernetes/manifests/kube-apiserver.yaml`

  ```yaml
  spec:
  containers:
  - command:
  - kube-apiserver
  ...
  - --enable-admission-plugins=NameSpaceAutoProvision,PodSecurityPolicy
  image: k8s.gcr.io/kube-apiserver-amd64:v1.11.3
  name: kube-apiserver
  ```

- Ref: <https://kubernetes.io/blog/2019/03/21/a-guide-to-kubernetes-admission-controllers/>

#### 4.1.1 **Pod Security Policies (PSP)**
[🔼 Back to top](#table-of-contents)

- Defines policies to controls security sensitive aspects of the pod specification
- PodSecurityPolicy is one of the admission controller
- enable at api-server using  `--enable-admission-plugins=NameSpaceAutoProvision,PodSecurityPolicy`
- Create Pod using PSP
  ```yaml
  apiVersion: policy/v1beta1
  kind: PodSecurityPolicy
  metadata:
    name: psp-test
    annotations:
      seccomp.security.alpha.kubernetes.io/allowedProfileNames: '*'
  spec:
    privileged: true
    allowPrivilegeEscalation: true
    allowedCapabilities:
    - '*'
    volumes:
    - '*'
    hostNetwork: true
    hostPorts:
    - min: 0
      max: 65535
    hostIPC: true
    hostPID: true
    runAsUser:
      rule: 'RunAsAny'
    seLinux:
      rule: 'RunAsAny'
    supplementalGroups:
      rule: 'RunAsAny'
    fsGroup:
      rule: 'RunAsAny'
  ```
  - Create Service Account
    `kubectl create sa psp-test-sa -n dev`
  - Create ClusterRole
    `kubectl -n psp-test create clusterrole psp-pod --verb=use --resource=podsecuritypolicies --resource-name=psp-test`
    or
    ```yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRole
    metadata:
      name: psp-pod
    rules:
    - apiGroups: ['policy']
      resources: ['podsecuritypolicies']
      verbs:     ['use']
      resourceNames:
      - psp-test
    ```
  - Create ClusterRoleBinding
    `kubectl -n psp-test create rolebinding -n dev psp-mount --clusterrole=psp-pod --serviceaccount=dev:psp-test-sa`
    or
    ```yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
      name: psp-mount
    roleRef:
      kind: ClusterRole
      name: psp-pod
      apiGroup: rbac.authorization.k8s.io
    subjects:
    - kind: ServiceAccount
      name: psp-test-sa
      namespace: dev
    ```
  - POD Access to PSP for authorization
    - Create Service Account or use Default Service account
    - Create Role with podsecuritypolicies, verbs as use
    - Create RoleBinding to Service Account and Role
- Ref: <https://kubernetes.io/docs/concepts/policy/pod-security-policy/>

#### 4.1.2 **Open Policy Agent (OPA)**
[🔼 Back to top](#table-of-contents)

- OPA for enforcing authorization policies for kubernetes
  - All images must be from approved repositories
  - All ingress hostnames must be globally unique
  - All pods must have resource limits
  - All namespaces must have a label that lists a point-of-contact
- Deploy OPA Gatekeeper in cluster

  ```sh
  kubectl apply -f https://raw.githubusercontent.com/open-policy-agent/gatekeeper/release-3.7/deploy/gatekeeper.yaml

  helm repo add gatekeeper https://open-policy-agent.github.io/gatekeeper/charts --force-update
  helm install gatekeeper/gatekeeper --name-template=gatekeeper --namespace gatekeeper-system --create-namespace
  ```

- Example constraint template to enforce to require all lables

  ```sh
  #create template
  kubectl apply -f https://raw.githubusercontent.com/open-policy-agent/gatekeeper/master/demo/basic/templates/k8srequiredlabels_template.yaml
  ```

  ```yaml
  apiVersion: constraints.gatekeeper.sh/v1beta1
  kind: K8sRequiredLabels
  metadata:
    name: ns-must-have-hr
  spec:
    match:
      kinds:
        - apiGroups: [""]
          kinds: ["Namespace"]
    parameters:
      labels: ["hr"]
  ```

  `kubectl get constraints`

- Ref: <https://kubernetes.io/blog/2019/08/06/opa-gatekeeper-policy-and-governance-for-kubernetes/>
- Ref: <https://open-policy-agent.github.io/gatekeeper/website/docs/install/>

#### 4.1.3 **Security Contexts**
[🔼 Back to top](#table-of-contents)

- Defines privilege, access control, Linux capabilities settings for a Pod or Container
- Set the security context for a Pod (applies to all containers)

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: security-context-demo
  spec:
    securityContext:
      runAsUser: 1000
      runAsGroup: 3000
      fsGroup: 2000
    volumes:
    - name: sec-ctx-vol
      emptyDir: {}
    containers:
    - name: sec-ctx-demo
      image: busybox
      command: [ "sh", "-c", "sleep 1h" ]
      volumeMounts:
      - name: sec-ctx-vol
        mountPath: /data/demo
      securityContext:
        allowPrivilegeEscalation: false
  ```

  ```sh
  kubectl exec -it security-context-demo -- sh
  ps
  cd demo
  echo hello > testfile
  ls -l
  id
  ```

- Set the security context for a Container (container level user is different)

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: security-context-demo-2
  spec:
    securityContext:
      runAsUser: 1000
    containers:
    - name: sec-ctx-demo-2
      image: gcr.io/google-samples/node-hello:1.0
      securityContext:
        runAsUser: 2000
        allowPrivilegeEscalation: false
  ```

  ```sh
  kubectl exec -it security-context-demo-2 -- sh
  ps aux
  id
  ```

- Set additional(CAP_NET_ADMIN & CAP_SYS_TIME) capabilities for a Container

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: security-context-demo-4
  spec:
    containers:
    - name: sec-ctx-4
      image: gcr.io/google-samples/node-hello:1.0
      securityContext:
        capabilities:
          add: ["NET_ADMIN", "SYS_TIME"]
  ```

- Assign SELinux labels to a Container

  ```yaml
  ...
  securityContext:
    seLinuxOptions:
      level: "s0:c123,c456"
  ```

- Set Seccomp profile to Container [Refer 3.4.1](#341-seccomp-profiles)
- Ref: <https://kubernetes.io/docs/tasks/configure-pod-container/security-context/>

### 4.2 Manage Kubernetes secrets
[🔼 Back to top](#table-of-contents)

- Types of Secrets
  - Opaque(Generic) secrets -
  - Service account token Secrets
  - Docker config Secrets
    - `kubectl create secret docker-registry my-secret --docker-server=DOCKER_REGISTRY_SERVER --docker-username=DOCKER_USER --docker-password=DOCKER_PASSWORD --docker-email=DOCKER_EMAIL`
  - Basic authentication Secret -
  - SSH authentication secrets -
  - TLS secrets -
    - `kubectl create secret tls tls-secret --cert=path/to/tls.cert --key=path/to/tls.key`
  - Bootstrap token Secrets -

- Secret as Data to a Container Using a Volume

  ```
  kubectl create secret generic mysecret --from-literal=username=devuser --from-literal=password='S!B\*d$zDsb='
  ```

  Decode Secret

  ```
  kubectl get secrets/mysecret --template={{.data.password}} | base64 -d
  ```

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: mypod
  spec:
    containers:
    - name: mypod
      image: redis
      volumeMounts:
      - name: secret-volume
        mountPath: "/etc/secret-volume"
        readOnly: true
    volumes:
    - name: secret-volume
      secret:
        secretName: mysecret
  ```

- Secret as Data to a Container Using Environment Variables

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: secret-env-pod
  spec:
    containers:
    - name: mycontainer
      image: redis
      # refer all secret data
      envFrom:
        - secretRef:
          name: mysecret-2
      # refer specific variable
      env:
        - name: SECRET_USERNAME
          valueFrom:
            secretKeyRef:
              name: mysecret
              key: username
        - name: SECRET_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysecret
              key: password
    restartPolicy: Never
    ```

- Ref: https://kubernetes.io/docs/concepts/configuration/secret
- Ref: https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data
- Ref: https://kubernetes.io/docs/tasks/configmap-secret/managing-secret-using-kubectl/

### 4.3 Use container runtime sandboxes in multi-tenant environments (e.g. gvisor, kata containers)
[🔼 Back to top](#table-of-contents)

- Sandboxing is concept of isolation of containers from Host
- docker uses default SecComp profiles restrict previleges. Whitlist or Blacklist
- AppArmor finegrain control of that container can access. Whitlist or Blacklist
- If large number of apps in containers, then SecComp/AppArmor is not the case

#### 4.3.1 gvisor

- gVisor sits between container and Linux Kernel. Every container has their own gVisior
- gVisor has 2 different components
  - **Sentry**: works as kernel for containers
  - **Gofer**: is file proxy to access system files. Middle man betwen container and OS
- gVisor uses runsc to runtime sandbox with in hostOS (OCI comapliant)

  ```yaml
  apiVersion: node.k8s.io/v1  # RuntimeClass is defined in the node.k8s.io API group
  kind: RuntimeClass
  metadata:
    name: myclass  # The name the RuntimeClass will be referenced by
  handler: runsc  # non-namespaced, The name of the corresponding CRI configuration
  ```

#### 4.3.4 kata containers

- Kata Install lightweight containers in VM, provisions own kernal for containers(like VM)
- Kata containers provide hardware virtualization support (cannot run in Cloud, GCP supports)
- Kata containers use kata runtime

  ```yaml
  apiVersion: node.k8s.io/v1  # RuntimeClass is defined in the node.k8s.io API group
  kind: RuntimeClass
  metadata:
    name: myclass  # The name the RuntimeClass will be referenced by
  handler: kata  # non-namespaced, The name of the corresponding CRI configuration
  ```

#### Install gVisor
- ```sh
  curl -fsSL https://gvisor.dev/archive.key | sudo gpg --dearmor -o /usr/share/keyrings/gvisor-archive-keyring.gpg
  echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/gvisor-archive-keyring.gpg] https://storage.googleapis.com/gvisor/releases release main" | sudo tee /etc/apt/sources.list.d/gvisor.list > /dev/null

  sudo add-apt-repository "deb [arch=amd64,arm64] https://storage.googleapis.com/gvisor/releases release main"

  sudo apt-get update && sudo apt-get install -y runsc

  cat <<EOF | sudo tee /etc/containerd/config.toml
  version = 2
  [plugins."io.containerd.runtime.v1.linux"]
    shim_debug = true
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
    runtime_type = "io.containerd.runc.v2"
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runsc]
    runtime_type = "io.containerd.runsc.v1"
  EOF

  sudo systemctl restart containerd

  echo 0 > /proc/sys/kernel/dmesg_restrict
  echo "kernel.dmesg_restrict=0" >> /etc/sysctl.conf

  {
  wget https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.13.0/crictl-v1.13.0-linux-amd64.tar.gz
  tar xf crictl-v1.13.0-linux-amd64.tar.gz
  sudo mv crictl /usr/local/bin
  }

  cat <<EOF | sudo tee /etc/crictl.yaml
  runtime-endpoint: unix:///run/containerd/containerd.sock
  EOF
  ```

#### Container Runtime

- docker run => docker CLI => REST API => Docker Daemon => check locally => Registery => call containerd  => convert image to OCI container => containerd-shim => runC will start container
  - we can override runtime when running container

  ```
  docker run --runtime kata -d nginx
  docker run --runtime runsc -d nginx
  ```

- Creating a Container Runtime for additional layers of isolation
- Create RuntimeClass to define specilized container runtime config
- Create Pod with specific runtime class

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: mypod
  spec:
    runtimeClassName: myclass
    containers:
    - image: busybox
      name: busybox
      command: ['sh', '-c', 'while true; do echo "Running..."; sleep 1h; done']
  ```
  `kubectl exec mypod -- dmesg`

- Ref: https://gvisor.dev/docs/user_guide/install/
- https://sbulav.github.io/certifications/cks-gvisor/
- Ref: https://kubernetes.io/docs/concepts/containers/runtime-class
- Ref: <https://github.com/kubernetes/enhancements/blob/5dcf841b85f49aa8290529f1957ab8bc33f8b855/keps/sig-node/585-runtime-class/README.md#examples>

### 4.4 Implement pod to pod encryption by use of mTLS
[🔼 Back to top](#table-of-contents)

- mTLS: Is secure communication between pods
- With service mesh Istio & Linkerd mTLS is easier, managable
  - mTLS can be Enforced or Strict
- Istio: https://istio.io/latest/docs/tasks/security/authentication/mtls-migration/
- Linkerd: https://linkerd.io/2.11/features/automatic-mtls/
- Article: https://itnext.io/how-to-secure-kubernetes-in-cluster-communication-5a9927be415b

**Steps for TLS certificate for a Kubernetes service accessed through DNS**

- Download and install CFSSL
- Generete private key using `cfssl genkey`
- Create CertificateSigningRequest
  ```
  cat <<EOF | kubectl apply -f -
  apiVersion: certificates.k8s.io/v1
  kind: CertificateSigningRequest
  metadata:
    name: my-svc.my-namespace
  spec:
    request: $(cat server.csr | base64 | tr -d '\n')
    signerName: kubernetes.io/kubelet-serving
    usages:
    - digital signature
    - key encipherment
    - server auth
  EOF
  ```
- Get CSR approved (by k8s Admin)
  - `kubectl certificate approve my-svc.my-namespace`
- Once approved then retrive from status.certificate
  - `kubectl get csr my-svc.my-namespace -o jsonpath='{.status.certificate}' | base64 --decode > server.crt`
- Download and use it
  ```
  kubectl get csr
  ```
- Ref: <https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/>

</details>
<hr />

## 5. Supply Chain Security - 20%
[🔼 Back to top](#table-of-contents)


### 5.1 Minimize base image footprint
[🔼 Back to top](#table-of-contents)

- Use **Slim/Minimal** Images than base images
- Use Docker **multi stage builds** for lean
- **Use Distroless:**
  - Distroless Images will have only your app & runtime dependencies
    - No package managers, shell, n/w tools, text editors etc
  - Distroless images are very small
- use `trivy` image scanner for container vulnerabilities, filesys, git etc
- Ref: <https://github.com/GoogleContainerTools/distroless>
- Ref: <https://github.com/aquasecurity/trivy>

### 5.2 Secure your supply chain: whitelist allowed registries, sign and validate images
[🔼 Back to top](#table-of-contents)

- **Approach 1 - using ImagePolicyWebhook Admission Controller**
  - Prerequisites - ImagePolicyWebhook Admission webhook server (deployment & service)
    ```sh
    #cfssl
    sudo apt update
    sudo apt install golang-cfssl
    ```
    ```sh
    #create CSR to send to KubeAPI
    cat <<EOF | cfssl genkey - | cfssljson -bare server
    {
      "hosts": [
        "image-bouncer-webhook",
        "image-bouncer-webhook.default.svc",
        "image-bouncer-webhook.default.svc.cluster.local",
        "192.168.56.10",
        "10.96.0.0"
      ],
      "CN": "system:node:image-bouncer-webhook.default.pod.cluster.local",
      "key": {
        "algo": "ecdsa",
        "size": 256
      },
      "names": [
        {
          "O": "system:nodes"
        }
      ]
    }
    EOF

    #create csr request
    cat <<EOF | kubectl apply -f -
    apiVersion: certificates.k8s.io/v1
    kind: CertificateSigningRequest
    metadata:
      name: image-bouncer-webhook.default
    spec:
      request: $(cat server.csr | base64 | tr -d '\n')
      signerName: kubernetes.io/kubelet-serving
      usages:
      - digital signature
      - key encipherment
      - server auth
    EOF

    #approver cert
    kubectl certificate approve image-bouncer-webhook.default

    # download signed server.crt
    kubectl get csr image-bouncer-webhook.default -o jsonpath='{.status.certificate}' | base64 --decode > server.crt

    mkdir -p /etc/kubernetes/pki/webhook/
    
    #copy to /etc/kubernetes/pki/webhook
    cp server.crt /etc/kubernetes/pki/webhook/server.crt

    # create secret with signed server.crt
    kubectl create secret tls tls-image-bouncer-webhook --key server-key.pem --cert server.crt
    ```
    - Using ImagePolicyWebhook Admission webhook server (deployment & service)
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: image-bouncer-webhook
    spec:
      selector:
        matchLabels:
          app: image-bouncer-webhook
      template:
        metadata:
          labels:
            app: image-bouncer-webhook
        spec:
          containers:
            - name: image-bouncer-webhook
              imagePullPolicy: Always
              image: "kainlite/kube-image-bouncer:latest"
              args:
                - "--cert=/etc/admission-controller/tls/tls.crt"
                - "--key=/etc/admission-controller/tls/tls.key"
                - "--debug"
                - "--registry-whitelist=docker.io,gcr.io"
              volumeMounts:
                - name: tls
                  mountPath: /etc/admission-controller/tls
          volumes:
            - name: tls
              secret:
                secretName: tls-image-bouncer-webhook
    ---
    apiVersion: v1
    kind: Service
    metadata:
      labels:
        app: image-bouncer-webhook
      name: image-bouncer-webhook
    spec:
      type: NodePort
      ports:
        - name: https
          port: 443
          targetPort: 1323
          protocol: "TCP"
          nodePort: 30020
      selector:
        app: image-bouncer-webhook
    ```
  - Add this to resolve service `echo "127.0.0.1 image-bouncer-webhook" >> /etc/hosts`
  - check service using`telnet image-bouncer-webhook 30020` or `netstat -na | grep 30020`
  - Create custom kubeconfig with above service, its client certificate
    - `/etc/kubernetes/pki/webhook/admission_kube_config.yaml`

    - ```yaml
      apiVersion: v1
      kind: Config
      clusters:
      - cluster:
          certificate-authority: /etc/kubernetes/pki/webhook/server.crt
          server: https://image-bouncer-webhook:30020/image_policy
        name: bouncer_webhook
      contexts:
      - context:
          cluster: bouncer_webhook
          user: api-server
        name: bouncer_validator
      current-context: bouncer_validator
      preferences: {}
      users:
      - name: api-server
        user:
          client-certificate: /etc/kubernetes/pki/apiserver.crt
          client-key:  /etc/kubernetes/pki/apiserver.key
      ```

  - Create ImagePolicyWebhook AdmissionConfiguration file(json/yaml), update custom kubeconfig file at
    - `/etc/kubernetes/pki/admission_config.json`
    - ```yaml
      apiVersion: apiserver.config.k8s.io/v1
      kind: AdmissionConfiguration
      plugins:
      - name: ImagePolicyWebhook
        configuration:
          imagePolicy:
            kubeConfigFile: /etc/kubernetes/pki/webhook/admission_kube_config.yaml
            allowTTL: 50
            denyTTL: 50
            retryBackoff: 500
            defaultAllow: false
      ```
  - Enable ImagePolicyWebhook in enable-admission-plugins in kubeapi server config at
  - Update admin-config file in kube api server admission-control-config-file
    - `/etc/kubernetes/manifests/kube-apiserver.yaml`

    ```yaml
    - --enable-admission-plugins=NodeRestriction,ImagePolicyWebhook
    - --admission-control-config-file=/etc/kubernetes/pki/admission_config.json
    ```
  - Test
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: dh-busybox
  spec:
    restartPolicy: Never
    containers:
    - name: busybox
      image: docker.io/library/busybox
      #image: gcr.io/google-containers/busybox:1.27
      command: ['sh', '-c', 'sleep 3600']
    ```
  - Ref: https://stackoverflow.com/questions/54463125/how-to-reject-docker-registries-in-kubernetes
  - Ref: https://github.com/containerd/cri/blob/master/docs/registry.md
  - Ref: https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#imagepolicywebhook
  - Ref: https://kubernetes.io/blog/2019/03/21/a-guide-to-kubernetes-admission-controllers/
  - Ref: https://computingforgeeks.com/how-to-install-cloudflare-cfssl-on-linux-macos/
  
- **Approach 2 - ConstraintTemplate**
  - Create ConstraintTemplate CRD to whitelist docker registries
    ```yaml
    apiVersion: templates.gatekeeper.sh/v1beta1
    kind: ConstraintTemplate
    metadata:
      name: k8sallowedrepos
    spec:
      crd:
        spec:
          names:
            kind: K8sAllowedRepos
          validation:
            # Schema for the `parameters` field
            openAPIV3Schema:
              properties:
                repos:
                  type: array
                  items:
                    type: string
    targets:
      - target: admission.k8s.gatekeeper.sh
        rego: |
          package k8sallowedrepos
          violation[{"msg": msg}] {
            container := input.review.object.spec.containers[_]
            satisfied := [good | repo = input.parameters.repos[_] ; good = startswith(container.image,repo)]
            not any(satisfied)
            msg := sprintf("container <%v> has an invalid image repo <%v>, allowed repos are %v",[container.name, container.image, input.parameters.repos])
          }
          violation[{"msg": msg}] {
            container := input.review.object.spec.initContainers[_]
            satisfied := [good | repo = input.parameters.repos[_] ; good = startswith(container.image,repo)]
            not any(satisfied)
            msg := sprintf("container <%v> has an invalid image repo <%v>, allowed repos are %v",
            [container.name, container.image, input.parameters.repos])
          }
    ```
  - Create a Resource Constraint with allowed docker registries
    ```yaml
    apiVersion: constraints.gatekeeper.sh/v1beta1
    kind: K8sAllowedRepos
    metadata:
      name: whitelist-dockerhub
    spec:
      match:
      kinds:
      - apiGroups: [""]
        kinds: ["Pod"]
    parameters:
      repos:
      - "docker.io"
    ```
  - Create a pod with valid registry and test
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: dh-busybox
    spec:
      restartPolicy: Never
      containers:
      - name: busybox
        image: docker.io/library/busybox
        command: ['sh', '-c', 'sleep 3600']
    ```

### 5.3 Use static analysis of user workloads (e.g.Kubernetes resources, Docker files)
[🔼 Back to top](#table-of-contents)

- **Tips for Static Analysis**
  - Avoid Host namespaces such as `hostNetwork: true, hostIPC: true, hostPID: true`
  - Avoid running pod as `securityContext.privileged: true`
  - Avoid running pod as root or 0 in `securityContext.runAsUser`
    - use nobody or some user xxxx
-   Do not use `:latest` tag instead use specific version
- **Static YAML analysis using poular tool Kubese**c
  - Kubesec scans and gives the risk score ``
  - Run Kubesec locally using ``

    ```sh
    # run scan using kubesec
    kubesec scan pod.yaml
    # run kubesec locally on 8080 port
    kubesec http 8080 &
    #kubesec API invoke and scan
    curl -sSX POST --data-binary @”pod.yaml" https://v2.kubesec.io/scan
    ```

  - Ref: <https://kubernetes.io/blog/2018/07/18/11-ways-not-to-get-hacked/#7-statically-analyse-yaml>
  - Ref: <https://kubesec.io/>

### 5.4 Scan images for known vulnerabilities
[🔼 Back to top](#table-of-contents)

- Vulnerability scanning tools are Trivy(*) & Anchore
- 1. Install Trivy (by Aquasec) on your control plane node
  - Scan an image using trivy
    - `trivy image busybox:1.33.1`
  - It results Low, Medium, High, Critical & VulnarabilityID
    - `trivy image-- severity CRITICAL,HIGH busybox:1.33.1`
    - `kubectl get pods -A -o jsonpath="{..image}" | tr -s '[[:space:]]' '\n' | sort -u`
- 2. Anchore CLI
  - Anchore CLI in mac using Docker Compose command
  ```sh
  curl -O https://engine.anchore.io/docs/quickstart/docker-compose.yaml
  docker-compose up -d
  docker-compose ps
  docker-compose exec api anchore-cli system status
  docker-compose exec api anchore-cli system feeds list
  docker-compose exec api anchore-cli system wait 
  docker-compose exec api anchore-cli image add nginx:1.19.0
  docker-compose exec api anchore-cli image wait nginx:1.19.0
  docker-compose exec api anchore-cli image content nginx:1.19.0 os
  docker-compose exec api anchore-cli image vuln  nginx:1.19.0 all
  docker-compose exec api anchore-cli evaluate check nginx:1.19.0
  ```
  - Anchore CLI in mac using Docker Compose with Anchore CLI commadn
  ```sh
  curl https://engine.anchore.io/docs/quickstart/docker-compose.yaml | docker-compose -p anchore -f - up -d
  docker run --rm -e ANCHORE_CLI_URL=http://anchore_engine-api_1:8228/v1/ --network anchore_default -it anchore/engine-cli
  anchore-cli --version
  anchore-cli image add nginx:1.19.0 && anchore-cli image wait nginx:1.19.0
  anchore-cli image vuln nginx:1.19.0 all
  anchore-cli image evaluate check nginx:1.19.0 all
  ```
  - Clair
  ```sh
  docker run -p 5432:5432 -d --name db arminc/clair-db:latest
  docker run -p 6060:6060 --link db:postgres -d --name clair arminc/clair-local-scan:latest
  clair-scanner -w example-alpine.yaml --ip YOUR_LOCAL_IP alpine:3.5

  ```
- Ref: https://kubernetes.io/blog/2018/07/18/11-ways-not-to-get-hacked/#10-scan-images-and-run-ids
- Ref: https://github.com/aquasecurity/trivy 
- Ref: https://github.com/anchore/anchore-cli#command-line-examples
- Ref: https://github.com/linuxacademy/content-cks-trivy-k8s-webhook
- Ref: https://project-serum.github.io/anchor/getting-started/installation.html#install-yarn
- Ref: https://docs.anchore.com/3.0/docs/installation/
- https://githubhelp.com/anchore/anchore-engine
</details>
   <hr />

## 6. Monitoring, Logging and Runtime Security - 20%
[🔼 Back to top](#table-of-contents)


### 6.1 Perform behavioral analytics of syscall process and file activities at the host and container level to detect malicious activities
[🔼 Back to top](#table-of-contents)

- Falco can detect and alerts on any behavior that involves making Linux system calls
- **Falco** operates at the user space and kernel space, major components...
  - Driver
  - Policy Engine
  - Libraries
  - Falco Rules
- Falco install 
- Helm Install Falco as DaemonSet

  ```sh
  helm repo add falcosecurity https://falcosecurity.github.io/charts
  helm repo update

  helm upgrade -i falco falcosecurity/falco \
    --set falcosidekick.enabled=true \
    --set falcosidekick.webui.enabled=true \
    --set auditLog.enabled=true \
    --set falco.jsonOutput=true \
    --set falco.fileOutput.enabled=true \
    --set falcosidekick.config.slack.webhookurl="<<web-hook-url>>" 
  ```

- **Falco Rules** Filters for engine events, in YAML format Example Rule

  ```yaml
  - rule: Unauthorized process
    desc: There is a running process not described in the base template
    condition: spawned_process and container and k8s.ns.name=ping and not proc.name in (apache2, sh, ping)
    output: Unauthorized process (%proc.cmdline) running in (%container.id)
    priority: ERROR
    tags: [process]
  ```

- Check the Logs
  - `kubectl logs --selector app=falco | grep Error`
  - 

- **Falco Configuration** for Falco daemon, YAML file and it has key: value or list
  - config file located at `/etc/falco/falco.yaml`
- Ref: https://falco.org/docs/getting-started
- Ref: https://github.com/falcosecurity/charts/tree/master/falco
- Ref: https://falco.org/blog/detect-cve-2020-8557

### 6.2 Detect threats within physical infrastructure, apps, networks, data, users and workloads
[🔼 Back to top](#table-of-contents)

### 6.3 Detect all phases of attack regardless where it occurs and how it spreads
[🔼 Back to top](#table-of-contents)

- Kubernetes attack matrix (9 Tactics, 40 techniques)
  -  Initial access
  -  Execution
  -  Persistence
  -  Privilege escalation
  -  Defense evasion
  -  Credential access
  -  Discovery
  -  Lateral movement
  -  Impact
- Use MITRE & ATT&CK framework of tacticques and techniques
- **Best Practices protection: apply native Kubernetes controls**
  1. Configure & Monitor RBAC - Least privilege, avoid overlaping role
  2. Configure Network Policies - Use default-deny-all & explicitly allow
  3. Harden pod configurations - security context for pod/container, use pod serviceaccount
  4. Detect Inscure Pods
- Ref: https://www.cncf.io/online-programs/mitigating-kubernetes-attacks/
- Ref: https://www.microsoft.com/security/blog/2020/04/02/attack-matrix-kubernetes/
- Ref: https://sysdig.com/blog/mitre-attck-framework-for-container-runtime-security-with-sysdig-falco/

### 6.4 Perform deep analytical investigation and identification of bad actors within environment
[🔼 Back to top](#table-of-contents)

- Monitor using sysdig k8s cluster, ns, svc, rc and labels
- sysdig capture system calls and other OS events
- Exploring a Kubernetes Cluster with csysdig
  `csysdig -k http://127.0.0.1:8080`
- Monitoring/Visualize Kubernetes with Sysdig Cloud
- Ref: https://kubernetes.io/blog/2015/11/monitoring-kubernetes-with-sysdig/
- Ref: https://docs.sysdig.com/

### 6.5 Ensure immutability of containers at runtime
[🔼 Back to top](#table-of-contents)

- immutable = cannot modify original state
- POD level using securityContext; key logic `readOnlyRootFilesystem = true,  privileged=false`

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: security-context-demo
  spec:
      volumes:
      - name: sec-ctx-vol
        emptyDir: {}
      containers:
      - name: sec-ctx-demo
        image: busybox
        command: [ "sh", "-c", "echo hello > /data/demo/hello.txt; sleep 1h" ]
        securityContext:
          privileged: true
          runAsUser: 0
          allowPrivilegeEscalation: true
          readOnlyRootFilesystem: false
        volumeMounts:
        - name: sec-ctx-vol
          mountPath: /data/demo
          
  ```

- Enforce using PSP(Pod Security Policies) - key logic `readOnlyRootFilesystem = true,  privileged=false; runAsUser=NonRoot`

  ```yaml
  apiVersion: policy/v1beta1
  kind: PodSecurityPolicy
  metadata:
    name: example
  spec:
    privileged: false
    readOnlyRootFilesystem: true
    runAsUser:
      rule: RunAsNonRoot
    seLinux:
      rule: RunAsAny
    supplementalGroups:
      rule: RunAsAny
    runAsUser:
      rule: RunAsNonRoot
    fsGroup:
      rule: RunAsAny
  ```

- Ref: <https://kubernetes.io/blog/2018/03/principles-of-container-app-design/>

### 6.6 Use Audit Logs to monitor access
[🔼 Back to top](#table-of-contents)

- The cluster audits the activities
  - generated by users,
  - by applications that use the Kubernetes API
  - control plane aswell
- When API Server performs action, it creates stages
  - **RequestReceived** - The stage for events generated as soon as the audit handler receives the request
  - **ResponseStarted** - response headers are sent & response body is sent. Only for long running (Ex: watch)
  - **ResponseComplete** - response body has been completed
  - **Panic** - Events generated when a panic occurred
- Create Policy Object configuration for Audit
  ```yaml
  apiVersion: audit.k8s.io/v1 # This is required.
  kind: Policy
  # Don't generate audit events for all requests in RequestReceived stage.
  omitStages:
    - "RequestReceived"
  rules:
    - namespace: ["prod-namespace"]
      verb: ["delete"]
      resources:
      - groups: " "
        resources: ["pods"]
        resourceNames: ["webapp-pod"]
      #None/Metadata/Request/RequestResponse
      level: RequestResponse
  ```
- Enable AuditLog in KubeAPI server level
  ```yaml
  - --audit-log-path=/var/log/k8-audit.log
  - --audit-policy-file=/etc/kubernetes/audit-policy.yaml
  - --audit-log-maxage=10
  - --audit-log-maxbackup=5
  - --audit-log-maxsize=100
  ```
- Ref: https://kubernetes.io/docs/tasks/debug-application-cluster/audit/
- Ref: https://cloud.google.com/kubernetes-engine/docs/concepts/audit-policy
- Ref: https://falco.org/docs/event-sources/kubernetes-audit/
- Ref: https://arabitnetwork.com/2021/03/13/k8s-enabling-auditing-logs-step-by-step/

### debugging api-server/kubelet failures

```sh
/var/log/pods
/var/log/containers
docker ps + docker logs
/var/log/syslog or journalctl -u kubelet
```
</details>
 <hr />

## IMP Notes Tips for CKSExam
[🔼 Back to top](#table-of-contents)

### **CKS Exam Catergories**
  1. **API Server Dependent**
     - CIS Benchmarking
     - PodSecurityPolicy
     - ImagePolicyWebhook/AdmissionController
     - AuditPolicy
     - Hardening access to Kubernetes API
     - Falco/SysDig
     - Tracing Container syscall
  2. **General**
     - NetworkPolicy
     - RBAC
     - Secret
     - ServiceAccount
     - AppArmor
     - SecComp
     - RuntimeClass
     - SecurityContext & Immutable Pods
     - Static Analysis
     - Trivy
  
### **CKS Exam Tips**
[🔼 Back to top](#table-of-contents)

- **Use vimrc oneliner command**  
  ```sh 
  echo "set ts=2 sw=2 sts=2 expandtab" > ~/.vimrc
  ```
- use 
  ```sh
  export oy="-o yaml"
  export now="--force --grace-period=0"
  ```
- **Troubleshoot api server using logs...**
  ```sh
  cd /var/log/pods/
  ls -l 
  cd kube-system_kube-apiserver-controlplane_xxx
  tail -f kube-apiserver/0.log
  ```
- **Troubleshoot api server using docker** `docker ps -a | grep -i kube-api`

- **Imperative command, open yaml, save/apply**
  ```sh
  kubectl run web --image nginx --dry-run=client -oyaml | vim - 
  :wq file1.yaml
  :wq kubectl apply -f -
  ```
- **Find process and remove**
  ```sh
  netstat -atnlp | grep -i 9090 | grep -w -i listen
  apt list --installed | grep -i nginx
  systemctl list-units --all | grep -i nginx
  ```
- **Find kubelet config path using** 
  - ```sh
    ps aux | grep -i kubelet | grep -w config
    ```
- **Find pod immutability issues using**
  - ```sh
    k get po --output=custom-columns="NAME:.metadata.name,SEC:spec.securityContext,SEC_CXT:.spec.containers[*].securityContext"
    ```
- **Find Falco rule using**
  - ```sh
    journalctl -fu falco | grep "xyz error"
    cat /etc/falco/falco_rules.yaml | grep "xyz error" -A5 -B5
    cat /var/log/syslog | grep falco | grep xyz | grep error

    # if falco installed as Helm/ds
    kubectl logs --selector app=falco | grep Error
    ```
- NOTE1: **AppArmor profile to copied in all nodes**
- NOTE2: **Trivy one liner** `trivy image -s CRITICAL nginx:1.14 | grep -i total`
- CKS Bookmarks Downloaded <a id="raw-url" href="resources/CKS-Bookmarks.html" download="CKS-Bookmarks.html">from here</a>
 <hr />

- [CCNF CKS Official Site](https://training.linuxfoundation.org/certification/certified-kubernetes-security-specialist/)
- [Refer Certification Preparation Material - my github page](https://github.com/ramanagali/Interview_Guide/blob/main/Certification_Preparation.md#cks)
- [CKS youtube playlist - my youtube channel](https://www.youtube.com/playlist?list=PLFkEchqXDZx6Bw3B2NRVc499j1TavjOvm)

## CKS - youtube playlist  - my youtube channel - Learn with GVR
  * [CKS youtube playlist - Learn with GVR](https://www.youtube.com/playlist?list=PLFkEchqXDZx6Bw3B2NRVc499j1TavjOvm)

[🔼 Back to top](#table-of-contents)
