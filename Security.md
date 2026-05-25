# AWS Cloud Security

## Part 1: AWS Cloud Security Overview

AWS cloud security covers multiple layers, including:

- Physical security of data centers
- Network security
- Hardware and managed infrastructure
- Identity and access management (IAM)
- Application security
- Data encryption

### Authentication vs Authorization

- **Authentication**: verifies who you are.
- **Authorization**: controls what you are allowed to do.

### IAM (Identity and Access Management)

IAM manages permissions and access across AWS.

- **User**: an identity for a person or service.
- **Group**: a set of users that share permissions.
- **Role**: temporary credentials that can be assumed by trusted entities.
- **Policy**: a document that defines allowed or denied actions.

### AWS IAM accounts

- **Root User**: has full account control, including deletion and billing access. Avoid using the root user for everyday tasks.
- **IAM User**: receives only the permissions explicitly granted to them.

## Part 2: AWS Policies and Roles

### Types of AWS Policies

1. **Identity-based policy**: attached to users, groups, or roles.
2. **Permission boundary**: sets the maximum permissions an identity can receive.
3. **Resource-based policy**: attached directly to resources like S3 buckets or Lambda functions.
4. **Service Control Policy (SCP)**: restricts service access for accounts in an AWS Organization.
5. **Resource Control Policy (RCP)**: restricts resource access for accounts in an AWS Organization.
6. **Session policy**: applied to temporary credentials used by SDKs, AWS CLI, or AWS CDK.
7. **ACLs (Access Control Lists)**: resource-level permissions used by services such as S3 and VPC.

## Demos

### Demo 1: IAM Users and Policies

1. Create two IAM users.
2. Attach policies to each user.
3. Change one users policy and confirm the other user is unaffected.

### Demo 2: EC2 Access to S3

1. Create or choose an EC2 instance.
2. On the EC2 instance, run:

   ```bash
   aws s3 ls
   ```

3. If access is denied, fix it using one of these options:

**Option A:**
- Run `aws configure` on the EC2 instance and enter AWS access keys.

**Option B:**
- Create an IAM role with S3 permissions, such as `AmazonS3ReadOnlyAccess`.
- Attach the role to the EC2 instance.
- Run `aws s3 ls` again.

4. You should now see the S3 bucket listing.

## Additional AWS Security Services

### AWS WAF (Web Application Firewall)

AWS WAF protects web applications from common threats like SQL injection, cross-site scripting (XSS), and malicious HTTP requests.

- Works with CloudFront, Application Load Balancer (ALB), and API Gateway.
- Helps filter traffic, block attacks, and enforce access rules.

### AWS Security Hub

AWS Security Hub centralizes security findings from multiple AWS services.

- Integrates with GuardDuty, Inspector, and IAM Access Analyzer.
- Provides a single dashboard for risk and compliance monitoring.
- Helps teams track issues and improve security posture.

### GuardDuty

- Threat detection service.
- Detects suspicious activity such as unauthorized access and unusual behavior.

### Inspector

- Security assessment service.
- Scans AWS resources for vulnerabilities and configuration issues.

### IAM Access Analyzer

- Access monitoring service.
- Identifies resources that are public or shared outside your account.

### Quick Summary

- **GuardDuty**: finds threats.
- **Inspector**: finds vulnerabilities.
- **IAM Access Analyzer**: finds risky access exposures.

