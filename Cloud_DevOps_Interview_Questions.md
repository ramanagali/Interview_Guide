# Cloud DevOps Engineer/Architect Interview Questions for Experienced Professionals

### Web Technologies
What happens when type google.com in browser and enter ?
<details>
<summary></summary>

* **type g**		
	* Browser Auto Complete function kicks in 
* **press enter**	
	* char converted to int 13, signal will be sent
* **parse URL**			
    * it has http or /(slash)|
* **URL or Search**		
  - uses default search engine if its search term
* **Convert non-ASCII** 
  * convert URL to a-z,A-Z,0-9,-,. to ASCII
* **HSTS Check**			
  * browser checks preloded HTTP Strict Trasnport Security policy
  * HTTP from the list
* **DNS Lookup**
  - verify DNS from browser cache
  - check local machine hostfile mapping for DNS lookup
  - if not local router or DNS server
  - Address Resolution(ARP) process for default Gateway
* **Address Resolution Protocol**
  - Check local ARP Cache for target IP
  - if not, any subnets target IP in the local route table
  - if not use default gateway subnet
  - grab local MAC address, look for target MAC Address
  - sends L2 (Data link)	
  - broadcast the ARP request to all other ports and wait for reploy
  - if any switch, the re-broadcast the ARP request to all other ports
  - ARP reply is received, now we know DNS server IP
  - DNS client(source port 1023) establish socket UDP port 53 on DNS Server 
* **Opening of Socket**
  - Once target IP of destination server is received
  - https(443) or http(80 default)
  - request L4 TCP socket stream to target IP
    - destination port 
    - source port added to header
  - request sent to L3(Network layer), 
    - adds targetIP & current IP in header
  - request will be sent to L2 (DataLink)
    - add MAC address of gateway
  - Physical layers converts to binary
  - vi ISP hits target server - TPC Conn FLOW
    - Client choose ISN & sends SYN packet to server 
    - Server adds ISN+1, ACK sends to Client
    - Client sends ACK,ISN+1, receiver ack number
    - Data Transfers, sends ACK
    - closer sends FIN packet
* **TLS Handshake**
  - Client sends TLS ClientHello, cipher algo & compression method
  - Server sends TLS ServerHello + public CA
    - client verifies against trusted CA,
    - generates pseudo random bytes for symmetric key
  - encrypt with public key 
  - Server decrypt using private key & makes own copy of symmetric master key
  - Client sends Finished message with symetric key
  - Server generetes hash, decrypts with client sent hash(symmetric key)
  - TLS session transmits with symmetric key
* **HTTP Protocol**
  - http reqeust will be sent
  - from server 200 ok response headers
  - sent payload of html
  - browser parses(image, css etc)
* **HTTP Server Request Handle**
  - HTTPD break downs GET, POST, HEAD,PUT, PAtH, DELETE, CONNECT, OPTIONS
  - rewrite modules
  - headers handlers etc
* **Browser**
  - HTML Parsinig
  - CSS interpretation
  - Page Rendering
  - GPU Rendering
  - Browser add-ons
</details>


### DevOps
* DevOps Deployment strategies?
  * **Recreate**: Version A is terminated then version B is rolled out.
  * **Rolling-update/incremental**: Version B is slowly rolled out and replacing version A.
  * **Blue/Green**: Version B is released alongside version A, then the traffic is switched to version B.
  * **Canary**: Version B is released to a subset of users, then proceed to a full rollout.
  * **A/B testing**: Version B is released to a subset of users under specific condition.
  * **Shadow**: Version B receives real-world traffic alongside version A and doesn’t impact the response.
* How you perform Hot deployment in Cloud?
  * using Canary 
* Explain Biggest issue in PROD ?
* How you scale prod ?
* DevOps vs Agile

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
	- git fetch retrieve the latest meta-data from original
	- git pull brings the changes from remote & adds in local but not commit
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
* How docker resolves the container names ?
* What is docker compose ?
  * dsfsdf
* Diff between docker CMD and ENTRYPOINT ?
  * sdfsdaf
* How can we make docker image lightweight ?
  * using multi stage builds

### CI CD
* What is the Branching Strategy to you use
* How you will fix prod issues
* What is the end to end flow of CI CD
* Explain CI
* Explain Continuous Delivery and Continuous deployment
	- Delivery automating the entire software release process
	- Delivery everthing is auto but  has manual step before deploying in PRD 
	- Deployment is a step up from Continuous Delivery 
	- Deployment evey step is auto
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
	- store in AWS Secret manager 
	- read from TF code using aws_secretsmanager_secret_version
	- using TF locals jsondecode and give reference
* What is local & remote provisioner
	- local exec executes code on the machine running terraform
	- runs on the provisioned resource
* What are the terraform modules, explain an an example
	- code reuse, versioning, support remote storage
* Where the terraform logs are stored
	- TF_LOG to JSON outputs logs at the TRACE level 
		- $ export TF_LOG="TRACE"
	- TF_LOG_PATH for persistent log path
		- $ export TF_LOG_PATH="terraform.txt"
	
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
	
* how to find system slow, EFS slow
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
	- http 80, https 443, nfs 2049, ICMP 7
* Linux Performance Monitoring?
  - vmstat (virtual memory statistic tool)
    ```sh
	sudo apt install sysstat         [On Debian, Ubuntu and Mint]
	sudo yum install sysstat         [On RHEL/CentOS/Fedora and Rocky Linux/AlmaLinux]
	sudo emerge -a app-admin/sysstat [On Gentoo Linux]
	sudo pacman -S sysstat           [On Arch Linux]
	sudo zypper install sysstat      [On OpenSUSE] 
	
	# List Active and Inactive memory 
	vmstat -a
	vmstat 2 6		# vmstat executes every 2 sec & 6 times
	vmstat -t 1 	# shows timestamps
	vmstat -s		# stats of various coutners
	vmstat -d		# disk stats
	vmstat -S M 1 5	# stats in megabytes

	iostat		#CPU and I/O statistics
	iostat -c 	# CPU Statistics
	iostat -d 	# # disk I/O stats
	iostat -p sda # I/O Statistics of Specific Device
	iostat -N # LVM stats
	```
* Linux Configuration &  Troubleshooting commands ?
	```sh
	ifconfig # assign ip,enable,disable interface

	# set IP
	ifconfig eth0 192.168.50.5 netmask 255.255.255.0	
	ifup eth0	# enable eth0
	ifdown eth0	# disable eth0
	ifconfig eth0 mtu XXXX	# set mtu

	ping -c 5 www.google.com # test connectivity 
	traceroute 4.2.2.2	# number of hops b/n destinations
	netstat -r 			# connection info, routing table information
	dig www.google.com	#query DNS related information (A,CNAME,MX)
	nslookup www.google.com	#find DNS related queries, shows A record
	route -n 	#shows default routing
	route add -net 10.10.10.0/24 gw 192.168.0.1 
	route del -net 10.10.10.0/24 gw 192.168.0.1
	route add default gw 192.168.0.1
	host www.google.com 	# find name to IP 
	arp -e		#Address Resolution Protocol default table
	ethtool eth0 	# view & set Network Interface Card NIC
	iwconfig [interface]	#configure a wireless network interface
	hostname	# to identify network
	nmcli	#manage network settings
	nmtui	#manage network devices
	```
	

  

### Kubernetes
* Explain kubernetes architecture ?
* If i give 3 servers, how you setup kubernetes ?
* kubernetes diff components ?
  - master, controller, scheduler, ETCD, flannel & caliko
  - node, container run time, kubelet, kubeproxy
* What are the taints & tolerations ?
	- Taint will be applied to nodes as key value paid, will allow to force set of pods
	- Tolerations will be applied to pods, allow the pods to schedule onto nodes with matching taints.
* Assigning Pods to Nodes - Node Affinity
	- nodeSelector is a field of PodSpec 
	- Node affinity (under pod spec) is similar to nodeSelector but more expressive syntax
	   	- requiredDuringSchedulingRequiredDuringExecution. it allows you to constrain which nodes your pod is eligible to be scheduled on, based on labels on the node
* How you acheive pod security
	- use securityContext in pods runAsUser, runAsGroup, runAsNonRoot & fsGroup
	- use securityContext => linux capabilities
* Explain about Stateful sets
	- like deployment but maintains a sticky identity for each Pods, not interchangeable & persistent identifier 
	- must need pv, headless service
	- Stable, unique network identifiers
	- Stable, persistent storage
	- Ordered, graceful deployment and scaling
	- Ordered, Automated rolling updates
* Headless service
	- used for Statefiul set pod
	- No Load balancing, directly interact with Pod
	- service discovery with service name
* What is Init container
	- Specialized containers that run before app containers in a Pod
	- used for setup scripts
* Diff between deployment, service and daemonset
	- Deployments manage stateless services
	- DaemonSets attempt to adhere to a one-Pod-per-node model
	- Service is an abstract way to expose an app running on a set of Pods as a network service
* Explain kubernetes reverse proxy
	- Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster
	- Ingress maps service
	- http path based
* Proxies in Kubernetes
	- kubectl proxy
	- apiserver proxy
	- kube proxy
* k8s create vs apply
	- imperative
	- declarative
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

*  Helm3 
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
* Explain 3 Way Handshake?
  * First step is 3-way handshake must be established
  * **1st Step**: 
    * Client sends SYN segment to the server 
    * ISN(InitialSeqNum)=7001, ACK=0, SYN=1)  
    * source port: client ephemeral port
    * destination port: 443
  * **2nd Step2**: 
    * Server replies SYN + ACK to client & ask for Open
    * SYN=1, ACK=1, ACK Number=7002, Server Seq #3001  
    * source port: 443
    * destination port: client ephemeral port
  * **3rd Step**: 
    * SYN-0, ACK=1, ACK number=3002, Initial Seq=7002
    * source port: client ephemeral port
    * destination port: 443
  	<img src="https://lh6.googleusercontent.com/-L9GyKGal2kX0x1uhEr_WcIPJNjaXt56MwI4dppR9LKS0SKciZ4ehop6uYdAM7RFm9PYoPcK445rVYeqzjAWSOaHNXd6wvgoWbVUVQnwLZe-M2iav6FZVIfTlE15ULPrWuEYi4Mw" alt="" width="602" height="376" loading="lazy" class="">
* How Https works?
* TCP vs UDP
  	| TCP  | UDP |
	| ------------- | ------------- |
	| Requires an established connection |Connectionless protocol|
	| Data sequencing| No Data sequencing|
	| Guaranteed delivery |Cannot guarantee delivery |
	| if data lost, Retransmission is possible| No retransmission|
	| error checking & ACK, SYN,SYN-ACK handshakes |error checking using checksum|
	| data read as byte, msgs in segments | UDP packets with defined boundaries|
	|Slower than UDP |Faster than TCP |
	| does not support Broadcasting| support Broadcasting| 
	| Used by HTTPS, HTTP, SMTP, POP, FTP, etc|Video conferencing, streaming, DNS, DHCP, TFTP, SNMP, RIP, and VoIP.|
	
* What are the Network Topology Types ?
  * Point to Point
  * Bus
  * Ring
  * Star
  * Tree
  * Mesh
  * Hybrid

### Bamboo

### Jenkins

### Packer

### AWS
* How you Host DB if DB has to be in the region where AWS/AZURE not avail
* Diff between Stateful and Stateless firewalls ?
* 


### GCP

### Azure

### SQL/RDBMS

### SRE

### DevSecOps
* What is the complete workflow in DevSecOps process?
* What is SAST ?
* What is DAST?
* What you monitor in Sonarqube
* What is the benefit of Fortify ?
* What you monitor in Sonarqube ?
* How you deal Cross site scripting ?
* What is SQL Injection?
* How do you secure git repo ?
* How do you secure pipeline?
* How do you check docker image for volunarables?
* Network level security?
* Load Testing?
* Stress Testing?

### Python


