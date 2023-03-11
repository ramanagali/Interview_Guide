# AWS Security Speciality Notes

- Infrastructure security protections accross accounts  	- AWS firewall manager
- Prevent Security Groups unauthorized changes 			- AWS Control Tower
- Security protection accorss accounts & resources in organization - AWS Firewall Manager acc,security mgmt policies

### KMS
- KMS programatic Access check 			- 
- safest process to delete KMS keys 		- Delete the KMS keys with a 30 day waiting period
- CloudHSM manual recovery 				- LunaSlotManager.reinitialize()
- CloudHSM appliance SSH 					- The public key is installed on the HSM during provision
- if 2 CloudHSM down 						- single HSM can maintain service
- HSM HA config 							- hands-off (HA recovery on) or manual (HA recovery off)
- CloudHSM multi region resiliancy		- CLoudHSM ability to copy keys to diff region, where - KMS does not
- KMS key delete in 24hours				- re import 
- Rotate KMS imported keys 				- automatic key rotation is not supported, reimport new key and point the alias
- CMK issues in Parameter Store 			- The CMK is Disabled or Pending Deletion or CMK doesnot exists
- KMS with 2 factor for decryption		- define MFA policy within key policy
- KMS Key policy 							- allows account to root access, IAM will manage for users
- Intransit Encryption					- AWS ACM certificate at ELB
- KMS CMK resilient accross Regions 		- re-wrap CMK DEK from source to target region
- S3 + KMS users change					- use KMS grants to AWS principals
- Compliance Documents					- AWS artifacts
- kms:ViaService                - limits use of an KMS key to requests from specified AWS services

### CloudFront
- CloudFront records in S3 				- end-user request for an object
- CloudFront restrict privately			- Create CF signed URL’s
- CloudFront + web app + protect users	- app to use CF key pair to set singed cookies

### VPC Networking
* NACL									- Inbound HTTP, outbound ephemral (1024 - 65535)
* custom NAT instance what must 			- The source/destination checks should be disabled on NAT instance
* VPN private and public traffic route	- RT + IG, VPN subnet to VPG, Subnet cannot span in AZ's
* VPN conn b/n on-prem & AWS 				- VPG, CG, VPN connection
* blast radius of a potential DDoS solution 	- SG,NACL, allow required ports
* VPC Flow logs how to add blocked traffic 	- log rejected traffic, all traffic
* VPC peering									- VPC-A and VPC-B same CIDR => VPC C can peer with A & B
* Network Error: Connection timed out 		- multiple access control lists for the subnet
* 1 RT with multiple subnets, 1 subnet 		- with 1 route table
* protect the subnets from specific IP addresses		- InBound NACL rule to deny
* unable to RDP from your to webserver in public sn 	- set public IP, windows firewall to RDP 
* Max SG's, NACL's over 								- Host based firewall
* IDS 										- VPC Traffic Mirroring and send traffic tto EC2 (which has IDS)

### EC2
* Splunk installed in EC2 to access S3 		- EC2 instance profile, kms key, s3 access
* EC2 Share images accross Organization 		- AWS Resource Access Manager 
* EC2 DDoS attacked, ensure minimize downtime - AWS Shield Advanced to block and mitigate future DDoS
* EC2 instance store what after reboot 			- data persisted
* EC2 prevent hacker to escalte root privileges 	- Linux iptables-host based restrictions
* EC2 instance reboot 							- Data persists in the instance store
* EC2 to S3, how to send securely? 			- VPC endpoint for Amazon S3
* EC2 access DynamoDB table 					- VPCE & IAM role
* EC2 access user session of DynamoDB table 	-  IAM Role with permissions for EC2
* EC2 instance detailed monitoring CW			- every minute metrics send to CW
* EC2 webserver protect from blast surface 	- ELB, SG, allow 80 & 443
* EC2 kernal panic error 						- A hardware device failure
* EC2 SSH private Key pair lost,protect 		- use EC2 RunCmd, Modify the public key in ~/.ssh/authorized_keys
* EC2 n/w rules if SG's are exceeded			- Use Built in firewall
* EC2 Brute force attach by 100's of IPs 		- Insall Intrusion Detection Systems (IDS) 
* EC2 analyzing traffic 						- Insall Host based Intrusion Detection Systems (IDS) 
* EC2 collect memory dump 					- Download and run the EC2Rescue
* EC2 SSH Key pair lost, find 				- search ec2-describe-instances --filters=key-name=KEY
* EC2 with EBS is stopping, starting when EBS removed		- kms:CreateGrant, kms:GrantIsForAWSResource
* EC2 CW agent not logging					- IAM trust cwlogs.amazonaws.com
* EC2 outgoing traffict  to specific URL’s 	- use AWS maketplace solution to URL whitelist
* EC2 Ping not working						- SG level, Allow Inbound ICMP traffic
* Block the EC2 instance metadata service		- local firewall rules using iptables

### EBS										- Create a new key-pair & launch new instance
* EBS-backed EC2	accidentally terminated		- The data is deleted
* after decrypt of EBS KMS key is deleted 	- data is not recoverable
* secure the EBS volume data at rest by explicit 			- Enable encryption with Cloud HSM
* Encrypted EBS volumes are dettaching to EC2	- kms:CreateGrant & kms:GrantIsForAWSResource'
* EBS Encryption
  * Data at rest inside the volume is encrypted
  * All data moving between the volume and the instance is encrypted
  * All snapshots created from the volume is encrypted
  * All volumes created from those snapshots are encrypted
  * Data at rest inside the volume is encrypted 
  * Data moving between the volume and the instance is encrypted

### IAM
* IAM role consists of 					- policy can be empty, trust policy cannot be empty for assume role
* S3 permission error Invalid principal in policy		- IAM role/user deleted or S3 ARn incorrect	
* IAM Credential report how often 		- every 4 hours
* IAM and MFA 							- via CLI & virtual & physical token
* suspicious activity in your account 	- 
* AD SSO with AWS cosnole 				- SAML 2.0
* cross-account trouble access 			- Role ARN or ExternalID wrong or AssumeRole not granted
* Root user account usage alerts 			- CW events => lambda => SNS
* Login to AWS using 3rd party & mobile 	- Congito & add social provider
* S3 Invalid principal in policy 			- Principal is an unsupported type, IAM user/role has been deleted
* S3 403 error from EC2 					- S3 bucket policy,KMS Key policy allow decrypt, EC2 Iam role permissions 
* S3 encrypt each file with a diff encryption key 	- use SSE-S3
* fastest way to block further access               - IAM =>Revoke all session

### on-premises related
* on-premises apps to AWS most secure way - VPN Gateway over AWS Direct Connect
* on-prem & AWS resources access with corporate creds 	-  secure Amazon Client VPN with corp creds to both AWS & on-prem
* Encrypt on-prem & AWS 						- AWS DC, VIF, Customer gateway, VPN connection over Direct connect
* on-premises ADFS and SSO to AWS console		- Cognito pool, connector @AD server, use CognitoSDK to auth with AD
* OpenID-Connect compatible social media		- Amazon Cognito acts as an identity broker does much of the fed work
* On-prem to EC2 encrypt in transit 			- CLB, TCP listener and terminate TLS

* AWS DB passwords, rotation etc 			- Secrets Manager
* AWS API notifications of accounts how ? - CW events rules, Kinesis stream to downstream
* designing for failure 					- anticipate failure and recover automatically

### CloudTrail
* Security of CloudTrail log files 		- KMS (SSE-KMS)
* CloudTrail logging from multiple regions - single CloudTrail trail & send to single S3
* monitor failed login attempts on aws accounts 	- CloudTrail => S3 => SNS =>  
* CloudTrail logs syntax 				- AccountID_CloudTrail_RegionName_YYYYMMDDTHHmmZ_UniqueString.FileNameFormat 
* CloudTrail failing to deliver to S3		- S3 Bucket policy should allow from CloudTrail
* CloudTrail logs							- Single S3 bucket, single trail for all regions

### Trusted Advisor, security related
* Trusted Advisor perform security audits, tracking usage patterns in S3 - bucket logging
* failed console sign-in attempts 		- CT => S3 => Athena 
* Control the blast radius of individual KMS 	- define data classification levels to keys, atleast 1 KMSK per lvl

* security audits and tracking usage patterns in S3 	- restriced permissions
* mobile chat app authenticate and authorize 		- Amazon Cognito, IAM roles, policies & AWS mobile CDK
* S3 bucket ACL 									- Bucket logging, bucket readable for static hosting
* Minimize security breaches accidental info disclosure 	- use encryption, aws permissions
* Amazon Inspector assessment tag findings with urgency  	- add key-value custom attribute to each finding
  
### Guardduty
* SSH brute force attack 					- Guardduty UnauthorizedAccess:EC2/SSHBruteForce
* malicious actor used API access keys 	- Guardduty UnauthorizedAccess:IAMUser/InstanceCredentialExfiltration
* GuardDuty prevent false alerts 			- use ElasticIP, GuardDuty side add in trusted IP address list
* GuardDuty cannot access DNS queries		- GuardDuty does not see these DNS requests
* GuardDuty Suppress the findings			- Auto Archive the finding


### ECS
* ECS vulnerable images monitoring 				- Amazon Inspector to scan ECR
* ECS attacked via port 5353,identify,stop?		- VPC flow logs 
* DDoS, SYN/ACK, UDP, reflection attacks and DNS query flood 			- R53 has by default
* S3 encryption to protect existing objects 	- Switch On Encryption, setup S3 batch job
* RDS sned metrics to CW frequency 		- every 1 min default

* AWS hosted app to use existing AD 		- AD Connector directory @ AWS 
* Access AWS resources Dev and PROD 		- IAM role to delegate access to the resources in the PROD and DEV accounts
* unused permissions across the IAMgroups	- Login Iam Console,access adviser in User Groups
* Amazon Inspector 						- pre-defined rules, packages, Assessment Templates, rules etc for Instances to test

* DDoS attack identify how n/w traffic & IP add how 		- VPC flow logs
* stolen root acount prevent unauthorized access 			- aviod root, iam roles
* AWS Systems Manager executre commands remotely 			- Run Command
* DyanmoDB table encryption 								- Use KMS ; AWS VPC endpoint for InTransit

### Lambda
* Lambda & S3 permissions in same account - Assign Lambda to use S3
* Lambda depcecated runteime				- Trusted Advisor security dashboard 
* Lambda misused to check 				- Lamba execution role 
* Lambda to use KMS 						- Lamda should have kms:Decrypt, KMS policy should have allow permissions
