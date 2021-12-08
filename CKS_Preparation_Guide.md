# Certified Kubernetes Security Specialist (CKS) Preparation Notes

Those who know about Kubernetes Admistriation, for the next level
Certified Kubernetes Security Specialist (CKS) exam point of view, below fine-tuned/shortened version one page study material to revise
<hr />

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
Insall ufw   apt-get intall ufw
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

- **Restirct allowed hostpaths with PodSecurityPolicy**
  - using PodSecurityPolicy can restrict AllowedHostPaths (used by hostPath volumes)
  - Ref: <https://kubernetes.io/docs/concepts/policy/pod-security-policy/#volumes-and-file-systems>

- **Identify and Fix Open Ports, Remove Packages**

```sh
Identify Open Ports, Remove Packages 
list all installed packages    apt list --installed 
list active services     systemctl list-units --type service
list the kernel modules    lsmod
search for service     systemctl list-units --all | grep -i nginx
stop remove nginx services   systemctl stop nginx
remove nginx service packages   rm /lib/systemd/system/nginx.service
remove packages from controlplane  apt remove nginx -y
check service listing on 9090   netstat -atnlp | grep -i 9090 | grep -w -i listen
check port to service mapping   cat /etc/services | grep -i ssh
check port listing on 22   netstat -an | grep 22  | grep  -w -i  listen
check lighthttpd service port   netstat -natulp | grep -i light
```

### 3.2 Minimize IAM roles

- **Least Privilege:** make sure IAM roles of EC2 permissions are limited,
- **Block Access:** If not IAM; block CIDR, Firewall or NetPol. Example block EC2 169.254.169.254
- Use AWS Trusted Advisor/GCP Security Command Center/Adviser
Ref: <https://kubernetes.io/docs/reference/access-authn-authz/authentication/>

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

  - Ref: <https://kubernetes.io/docs/tutorials/clusters/seccomp/>

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
  check status   systemctl status apparmor
  check enabled in nodes  cat /sys/module/apparmor/parameters/enabled
  check profiles   cat /sys/kernel/security/apparmor/profiles

  installed   apt-get install apparmor-utils 
  create apparmor profile  aa-genprof /root/add_data.sh
  apparmor module status  aa-status
  def Profile file directory  /etc/apparmor.d/
  load profile file  apparmor_parser -q /etc/apparmor.d/usr.sbin.nginx
  load profile    apparmor_parser /etc/apparmor.d/root.add_data.sh
  disable profile   apparmor_parser -R /etc/apparmor.d/root.add_data.sh
  create     apparmor-deny-write
          apparmor-allow-write
  ```

- Ref: <https://kubernetes.io/docs/tutorials/clusters/apparmor/>

</details>
<hr />

## 4. Minimize Microservice Vulnerabilities - 20%

<details>
<summary></summary>

### 4.1 Setup appropriate OS level security domains e.g. using PSP, OPA, security contexts

#### **Admission Controller**

- Implment security measures to enforce. Triggers before creating a pod
- Enable a Controller in kubeadm cluster /etc/kubernetes/manifests/kube-apiserver.yaml

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

- Defines policies to controls security sensitive aspects of the pod specification
- PodSecurityPolicy is one of the admission controller
- enable at api-server using  `--enable-admission-plugins=NameSpaceAutoProvision,PodSecurityPolicy`
- Create Pod using PSP

  ```yaml
  apiVersion: policy/v1beta1
  kind: PodSecurityPolicy
  metadata:
    name: privileged
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

  - POD Access to PSP for authorization
    - Create Service Account or use Default Service account
    - Create Role with podsecuritypolicies, verbs as use
    - Create RoleBinding to Service Account and Role
- Ref: <https://kubernetes.io/docs/concepts/policy/pod-security-policy/>

#### 4.1.2 **Open Policy Agent (OPA)**

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
- Ref: <https://kubernetes.io/docs/tasks/configure-pod-container/security-context/>

### 4.2 Manage Kubernetes secrets

- Types of Secrets
  - Opaque(Generic) secrets -
  - Service account token Secrets
  - Docker config Secrets -
  - Basic authentication Secret -
  - SSH authentication secrets -
  - TLS secrets -
  - Bootstrap token Secrets -  

- Secret as Data to a Container Using a Volume

  ```
  kubectl create secret generic mysecret --from-literal=username=devuser --from-literal=password='S!B\*d$zDsb='
  ```

  Decode Secret  

  ```
  kubectl get secrets/mysecret --template={{.data.password}} | base64 -D
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

- Ref: <https://kubernetes.io/docs/concepts/configuration/secret/>
- Ref: <https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/>

### 4.3 Use container runtime sandboxes in multi-tenant environments (e.g. gvisor, kata containers)

- Sandboxing is concept of isoloation of containers from Host
- docker uses default SecComp profiles restrict previleges. Whitlist or Blacklist
- AppArmor finegrain control of that container can access. Whitlist or Blacklist
- If large number of apps in containers, then SecComp/AppArmor is not the case

#### 4.3.1 gvisor

- gVisor sits between container and Linux Kernal. Every container has their own gVisior
- gVisor has 2 different componets
  - Sentry: works as kernal for containers
  - Gofer is file proxy to access system files. Middle man betwen container and OS
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

#### Container Runtime

- docker run => docker CLI => REST API => Docker Daemon => check locally => Registery => call containerd  => convert image to OCI container => containerd-shim => runC will start container
  - we can override runtime when running container

  ```
  docker run --runtime kata -d nginx
  docker run --runtime runsc -d nginx
  ```

- Creating a Container Runtime for additional layes of isolation
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
    - image: nginx
      name: nginx
  ```

- Ref: <https://kubernetes.io/docs/concepts/containers/runtime-class/>
- Ref: <https://github.com/kubernetes/enhancements/blob/5dcf841b85f49aa8290529f1957ab8bc33f8b855/keps/sig-node/585-runtime-class/README.md#examples>

### 4.4 Implement pod to pod encryption by use of mTLS

- mTLS: Is secure communication between pods
- With service mesh Istio & Linkerd mTLS is easier, managable
  - mTLS can be Enforced or Strict
  
**Steps for TLS certificate for a Kubernetes service accessed through DNS**

- Download and install CFSSL
- Generete private key using `cfssl genkey`
- Create CertificateSigningRequest
- Get CSR approved (by k8s Admin)
- Once approved then retrive from status.certificate
- Download and use it
- Ref: <https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/>

</details>
<hr />

## 5. Supply Chain Security - 20%

<details>
<summary></summary>

### 5.1 Minimize base image footprint

- Use Slim/Minimal Images than base images
- Use Docker multi stage builds for lean
- **Use Distroless:**
  - Distroless Images will have only your app & runtime dependencies
    - No package managers, shell, n/w tools, text editors etc
  - Distroless images are very small
- use trivy image scanner for container vulnerabilities, filesys, git etc
- Ref: <https://github.com/GoogleContainerTools/distroless>
- Ref: <https://github.com/aquasecurity/trivy>
  
### 5.2 Secure your supply chain: whitelist allowed registries, sign and validate images

- **Approach 1 - using ImagePolicyWebhook Admission Controller**
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
                - "--registry-whitelist=docker.io,k8s.gcr.io"
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
          nodePort: 30080
      selector:
        app: image-bouncer-webhook
    ```

  - Create custom kubeconfig with above service, its client certificate
    - `/etc/kubernetes/pki/admission_kube_config.yaml`

    ```yaml
    apiVersion: v1
    kind: Config
    clusters:
    - cluster:
        certificate-authority: /etc/kubernetes/pki/server.crt
        server: https://image-bouncer-webhook:30080/image_policy
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

  - Create ImagePolicyWebhook AdmissionConfiguration file, update custom kubeconfig file at
    - `/etc/kubernetes/pki/admission_configuration`

    ```yaml
    apiVersion: apiserver.config.k8s.io/v1
    kind: AdmissionConfiguration
    plugins:
    - name: ImagePolicyWebhook
      configuration:
        imagePolicy:
          kubeConfigFile: /etc/kubernetes/pki/admission_kube_config.yaml
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
    - --admission-control-config-file=/etc/kubernetes/pki/admission_configuration.yaml
    ```

  - Ref: <https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#imagepolicywebhook>
  
- **Approach 2 - ConstraintTemplate**
  - Create ConstraintTemplate CRD to whitelist docker registries
  - Create a Resource Constraint with allowed docker registries
  - Create a pod with valid registry and test
  
### 5.3 Use static analysis of user workloads (e.g.Kubernetes resources, Docker files)

- Static YAML analysis using poular tool Kubesec
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

- Vulnerability scanning tools are Trivy(*) & Anchore
- Install Trivy (by Aquasec) on your control plane node
- Scan an image using trivy
  - `trivy image busybox:1.33.1`
- It results Low, Medium, High, Critical & VolnarabilityID
  - `trivy image-- severity CRITICAL,HIGH busybox:1.33.1`

- Ref: <https://kubernetes.io/blog/2018/07/18/11-ways-not-to-get-hacked/#10-scan-images-and-run-ids>
- Ref: <https://github.com/aquasecurity/trivy>
- Ref: <https://github.com/anchore/anchore-cli#command-line-examples>

</details>
   <hr />

## 6. Monitoring, Logging and Runtime Security - 20%

<details>
<summary></summary>

### 6.1 Perform behavioral analytics of syscall process and file activities at the host and container level to detect malicious activities

- Falco can detect and alerts on any behavior that involves making Linux system calls
- **Falco** operates at the user space and kernel space
  - Policy Engine
  - Libraries
  - Falco Rules
- Helm Install Falco as DaemonSet

  ```sh
  helm repo add falcosecurity https://falcosecurity.github.io/charts
  helm repo update
  helm install falco falcosecurity/falco
  ```

- **Falco Rules** Filters for engine events, in YAML format Example Rule  

  ```yaml
  - rule: Write below etc
    desc: an attempt to write to any file below /etc
    condition: write_etc_common
    output: "File below /etc opened for writing (user=%user.name command=%proc.cmdline parent=%proc.pname pcmdline=%proc.pcmdline file=%fd.name program=%proc.name gparent=%proc.aname[2] ggparent=%proc.aname[3] gggparent=%proc.aname[4] container_id=%container.id image=%container.image.repository)"
    priority: ERROR
    tags: [filesystem, mitre_persistence]
  ```

- **Falco Configuration** for Falco daemon, YAML file and in  key: value/list  
  - config file located at `/etc/falco/falco.yaml`
- Ref: <https://falco.org/docs/getting-started/>
- Ref: <https://github.com/falcosecurity/charts>
- Ref: <https://falco.org/blog/detect-cve-2020-8557/>
  
### 6.2 Detect threats within physical infrastructure, apps, networks, data, users and workloads

### 6.3 Detect all phases of attack regardless where it occurs and how it spreads
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
- Monitor using sysdig k8s cluster, ns, svc, rc and labels
- sysdig capture system calls and other OS events
- Exploring a Kubernetes Cluster with csysdig 
  `csysdig -k http://127.0.0.1:8080`
- Monitoring/Visualize Kubernetes with Sysdig Cloud 
- Ref: https://kubernetes.io/blog/2015/11/monitoring-kubernetes-with-sysdig/
- Ref: https://docs.sysdig.com/

### 6.5 Ensure immutability of containers at runtime
- immutable = cannot modify original state
- POD level using securityContext; readOnlyRootFilesystem = true,  privileged=false

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: security-context-demo
  spec:
    securityContext:
      readOnlyRootFilesystem: true
        privileged: false
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

- Enforce using PSP(Pod Security Policies) - readOnlyRootFilesystem = true,  privileged=false; runAsUser=NonRoot

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
- The cluster audits the activities 
  - generated by users, 
  - by applications that use the Kubernetes API
  - control plane aswell
- When API Server performs action, it creates stages
  - **RequestRecived** - The stage for events generated as soon as the audit handler receives the request
  - **ResponseStarted** - response headers are sent & response body is sent. only for long running (Ex: watch) 
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
- Enble AuditLog in KubeAPI server level
  ```yaml
  - --audit-log-path=/var/log/k8-audit.log
  - --audit-policy-file=/etc/kubernetes/audit-policy.yaml
  - --audit-log-maxage=10
  - --audit-log-maxbackup=5
  - --audit-log-maxsize=100
  ```
- Ref: https://kubernetes.io/docs/tasks/debug-application-cluster/audit/
</details>
 <hr />

- [Refer Certification Preparation Material
](#https://github.com/ramanagali/Interview_Guide/blob/main/Certification_Preparation.md#cks)
