# Cloud DevOps Engineer Architect Interview Questions

### DevOps
* DevOps rollback strategy
* How you perform Hot deployment
* Biggest issue in PROD
* Scale prod
* DevOps vs Agile
* What you monitor in Sonarqube

### Git
* Diff between git pull and git clone
   	- pull is remote branch changes will be added locally & commit
   	- fetch is fetches but not commit
* How to delete previous commit from pushed repo
	- git reset --hard <commit>
* Diff between git merge and git rebase
	- merge combines source branch to a target branch, preserves the entire history
	- rebase overwrites the changes
	
* Diff between git pull vs git fetch
* git clone specific branch
  	- git clone -b brnachname
* git stash
 	- temparary save
* git how to revert specific branch 
	- git log             	(Shows the commit logs)
	- git revert commitID         (directly commits/reverts )
	- git revert -n commitID      (merges the changes but not commit)
	- git reset  --hard commitID  (resets & reverts the changes)
	
* git differnece
	- git diff	(Shows the changes between unstaged files and the commits)
	- git diff --staged	(Shows the changes between staged(ready-to-be-committed) files and the commits)

* git find list of commited files
	- git log --oneline

### Docker
* What is default docker network applies when creating containers
* How docker resolves the container names
* What is docker compose
* Diff between docker CMD and ENTRYPOINT
* How can we make docker image lightweight

### CI CD
* What is the Branching Strategy to you use
* How you will fix prod issues
* What is the end to end flow of CI CD
* Explain CI
* Explain CD
* Where to store artefacts

### Ansible
* How you used Ansible in your project
* Archtecture of Ansible
* What are ansible facts?
* Diff between Ansible & Terraform
* What are the Ansible roles
* What are the Ansbile valuts
  - if inventory is dyanmic
* How to enable detailed logs
* Ansible tower
* Dynamic Inventory
* Ansible how you avoid prompt
* ansible copy
* Ansible vs chef vs puppet


### Terraform
* How do you store secrets in terraform
* What is local & remote provisioner
* What are the terraform modules, explain an an example

## Linux
* How ssh works
* explain private & public key role in ssh 
* Linux command to check the logs
* tail
* follow
* how to find system slow, NFS slow
* how to run sh in background
* How to check server is slow
* TCP vs UDP
* port for http, https, nfs, ICMP
* bamboo agent setup steps


### Kubernetes
* Explain kubernetes architecture
* If i give 3 servers, how you setup kubernetes
* kubernetes diff components
  - master, controller, scheduler, ETCD, flannel & caliko
  - node, container run time, kubelet, kubeproxy
* What are the taints & tolerations
* How you acheive pod security
* Stateful sets
* what is Init container
* Diff between deployment, service and daemonset
* Explain kubernetes reverse proxy
* k8s create vs apply
* k8s pod life cycle
* k8s startup probe liveness  & ready probe
* k8s High avail: 
	- liveness & readyness probe
	- HoriPodAutoscaler VirtualPodAutoscaler
	- network policies
	- use volumes for stateful & helper 
* k8s namespaces in default installation
  - dafault
  - kubesystem
  - kubepublic
  - kubenoderlease
* why we attach resource quota. create resource quota for namespace
* k8s sidecar containers
* K8s volume types
* k8s adapter containers
* k8s network policy
  - ingress & egress
  - ipBlock
  - namespaceSelector
  - podSelector
  - ports
* K8s pod disruption budget
  - volunatry and non voluntary... in case of any down... minimum 1 should be there
* k8s operators or custom resource definitions
  - in case of stateful applications

* k8s control loop mechanism
  - default mechanism... observe and take action on 

* Helm Helm3 
  - no tiller, 3-way-verification, schema verification, release namespace.

* use of networking solutions calico/flannel
  - 4 nodes
  - make sure unique IP's created

* use of PV & PVC (local & remote storge)
  - volumes 
  - PV use central volume like EFS/EBS
  - PVClaim - attach to pod
* which one is default deployment strategy? how it works?
* command to check the container logs in pod?
* what are the types of services present in kubernetes?
* What is the link between pod and service?

### Network
* How Https works
* Diff between SecGroup vs NACL
* Explain stateful vs stateless firewall
* 

### Bamboo

### Jenkins

### Packer

### AWS
* Where to Host DB if DB has to be in the region where AWS/AZURE not avail
* Scaling in PRD


### GCP

### Azure

### SQL/RDBMS

### SRE

### DevSecOps
* What is the benefit of Fortify
* What you monitor in Sonarqube
* How you deal Cross site scripting

### Python


