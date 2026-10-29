# AWS security best practices

By the end of this guide, you should be able to:

| No. | Security best practice | AWS service or feature |
|---:|---|---|
| 1 | Apply the principle of least privilege | AWS IAM |
| 2 | Secure the AWS root user | AWS Account Management, IAM |
| 3 | Enable multi-factor authentication (MFA) | IAM, IAM Identity Center |
| 4 | Secure network access | Amazon VPC, Security Groups, Network ACLs |
| 5 | Encrypt data at rest and in transit | AWS KMS, TLS |
| 6 | Protect sensitive information | AWS Secrets Manager, Systems Manager Parameter Store |
| 7 | Enable security logging and auditing | AWS CloudTrail, AWS Config |
| 8 | Detect threats | Amazon GuardDuty, AWS Security Hub |
| 9 | Protect applications and APIs | AWS WAF, AWS Shield |
| 10 | Secure storage | Amazon S3, EBS |
| 11 | Patch and harden systems | AWS Systems Manager |
| 12 | Secure workloads running on EC2 and containers | EC2, ECR, ECS, EKS |
| 13 | Prepare for security incidents | CloudTrail, GuardDuty, incident response procedures |
| 14 | Implement backup and disaster recovery | AWS Backup |
| 15 | Monitor security costs and resource exposure | AWS Budgets, AWS Trusted Advisor |

## 1. What is the principle of least privilege?

The principle of least privilege (PoLP) means giving a user, application, service, or role only the permissions required to perform its assigned tasks—and no additional permissions.
For example, suppose a developer needs to download files from an Amazon S3 bucket.

The developer should be able to read files from that specific bucket, but should not automatically receive permission to:
- Delete S3 buckets.
- Modify IAM users.
- Create administrator accounts.
- Access unrelated production databases.
- Disable security monitoring.
- Modify AWS account security settings.
- 
Incorrect approach
Giving the developer the AWS-managed AdministratorAccess policy.
This policy provides broad administrative permissions and is excessive for a user who only needs to download files.

Correct approach
Create a policy that grants read-only access to the specific S3 bucket and its objects.
The following example grants read access to objects in a bucket named company-app-documents.

Important: Replace the example bucket name with your own bucket name before using the policy.

```text
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ListSpecificBucket",
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket"
      ],
      "Resource": "arn:aws:s3:::company-app-documents"
    },
    {
      "Sid": "ReadSpecificBucketObjects",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject"
      ],
      "Resource": "arn:aws:s3:::company-app-documents/*"
    }
  ]
}
```
This policy allows the principal to list the bucket's objects and retrieve their contents. It does not grant permissions to upload or delete objects.

Note that effective access also depends on other applicable policies, resource policies, permissions boundaries, service control policies, and other authorization controls.

## AWS Identity and Access Management (IAM) 
**IAM** is a web service that helps you securely control access to AWS resources. With IAM, you can manage permissions that control which AWS resources users can access. You use IAM to control who is authenticated (signed in) and authorized (has permissions) to use resources. IAM provides the infrastructure necessary to control authentication and authorization for your AWS accounts.

IAM answers three important questions:
1. Who are you? — Authentication.
2. What are you allowed to do? — Authorization.
3. Which resources can you access? — Resource-level permissions.

AWS IAM Core Components
There are five important IAM components.

| Component | Purpose | Example |
|---|---|---|
| IAM User | An identity for a person or application when needed | A legacy application identity |
| IAM Group | Organizes IAM users with similar permissions | Developers |
| IAM Role | Provides temporary credentials to an authorized identity or service | EC2 accessing an S3 bucket |
| IAM Policy | Defines allowed or denied actions | Allow reading specific S3 objects |
| IAM Identity Center | Centralizes workforce access to multiple AWS accounts | One employee accessing development and production accounts |


### Why is root user security important?
The AWS root user is the original identity created when an AWS account is established. It has extensive privileges and should not be used for everyday administrative tasks.

If an attacker gains root access, the attacker may be able to perform highly destructive actions, depending on the service and applicable controls.
AWS recommends protecting root credentials, enabling MFA, avoiding root access keys, and using root only when necessary.

#### Step-by-step implementation
Step 1: Sign in as the root user.

Open the AWS Management Console and sign in using the account's root email address and password.

Step 2: Enable MFA.
1. Open Account settings or the root user's security credentials area.
2. Locate Multi-factor authentication (MFA).
3. Choose to assign or add an MFA device.
4. Follow the prompts to configure a supported authenticator app or security key.
5. Test the sign-in process.
Where practical, register more than one supported MFA device to improve recovery options.

Step 3: Remove root access keys.
1. Open the root user's security credentials page.
2. Locate access keys.
3. If root access keys exist, identify and migrate any dependent process.
4. Deactivate and remove the keys after confirming they are no longer required.
Do not create root access keys for normal automation.

Step 4: Use a separate administrative identity.

Configure IAM Identity Center or another suitable identity provider for everyday administration. Give administrators access through appropriately scoped permission sets and roles.

Step 5: Monitor root account activity.

Enable CloudTrail logging and create alerts for root user activity. Root usage should be investigated unless it is an expected, authorized action.
Security checklist
- [ ] Root MFA enabled.
- [ ] Root password is strong and unique.
- [ ] Root access keys do not exist.
- [ ] Root credentials are securely stored.
- [ ] Account recovery email and phone are protected.
- [ ] Root user activity is monitored.
- [ ] Everyday work uses a separate identity.

#### Enable Multi-Factor Authentication (MFA)
##### What is MFA?
Multi-factor authentication adds a second verification factor to a sign-in process.

For example, a user enters a password and then verifies their identity using an authenticator app or a hardware security key.

Even if an attacker steals the password, MFA can make unauthorized access more difficult.
##### Recommended approach
For workforce access, prefer centralized identity management and temporary credentials. Require MFA through your identity provider or AWS identity configuration.

Use phishing-resistant authentication, such as supported FIDO security keys or passkeys, where available.

##### Why avoid long-lived access keys?
An AWS access key consists of an access key ID and a secret access key. If these credentials are exposed in a public repository, log file, laptop, or application configuration, an attacker may use them to access AWS.

For supported workloads, AWS recommends temporary credentials through IAM roles instead of long-lived credentials. Amazon Web Services, Inc.
Incorrect configuration
```
AWS_ACCESS_KEY_ID = "EXAMPLE_ACCESS_KEY"
AWS_SECRET_ACCESS_KEY = "EXAMPLE_SECRET_KEY"
```
Never place actual credentials directly in source code.
Better configuration
Use an IAM role attached to the workload, or another supported temporary-credential mechanism.

## Secure Network Access with Amazon VPC

### Understand Amazon VPC
**What is Amazon VPC** Amazon Virtual Private Cloud allows you to create a logically isolated virtual network in AWS.

You control important networking components, including:
- IP address ranges.
- Subnets.
- Route tables.
- Internet connectivity.
- Network access controls.
- Private connectivity to AWS services.
- Traffic monitoring.
  
For example, suppose you are deploying a web application on AWS. You want users to access the website publicly, but you do not want them to connect directly to your application servers or database.

A VPC lets you separate these resources and enforce network security boundaries.

Important VPC components


| Component | Purpose | Security benefit |
|---|---|---|
| VPC | Isolated virtual network | Defines the network boundary |
| CIDR block | Defines the IP address range | Enables planned IP allocation |
| Public subnet | Hosts resources with a route to an Internet Gateway | Supports controlled public-facing services |
| Private subnet | Hosts resources without a direct route to an Internet Gateway | Reduces direct Internet exposure |
| Route table | Controls where network traffic is sent | Restricts network paths |
| Internet Gateway (IGW) | Connects eligible VPC resources to the Internet | Enables public Internet connectivity |
| NAT Gateway | Enables outbound IPv4 connectivity from private subnets | Avoids direct inbound Internet connections to private instances |
| Security Group | Stateful network firewall for supported resources | Controls inbound and outbound traffic |
| Network ACL (NACL) | Stateless subnet-level traffic filter | Adds an additional network filtering layer |
| VPC Flow Logs | Records information about IP traffic | Supports monitoring and investigation |
| VPC endpoint | Provides private connectivity to supported AWS services | Can reduce dependence on public Internet paths |


Key principle: A subnet is not automatically public just because it is named public-subnet. Its route table and associated network configuration determine whether it has a public Internet route.


<img width="1408" height="705" alt="image" src="https://github.com/user-attachments/assets/8e7e2431-b62f-4729-b966-cf93c28bfd2f" />

```
                       Internet
                          |
                          v
                 Internet Gateway
                          |
                          v
                 Public Subnet
                Application Load Balancer
                          |
                          v
                  Private Subnet
                  Application Servers
                          |
                          v
                   Private Subnet
                     RDS Database
```

This architecture limits direct exposure of the application servers and database.



 
## CIDR Blocks and IP Addressing

CIDR stands for **Classless Inter-Domain Routing**.
A CIDR block defines a range of IP addresses.
For example:

`10.0.0.0/16`

This IPv4 CIDR block contains 65,536 addresses, including network and other reserved addresses.
A smaller subnet could use:

`10.0.1.0/24`

This contains 256 IPv4 addresses before accounting for AWS-reserved addresses.

### Why CIDR planning matters

Proper IP address planning helps you:

- Divide a network into smaller segments.
- Avoid overlapping network ranges.
- Connect VPCs through peering or transit networks.
- Establish connectivity with on-premises networks.
- Organize resources according to security requirements.

**Security consideration** Overlapping CIDR ranges can complicate routing and connectivity between networks. A well-planned IP addressing strategy reduces these problems.

---

# Understanding Public and Private Subnets

Subnet design is one of the most important concepts in secure VPC architecture.

## What is a Subnet?

A subnet is a segment of a VPC's IP address range.

Subnets help organize resources into separate network segments based on their purpose.

For example, a company might create separate subnets for:

- Public-facing web applications.
- Internal application servers.
- Databases.
- Management and administration systems.

Subnet separation supports network organization, but **a subnet is not, by itself, a complete security boundary**.

## Public Subnet

A public subnet is a subnet whose associated route table has a route directing internet-bound IPv4 traffic to an Internet Gateway.

Resources also need suitable addressing and security configuration to communicate directly with the internet.

### Typical resources

- Public-facing load balancers.
- Internet-facing web servers, when required.
- Network appliances that need public connectivity.

### Security considerations

Public-facing resources are more exposed to internet-originated traffic.

To reduce risk:

- Expose only the services that must be publicly accessible.
- Restrict administrative access.
- Use security groups to control inbound connections.
- Monitor network traffic.
- Avoid placing databases and other sensitive backend systems in directly internet-accessible configurations.

**Important:** A public subnet does not mean every resource in that subnet is automatically accessible from the internet.

## Private Subnet

A private subnet generally has no direct route to an Internet Gateway for internet-bound traffic.

It is commonly used for resources that do not need direct inbound internet connectivity.

### Typical resources

- Application servers.
- Database servers.
- Internal APIs.
- Background processing services.
- Internal business applications.

A private subnet can still have outbound connectivity through a NAT Gateway or other network path, depending on its routing configuration.

### Security benefits

- Reduces direct internet exposure.
- Separates backend systems from public-facing components.
- Supports layered access controls.
- Helps limit the potential impact of a compromised public-facing system.

### Important distinction

**Private does not mean completely isolated.**

A resource in a private subnet may still communicate with other VPC resources, connected networks, or external destinations through configured routes and permitted security rules.

## Public vs. Private Subnets

| Feature | Public subnet | Private subnet |
|---|---|---|
| Typical purpose | Public-facing services | Internal workloads |
| Route to Internet Gateway | Usually present for internet-bound IPv4 traffic | Usually absent for direct internet access |
| Direct inbound internet access | Possible when addressing, routing, and security permit | Generally not available through a direct internet route |
| Outbound internet access | Possible with suitable routing and security | Possible through NAT or other configured paths |
| Common resources | Public load balancers | Application servers and databases |
| Security objective | Minimize exposure | Limit direct external reachability |

### Example architecture

Consider an online shopping platform:

- **Public tier:** A load balancer receives customer requests.
- **Private application tier:** Application servers process orders.
- **Private database tier:** Database servers store customer and order records.

Customers connect to the public endpoint. The application tier communicates with the database tier through specifically authorized connections.

This design separates internet-facing access from sensitive internal processing.

---

# Internet Gateway (IGW)

## What is an Internet Gateway?

An **Internet Gateway (IGW)** is a VPC component that enables communication between resources in a VPC and the public internet when appropriate routing, addressing, and security rules are configured.

It is horizontally scaled, redundant, and highly available.

### Main responsibilities

- Provides a target for internet-bound routes in VPC route tables.
- Enables internet connectivity for suitably configured resources.
- Supports internet communication for public IPv4 and IPv6 configurations, subject to their respective routing and addressing requirements.

### Security considerations

An Internet Gateway does not act as a firewall for your application.

The presence of an IGW does not automatically expose every resource in a VPC to the internet. Exposure depends on routing, IP addressing, security groups, network ACLs, and other applicable controls.

**Example:** A public-facing web application may need internet access, while a database should not need direct inbound internet connectivity.

---

# NAT Gateway and Outbound Internet Access

## What is a NAT Gateway?

A **Network Address Translation (NAT) Gateway** enables resources in a private subnet to initiate connections to external IPv4 destinations without requiring those resources to have directly reachable public IPv4 addresses.

For example, a private application server might need to download operating system updates from an external repository.

A NAT Gateway can provide the required outbound connectivity.

### How it works conceptually

1. A private resource initiates an outbound connection.
2. The private subnet's route table directs the traffic to a NAT Gateway.
3. The NAT Gateway translates the source address for the outbound connection.
4. The destination server receives the request.
5. Response traffic returns through the NAT Gateway to the originating resource.

### Security benefits

- Private instances do not need public IPv4 addresses for this outbound connectivity.
- Unsolicited inbound connections from the internet cannot normally be initiated through a public NAT Gateway to the private resource.
- Outbound access can be separated from direct inbound access.

### Important limitation

A NAT Gateway is **not a general-purpose firewall**. It does not replace security groups, network ACLs, or dedicated outbound traffic inspection.

If a compromised private resource can make outbound connections, it may still communicate with external destinations permitted by the network configuration.

### NAT Gateway vs. Internet Gateway

| Feature | Internet Gateway | NAT Gateway |
|---|---|---|
| Primary purpose | Connect a VPC to the internet | Enable outbound IPv4 connectivity from private resources |
| Common use | Public-facing services | Private servers needing external access |
| Public IP on private workload required for this path | Depends on the internet connectivity design | No |
| Replaces security groups? | No | No |
| Filters malicious application traffic by itself? | No | No |

---

# Route Tables and Network Traffic Flow

## What is a Route Table?

A route table contains rules that determine where network traffic is directed.

Each route includes a destination network and a target.

A subnet uses an associated route table to determine how its outbound traffic is routed.

### Common route targets

- Local VPC routing.
- Internet Gateway.
- NAT Gateway.
- Virtual Private Gateway.
- Transit Gateway.
- VPC endpoint, depending on endpoint type.

## Why Are Route Tables Important for Security?

Security groups determine which traffic is permitted at a resource's network interface, while route tables determine where traffic can be directed.

Both must be considered when evaluating network reachability.

### Example: Public subnet

A public subnet might have:

- A local route for communication within the VPC.
- A default IPv4 route to an Internet Gateway.

### Example: Private application subnet

A private application subnet might have:

- A local route for internal VPC communication.
- A default IPv4 route to a NAT Gateway for outbound internet connectivity.

### Example: Isolated database subnet

A database subnet might have only the local VPC route and any additional routes strictly required by the application architecture.

It would not need a default internet route if it has no internet connectivity requirement.

### Security best practice

**Use the minimum routes necessary to support the workload.**

Unnecessary routes can create connectivity paths that increase exposure or make network behavior harder to understand.

---

# Security Groups: The Primary Resource-Level Firewall

## What is a Security Group?

A security group is a virtual firewall that controls inbound and outbound traffic for supported AWS resources, such as Amazon EC2 instances through their network interfaces.

Security groups are **stateful**.

This means that when a connection is permitted in one direction, its response traffic is automatically permitted by connection tracking, regardless of whether a separate rule would otherwise permit that response direction.

### Security group rules

Security groups support two main categories of rules.

**Inbound rules:** Control which traffic is allowed to reach the resource.

**Outbound rules:** Control which traffic the resource is allowed to initiate.

Rules can specify:

- Protocol.
- Port or port range.
- Source for inbound traffic.
- Destination for outbound traffic.
- IP address ranges or, where supported, references to other security groups or managed prefix lists.

Security groups support allow rules rather than explicit deny rules. Traffic not permitted by the applicable rules is denied. [AWS Documentation](https://docs.aws.amazon.com/en_en/vpc/latest/userguide/vpc-security-groups.html?utm_source=chatgpt.com)


## Understanding Common Ports

| Port | Protocol | Typical purpose | Security consideration |
|---|---|---|---|
| 22 | TCP | SSH administration | Restrict access to approved administration sources. |
| 80 | TCP | HTTP web traffic | Use HTTPS for sensitive application communication. |
| 443 | TCP | HTTPS web traffic | Permit access according to the application's exposure requirements. |
| 3389 | TCP | Windows Remote Desktop | Restrict access to approved management sources. |
| 5432 | TCP | PostgreSQL | Allow only the application components that need database access. |
| 3306 | TCP | MySQL | Avoid unnecessary exposure to public networks. |

These are typical defaults; applications can use other ports.

## Principle of Least Privilege

The principle of least privilege means granting only the access necessary to perform a specific task.

In VPC security, this means avoiding unnecessarily broad source ranges, destinations, protocols, and port ranges.

### Example: Web application

A secure design might permit:

- Public users to access the public web endpoint over HTTPS.
- The web tier to communicate with the application tier on its required application port.
- The application tier to connect to the database on its required database port.
- Approved administrators to use a controlled management path.

The database should not accept direct connections from arbitrary internet addresses.

## Why Is `0.0.0.0/0` Important?

`0.0.0.0/0` represents all IPv4 addresses.

If used as the source of an inbound security group rule, it permits traffic from any IPv4 source for the specified protocol and ports, subject to other applicable controls.

For IPv6, `::/0` represents all IPv6 addresses.

Broad rules are not inherently wrong in every situation. For example, a public HTTPS endpoint may need to accept requests from users around the world.

However, opening administrative or database ports to all addresses creates unnecessary risk.

**Best practice:** Make every security group rule correspond to a documented business or technical requirement.

---

# Network Access Control Lists (Network ACLs)

## What is a Network ACL?

A Network Access Control List, or **network ACL (NACL)**, is a network traffic filter associated with a subnet.

Unlike security groups, network ACLs are stateless and support both allow and deny rules.

### Main characteristics

- Operates at the subnet level.
- Evaluates inbound and outbound traffic.
- Supports explicit allow and deny rules.
- Evaluates rules in ascending numerical order.
- Stops evaluating rules when a matching rule is found.
- Does not automatically allow response traffic.

The last point means that inbound and outbound rules must be designed to permit the necessary request and response traffic. [AWS Documentation](https://docs.aws.amazon.com/en_en/vpc/latest/userguide/infrastructure-security.html?utm_source=chatgpt.com)


## Security Group vs. Network ACL

| Feature | Security group | Network ACL |
|---|---|---|
| Scope | Resource network interface | Subnet |
| Stateful or stateless | Stateful | Stateless |
| Explicit deny rules | No | Yes |
| Rule evaluation | All applicable rules are considered | Rules evaluated in ascending number until a match |
| Return traffic | Automatically allowed for tracked connections | Must be explicitly permitted |
| Common role | Fine-grained resource access | Additional subnet-level control |

## How Do They Work Together?

Imagine an application server in a private subnet.

When another resource attempts to connect:

1. The routing configuration determines the network path.
2. The subnet's network ACL evaluates the traffic crossing the subnet boundary.
3. The application's security group evaluates whether the connection is permitted.
4. The destination resource processes the request if it reaches the service.

The exact traffic path depends on the architecture and where the communicating resources are located.

### Recommended approach

Use security groups as the primary resource-level traffic control. Add network ACLs where subnet-level filtering provides meaningful additional protection.

Avoid overly complex NACL rules that could unintentionally block legitimate application traffic or its responses.

---
 

# VPC Endpoints and Private Connectivity to AWS Services

## What is a VPC Endpoint?

A VPC endpoint allows supported resources in a VPC to connect to certain AWS services without requiring the traffic to traverse the public internet.

This can reduce unnecessary internet exposure and simplify network architecture.

For example, a private application may need to access Amazon S3 to retrieve files or store application data.

Without a suitable endpoint, the architecture may require an alternative network path, such as a NAT Gateway.

A properly configured VPC endpoint can provide a private connectivity path for supported services.

## Types of VPC Endpoints

### A. Gateway endpoints

Gateway endpoints support:

- Amazon S3.
- Amazon DynamoDB.

They use route table entries to direct traffic to the supported service.

### B. Interface endpoints

Interface endpoints use AWS PrivateLink and create endpoint network interfaces in selected subnets.

They support private connectivity to many AWS services and supported endpoint services.

### Security benefits

- Reduce reliance on internet-facing network paths.
- Support private service access.
- Allow endpoint policies to restrict permitted actions for supported services.
- Help keep internal service communication within the intended network design.

### Important limitation

A VPC endpoint does not automatically grant permission to access the connected service.

Access may still depend on IAM policies, resource policies, endpoint policies, security groups for interface endpoints, and the service's own authorization controls.

**Key principle:** Private network connectivity and authorization are separate controls; both must be configured appropriately.

---

---

# VPC Peering, Transit Gateway, VPN, and Direct Connect

Secure network access involves more than controlling internet connectivity. Organizations often need to connect multiple VPCs and on-premises networks.

## VPC Peering

**VPC Peering** establishes network connectivity between two VPCs using private IP addresses.

For example, an organization might have:

- One VPC for production applications.
- Another VPC for shared services.

VPC peering can allow these environments to communicate when the appropriate routes and security rules are configured.

### Security considerations

- VPC peering does not automatically permit all traffic.
- Route tables and security rules must permit the intended communication.
- Overlapping IPv4 or IPv6 address ranges can prevent peering.
- VPC peering does not provide transitive routing through a third VPC.

**Use case:** Connecting two VPCs that need a relatively straightforward private network connection.

## AWS Transit Gateway

AWS Transit Gateway acts as a central network hub that connects multiple VPCs and supported external networks.

Instead of creating a large number of individual peering connections, an organization can use Transit Gateway to simplify connectivity across many networks.

### Security benefits

- Centralizes network connectivity.
- Supports network segmentation using separate Transit Gateway route tables.
- Helps control which networks can communicate.
- Simplifies network architecture as the number of VPCs grows.

### Example

A company operates separate VPCs for development, testing, production, and shared services.

Transit Gateway can provide the connectivity architecture while routing and other security controls restrict unnecessary communication between environments.

**Important:** Centralized connectivity does not automatically mean centralized security. Routes and security policies must still be designed correctly.

## AWS Site-to-Site VPN

AWS Site-to-Site VPN creates an encrypted connection between an on-premises network and an AWS network over the internet.

It commonly uses IPsec VPN tunnels.

### Use cases

- Connecting a company office to AWS.
- Connecting an existing data center to a VPC.
- Enabling hybrid cloud applications.
- Supporting secure communication between corporate and cloud resources.

### Security considerations

Encryption protects traffic in transit, but VPN connectivity does not automatically authorize every user or device to access every cloud resource.

Routing, security groups, network ACLs, identity controls, and endpoint security remain important.

## AWS Direct Connect

AWS Direct Connect provides a dedicated network connection between an organization's network and AWS.

It can offer more predictable network performance and support private connectivity architectures.

### Security considerations

- A dedicated connection is not the same as encryption.
- Encryption requirements must be evaluated separately.
- Network routing and access controls remain necessary.
- Access to AWS services and resources must still be authorized.

### Connectivity comparison

| Service | Primary purpose | Typical use case |
|---|---|---|
| VPC Peering | Private connectivity between two VPCs | Simple VPC-to-VPC communication |
| Transit Gateway | Centralized connectivity across many networks | Enterprise network architecture |
| Site-to-Site VPN | Encrypted connectivity over the internet | Hybrid cloud connectivity |
| Direct Connect | Dedicated network connection to AWS | Enterprise connectivity with specific performance or connectivity requirements |
| PrivateLink | Private access to supported services | Service access without general network connectivity to the provider VPC |

For more information, see the official [AWS VPC connectivity documentation](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html?utm_source=chatgpt.com).

---

# AWS Network Firewall and Web Application Firewall

Two important security services are AWS Network Firewall and AWS WAF. They solve different problems.

## AWS Network Firewall

AWS Network Firewall is a managed network firewall that can inspect and filter traffic flowing through configured network paths.

It supports stateful and stateless traffic filtering and can provide intrusion detection and prevention capabilities.

### What does it protect against?

Depending on its configuration and rule sets, it can help detect or block:

- Unauthorized network connections.
- Traffic to prohibited destinations.
- Suspicious traffic patterns.
- Certain known malicious network signatures.
- Connections that violate organizational network policies.

### Example

An organization wants to restrict outbound connections from private application servers.

Security groups can restrict traffic by protocol and port, while AWS Network Firewall can provide additional inspection and filtering along the configured outbound path.

**Important:** Traffic must actually traverse the firewall for that firewall to inspect it. Route tables and network architecture determine whether traffic follows the intended inspection path.

## AWS WAF

AWS WAF is a web application firewall designed to protect supported web application endpoints.

It operates at the HTTP/HTTPS application layer rather than serving as a general-purpose VPC firewall.

### Typical protection capabilities

Depending on its rules and configuration, AWS WAF can help protect web applications against:

- SQL injection patterns.
- Cross-site scripting patterns.
- Malicious HTTP requests.
- Requests originating from prohibited IP addresses or ranges.
- Excessive request rates.

### Network Firewall vs. AWS WAF

| Feature | AWS Network Firewall | AWS WAF |
|---|---|---|
| Primary focus | Network traffic | HTTP/HTTPS web requests |
| Typical scope | Configured network traffic paths | Supported web application endpoints |
| Common protection | Network-level filtering and inspection | Application-layer request filtering |
| Example | Restrict outbound network connections | Block suspicious web requests |
| Replaces security groups? | No | No |

### Key concept

Network Firewall, WAF, security groups, and network ACLs are complementary controls. Each addresses a different part of the security problem.

---

# VPC Flow Logs and Network Monitoring

## 13.1 What are VPC Flow Logs?

VPC Flow Logs capture metadata about IP traffic to and from monitored VPC network interfaces.

They can be created at the VPC, subnet, or network-interface level.

Flow Logs are useful for understanding network activity and investigating connectivity problems.

### Information they can contain

Depending on the selected log format, records may include:

- Source IP address.
- Destination IP address.
- Source and destination ports.
- IP protocol.
- Packet and byte counts.
- Traffic start and end times.
- Traffic acceptance or rejection status.
- Relevant network-interface information.

Flow Logs do not capture the full contents of network packets by default.

## Why Are Flow Logs Important?

Suppose an organization discovers unexpected traffic associated with a database server.

Flow Logs may help the security team determine:

1. Which IP addresses communicated with the database.
2. Which ports were involved.
3. Whether observed traffic was accepted or rejected.
4. When the traffic occurred.
5. Whether the activity matches the expected application architecture.

This information can support investigation and remediation.

### Common destinations

Flow Logs can publish data to supported destinations such as:

- Amazon CloudWatch Logs.
- Amazon S3.
- Amazon Data Firehose.

### Important limitation

Flow Logs are not a complete packet capture or a real-time guarantee of every connection event. They also do not, by themselves, block malicious traffic.

They provide network visibility that can be combined with detection and response systems.

For official details, see [Amazon VPC Flow Logs documentation](https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html?utm_source=chatgpt.com).

---

# Amazon GuardDuty and Network Threat Detection

## What is Amazon GuardDuty?

Amazon GuardDuty is a managed threat detection service that analyzes supported data sources to identify potentially malicious or unauthorized activity in AWS environments.

Depending on the enabled protections, it can identify suspicious activity associated with AWS accounts, workloads, and network behavior.

### Example findings

GuardDuty may identify activity associated with:

- Known malicious IP addresses.
- Suspicious network communication.
- Potentially compromised EC2 instances.
- Unusual API activity.
- Certain attempts to discover or access resources.

### How it complements VPC security

Security groups and network ACLs establish network traffic rules.

Flow Logs provide visibility into traffic metadata.

GuardDuty analyzes supported signals to identify potential threats.

These controls work together, but they do not have identical functions.

**Remember:**

- Security groups control permitted traffic.
- Network ACLs filter traffic at the subnet level.
- Flow Logs record traffic metadata.
- GuardDuty detects potential threats.

---

# 19. Step-by-Step Security Design Process

The following steps provide a theoretical methodology for planning a secure VPC.

## Step 1: Identify Business and Security Requirements

Determine:

- Which applications will run in the VPC?
- Which resources need public access?
- Which resources contain sensitive information?
- Which systems must communicate?
- Which administrators require access?
- Which compliance requirements apply?

**Expected outcome:** A clear list of required communication paths and security boundaries.

## Step 2: Design the IP Addressing Scheme

Choose VPC CIDR ranges and subnet sizes.

Consider:

- Current workload requirements.
- Future growth.
- Connectivity to other VPCs.
- On-premises network integration.
- IPv4 and IPv6 requirements.

**Expected outcome:** A non-overlapping, scalable network addressing plan.

## Step 3: Separate Workloads into Network Tiers

Classify resources into appropriate network segments.

For a three-tier application, these might include public-facing, application, and database tiers.

Place workloads that require different levels of network exposure into appropriate subnets.

**Expected outcome:** Reduced direct exposure and clearer network boundaries.

## Step 4: Define Routing Requirements

For each subnet, determine:

- Which internal networks it needs to reach.
- Whether it needs inbound or outbound internet connectivity.
- Whether it needs access to AWS services through VPC endpoints.
- Whether it needs VPN, Transit Gateway, or other connectivity.

**Expected outcome:** A routing design that supports required communication without unnecessary paths.

## Step 5: Define Security Group Rules

For each workload, document the required inbound and outbound communication.

Consider:

- Source and destination.
- Protocol.
- Port.
- Business justification.
- Responsible owner.

**Expected outcome:** Resource-level network access based on least privilege.

## Step 6: Evaluate Network ACL Requirements

Determine whether additional subnet-level filtering provides meaningful protection.

Account for stateless filtering and the return traffic required by legitimate connections.

**Expected outcome:** An additional network control layer without unnecessary complexity.

## Step 7: Plan Secure Administrative Access

Choose an appropriate management approach, such as Systems Manager Session Manager or a tightly controlled bastion host.

Determine which identities can administer which resources.

**Expected outcome:** Administrative access that is separate from public application access.

## Step 8: Restrict Outbound Connectivity

Identify why each workload needs external connectivity.

Consider:

- NAT Gateways for required outbound IPv4 internet access.
- VPC endpoints for supported AWS services.
- AWS Network Firewall for traffic inspection and filtering.
- Domain-based filtering where appropriate.

**Expected outcome:** Reduced opportunities for unauthorized outbound communication.

## Step 9: Enable Monitoring and Threat Detection

Plan monitoring for:

- Network traffic metadata.
- AWS administrative API activity.
- Potentially malicious behavior.
- Unexpected access patterns.
- Security configuration changes.

**Expected outcome:** Visibility into network activity and potential threats.

## Step 10: Review and Validate the Architecture

Evaluate the completed design against the original requirements.

Ask:

- Can public users reach only the intended public endpoints?
- Can application servers reach only the resources they need?
- Can the database be accessed only through approved paths?
- Are IPv4 and IPv6 rules both appropriate?
- Are administrative permissions restricted?
- Is outbound access justified?
- Can the security team investigate suspicious traffic?

**Expected outcome:** A documented network architecture with clear access boundaries and monitoring responsibilities.

AWS provides further guidance in its official [VPC security best practices](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-best-practices.html?utm_source=chatgpt.com).

--- 
