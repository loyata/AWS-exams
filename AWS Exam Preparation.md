## PART 1: Classic Managed AWS

### Overview

Hierarchy

- Region
- Availability Zone, each region usually 3, min 2, max 6, AZs are connected with high bandwidth and low latency bbut functioned separated from each other in case of disasters
- Data Center
- Edge Locations, help to deliver to end users with lower latency

Region Choice

- data compliance with government
- proximity to customers to reduce latency
- region with available services
- pricing

Global Service: IAM, Route 53, CloudFront, WAF

Region-based Service: EC2, Elastic Beanstalk, Lambda, Rekognition

### IAM

A global service

Root user is created with the account and should not be used. A administrator IAM user should be created, and the group should be given **AdministratorAccess** policy.

Users can be grouped, but group cannot contain other groups. Users don't have to belong to a group, can can belong to many groups.

Privilege Principle: don't give more permissions than a user needs.

IAM: User, Groups(teams, users) and Roles(machines). You apply permissions using policies in JSON to users, groups and roles. 

IAM policy is inheritable 
- Version
- Id
- Statement
  - Sid
  - Effect **(Allow/Deny)**
  - Principle **(account/user/role this policy applies to)**
  - Action**(List of API calls)**
  - Resource**(target of API calls)**
  - Condition**(only if ...)**

Policies applying to a user directly is an inline policy. Policy can be created under IAM/Policy.

Password policies include specific character type, allow IAM users to change password, require to change after some time, prevent re-use.

MFA = password + security devices

IAM Federation is using SAML standard to use your IAM solution for example LDAP active directory.

Three ways to access AWS

- AWS management console(password + MFA)
- AWS command line interface(CLI) , for command line(access keys)
- AWS software developer kit(SDK) , for code (access keys)

AWS CLI is built on Python

Access Keys create: IAM/Access management/Users/Security credentials, can only be downloaded once.

To use CLI:
```shell
# type ID ACCESSKEY REGION
aws configure
# list all users
aws iam list-users
```

CloudShell, no need to log in, default region is the current region

IAM roles is for **AWS services** to perform actions, like **EC2 and Lambda**

IAM security tools

- Credential report (account-level), all users are listed
- Access Advisor (user-level), permissions 

Shared Responsibility Model

- AWS: Infrastructure, Configuration and compliance
- AWS Customer: Users, group, roles management; MFA; password; permissions

One physical user = One AWS user

IAM Policy Simulator: check if a certain action is allowed under policies.

### EC2

Elastic Compute Cloud, IaaS, AZ locked

Related: EBS, ELB, ASG

Configuration:

- OS: Linux, MacOS, Windows
- Compute powers: CPU
- random-access memory: RAM
- storage: EBS, EFS, EC2 Instance Store
- Firewall: **security group**
- Bootstrap: **EC2 User Data** (run a script at first start to install/download )

<img src="http://www.ruanyifeng.com/blogimg/asset/2017/bg2017072301.jpg" alt="img" style="zoom: 50%;" />



<img src="http://www.ruanyifeng.com/blogimg/asset/2017/bg2017072306.png" alt="img" style="zoom: 50%;" />

EC2 are visited via **public IP**

Stop Instance: temporary shutdown;
Terminate Instance: permanent shutdown;

Instance Type: m5.2xlarge

- m: instance class
- 5: generation
- 2xlarge: size

compute optimized: high performance
- batch processing
- transcoding
- computing
- ML
- game

memory optimized: fast process, large data in memory
- high performance sql/no-sql db(twice)
- distributed web scale cache store
- business intelligence db
- real-time big unstructured data

storage optimized: high w/r to large data on local storage
- OLTP
- sql/no-sql db(twice)
- Cache for in-memory db
- data warehouse
- distributed file system

Security groups **contains allow only**, can refernce IP or security group, regulates **IP range, port, inbound, outbound**. Instance and security groups are m/m. Locked to region/VPC.  **Timeout issue is caused by EC2 security group.** Connection error means application error or not launched. Inbound auto blocked, outbound auto authorised.

22 SSH/SFTP, 21 FTP, 80 HTTP, 443 HTTPS, **3389 RDP(windows)**

To log into EC2 instance:
```shell
chmod 0400 <credential.pem>
ssh -i <credential.pem> ec2-user@<publicIP>
```

Never enter IAM credentials inside EC2 instances, instead using IAM roles.

| Type                  | Description                                                  | Price |
| --------------------- | ------------------------------------------------------------ | ----- |
| On demand             | pay for what you use, expensive, short-term and uninterrupted workloads | 1     |
| Reserved              | reserve an instance , stable usage, 1 year or 3 year, use case: run a server for a year; Convertible Reserved Instance can change type, with less discount | 0.3   |
| Savings Plans         | certain type of usage, beyond pay on-demand                  | 0.3   |
| Spot instances        | biding, failure tolerable                                    | 0.1   |
| Dedicated Hosts       | reserve an instance + own software license                   | >=1   |
| Capacity Reservations | reserve some capacity with access                            | 1     |

EBS(Elastic Block Store): Network Drive, can only mount to one instance at a time, except for EBS multi-attach(one instance can have many), **AZ locked**, provisioned capacity, not necessarily attached.

Deletion: By default, root EBS is deleted when EC2 terminates, others are not deleted.

Snapshots: used for backup, can be copied to other region/AZ. Snapshots can be moved to storage tiers(archived) which are 75% cheaper, and takes 24-72 hr to recover. Recycle bin(1 day - 1 year) can be used to avoid accidentally deletion.

AMI(Amazon machine image): built form current EC2, customized EC2. own software + faster boot, **region locked**, a marketplace available.

EC2 Instance Store: hardware, high-performace, better I/O(buffer/cache), die with EC2, risk of hardware faillure.

| volumn  | performance              | boot | max IOPS | max size |
| ------- | ------------------------ | ---- | -------- | -------- |
| gp2/gp3 | general                  | yes  | 16k | 16T      |
| io1/io2 | highest, high i/o, EBS multi-attach | yes  | 1/2: 64k; express: 256k | 1/2: 16T; block express: 64T |
| st1     | frequently accessed, throuhput: 500MB/s | no   | 500 | 16T |
| sc1     | less frequently accessed, throuhput: 250MB/s | no   | 250 | 16T |

EBS multi-attach: io1/2, higher application availability or concurrency.

EFS(Elastic File System): can be mounted on many EC2, One zone or **multi-AZ**, expensive, **only Linux**(POSIX), no provisioning, EFS-IA(infrequent access) tier for cost-saving.
use case: share files betweem EC2 in different AZs

RAID 0: for better IO; RAID 1: backup for more availability

### ELB + ASG

vertical scalability: increase the size of the instance(non distributed, e.g. database)
horizontal scalability: more instances(e.g. ASG)
high availability: running in >2 AZs, in case the loss of certain data center

Health Check: check if a instance ELB is forwarding traffic is available to reply to request,  health check is done on a port and a route, if response is not 200, it is not healthy.

  - CLB(classic) 

    - HTTP, HTTPS, TCP, SSL     

    - **outdated**
    - fixed hostname
    - a security pattern is setting the instance's security group inbound rule only allows HTTP request from the CLB, instances can cross AZ
  - ALB(application) 

    - HTTP, HTTPS, WEBSOCKET
    - balance across machines(target groups) or on the same machine(**containers/docker**)
    - suitable for **microservices**
    - routing based on **path** in URL or **hostname** in URL or **query strings** + **HTTP header, HTTP method, source IP**
    - target groups: EC2 instances(HTTP), ECS tasks(HTTP), Lambda(HTTP->JSON), IP addresses(private), ALB
    - fixed hostname
    - ELB uses private IP to communicate with target group, true ip of the client in the header **X-Forwarded-For**, port X-Forwarded-Port; protocol X-Forwarded-Proto
    - multiple listener, multiple target groups
  - NLB(network) 

    - TCP,UDP,TLS
    - handle great TCP/UDP traffic
    - lower latency
    - **one static ip per AZ**, helpful for whitelisting ip, but **also provide a DNS name**
    - target: EC2, IP addresses, ALB
  - GWLB(gateway) 

    - IP, layer 3
    - Forward traffic to third-party applications for analysis and return, forward to target groups
    - = Transport Network Gateway + Load Balancer
    - Geneve protocal, port 6081

Sticky session ensures traffic for the same client is always redirected to the same target (e.g., EC2 instance). This helps that the client **does not lose his session data**. Works for CLB, ALB. Cookie used for stickiness may contain a expiration date. Shortcoming is possible imbalance. There are **application-based cookies**(**custom cookie** generated by the app, **must not contain AWSALB, AWSALBAPP, AWSALBTG**; or **application cookie**, generated by ELB named AWSALBAPP), **duration-based cookies** generated by ELB, named AWSALB/AWSCLB.



If Cross Zone Balancing enabled, each load balancer distributes evenly across all registered instances in all AZs. Otherwise, distributed according to the node of load balancers. ALB(default on, cannnot turn off, no charge), NLB(default off, charge), CLB(default off, no charge).

<img src="/Users/alex/Library/Application Support/typora-user-images/image-20220717011841798.png" alt="image-20220717011841798" style="zoom:33%;" />

SSL(secure socket layer) is used for in-flight encryption, TLS(transport layer security) is a newer version of SSL. Public SSL certificates are issued by Cerificate Authorities, they have an expiration date that must be renewed(X.509). Load Balancer will check the certificate. The certificate can be managed using ACM(AWS Certificate Manager), or uploaded by AWS client, but there must be a default certificate. SNI(Server Name Indication) helps to solve the problem of loading multiple certificates onto one web server, client must indicate the hostname. SNI only works for ALB, NLB. For CLB, multiple certificates means multiple CLB.

 HTTPS encryption https://zhuanlan.zhihu.com/p/43789231
Extension: Why HTTPS needs a CA?
Symmetric encryption: secret key may be intercepted
Asymmetric encryption: One pair of public and private keys can only solve one-way encrypted communication, two pairs are too costly to compute
Symmetric + Asymmetric encryption: Bob generates a bunch of public and private keys, gives the public key to Alice, Alice generates the secret key X, encrypts it with the public key and gives it to Bob, Bob decrypts it with the private key and gets X, then uses symmetric encryption. But there is still man-in-the-middle attack. The man-in-the-middle Evil can replace the public key with his own.
So the problem is how to verify that Bob's public key belongs to Bob. Bob is the equivalent of a website. Before a website uses HTTPS, it needs to apply for a digital certificate from CA, which contains the certificate holder information, public key information, etc. The client holds the CA's public key. The CA will encrypt the hash value of the certificate with the private key and give it to the client together with the certificate. After the customer decrypts the certificate with the public key, the certificate will be hashed and compared. AWS Certificate Manager helps to manage certificates.

Connection Draining/Deregistration delay, ELB will stop sending requests to EC2, default 300s, (1-3600)



Use auto scaling to automatically register new instances to the load balancer. Scale out is adding instances and scale in removing. Users need to define auto scaling groups ASG and the balancer will use it to register new instances.

ASG needs **a launch template**(including AMI, User Data, EBS volumns, security group, SSH key, IAM role, network, load balancer), **size(min/max/initial)** and **scaling policy**. Scaling can be triggered by **CloudWatch alarms**(**CPUUtilization, RequestCountPerTarget, Average Network In/Out** and customized).

There are 3 types of dynamic scaling policy:

- Target tracking scaling: to keep a certain CloudWatch metric
- Simple/Step Scaling: +2, -1 unit, based on CloudWatch Alarm
- Scheduled Actions: based on usage patterns, date/time

Predictive Scaling: ML, continous forecast.

Cooldown: Stoping launching new instances, default 300s.

If you create ASG for you instances, with a min number if you terminate them, the AGS will provision a new one to meet the min req. **ASG will terminate unhealthy instances.**

Load balancers and ASG are under the menu in EC2. First you need to create a launch template and then the ASG. The template is to create new EC2 instances, so the config is the same as for creating an EC2 instance, this way you enforce that all of them are the same type. When creating a ASG it is recommended that you go to advanced details and select load balancing so it works with the ELB, just select the target group previously created for the ELB. You can reuse the health check for ELB. You can attach and detach instances to the ASG, it is best practice that all ec2 instances are part of an ASG and ASG are part of target group for the ELB. So load balancers and ASG work together.

EC2 instances can add an inbound rule with ALB's security group to ensure only the ALB can access the EC2 instances.



### RDS

RDS stands for Relational Database Service, supported DB includes **Postgres, MySQL, MariaDB, Oracle, Microsoft SQL Server, Aurora(Amazon).** Cannot SSH into RDS instances. Backups are automatically enabled, a daily full-backup and transaction logs every 5 min, so DB can be restored to any time(**> 5 min ago**), retention is 7 days, 35 days optional. Snapshots are triggered by used for any retention. RDS supports **Storage Auto Scaling**, only need to set a maximun storage threshold, happens only if storage < 10% + 5 min low storage + 6 hour since last modification, very handy for unpredictable workloads.

Read Replicas can be created same AZ, another AZ or another Region(with a fee), max 5, **Async**, SELECT operation only, use case is running new workload and do not affect the app or boost read capability. Multi-AZ is to backup and increate availability, **Sync**, auto failover, use case Disaster Recovery, **keep the same connection string**. From Single-AZ to multi-AZ there is no downtime operation, internally a snapshot is taken, and a backup DB in another AZ is restored from the snapshot.

RDS has at-rest encryption with AWS KMS **AES-256**, only fitst create the DB. If the master is not encrypted, the read replicas cannot be encrypted. Transparent Data Encryption (TDE) available for Oracle and SQL Server. For in-flight encryption use **SSL certificates**.

- PostgreSQL: rds.force_ssl=1 

- MySQL: Within the DB: GRANT USAGE ON *.* TO 'mysqluser'@'%' REQUIRE SSL;(require ssl to all users)

Encrypt an un-encrypted RDS database: 1) snapshot un-encrypted DB 2) copy the snapshot 3) encrypt copied snapshot 4) restore an encrypted DB from 3), migrate app, delete the old DB. 

RDS DB normally deploys within privite subnet, security groups: which IP/port/security group can communicate with RDS; IAM policies: who can manage RDS. IAM Authentication only works for MySQL & PostgreSQL, no password needed, will issue a 15 min token.

Aurora support **MySQL& PostgreSQL**, storage (10 GB increment to 128 TB), can create **15 replicas**. High availability is native, total **6 copies aross 3 AZs**, only one **master**, Replication(cross-region) + Self Healing + Auto Expanding.

<img src="/Users/alex/Library/Application Support/typora-user-images/image-20220718115151239.png" alt="image-20220718115151239" style="zoom:33%;" />



ElastiCache is to get managed in-memory databases(Redis/Memcached) with really high performance, low latency, helps reduce load off of databases for **read intensive** workloads. Using ElastiCache involves heavy application **code changes**. 

Architecture 1, DB Cache: app queries ElastiCache, if not hit, get data form RDS and store in ElastiCache. Cache must have **an invalidation strategy** to ensure the most-current data.

Architecture 2, User Session Store(**use case: login**): • User logs into any of the application • The application writes the session data into ElastiCache • The user hits another instance of our application • The instance retrieves the data and the user is already logged in.

Redis has **multi-AZ** with failover, **Read Replicas** for HA, can **Backup and Restore**, and data is durable. Memcached is for sharding with multi-node, **no HA, no persistent, no backup and restore**.

Caching strategies:

**Lazy loading/Cache-Aside**: process is same as DB Cache. Pros: only requested data is cached, no failures are not fatal. Cons: **Cache miss(read) results in latency(read penalty); cached data may be outdated**.

**WriteThrough**: add or update cache when database is updated(write). Pros: **No outdated data.Write causes latency.** Cons: Certain data will not be in the cache until added/updated in the DB. Can combine with Lazy Loading. Also a lot of data never be read.

To evict data from cache, 1) delete manully 2)LRU 3) **TTL**

ElastiCache Security: **Do not suport IAM authentication**, can use **Redis Auth** to enforce input of password, Memcached supportes SASL-based auth.

ElastiCache Replication has two modes: Cluster Mode Disabled(scale reads), 5 replicas, async, master for w/r, other nodes read-only, one shard(all notes have all data), multi-AZ enabled.   Cluster Mode Enabled(scale writes): Data is partitioned,each shard has up tp 5 replicas, multi-AZ enabled.



### Route 53

DNS is a domain name system that translate hostname to ip address.

```
The query process(a.b.com.   IP 9.10.11.12)
1. The web browser queries local DNS server(managed by a company or ISP)
2. local DNS server -(a.b.com.)> root server
3. root server -(com.)> local server
4. local server -(a.b.com.)> TLD server
5. TLD server -(b.com.)> local server
6. local server -(a.b.com.)> SLD server(b.com.)
7. SLD server -(ip)> local server

recursion: user <-> local DNS server
iteration: local DNS server <-> root, TLD, SLD
```

Route 53 is a managed Authoritative DNS also a Domain Registrar. Health Check available.

- domain name: example.com
- record type : A/AAAA/CNAME/NS
  - A: hostname(Record Name) -> IPv4(Value)
  - AAAA: hostname(Record Name) -> IPv6(Value)
  - CNAME: hostname(Record Name, **must not be the top node**) -> hostname(Value)
  - NS: Name servers for the hosted zone(your domain name)
- value: 12.34.56.78
- routing policy
- TTL: how long the result cached at DNS resolvers, high ttl: outdated, low ttl:expensive

Hosted Zone:  how to route traffic to a domain and its subdomains. public(route on the internet), private(route **within VPC**)

Under Cloudshell, **nslook** and **dig** command can be used to see records.

CNAME: Points a hostname to any other hostname. (app.mydomain.com => blabla.anything.com) • ONLY FOR NON ROOT DOMAIN (aka. something.mydomain.com) 

Alias: Points a hostname to an AWS Resource (app.mydomain.com => blabla.amazonaws.com) • Works for ROOT DOMAIN and NON ROOT DOMAIN (aka mydomain.com) • Free of charge • **Native health check** • cannot specify TTL • ELB, CloudFront, API Gateway, Elastic BeanStalk, S3 Website, VPC endpoint, **but not an EC2 DNS name**, can be achieved by CNAME as well

Health Checks enable automated DNS failover, integreated with CloudWatch metrics. Health checks can monitor an endpoint(application, server, other AWS resource), other health checks(called calculated heath check) or CloudWatch Alarms. 

About **15** global health checkers will check the endpoint health. The interval is default 30s(can set to 10s, 30 or 10), support HTTP/HTTPS/TCP, if > 18% report healthy(returns 2XX 3XX status code), then healthy.Health Checks can be setup to pass / fail based on the text in the first 5120 bytes of the response. **Must allow incoming requests(e.g. in security groups) for health checks.**

Calculated Health Checks combine the result with OR, AND, NOT, max 256 child health checks, can specify how many of the health checks need to pass to make the parent pass. The use case is to perform maintenance to your website without causing all health checks to fail.

Route 53 health checkers are outside VPC and cannot access private endpoints. The solution is creating a CloudWatch Metric and associate a CloudWatch Alarm, then create a Health Check that checks the alarm itself.

Routing policy

- simple
  - can specify multiple values in the same record
  - if return multiple values, client will choose one randomly
  - no health check
- weighted
  - allocate according to weight, if one assigned 0, stop sending to it; if all assigned 0, returns equally.
  - health check
- latency
  - redirect to the resource that has least latency to us
  - health check
- failover
  - a primary/secondary, combined with a health check
  - automate failover
- geolocation
  - based on user location
- geoproximity
  - bias(-99~99) to shift traffic to resources
- traffic flow
  - Simplify the process of creating and maintaining records in large and complex configurations
  - a visual editor
  - Configurations can be saved as **Traffic Flow Policy**, can be applied to different Route 53 Hosted Zones and support versioning
- multi-value
  - use when routing traffic to multiple resources
  - health check, up to 8 return
  - not suitable for ELB

You can use another DNS service to manage your DNS records, e.g.  purchase the domain from GoDaddy and use Route 53 to manage your DNS records. **Domain Registrar != DNS Service**

1. Create a **Public** Hosted Zone in Route 53  
2.  Update NS Records on 3rd party website to use Route 53 Name Servers



### VPC(Virtual Private Cloud)

VPC is a **region** level private network to deploy your resources. Subnets allow to partition inside VPC, and are defined in the **AZ** level. One AZ can have several subnets. Public subnets can be accessed from the internet, while private subnets not. VPC has a CIDR range. Route tables needed for communication between VPC/Internet  or  Subnet/Subnet.

<img src="/Users/alex/Library/Application Support/typora-user-images/image-20220718175150861.png" alt="image-20220718175150861" style="zoom: 33%;" />

Internet gateways helps VPC instances connect with the Internet. Public Subnets have a route to the internet gateway. NAT Gateways (AWS-managed) & NAT Instances (self-managed) allow your instances in your Private Subnets to access the internet while remaining private. NAT is depolyed in public subnets. We create a route from the private subnet to the NAT instance.

<img src="/Users/alex/Library/Application Support/typora-user-images/image-20220718175731412.png" alt="image-20220718175731412" style="zoom:25%;" />

Network ACL is a **firewall** controlls traffic from and to the subnet with **Allow and Deny** rules. It is attached at the **Subnet level**, rules only includes **IP addresses**, and stateless. Security Groups are firewalls controls traffic from and to **EC2/ENI(Elastic Network Interface) instances**,  only **allow**, stateful. VPC/Subnet/ENI Flow Logs help to monitor and troubleshoot VPC connectivity issues.

VPC peering connects two VPCs even in 2 different regions, must not have overlapping CIDR, and the connection is not **transitive**(A-B, B-C, A!-C). VPC endpoints allow **private** access from within VPC to AWS services. Two services available for VPC endpoints are S3 and DynamoDB. There are 2 ways to connect from on-premise data center to VPC, first is Site to Site VPN, connection encrypted automatically, goes over public internet, another is Direct Connect, a physical private connection goes over a private network(take a month to set up). Both of them do not access VPC endpoints.

<img src="/Users/alex/Library/Application Support/typora-user-images/image-20220718183041277.png" alt="image-20220718183041277" style="zoom: 33%;" />



LAMP Stack: Linux + Apache + MySQL + PHP + ElastiCache + RDS





### S3

Amazon S3 allows people to store objects (files) in “buckets” (directories), buckets must have a globally unique name. Naming convention: No uppercase, no underscore, no IP, 3-63, start with lowercase or number. Objects (files) have a Key, is composed of prefix + object name. s3://my-bucket/<span style="color: orange">my_folder1/another_folder</span>/<span style="color: green">my_file.txt</span>. 

Object max size is **5GB**, if exceeds, must use **multi-part upload**. Multi-part upload is recommended for file > 100MB, can help speed up transfers. Another way to speed up transferring files is  **S3 Transfer Acceleration**(underlying transfer file to AWS edge location, can integrate with Multi-part upload). **S3 Byte-Range Fetches** can help to speed up downloads and retrieve partial data(e.g. head of a file, first bytes...)

To retrieve less data, use **S3 Select**(underlying server side filtering), can filter by row&columns.

It is a good practice to version the buckets in case of accidently deletion and easy rollback. Same key overwrite will increment the version(1 -> 2). Any file that is not versioned prior to enabling versioning will have version “null”. Deleting an unlisted item is not true delete, it just add a deletion marker. To undo, just delete the deletion marker. Deleting a deletion marker or a specific version ID is permanent deletion. 

There are 4 methods of encrypting objects in S3 

- SSE-S3
  - encrypts S3 objects using keys handled & managed by AWS 
  - HTTP/HTTPS
  - server-side
  - AES-256
  - header: “x-amz-server-side-encryption": "AES256"

- SSE-KMS
  - leverage AWS Key Management Service to manage encryption keys 
  - HTTP/HTTPS
  - KMS Advantages: user control + audit trail
  - server-side
  - header: “x-amz-server-side-encryption": ”aws:kms"

- SSE-C

  - when you want to manage your own encryption keys 

  - **HTTPS only**
  - key must in the header
  - server-side

- Client Side Encryption 
  - client encryption before upload

User-based security: IAM
Resource-based security: Bucket Policy and Object Aceess Control List(ACL)

an IAM principal can access an S3 object if

- the user IAM permissions allow it OR the resource policy ALLOWS it
- there’s no explicit DENY

Bucket policy uses JSON based policies 

- Resources: buckets and objects

- Actions: Set of API to Allow or Deny

- Effect: Allow / Deny

- Principal: The account or user to apply the policy to

The policy can help to grant public access to the bucket, **force encryption** or cross account access.

To force encrytion, apart from using bucket policy to refuse any API call to PUT an S3 object without encryption headers, another way is to use **default encryption**.(will only operate on un-encrypted files, will not chage encrypted files)

"allow anyone do getObject from bucket examplebucket"

<img src="/Users/alex/Library/Application Support/typora-user-images/image-20220718195325808.png" alt="image-20220718195325808" style="zoom:25%;" />

To prevent company data leaks, there are bucket settings for **blocking public access**. S3 supports VPC endpoints. For user security, MFA can be enabled or use **Pre-signed URLs(keyword: valid for a limited time)**. Presignened URLs are generated by SDK(upload) or CLI(download). Valid for **3600s**, and adjustable.

**S3 access logs** can be stored in other S3 bucket(Amazon S3/Buckets/Properties/Server access logging), API calls can be logged in AWS CloudTrail. Do not log into current bucket, or will lead to **a logging loop**. 

S3 can host **static websites** and have them accessible on the www, if get a 403 (Forbidden) error, make sure 1) turn of blocking public access 2) the bucket policy allows public reads

**CORS**: An origin is a scheme (protocol), host (domain) and port. The requests won’t be fulfilled unless **the other origin** allows for the requests, using CORS Headers (ex: Access-Control-Allow-Origin), use a json file, "AllowedOrigins": ["xxx"]

Amazon S3 is strongly consistent, after PUT/DELETE any subsequent read immediately receives the lastest version of the project, and subsequent list request immediately request changes.

S3 MFA-Delete can only enabled by **root account** using **CLI** after versioning is enabled. You will need MFA to **permanently delete an object** version or **suspend versioning** after MFA-Delete. No MFA needed to enable versioning or list deleted versions.

To use S3 Replication, must **enable versioning**, buckets can be in different accounts, **asynchronous**, and S3 should have **IAM permissions**. Cross Region Replication(CRR) is used for compliance and low latency; Same Region Replication(CRR) is used for log aggregation and live replication between production and test accounts. After activating, only new objects are replicated, **S3 Batch Replication** can copy exsiting objects. Delete markers are replicated **optionally**, but deletions with version ID are not replicated. Deletion is not chained.

Durability: Ability to avoid data lost
Availibility: Ability to present readily data

- standard - general purpose
  - frequently accessed data
  - low latency
  - big data, game...
  
- standard infrequent access
  - less frequently accessed but need rapid access
  - cheaper
  - backup and disaster recovery
  
- one zone infrequent access
  
  - extreme high durability
  
  - storing second backup

All glacier classes pays for **storage cost + retrieval cost**, archiving, backup

- glacier instant retrieval
  - very quick retrieval
  - min storage 90 days

- glacier flexible retrieval
  - longer retrieve time,**free** Expedited (1 to 5 minutes), Standard (3 to 5 hours), Bulk (5 to 12 hours)
  - min storage 90 days

- glacier deep archive
  - longest retrieval time Standard (12 hours), Bulk (48 hours)
  - min storage 180 days
  - cheapest

- intelligent tiering

  - no retrieval fee

  - monitoring cost apply

Storage Cost: Standard > Standard-IA > One Zone-IA > Glacier Instant Retrieval > Glacier Flexible Retrieval > Intelligent - Tiering > Glacier Deep Archive

Moving objects can be automated using **a lifecycle** configuration. 

Transition actions: It defines when objects are transitioned to **another storage class**. 

Expiration actions: configure objects to expire (**delete**) after some time. Use case: Delete Log files, old versions of files, imcomplete uploads.

S3 can achieve 3,500 PUT/COPY/POST/DELETE and 5,500 GET/HEAD requests per second per prefix in a bucket.  **SSE-KMS may impact the S3 performance.**

**S3 Event Notification** perform tasks when events happen(e.g. ObjectCreated/Removed/Restore/Replication), with name filtering(*.jpg), unlimited number, and deliver results to SNS/SQS/Lambda.Use case: generate thumbnails.

 **EventBridge provides advanced filtering with JSON rules, and send to more destinations.**

**Athena** is a serverless service to perform **analytics** against **S3 objects** with **SQL** language.

### CLI & SDK

--dry-run can be used to simulate API calls **without actually performing the command**, test if user/role have the permission. If the permission is right, CLI will get DryRunOperation error.

STS helps to decode error message.(IAM-DecodeAuthorizationMessage needed)

```shell
aws sts decode-authorization-message --encoded-message <encoded_msg>
```

EC2 instance MetaData can be retrieved from URL http://169.254.169.254/latest/meta-data. No IAM role needed. You can retrieve the IAM Role name from the metadata, but you cannot retrieve the IAM Policy.  Metadata = Info about the EC2 instance; Userdata = launch script of the EC2 instance

```shell
aws configure --profile <other_account> # Switch to different account in CLI
cat config # show account info
```

To use MFA with the CLI, must create a temporary session with **aws sts get-session-token**

SDK supports Java/.Net/Node.js/PHP/Python/Go/Ruby/C++, if a region is not specified, the default is us-east-1

When we exceed a API call limit(throttling), temporary solution is using **Exponential Backoff**(5XX error only), permanent solution is requsting an **API throttling limit increase**. When we exceed Service Quotas, we can **open a ticket** or use **Service Quotas API**

In CLI, CLI will look for credentials in the order of Command Line Options, Environment variables, credential file, configuration file, container credential , Instance profile credential(IAM)
In CDK, CDK will look for credentials in the order of Java System properties, Environment variables, credential file, container credential, Instance profile credential(IAM)

When using AWS HTTP API, request should be signed, some request to S3 don't need to be signed, and SDK,CLI sign HTTP for you, otherwise should use SigV4. If using working within AWS, use IAM Roles, **if working outside AWS(e.g. on-premise), use environment variables / named profiles**.



### CloudFront

CloudFront is a content delivery network(CDN), **content is cached at the edge**, can protect **DDoS** and integrate with AWS Web Application Firewall.

One origin is S3 Bucket, **Origin Access Identity**(OAI, an IAM role) ensures S3 allows only communication from CloudFront. CF can also serve as ingress to upload files.

Another origin is HTTP endpoint(e.g.ALB, EC2, S3 Website)

 Process
  - client send HTTP requests to edge

  - edge forward query strings/headers to origin

  - origin response

  - edge store caches

    

Edge communicates with S3 under private AWS(OAI eusures security). When communicating with EC2, EC2 must allow IP of edges, and must be public. When communicating with ALB, ALB must be public and allow edge IPs, but EC2 instances can be private.

CloudFront can whitelist or blacklist certain countries.

Comparing with S3 CRR: CloudFront is a Global Edge Network with TTL, great for a **static content** that must be everywhere. S3 CRR must set for each region, read-only, quick update, and great for **dynamic content** that needs to have low-latency at certain regions.

Caching is based on **header, cookies and query strings**, TTL between 0 sec to 1 year. You can invalidate part of the cache using **CreateInvalidation** API. A commom strategy is to separate static and dynamic requests. 

For CloudFront Security, **Viewer Protocol Policy** is between client & edge, redirect HTTP/HTTPS, or use HTTPS only;  **Origin Protocol Policy** is between edge and origin, HTTPS only or match viewer. S3 website supports HTTP only.

CloudFront Signed URL/Cookies is used to share restrictive content(**keyword: distribution**). one URL grants access to one file, wheras one cookie grants access to many files. CloudFront Signed URL allow **access to any path**(S3, HTTP endpoint...), can only managed by **root user,** and filter by IP, path... and use caching features; S3 URL grants permission as the people who signed the URL, uses IAM, and with limited lifetime. If CF is used, must use CF URL.

The signers of CF URL can be a trusted key groups or an account contains a CF Key Pair. Key pairs are generated by AWS client self, private key to sign the url, public key to verify.

<img src="/Users/alex/Library/Application Support/typora-user-images/image-20220719193804459.png" alt="image-20220719193804459" style="zoom: 33%;" />

**CloudFront Origin Groups:** auto do failover from primary group to secondary group.

**CloudFront Field Level Encryption** add additional layer of security along with HTTPS, sensitive information encrypted at the edge close to user.(e.g. Credit Card info), Async encryption.



### ECS, ECR, Fargate

**Amazon ECS**(Elastic Container Service) is a container platform for container services; **Amazon EKS**(elastic kubernetes service) is Amazon's managed kubernetes(open-source); **Amazon Fargate**, serverless container platform, **Amazon ECR**(Elastic Container Registry) , store container images

ECR is a private Docker repo; dockerHub is public.

VM vs Docker, Hypervisor ==> VMWare, VirtualBox...

<img src="/Users/alex/Library/Application Support/typora-user-images/image-20220719201544404.png" alt="image-20220719201544404" style="zoom: 25%;" />

#### ECS

Hierachy: ECS Cluster - (EC2 Container Instances) - ECS Tasks

Launch Docker images on AWS means **launch ECS tasks on ECS Clusters**. For **EC2 launch type**, you must provision and maintain the infrastructure. One ECS Cluster contains several EC2 instances, and each EC2 instance must run the **ECS Agent** to register the EC2 Cluster. For **fargate launch type**, no need to provision, all **serverless**, task definition only.

**EC2 Instance Profile** is ECS's IAM role. EC2 launch type only. One instance may run multiple EC2 Tasks. EC2 Instance Profile is used by ECS Agent to make API calls to ECS services, send logs to CloudWatch Logs, pull docker images from ECR, or reference sensitive data. Each Task get **ECS Task Role**, defined in **task definition**, apply for both launch types. Use different roles to link to different ECS services(e.g. Tasks A role allows S3, Task B role allows DynamoDB). ALB, NLB supported. NLB is for high throughput and paired with AWS private Link. **EFS** supports both launch types, multi-AZ,  can be used to share files across tasks. S3 cannot serve as ECS's file system.

<img src="/Users/alex/Library/Application Support/typora-user-images/image-20220720104532861.png" alt="image-20220720104532861" style="zoom:33%;" />



Task Definition determines which docker image to use. Tasks need to be deployed to ECS Clusters. ECS can use **AWS Application Auto Scaling** to scale **task numbers**, based on **CPUUtilization, Memory(RAM) Utilization and ALB Request Count per Target.** To scale EC2 Instances, we can use ASG or **ECS Cluster Capacity Provider**.         *ECS Service Auto Scaling (task level) ≠ EC2 Auto Scaling (EC2 instance level)*

ECS Rolling Update has 2 parameters. Minimum healthy check determines how many task can be terminated, maximun percent determines how many new tasks can be created.

<img src="/Users/alex/Library/Application Support/typora-user-images/image-20220720111936757.png" alt="image-20220720111936757" style="zoom: 15%;" />

<img src="/Users/alex/Library/Application Support/typora-user-images/image-20220720111958117.png" alt="image-20220720111958117" style="zoom:15%;" />



ECS tasks can be invoked by EventBridge or combine with SQS.

**Task Definitions** can also be the form of JSON, telling ECS how to run a Docker container. (Image Name, Port Binding(Host(EC2) Port and Container Port), RAM/CPU, Environment Variables, IAM role, CloudWatch). A task definition can define up to 10 containers. 

EC2 Launch Type: **Dynamic Host Port Mapping** helps to solve situations when only the container ports are defined in the task definition, with which ALB knows the right EC2 port. EC2 instances' security group should allow any port.(page 324)

Fargate Launch Type: Each ECS Task has a unique private IP, there is no host. **Host port should be 0 or do not set.** Containers share same port.

IAM are assigned per Task Definition. IAM roles are defined on the task definition.

Environment Variables can be hardcoded inside the Task Definition, or store them inside SSM Parameter Store or Secret Manager, and reference them inside Task Definition. Environment files(bulk files) are stored inside S3, and referenced inside Task Definition.

EFS persistent multi-AZ shared storage for your tasks & containers.  For **the same EC2 task**, **EC2 Instance Storage** share files between containers; for Fargate Tasks, **ephemeral storage**(20G- 200G) share files.(keyword: side car container, share between containers)

Task Placement Strategy and Task Placement Constraints help to determine where to place/terminate new tasks. 

Strategy

- **Binpack** place on the least availble amount of CPU/Memory, minimize EC2 instances in use
- **Random** places randomly
- **Spread** place the task evenly based on value(e.g.AZ)

Constraint

- **distinctInstance** never two task in the same instance
- **memberOf** place task on instances that satisfy an expression

#### ECR

Private repo or public(Amazon ECR Public Gallery) storing docker images, backed by S3

ECR permission error are caused by IAM

```shell
aws ecr | docker login # login CLI
docker pull/push # pull/push from CLI
```

#### EKS

Use case: migrate from on-premises that **using Kubernetes** to AWS

Can be used in any cloud(AWS, Azure, GCP...)

Cluster has service/task, service is running a long-running app, task is handle some jobs

### BeanStalk

BeanStalk is a managed service to deploy an application from a developer centric view. Automatically create EC2, ASG, Security groups, ALB... The underlying is CloudFormation.

Amazon Elastic Beanstalk has two types of environment tiers to support different types of web applications. Web servers are standard applications that listen for and then process HTTP requests, **using ELB**, typically over port 80. Workers are specialized applications that have **a background processing task** that listens for messages **on an Amazon SQS queue**. Scaling is based on the number of SQS messages A worker application that processes long-running workloads on demand or performs tasks on a schedule. Worker Environment can combine with Web Tier, it does periodic tasks in **cron.yaml**.

When setting up a new environment, **the choice of Load Balancer is final.**

#### ![image-20220720183130416](/Users/alex/Library/Application Support/typora-user-images/image-20220720183130416.png)

bucket size: how many instances can be taken down

- Immutable: Deploy on new instances on a new ASG, high cost, quick rollback in case of falure

- Traffic Spliting: **Canary Testing**, deploy to new ASG, quick automated rollback

- Blue/Green: swap environment URLs

We can install an additional CLI called the “EB cli” which makes working with Beanstalk from the CLI easier.

To deploy, dependency files need be created(requirements.txt for Python, package.json for Node.js), package code+dependency file as .zip, and upload .zip file

BeanStalk uses lifecycle policy to delete old versions(max 1000), based on **time**(remove old version) or **space**(when there are too many versions) Option not to delete the source bundle in S3 to prevent data loss.

EB extentions can be used to add some resources(e.g. environment variables), 1) in the .ebextentions/ directory 2) YAML/JSON 3) end with .config 

EB Cloning clones an environment with same configuration, use case : create a test env. ELB, RDS(except data), Enviroment Variables will be reserved.

As ELB is final, to change ELB type need to perform migration, 1) create a new env except for the ELB 2) deploy app 3) CNAME swap

In production environment stage it is a better practice to seperate the RDS from the env in case they have the same lifecycle. To decouple, 1) create a snapshot of RDS DB for backup 2) protect RDS from deletion 3) Create a new environment without RDS, and point the application to current RDS 4) CNAME swap or update Route 53 5) terminate old environment.

EB Environment can run docker. For Single Docker, provide **Dockerfile** or **Dockerun.aws.json(v1)**, ECS not included. For MultiDocker, will create ECS Clusters, EC2 Instances, ALB, Task Definitions, and require **Dockerun.aws.json(v2)** to generate Task Definition.

To enable HTTPS in EB, SSL certificate should be loaded to the load balancer by console or EB extension .ebextensions/securelistener-alb.config. Redirecting HTTP to HTTPS can be either realized by configuring Instances or ALB.

BeanStalk provides Custom Platform for app with not supported language and not using Docker. To create own platform need **Platform.yaml(AMI) // Packer Software**.

### CI/CD

Continuous Integration, Continous Delivery, It is a set of processes to automate the build, test and deployment of software.

![image-20220720212735619](/Users/alex/Library/Application Support/typora-user-images/image-20220720212735619.png)



```
• AWS CodeCommit – storing code
• AWS CodePipeline – automating pipeline from code to Elastic Beanstalk
• AWS CodeBuild – building and testing code
• AWS CodeDeploy – deploying the code to EC2 instances (not Elastic Beanstalk)
• AWS CodeStar – manage software development activities in one place
• AWS CodeArtifact – store, publish, and share software packages(dependency management)
• AWS CodeGuru – automated code reviews using Machine Learning
```

AWS CodeCommit is Amazon's private **Git** repo. It is integrated with CodeBuild. It uses **SSH & HTTPS** for authorization.  IAM roles/users limit the access to repos. Repos are encrypted automatically at rest using KMS, and SSH encrypted in transit. Use IAM to grant cross-account access.Use **AWS SNS/Lambda** to check if there is credential in the commit.

AWS CodePipeline is a visual workflow to manage CICD, from source(e.g. CodeCommit, Github..) -> build (e.g. CodeBuild, Jenkins)-> test(CodeBuild, AWS Device Farm) -> depoly(CodeDeploy, Elastic BeanStalk, CloudFormation....). Each stage will create an **artifact** that stored in an **S3 bucket**, then pass to next stage. Clients can use **CloudWatchEvents** to monitor event(e.g. failed pipelines, cancelled stages). Check the IAM if pipeline doesn't work. CloudTrail can be used to audit API calls. 

Action groups are within stages. A stage can have multiple action groups.

AWS CodeBuild helps to build and test code. Build instruction in **buildspec.yml(must at code root).** Output logs in S3 & CloudWatch Logs. CloudWatch Metrics => monitor build statistics; CloudWatch Events => detect failures + notify; CloudWatch Alarms => notify if too many failures. CodeBuild will fetch code+buildspec.yml from CodeCommit, and fetch a docker image(perpackaged or customized) to run the build.**Optionally build cache(e.g. dependencies) can be stored in a S3 Bucket**. When the build is done, artifacts will go to S3. CodeBuild Agent allows **Local Build**. For access to internal services can build inside a VPC.

buildspec.yml must be at the **root**. 

- env
  - variables:plaintext
  - parameter-store: SSM Parameter Store
  - secrets-manager: AWS Secrets manager
- phases
  - install: install dependencies
  - pre_build: command execute before build
  - Build: actual build command
  - post_build: finishing touches(e.g. zip output)
- artifacts: what to load to S3
- cache: file to cache

```shell
grep -Fq "xxx" index.html # 检查index.html中是否有xxx
```

AWS CodeDeploy: Compute platform is where to run CodeDeploy Agent, including EC2/On-Premise, Lambda and ECS. Each EC2 instance of on-premises server must be running **CodeDeploy Agent, and must be tagged. appsepc.yml** is used for deploy. Internally, the agent is continously polling AWS CodeDeploy for work to do. If triggered, agent will pull code form S3/github and run deployment instructions from appsec.yml. 

The deploy process: 1) create IAM Instance Profile to allow EC2 instance that runs CodeDeploy Agent to have access to S3; create service role for CodeDeploy 2) configure EC2's security group 3) SSH to EC2 and install ruby/CodeDeploy Agent 4) Create Deploy Groups(dev, prod, test...), apply IAM, and set key/value, all EC2 instances that matches tags will be deployed. 4)Create Deployment, specify a S3 bucket that has code+appspec.yml

appspec.yml: 

- files: S3 source
  - source
  - destination
- hooks: operations to deploy
  - BeforeInstall
  - Install
  - AfterInstall
  - ApplicationStart
  - **ValidateService**

```
deployment configuration: 
Deploy to EC2:
One at a time(error then stop)
Half at a time(50%)
All at a time(good for dev)
Deploy to ASG
In-Place: Replace old instances and auto deploy new instances
Blue-Green: Crete a new ASG, ELB only
```

New deployments will first be deployed to failed instances, to rollback, **redeploy old deployments or enable automated rollback**. • If a roll back happens, CodeDeploy redeploys the last known good revision as a new deployment (not a restored version)

AWS CodeStar is an integrated solution that groups: GitHub, CodeCommit, CodeBuild, CodeDeploy, CloudFormation, CodePipeline, CloudWatch. Can Create projects quickly. Support C#, Go, HTMLL, JAVA,Node, PHP, Python, Ruby. Support issue tracking.

AWS CodeAritifacts a artifact management serive(storing and retrieving dependecies), serve as a proxy inside VPC(developer - CodeArtifact - pip/maven/npm), 1) for security 2) caching, in case public repo disappears. AWS CodeBuild can also fetch dependencies from CodeArtifact.

AWS CodeGuru is a ML-based service for automated code reviews(CodeGuru Reviewer) and app performance recommendations(CodeGuru Profiler).



#### CloudFormation

Infrastructure as code => coding AWS's instructures for better migration, reproduction.., estimation of cost based on the CF template. CF templates are uploaded to S3. Once uploaded, they are immutable. To update, you will load a new template, and **CF will find out the difference and proceed the update**.

In CloudFormation, you create stacks from templates. Stack is a single unit for managing related resources. Stacks are identified by a name. **Deleting a stack will delete all resources.** The recommended way to deploy a template is editting templates in a YAML/JSON file and use CLI to deploy.

Templete does not support dynamic resources.

Template YAML:

- key-value pairs, including nested

- \- : it is an arry

- \|: multi-line string

- Component: Resources, Mappings, Parameters, Outputs, Conditions

- ! Ref:  to reference a parameter, can **ref a pre-defined a parameter, or a resource**, there are also pseudo Parameters provided by Amazon that you don't need to create(e.g. **AccountId**, ARN, **Region**, **StackId, StackName**, no AccountName!!!)

- Mappings are fixed variables(predefined, like a dictionary) within CF templates, to differentiate regions, AZs, accounts... (mapping vs ref: mapping is for you know what values would possibly be taken)
  - **!FindInMap[Map Name, TopLevelKey, SecondLevelKey]**
  - ![image-20220721160645789](/Users/alex/Library/Application Support/typora-user-images/image-20220721160645789.png)
  
  - ! FindInMap[RegionMap, ! Ref "AWS::Region", 32]
  
 - Outputs are optional. Export some values and import to other stacks. (cross-stack collabration). A Stack cannot be deleted if refered by others.

   - **! ImportValue** XXX

 - Conditions give a true/false, limit the creation of resources, applied inside the Resources

   - Fn::And
   - Fn::Equals
   - Fn::If
   - Fn::Not
   - Fn::Or

 - **!GetAtt**: Get the attribute of a resource declared in the template(e.g. AZ)

 - !Join[":",[a,b,c]]  => a\:b\:c

 - !Sub: substitution
```yaml
---
# for the whole yaml file, use case: migrate to another company/ some params cannot be defined in advance
Parameters:
  SecurityGroupDescription:
    Description: Security Group Description
    Type: String

# Resources are AWS Components
Resources:
  MyInstance: # name defined by user
    # Specify type
    Type: AWS::EC2::Instance
    # Properties for a single Resource
    Properties:
      AvailabilityZone: us-east-1a
      ImageId: ami-a4c7edb2
      InstanceType: t2.micro
      SecurityGroups:
        - !Ref SSHSecurityGroup
        - !Ref ServerSecurityGroup

  MyEIP:
    Type: AWS::EC2::EIP
    Properties:
      InstanceId: !Ref MyInstance
      
  SSHSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Enable SSH access via port 22
      SecurityGroupIngress:
      - CidrIp: 0.0.0.0/0
        FromPort: 22
        IpProtocol: tcp
        ToPort: 22

  
  ServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: !Ref SecurityGroupDescription
      SecurityGroupIngress:
      - IpProtocol: tcp
        FromPort: 80
        ToPort: 80
        CidrIp: 0.0.0.0/0
      - IpProtocol: tcp
        FromPort: 22
        ToPort: 22
        CidrIp: 192.168.1.1/32
```

Rollback: if creation fails, everything get deleted; if update fails, stack automatically rolls back to the previous known working state.

When updating slacks, **ChangeSets** give a preview what will happen. **Nested stacks** are part of **other** stacks, (e.g. a reusable ALB, SG). **Cross stacks** are used when you export values to many stacks.**StackSets** is used to CRUD stacks across regions and accounts.

**CloudFormation Drift** helps to protect againt manual configuration changes outside CF.
