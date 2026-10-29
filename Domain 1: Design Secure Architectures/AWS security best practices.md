# AWS security best practices (the principle of least privilege)  

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



