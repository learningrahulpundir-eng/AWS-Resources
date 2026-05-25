# AWS Cloud Security

## Part 1: AWS Cloud Security Overview

AWS cloud security includes:
- Physical data center security
- Networking security
- Hardware and managed infrastructure
- IAM user and permission management
- Application security
- Encryption

### Authentication vs Authorization
- **Authentication**: who are you?
- **Authorization**: what can you do?

### IAM (Identity and Access Management)
IAM controls who can access what.

- **User**: individual identity that represents a person or service
- **Group**: collection of users to apply common permissions
- **Role**: temporary credentials that can be assumed by users or services
- **Policy**: set of permissions expressed as instructions

### AWS IAM accounts
- **Root User**: has full account access, including the ability to delete the account. Do not use for daily activities.
- **IAM User**: can access only the AWS features and resources granted by permissions assigned to them.

## Part 2: AWS Policies and Roles

### Types of AWS Policies
1. **Identity-based policy**: attached to users, groups, or roles.
2. **Permission boundary**: maximum permissions an identity can receive.
3. **Resource-based policy**: attached directly to resources like S3 buckets or EC2 instances.
4. **Service Control Policy (SCP)**: limits services that can be used by IAM entities within an AWS Organization.
5. **Resource Control Policy (RCP)**: limits resources that can be accessed by IAM entities within an AWS Organization.
6. **Session policy**: used with temporary credentials in SDKs, AWS CLI, or AWS CDK.
7. **ACLs (Access Control Lists)**: resource-level controls used in services such as S3 or VPC.

## Demos

### Demo 1: IAM Users and Policies
1. Create two IAM users.
2. Assign policies to each user.
3. Modify one user policy and observe whether it affects the other user.

### Demo 2: EC2 Access to S3
1. Create or use an existing EC2 instance.
2. On the EC2 instance, run:

   ```bash
   aws s3 ls
   ```

   If you see an access error, you can fix it in one of two ways.

**Option A:**
- Run `aws configure` on the EC2 instance and provide AWS access keys.

**Option B:**
- Create an IAM role with the required S3 permissions (for example, AmazonS3ReadOnlyAccess).
- Attach that role to the EC2 instance.
- Then run:

   ```bash
   aws s3 ls
   ```

You should now see the S3 bucket listing.

