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
- Control plane components a ```/etc/kubernetes/manifests/```

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

Ref: <https://github.com/kubernetes/kubernetes/releases>
Ref: <https://github.com/kubernetes/kubernetes/tree/master/CHANGELOG#changelogs>
</details>
<hr />

## 2. Cluster Hardening - 15%

<details>
<summary></summary>

### 2.1  Restrict access to Kubernetes API

- Control anonymous requests to Kube-apiserver by using
```sh --anonymous-auth=false```
- non secure access to the kube-apiserver

  1. **localhost**
      - port 8080
      - no TLS
      - default IP is localhost, change with `--insecure-bind-address`
  2. **secure port**
      - default is 6443, change with `--secure-port`
      - set TLS certificate with `--tls-cert-file`
      - set TLS certificate key with `--tls-private-key-file` flag

Ref: <https://kubernetes.io/docs/concepts/security/controlling-access/#api-server-ports-and-ips>

Ref: <https://kubernetes.io/docs/concepts/security/controlling-access/#api-server-ports-and-ips>

### 2.2 Use Role-Based Access Controls to minimize exposure

Roles live in namespace, RoleBinding specific to ns
ClusterRoles live across all namespace, ClusterRoleBidning
ServiceAccount should have only necessary RBAC permissions

**Solution**

- Create virutal users using ServiceAccount for specific namespace
- Create Role in specific namespace
  - has resources (ex: deployment)
  - has verbs (get, list, create, delete))
- Create RoleBinding n specific namespace & link Role & ServiceAccount
  - can be user, group or service account
- specify service account in deployment/pod level

```yaml
spec:
  serviceAccountName: deployment-viewer-sa
```

Ref: <https://kubernetes.io/docs/reference/access-authn-authz/rbac/>

### 2.3 Exercise caution in using service accounts e.g. disable defaults, minimize permissions on newly created ones

- Create ServiceAccount to automount to any pod `automountServiceAccountToken: false`

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: build-robot
automountServiceAccountToken: false
```

- Create Pod with serviceAccountName: default, automountServiceAccountToken: false

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  serviceAccountName: build-robot
  automountServiceAccountToken: false
```

Ref: <https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#use-the-default-service-account-to-access-the-api-server>

### 2.4 Update Kubernetes frequently

- Minor versions(bug fixes) must be patched regularly

- Latest 3 Minor versions receive patch support
- Minor versions receive patches for ~1year

Ref: <https://v1-21.docs.kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/>

</details>
<hr />

## 3. System Hardening - 15%

<details>
<summary></summary>

### 3.1 Minimize host OS footprint (reduce attack surface)

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
  containers:
  - name: nginx
    image: nginx:latest
  securityContext:
    # container will ran as root (Default is false)
    privileged = ture
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
- **Remove Obsolete/unncessary Software**
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
- **Restrict Obsolete Kernal Modules**
```sh
# list all kernal modules
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
Insall ufw			apt-get intall ufw
				systemctl enable ufw
				systemctl start ufw

check ufw firewall status 	ufw status/ufw status numbered
				ufw default allow outgoing
				ufw default deny incoming

Allow specific (80) 		ufw allow from 172.1.2.5 to any port 22 proto tcp
				ufw allow 1000:2000/tcp
				ufw allow from 172.1.3.0/25 to any port 80 proto tcp
				ufw allow 22
default deny 8080		ufw deny 80
activate ufw firewall		ufw enable
				ufw delete deny 80
				ufw delete 5
reset ufw			ufw reset
activate ufw firewall		ufw disable
```

- **Restirct allowed hostpaths with PodSecurityPolicy**
  - using PodSecurityPolicy can restrict AllowedHostPaths (used by hostPath volumes)
  - Ref: https://kubernetes.io/docs/concepts/policy/pod-security-policy/#volumes-and-file-systems

- **Identify and Fix Open Ports, Remove Packages**
```sh
Identify Open Ports, Remove Packages 
list all installed packages 			apt list --installed 
list active services 				systemctl list-units --type service
list the kernel modules 			lsmod
search for service 				systemctl list-units --all | grep -i nginx
stop remove nginx services			systemctl stop nginx
remove nginx service packages			rm /lib/systemd/system/nginx.service
remove packages from controlplane		apt remove nginx -y
check service listing on 9090			netstat -atnlp | grep -i 9090 | grep -w -i listen
check port to service mapping			cat /etc/services | grep -i ssh
check port listing on 22			netstat -an | grep 22  | grep  -w -i  listen
check lighthttpd service port			netstat -natulp | grep -i light
```
### 3.2 Minimize IAM roles

- **Least Privilege:** make sure IAM roles of EC2 permissions are limited, 
- **Block Access:** If not IAM; block CIDR, Firewall or NetPol. Example block EC2 169.254.169.254 
- Use AWS Trusted Advisor/GCP Security Command Center/Adviser
Ref: https://kubernetes.io/docs/reference/access-authn-authz/authentication/

### 3.3. Minimize external access to the network
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

#### 3.4.1 **SECCOMP PROFILES** 
- restricting the system calls it is able to make from userspace into the kernel
- SECCOMP can operate with 3 modes
  - Mode **0 - disabled**
  - Mode **1 - strict mode**
  - Mode **2 - filtered mode**
- SECCOMP profile default directory `/var/lib/kubelet/seccomp/profiles`
- SECCOMP Profiles are 3 types
  - Default Profile
  - Audit - audit.json
  - Violation- violation.json
  - Custom - fine-grained.json
- SECCOMP profile action can be
  - "action": "SCMP_ACT_ALLOW"
  - "action": "SCMP_ACT_ERRNO"
  - "defaultAction": "SCMP_ACT_LOG"
- - Create POD with Specific Profile
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
  starce -c touch /tmp/test.log	

  # check SECCOMP is supported by kernal
  grep -i seccomp /boot/config-$(uname -r)

  #Run Kubbernetes POD (amicontained - open source inspection tool)
  kubectl run amicontained --image r.j3ss.co/amicontained amicontained -- amicontained
  kubectl logs amicontained

  #default location 		
  /var/lib/kubelet/seccomp

  #use in pod			
  localhostProfile: profiles/audit.json
  ``` 
  - Ref: https://kubernetes.io/docs/tutorials/clusters/seccomp/
 #### 3.4.2 **APPARMOR** 	
  - Kernal Security Module to granualr access control for programs on Host OS
  - **AppArmor Profile** - Set of Rules, to be enabled in nodes
  - AppArmor Profile loaded in 2 modes
    - **Complain Mode** - Discover the program
    - **Enfore Mode** - prevent the program
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
  - load the profile on all our nodes default directory /etc/apparmor.d
  `sudo apparmor_parser /etc/apparmor.d/deny-write`
  - apply to pod
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: hello-apparmor
    annotations:
      # Tell Kubernetes to apply the AppArmor profile "k8s-apparmor-example-deny-write".
      # Note that this is ignored if the Kubernetes node is not running version 1.4 or greater.
      container.apparmor.security.beta.kubernetes.io/hello: localhost/k8s-apparmor-example-deny-write
  spec:
    containers:
    - name: hello
      image: busybox
      command: [ "sh", "-c", "echo 'Hello AppArmor!' && sleep 1h" ]
  ```
  - useful commands  
  ```
  check status			systemctl status apparmor
  check enabled in nodes		cat /sys/module/apparmor/parameters/enabled
  check profiles			cat /sys/kernel/security/apparmor/profiles

  installed			apt-get install apparmor-utils 
  create apparmor profile 	aa-genprof /root/add_data.sh
  apparmor module status		aa-status
  def Profile file directory 	/etc/apparmor.d/
  load profile file		apparmor_parser -q /etc/apparmor.d/usr.sbin.nginx
  load profile 			apparmor_parser /etc/apparmor.d/root.add_data.sh
  disable profile 		apparmor_parser -R /etc/apparmor.d/root.add_data.sh
  create 				apparmor-deny-write
          apparmor-allow-write
  ```
- Ref: https://kubernetes.io/docs/tutorials/clusters/apparmor/

</details>
<hr />

## 4. Minimize Microservice Vulnerabilities - 20%

<details>
<summary></summary>

### 4.1 Setup appropriate OS level security domains e.g. using PSP, OPA, security contexts

#### 4.1.1 **Pod Security Policies**

#### 4.1.2 **Open Policy Agent**

#### 4.1.3 **Security Contexts**
- Defines privilege and access control settings for a Pod or Container
- give Linux capabilities 
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
- Set the security context for a Container 
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
