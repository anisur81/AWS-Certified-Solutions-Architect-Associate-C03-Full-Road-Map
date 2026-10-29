# AWS security best practices

By the end of this guide, you should be able to:

| No. | Security best practice | AWS service or feature |
|---:|---|---|
| 1 | Apply the principle of least privilege | AWS IAM |
| 2 | Secure the AWS root user | AWS Account Management, IAM |
| 3 | Enable multi-factor authentication (MFA) | IAM, IAM Identity Center |
| 4 | Use centralized identity management | AWS IAM Identity Center |
| 5 | Secure network access | Amazon VPC, Security Groups, Network ACLs |
| 6 | Encrypt data at rest and in transit | AWS KMS, TLS |
| 7 | Protect sensitive information | AWS Secrets Manager, Systems Manager Parameter Store |
| 8 | Enable security logging and auditing | AWS CloudTrail, AWS Config |
| 9 | Detect threats | Amazon GuardDuty, AWS Security Hub |
| 10 | Protect applications and APIs | AWS WAF, AWS Shield |
| 11 | Secure storage | Amazon S3, EBS |
| 12 | Patch and harden systems | AWS Systems Manager |
| 13 | Secure workloads running on EC2 and containers | EC2, ECR, ECS, EKS |
| 14 | Prepare for security incidents | CloudTrail, GuardDuty, incident response procedures |
| 15 | Implement backup and disaster recovery | AWS Backup |
| 16 | Monitor security costs and resource exposure | AWS Budgets, AWS Trusted Advisor |

## What is the principle of least privilege?

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
Network security determines which systems can communicate with your workloads and which services are exposed to the internet.

AWS network security commonly uses:
- Amazon VPC: A logically isolated network for your AWS resources.
- Security groups: Stateful virtual firewalls attached to supported resources.
- Network ACLs: Stateless controls applied at the subnet level.
- Private subnets: Subnets without a direct route to an internet gateway.
- VPC endpoints: Private connectivity to supported AWS services.

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

his architecture limits direct exposure of the application servers and database.
