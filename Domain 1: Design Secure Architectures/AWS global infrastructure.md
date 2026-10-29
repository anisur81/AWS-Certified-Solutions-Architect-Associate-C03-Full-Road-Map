# AWS Global Infrastructure — Step-by-Step Guide with Details

Learning Objectives
AWS Global Infrastructure is a fundamental topic for the AWS Certified Solutions Architect – Associate (SAA-C03) exam. 
It helps you understand how to design AWS workloads for high availability, fault tolerance, low latency, disaster recovery, and security.

By the end of this guide, you should be able to:
1. Understand the AWS Global Infrastructure Hierarchy
2. Explain AWS Regions and Availability Zones (AZs).
3. Understand AWS data centers and edge locations.
4. Select an appropriate AWS Region for a workload.
5. Design highly available applications across multiple AZs.
6. Understand regional versus global AWS services.
7. Understand AWS Wavelength Zones
8. Understand AWS Outposts
9. Understand AWS Global Network and CloudFront
10. Understand AWS Infrastructure Resilience


## Understand the AWS Global Infrastructure Hierarchy
 
 | Component | Meaning | Primary purpose |
|---|---|---|
| AWS Global Infrastructure | AWS's worldwide infrastructure | Run applications and deliver services globally |
| Region | A separate geographic area containing multiple AZs | Geographic placement and regional resilience |
| Availability Zone (AZ) | An isolated infrastructure location within a Region | High availability and fault isolation |
| Data center | A physical facility containing computing infrastructure | Hosts servers, storage, and networking |
| Edge location | A location used by services such as CloudFront to deliver content closer to users | Lower latency and faster content delivery |
| Regional edge cache | An intermediate caching layer used by CloudFront | Improve cache efficiency for some content |


Important: A Region is not the same as an Availability Zone, and an Availability Zone is not the same as a data center.
An AZ consists of one or more data centers with independent, redundant power, networking, and connectivity. AWS designs AZs to be isolated from failures 
in other AZs while remaining connected within the Region.


<img width="1672" height="940" alt="image" src="https://github.com/user-attachments/assets/3cdaa2a4-c5b4-4b09-84f1-59814db76dc8" />

## What is an AWS Region?
An AWS Region is a separate geographic area in which AWS operates infrastructure and offers services.
For example, AWS has Regions in North America, Europe, Asia Pacific, and other parts of the world.

A Region generally contains multiple Availability Zones. You choose a Region when creating many AWS resources, such as an EC2 instance, an RDS database, or a VPC.

## Why do Regions matter?

1. Geographic proximity
Choose a Region close to your users when appropriate to reduce network latency.
Example: If most of your application's users are in Bangladesh, you might evaluate nearby Asia Pacific Regions such as Singapore, Mumbai, and Hyderabad.

2. Compliance and data residency
Some organizations must keep certain data within a specific country or geographic boundary. You must evaluate applicable laws, contractual requirements, AWS service capabilities, and where data is stored or processed.

3. Service availability
Not every AWS service or feature is available in every Region. Check the availability of the required service and features before selecting a Region.

4. Cost
Pricing can vary by Region. Compare compute, storage, networking, and data-transfer costs before deciding.

5. Disaster recovery
A second Region can help protect against a regional outage. However, you must design and test replication, backups, failover, and recovery procedures.

## How to select an AWS Region
Use this decision process:
1. Identify where your users are located.
2. Determine applicable data residency and compliance requirements.
3. Confirm that the AWS services and features you need are available.
4. Compare expected costs.
5. Evaluate latency and network connectivity.
6. Define your recovery objectives.
7. Test the architecture in the selected Region.

Understand Availability Zones (AZs)
What is an Availability Zone?
An Availability Zone is an isolated location within an AWS Region. It consists of one or more data centers with independent, redundant power, networking, and connectivity.

AWS connects AZs within a Region using high-bandwidth, low-latency networking. This makes it possible to design applications that continue operating when one AZ experiences a failure


                 AWS Region
                ap-southeast-1
                  Singapore
                      |
           +----------+----------+
           |                     |
     Availability Zone A    Availability Zone B
           |                     |
     +-----------+         +-----------+
     | Web Server|         | Web Server|
     | EC2       |         | EC2       |
     +-----------+         +-----------+
           |                     |
           +----------+----------+
                      |
                Application
                   Users


## Why use multiple AZs?
Suppose your application runs on one EC2 instance in AZ A.

If AZ A becomes unavailable, your application may become unavailable.

If you deploy application instances across AZ A and AZ B and configure a load balancer and appropriate health checks, traffic can be directed to healthy instances in the remaining AZ.

Important: Simply launching instances in multiple AZs does not automatically guarantee failover. You must configure load balancing,
application health checks, data availability, and recovery behavior.

| Feature | Single-AZ architecture | Multi-AZ architecture |
|---|---|---|
| Deployment | One AZ | Two or more AZs |
| Availability | Vulnerable to an AZ failure | Designed to tolerate an AZ failure |
| Complexity | Lower | Higher |
| Cost | May be lower | May be higher |
| Suitable for | Learning labs and some noncritical workloads | Production workloads that require high availability |


## Understand Availability Zone Names and AZ IDs
You may see names such as:
- ap-southeast-1a
- ap-southeast-1b
- ap-southeast-1c
These are Availability Zone names within the Singapore Region.

However, an AZ name such as ap-southeast-1a may refer to a different physical AZ in another AWS account. AWS uses account-specific AZ-name mappings in some Regions.
What is an AZ ID?
An AZ ID identifies an Availability Zone consistently across AWS accounts.

### When are AZ IDs useful?
AZ IDs are especially useful when:
- Sharing network resources across AWS accounts.
- Coordinating deployments across accounts.
- Ensuring that resources are placed in the same physical AZ when required.
Exam tip: Remember that an AZ name is not necessarily a consistent physical identifier across different AWS accounts.

## Understand AWS Data Centers
A data center is a physical facility that houses computing equipment and supporting infrastructure.

AWS data centers contain infrastructure such as:
- Physical servers.
- Networking equipment.
- Storage systems.
- Power and backup-power systems.
- Cooling systems.
- Physical security controls.
  
How data centers relate to Regions and AZs

AWS Region
   |
   +-- Availability Zone A
   |      +-- Data Center 1
   |      +-- Data Center 2
   |
   +-- Availability Zone B
   |      +-- Data Center 3
   |      +-- Data Center 4
   |
   +-- Availability Zone C
          +-- Data Center 5
          +-- Data Center 6

  This is a conceptual example, not a representation of a particular AWS Region.

  ## Why does AWS use multiple data centers and AZs?
The design helps isolate failures and supports the deployment of resilient applications.

For example, a workload distributed across multiple AZs can be designed to continue operating if one AZ experiences a problem.
However, physical redundancy alone does not make an application highly available. Application architecture, networking, data replication, 
monitoring, and failover configuration also matter.

### Data center versus edge location

| Feature | Data center | Edge location |
|---|---|---|
| Primary purpose | Run underlying cloud infrastructure | Support content delivery and edge networking |
| Typical resources | Servers, storage, networking equipment | Edge networking and supported edge services |
| Application hosting | Provides infrastructure for supported compute services | Not a general replacement for an AWS Region |
| Content caching | Not its defining purpose | Commonly used by CloudFront |
| Geographic strategy | Organized into AZs and Regions | Distributed near users and networks |
| Example | Infrastructure supporting EC2 instances | CloudFront point of presence |

## Understand AWS Edge Locations
What is an edge location?
An edge location is a point of presence in the AWS global network. Services such as Amazon CloudFront use these locations to deliver content closer to users.

For example, suppose your website's origin server is in Singapore, but a user visits from another country.
Without caching, requests for content may need to travel back to the origin server. With CloudFront caching, eligible content may be served from a nearby edge location.


User
  |
  v
Nearby CloudFront Edge Location
  |
  |-- Cached content available?
  |          |
  |         Yes
  |          |
  |      Return content
  |
  No
  |
  v
Origin Server
  |
  v
Return content and cache it when appropriate

### Data center versus edge location


| Feature | Data center | Edge location |
|---|---|---|
| Primary purpose | Run underlying cloud infrastructure | Support content delivery and edge networking |
| Typical resources | Servers, storage, networking equipment | Edge networking and supported edge services |
| Application hosting | Provides infrastructure for supported compute services | Not a general replacement for an AWS Region |
| Content caching | Not its defining purpose | Commonly used by CloudFront |
| Geographic strategy | Organized into AZs and Regions | Distributed near users and networks |
| Example | Infrastructure supporting EC2 instances | CloudFront point of presence |

### When should you use edge locations?
Use CloudFront and edge delivery when you need:
- Faster delivery of static website content.
- Global distribution of downloadable files.
- Reduced load on your origin servers.
- Support for global audiences.
- Integration with AWS edge security features, such as AWS WAF.
- Improved delivery performance for supported dynamic and streaming workloads.
SAA-C03 exam tip: If a question asks how to reduce latency when delivering website images to users around the world, consider Amazon CloudFront rather than creating an EC2 server in every country.

## Understand Local Zones
What is an AWS Local Zone?
An AWS Local Zone extends selected AWS services closer to a large population center or a specific location where users need low-latency access.

Local Zones are associated with a parent AWS Region and allow you to run supported compute and other resources closer to users while connecting to services in the parent Region. Availability varies by location. AWS Documentation
Example
Imagine you operate a video-editing application where users need responsive access to compute resources near a particular city.

A Local Zone may help reduce the network latency between users and the application.

### How Local Zones work

              Parent AWS Region
                     |
          +----------+----------+
          |                     |
       AZ A                    AZ B
          |
          +---- AWS Local Zone
                    |
             Compute resources
                    |
                Local users

This diagram is conceptual. Actual service availability and network design depend on the selected Local Zone.
When should you use Local Zones?
Consider Local Zones when:
- Users need low-latency access from a particular metropolitan area.
- Applications require local compute resources.
- Supported AWS services in the Local Zone meet the workload's needs.

| Feature | Availability Zone | Local Zone |
|---|---|---|
| Primary purpose | Resilience and high availability | Place selected resources closer to users |
| Relationship | Part of an AWS Region | Associated with a parent Region |
| Typical use | Multi-AZ application deployment | Low-latency applications in specific locations |
| Availability | Depends on the Region and account | Depends on the specific Local Zone and supported services |


## Understand AWS Wavelength Zones
What is AWS Wavelength?
AWS Wavelength brings selected AWS infrastructure and services into telecommunications providers' 5G networks. It is designed for applications that need very low latency for mobile devices and end users. AWS Documentation
Example use cases
- Real-time mobile gaming.
- Connected vehicles.
- Interactive augmented reality.
- Certain industrial IoT applications.
- Applications requiring very low latency over 5G.
How Wavelength works

Mobile device
     |
   5G network
     |
Telecommunications provider
     |
  Wavelength Zone
     |
  AWS Region


  | Feature | Local Zones | Wavelength Zones |
|---|---|---|
| Main purpose | Bring compute closer to users in specific locations | Bring AWS infrastructure into 5G networks |
| Typical use | Media processing, interactive applications | Mobile applications requiring very low latency |
| Network focus | Proximity to local users | Proximity to 5G-connected devices |
| Availability | Specific locations | Supported telecommunications locations |

## Understand AWS Outposts
What is AWS Outposts?
AWS Outposts brings supported AWS infrastructure and services into a customer's on-premises facility or other supported local environment.

It allows organizations to run certain AWS workloads locally while using AWS services and management capabilities. Outposts requires appropriate connectivity to its parent Region. AWS Documentation
Example
A company has a factory with equipment that must communicate with local applications quickly. Some data also needs to be processed on-site.

The company can evaluate AWS Outposts to run supported AWS workloads in its own facility.
How Outposts works

             AWS Region
                  |
           AWS connectivity
                  |
          Company network
                  |
             AWS Outposts
                  |
       Local applications and data

 When should you consider Outposts?
Consider Outposts when:
- Applications need to run on-premises.
- Local data processing is important.
- Applications have strict latency requirements for local systems.
- Specific operational or data-location requirements call for local infrastructure.
Outposts is not identical to a normal AWS Region or an Availability Zone. It is an extension of AWS infrastructure into a supported local environment.      

## Understand AWS Global Network and CloudFront
AWS connects its infrastructure through a global network. Services such as Amazon CloudFront use this network to help deliver content efficiently to users around the world. Amazon Web Services, Inc.
What is a content delivery network?
A content delivery network (CDN) distributes content through geographically distributed locations.

CloudFront can cache eligible content at edge locations and retrieve content from an origin when needed.
Example architecture
Suppose your website uses:
- Amazon S3 to store images and static website assets.
- An application server in an AWS Region.
- CloudFront to distribute content.
- Route 53 to provide DNS routing.
A simplified architecture is:

                 Users worldwide
                        |
                        v
                  Amazon Route 53
                        |
                        v
                   CloudFront
                        |
               +--------+--------+
               |                 |
          Cached content     Cache miss
               |                 |
               v                 v
             Users         Origin in AWS
                            Region
                               |
                       +-------+-------+
                       |               |
                    Amazon S3      Application
                                   Load Balancer
                                       |
                                     EC2



This is a conceptual example; the actual design depends on the application's configuration.
Benefits
1. Lower latency: Cached content may be served closer to users.
2. Reduced origin traffic: Cache hits reduce the number of requests reaching the origin.
3. Scalability: CloudFront distributes content through its global edge network.
4. Security integration: CloudFront can integrate with AWS security services such as AWS WAF and AWS Shield.
CloudFront is not a replacement for multi-AZ application architecture. It addresses content delivery and related edge capabilities, while multi-AZ deployment helps with regional application availability.


## Understand Regional and Global AWS Services
One of the most important concepts for the AWS Solutions Architect Associate exam is understanding whether a resource is regional, zonal, or global.
Regional resources
Regional resources are associated with a specific AWS Region.

Examples include:
- Amazon EC2 instances.
- Amazon VPCs.
- Amazon RDS databases.
- Regional Amazon S3 buckets.
Some regional services can replicate data or resources to another Region, but that does not mean replication happens automatically.
Zonal resources
Zonal resources are associated with a particular Availability Zone.

For example, an EC2 instance runs in a particular AZ, and a subnet is associated with one AZ.
Global services
Some AWS services provide capabilities that span multiple Regions or use a global control plane.

Examples include:
- AWS Identity and Access Management (IAM).
- Amazon CloudFront.
- Amazon Route 53.
The exact scope and behavior depend on the service and resource type.
Comparison table

| Scope | Meaning | Example |
|---|---|---|
| Global | Provides capabilities across Regions | IAM, CloudFront |
| Regional | Associated with a specific Region | VPC, EC2 regional resources |
| Zonal | Associated with a specific AZ | EC2 instance, subnet |


## Design a Highly Available AWS Architecture
Now combine the concepts you have learned.
Scenario
You are designing an e-commerce website for customers in Bangladesh and other countries.

Requirements:
- The website should remain available if one AZ fails.
- Images and static content should load quickly.
- The application should scale as traffic increases.
- The database should have appropriate backup and recovery capabilities.
- The architecture should support future disaster recovery.

<img width="3408" height="1736" alt="image" src="https://github.com/user-attachments/assets/bf71aa41-1f56-440b-ad40-4677597d7256" />



<img width="1011" height="553" alt="image" src="https://github.com/user-attachments/assets/a6a60404-001b-44b2-a4e0-b343308c0621" />


### What happens if one AZ fails?
With suitable multi-AZ configuration:
- The load balancer can stop sending traffic to unhealthy targets.
- Application instances in another AZ can continue handling requests.
- The database's availability depends on its configuration and failover behavior.
Exam point: Multi-AZ deployment primarily addresses availability within a Region. Multi-Region architecture is a separate design decision for regional resilience and disaster recovery.

## Understand AWS Infrastructure Resilience
Three terms frequently appear in AWS architecture questions.
High availability
The application is designed to remain accessible despite certain component failures.

Example: Deploying application servers across multiple AZs.
Fault tolerance
The system is designed to continue operating through specified failures with minimal or no interruption.

Example: A carefully engineered system with redundant components and automated failover.

Fault tolerance requirements are generally stricter than simply having backups.
Disaster recovery
The processes and infrastructure used to restore a workload after a major disruption.

Example: Maintaining a recoverable copy of an application and its data in another Region.
RTO and RPO
- RTO (Recovery Time Objective): The target time within which a service should be restored.
- RPO (Recovery Point Objective): The maximum acceptable amount of data loss measured in time.
For example, an organization might set an RTO of one hour and an RPO of fifteen minutes. Its architecture and recovery procedures must be designed and tested against those objectives.  

| Requirement | Possible design consideration |
|---|---|
| Tolerate one AZ failure | Multi-AZ application architecture |
| Recover deleted data | Backups and tested restoration |
| Recover from regional disruption | Cross-Region disaster recovery |
| Minimize latency for global users | CloudFront and appropriate Region selection |
| Process data close to a specific city | Evaluate Local Zones |
| Run workloads on-site | Evaluate AWS Outposts |


