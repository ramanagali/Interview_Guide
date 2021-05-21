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
	- default bridge network are usually used when your apps run in standalone containers that need to communicate
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
	- server configured with public key & sshd service running
	- client will verify server in network ~/.ssh/known_hosts
	- negotiate with shared session key
	- client sends ID of the key pair
	- sserver checks the authorized_keys file
	- server generate random num with public key and sends msgs
	- client decrypt msg get random num
	- client combine random num + session key sends MD5 hash
	- server uses same shared session key
	
* explain private & public key role in ssh 
	- private key stores at client
	- public key will be at server
* Linux command to check the logs
	- tail -f /var/log/mail.log
	
* how to find system slow, NFS slow
	- nfsstat -s (server)
	- nfsstat -c (cient)
	- nfsiostat (check its performance)
* how to run shell script in background
	- nohup script.sh &
	- nohup /path/to/your/script.sh > /dev/null 2>&1 & 
	- script.sh & disown &
* How to check server is slow?
	- cat /proc/cpuinfo 
	- lscpu
	- service --status-all
	- chkconfig --list
	- uptime
	- Step 1: Check I/O wait and CPU Idletime using top 
		- "wa" (I/O wait) 
		- "id" (CPU idletime)
	- Step 2: IO Wait is low and idle time is low: check CPU user time
		- %us
	- Step 3: IO wait is low and idle time is high
		- slowness isn't due to CPU or IO problems, it's likely an app-specific issue
		- starce or lsof
	- Step 4: IO Wait is high: check your swap usage
		- top or free -m
	- Step 5: swap usage is high 
		- means out of RAM
	- Step 6: swap usage is low
		- means real" IO wait 
		- use iotop
	- Step 7: Check memory usage
* TCP vs UDP
	- TCP is a connection-oriented protocol, whereas UDP is a connectionless protocol.
	- The speed for TCP is slower while the speed of UDP is faster
	- TCP uses handshake protocol like SYN, SYN-ACK, ACK while UDP uses no handshake protocols
	- TCP does error checking and also makes error recovery, on the other hand, UDP performs error checking, but it discards erroneous packets.
	- TCP has acknowledgment segments, but UDP does not have any acknowledgment segment.
	- TCP is heavy-weight, and UDP is lightweight.
	- It is a connection-oriented protocol.	It is a connectionless protocol.
	- TCP reads data as streams of bytes, and the message is transmitted to segment boundaries.	UDP messages contain packets that were sent one by one. It also checks for integrity at the arrival time.
	- TCP messages make their way across the internet from one computer to another.	It is not connection-based, so one program can send lots of packets to another.
	- TCP rearranges data packets in the specific order.	UDP protocol has no fixed order because all packets are independent of each other.
	The speed for TCP is slower.	UDP is faster as error recovery is not attempted.
	Header size is 20 bytes	Header size is 8 bytes.
	- TCP is heavy-weight. TCP needs three packets to set up a socket connection before any user data can be sent.	UDP is lightweight. There are no tracking connections, ordering of messages, etc.
	- TCP does error checking and also makes error recovery.	UDP performs error checking, but it discards erroneous packets.
	Acknowledgment segments	No Acknowledgment segments
	Using handshake protocol like SYN, SYN-ACK, ACK	No handshake (so connectionless protocol)
	- TCP is reliable as it guarantees delivery of data to the destination router.	The delivery of data to the destination can't be guaranteed in UDP.
* port for http, https, nfs, ICMP

### Kubernetes
* Explain kubernetes architecture
* If i give 3 servers, how you setup kubernetes
* kubernetes diff components
  - master, controller, scheduler, ETCD, flannel & caliko
  - node, container run time, kubelet, kubeproxy
* What are the taints & tolerations
	- Taint will be applied to nodes as key value paid, will allow to force set of pods
	- Tolerations will be applied to pods, allow the pods to schedule onto nodes with matching taints.
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


