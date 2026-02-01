## 1. Public & Private Services
![[Pasted image 20260124082407.png]]
+ 3 network zones
+ Called "Public" or "Private" is based on **Networking perspective**
+ IGW is a gate for Private zone to the outside world
+ EC2's **Public IP address** is the way we can access to EC2 from the Internet by **projecting EC2 instance into AWS public zone**
+ On premises (Your company physical servers) can access to Private zone via VPN or Direct Connect

## 2. AWS Regions, Edge Locations and Availability Zones
Term to understand: "Globally Resilient, Regional Resilient and AZ resilient"

+ EC2 Instance in Sydney != EC2 Instance in Virginia
Why these three (Regions, Edge Locations and AZ) exist?
+ Fault isolation
+ Different governance
+ Performance

![[Pasted image 20260124092551.png]]

| Globally Resilient                                                   | Regional Resilient                                                                 | AZ Resilient                                 |
| -------------------------------------------------------------------- | ---------------------------------------------------------------------------------- | -------------------------------------------- |
| + Can **failover** other regions<br>+ No concept of picking a region | + Can **failover** across AZs<br>+ Requires manual setup for cross-region failover | + **Down** when AZ is down<br>               |
| ex: IAM, Route53                                                     | ex: VPC, RDS (Multi-AZ), ALB, EKS                                                  | ex: EC2 (single AZ), EBS volume, NAT Gateway |

## 3. EC2
### Instance lifecycle
![[Pasted image 20260125184823.png]]
- Stopped state does not wipe out memory => EBS is still get charged
### AMI
AMI contains:
+ **Permissions**: Public - Owner only - Explicit account
+ **Root volume**: Drive - This is the **main disk** that the EC2 instance boots from.
+ **Block device mapping**: Determine which volume is **root** and which is **data volume**
## 4. CloudFormation (CFN)
### What is it?
+ CloudFormation is an Infrastructure as Code (IaC) 
+ Allows **automation** in infrastructure creation, update and deletion.
### How it works?
![[Pasted image 20260126195206.png]]
+ CloudFormation add/update/delete physical resources via **Stack**
+ **Stack** is created via **Template**
+ **Template** can be defined in YAML or JSON 
![[Pasted image 20260126122211.png]]

### 5. Cloudwatch
+ CloudWatch provides **metric**, **log** and **event management** services.
![[Pasted image 20260128124917.png]]
+ Services and custom data can be managed through **Metrics**
+ **Metrics** can be **viewed** via Console or API 
+ Metrics can be used to **trigger** **actions** via **Alarm** (action can be Auto scaling ec2, send SNS)

*Term to understand: "Data points, Dimension within CPU Utilization metric"*
+ Datapoint: A data in a specific timestamp
![[Pasted image 20260128130343.png]]
+ Dimension: For that CPU Utilization **metric**, within AWS/EC2 **namespace**, at **Dimension** of InstanceId, the **datapoints** are ... 
![[Pasted image 20260128130305.png]]

### Alarm state
![[Pasted image 20260128133348.png]]
+ 3 states: 
	+ OK
	+ Alarm
	+ Insufficient data: Not enough initialized data for measurement

### 6. Shared responsibility Model
The Shared Responsibility Model - is how AWS provide clarity around *which areas of **systems security** are **theirs***, and *which are owned by the **customer***.
![[Pasted image 20260128140643.png]]

=> What to note out:
*Whenever using a service, keep Shared Responsibility Model in mind - which elements you gonna manage by yourself and which AWS will manage for you*

### 7. High-Availability & Fault Tolerant & Disaster Recovery

#### High-Availability (easy - less costly)
✘ to stop failure
✘ customer never experience outages
✘ about UX, about operating through failure

✔ Providing services **as often as possible**
✔ Service can be fixed **as quickly as possible** (by automation)
✔ ==Maximum system **online time/ uptime**== = minimize the outage

Ex: Go *replace* server entirely instead of fixing it - An ==active - standby== architect (spare architect)
-> When you just want to minimize the downtime
#### Fault-Tolerant (complex - expensive)
✔ FT = HA (minimize outage) + tolerate the failure 
✔ System **should operate normally** **even fault occurred** and is being fixed
✔ ==Operating through failure==

Ex: Life-critical situations (medical, plane). ==active - active== architect
-> When life is crucial 
#### Disaster Recovery (Plan)
✘Design a system to cope, operate through disaster

✔==What to do when disaster occurs== - Pre-planning

Ex: Taking **regular standby/ ==backup==**, in offsite location (other locations, not same site as primary one)
=> Used when above twos don't work

### 8. Route 53
+ A Global resilient service
#### Feat(s):
**8.1 Register domain**
Term to understand: 
"*.com zone has multiple nameserver records"*
*"these records point to amazon servers hosting the amazone.com zone"*
+ **zone** = all DNS records for a domain = table
+ A **nameserver** = server (computer) that stores the DNS zone

*How DNS works?*
![[Pasted image 20260129152700.png]]
+ So to know which IP is netflix.com, we need to locate **Netflix Zone** that hold the record 
	-> That's DNS job
![[Pasted image 20260129153908.png]]
+ Your computer asks a **DNS resolver** - a caching machine, to find an IP for a name. 
+ If not cached, it must walk the tree
+ The resolver **==walks the tree== (root → TLD → domain’s authoritative servers), gets the answer**, then **caches it** so next time is faster.
+ **Root** contains NS record for **.com NS**, and **.com server** contain NS record for **netflix DNS NS**
+ **Netflix DNS NS** will give the final answer
![[Pasted image 20260129152218.png]]
+ Usually, query to netflix.com leads to **another domain name** which leads to **another query**
	-> That's why some servers are very slow 

*How can we register a new domain?*
![[Pasted image 20260129160924.png]]
+ **Domain Registar** - where you ==pay for a domain== (Ex: GoDaddy, R53, Hover)
+ **Registry / TLD operator** (the owner/operator of the TLD, e.g. .com is run by Verisign)
+ **DNS hosting provider** (==provide server that contain **DNS Zone** (for the being-registered domain)== , e.g. Route53 hosted zone, Cloudflare DNS, etc.)
+ ==A domain becomes usable when the **.com registry is told which *authoritative name servers (NS)* host the domain’s zone**==
So: 
+ Pay for a domain - ex: *cats.com*
+ Need a **authoritative NS** for the being-registered domain - ex: *rita.ns.cloudflare.com*
	1. If registrar = DNS host (same company), this may be **auto-created**.
	2. If different companies, you must create it yourself and **inform NS values**.
+ Provide that info to **.com registry**, to get that  name in TLD Zone
+ Now `cats.com` is **live**: resolvers can go root → `.com` → your NS (*rita.ns.cloudflare.com*) → get real records (*cats.com*).

| ==1.2.1.2== | ==cats.com== |
| ----------- | ------------ |
|             |              |
|             |              |
(*==rita.ns.cloudflare.com ==*)

| ==2.1.23.3== | ==rita.ns.cloudflare.com== |
| ------------ | -------------------------- |
|              |                            |
(*.com*)
These 3 things should be define

*How to use Route53 for domain registry?*
1. Register a **domain** 
2. R53 create a zone file (abc.org) - contains all info about that registered domain + 4 NS
3. Put the zone file into 4 NS
4. Add 4 NS to TLD 

**8.2 Hosted Zones**
**8.3 DNS Record Types**
+ NS
+ A(v4) & AAAA(v6)
+ CNAME
+ TXT
+ MX

**TTL** - Time To Live of a record in cache server (Resolver)