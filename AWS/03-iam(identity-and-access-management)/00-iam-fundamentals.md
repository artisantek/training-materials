# Authentication and Authorization Fundamentals


### **Authentication (AuthN)**

Authentication is the process of verifying your identity. It answers the question: **"Who are you?"**

You prove your identity by providing credentials that only you should know or possess, such as:

- A username and password
- An access key
- A multi-factor authentication (MFA) code from your phone
- A biometric scan (fingerprint, face ID)

Authentication is the **first step**—the "front door" of security. If you fail authentication, you cannot access the system at all.


### **Authorization (AuthZ)**

Authorization is the process of granting permission to access specific resources or perform certain actions. It answers the question: **"What are you allowed to do?"**

Authorization occurs **after** successful authentication. It is based on the policies, roles, or permissions assigned to your now-verified identity. Authorization determines things like:

- Can you read a file?
- Can you create a new server?
- Can you delete a user?

> **Note:** You can be authenticated but have zero authorization to do anything.


<img src="images/auth-vs-auth.png" alt="Authentication vs Authorization" width="400"/>

---

# How IAM Works: Authentication vs Authorization

  ![How IAM Works](images/How-IAM-works.png)


Let's break down the diagram above:

#### **Part 1: The Login Process (Top Section)**

- **User & Login:**  
  The process starts when a user attempts to log in by providing credentials (such as a username and password) to prove their identity.

- **AWS IAM: The Security System:**  
  The login request is sent to the central AWS IAM service. IAM performs two key functions:
  1. **Authentication:** IAM first checks if the user's credentials are correct.
  2. **Authorization:** IAM is also responsible for checking what permissions the user has (explained in the next step).

- **Authentication Arrow:**  
  The arrow labeled "Authentication" leaving the IAM box shows that authentication was successful. The user's identity is now confirmed.

#### **Part 2: The Authenticated User and Their Actions (Bottom Section)**

- **Authenticated User:**  
  The green box represents the user after a successful login. AWS now knows who the user is, but this does **not** mean they can do everything.

- **Accessing Services (Authorization in Action):**  
  When the authenticated user tries to perform any action (such as accessing EC2, S3, or Lambda), IAM performs an **authorization** check to determine if the user has permission for that action.

> **In summary:**  
> Authentication answers **"Who are you?"**  
> Authorization answers **"What are you allowed to do?"**  
> Both steps are enforced by IAM, as shown in the diagram above.

---

# IAM (Identity and Access Management) Fundamentals

This comprehensive guide covers AWS Identity and Access Management (IAM) fundamentals, including users, groups, roles, policies, and security best practices.

## What is IAM and Why Do We Need It?
AWS IAM is a web service that helps you securely control access to AWS resources. It enables you to manage authentication (who is signed in) and authorization (what resources they can access) for permissions through a unified system of users, groups, roles, and policies.

Think of **IAM (Identity and Access Management)** as the **security system** for your AWS account. Just like a building has security guards, key cards, and different access levels for different areas, IAM controls **who can access what** in your AWS environment.

---

## Understanding IAM: A Real-World Analogy

Think of **AWS IAM (Identity and Access Management)** as the master security system for a high-tech corporate headquarters. It’s not just a lock on the front door; it's the intelligent system that controls access to every room, file cabinet, and piece of equipment inside.

-   **🏢 The Corporate Building** is your **AWS Account**—the entire environment containing your valuable applications and data.
-   **👤 Employees** are your **IAM Users**. They are the individuals (like developers or administrators) who need long-term, persistent access to do their jobs.
-   **🗂️ Employee Departments** are **IAM Groups**. Just like you'd grant everyone in the 'Marketing' department access to the same file shares, you can put IAM Users into groups (e.g., `Developers`, `Testers`, `Admins`) to manage their permissions collectively.
-   **🎭 Visitor & Contractor Badges** are **IAM Roles**. You wouldn't give a visitor a permanent employee keycard. A role is a temporary set of permissions that anyone (or any application) can assume to perform a specific task and then give up. This is perfect for granting temporary or cross-account access.
-   **📜 The Rules on the Badge** are **IAM Policies**. A policy is a document that explicitly defines what doors a badge can open. It answers the question, "Is this person *authorized* to do this?"

---

## Why IAM is Essential for Your AWS Environment

IAM is not just a feature; it's a foundational pillar of a well-architected AWS cloud. Here’s why it’s so critical:

-   **🔐 Enforce a Strong Security Posture**
    IAM operates on a "deny by default" model. This means no one can do anything until you explicitly grant them permission, creating a secure foundation for your entire account.

-   **🎯 Implement the Principle of Least Privilege**
    Don't just give a developer access to all of your databases. IAM lets you grant the *minimum* permissions necessary for a user to perform their job—down to a single action on a specific resource. This granular control dramatically reduces your security risk.

-   **👥 Simplify Access Management at Scale**
    Instead of manually assigning permissions to 20 engineers one by one, place them in a `Developers` group and apply policies to the group. When a new developer joins, just add them to the group to grant them the same permissions instantly.

-   **🔄 Securely Grant Temporary Credentials**
    Hardcoding access keys in applications is a major security risk. With IAM Roles, your applications can request temporary credentials that automatically expire, eliminating the danger of leaked long-term keys.

-   **📊 Enable Auditing and Compliance**
    IAM integrates with AWS CloudTrail to log every single action taken in your account. This provides a detailed audit trail of "who did what, and when," which is critical for security analysis, troubleshooting, and meeting regulatory compliance requirements.

-   **💰 Prevent Accidental and Malicious Spending**
    By restricting who has permission to create or modify resources, you prevent costly configuration mistakes and unauthorized usage that can lead to unexpected and significant bills.

## Core IAM Components

<img src="images/IAM-types.png" alt="IAM Types" width="600"/>

## Principals ("Who")
Principals are entities that can make requests to AWS services.

### 1. **Users**
- **Definition:** Individual identities with permanent credentials (username/password or access keys).
- **Importance:**
  - Represent actual people or applications.
  - Each user has unique credentials.
  - Can be assigned directly to policies or added to groups.
- **Key Points:**
  - Follow the principle of least privilege.
  - Enable MFA for enhanced security.
  - Rotate access keys regularly.
  - Maximum **5,000 users** per AWS account.

### 2. **User Groups**
- **Definition:** Collections of IAM users that share the same permissions.
- **Importance:**
  - Simplifies permission management.
  - Ensures consistent access control.
  - Makes onboarding/offboarding easier.
- **Key Points:**
  - Groups cannot be nested.
  - Users can belong to multiple groups.
  - Groups cannot be principals in resource-based policies.
  - Maximum **300 groups** per account.

### 3. **Roles**
- **Definition:** Temporary identities with no permanent credentials that can be assumed.
- **Importance:**
  - Enable secure cross-account access.
  - Allow EC2 instances to access AWS services.
  - Support federation with external identity providers.
- **Key Points:**
  - Use STS (Security Token Service) for temporary credentials.
  - Can be assumed by users, applications, or AWS services.
  - Include trust policies defining who can assume them.
  - Session duration configurable (15 min to 12 hours).

---

## Policies ("What")
Policies are JSON documents that define permissions.

### 1. **Identity-based Policies**
Attached to users, groups, or roles to grant permissions.

#### a) **AWS Managed Policies**
- **Definition:** Pre-built policies created and maintained by AWS.
- **Importance:**
  - Cover common use cases.
  - Automatically updated by AWS.
  - Good starting point for permissions.
- **Key Points:**
  - Cannot be modified.
  - Examples: `AdministratorAccess`, `ReadOnlyAccess`.
  - Versioned and tracked by AWS.

#### b) **Customer Managed Policies**
- **Definition:** Custom policies created and maintained by you.
- **Importance:**
  - Tailored to specific organizational needs.
  - Reusable across multiple entities.
  - Version controlled.
- **Key Points:**
  - Maximum 5 versions retained.
  - Can be attached to multiple entities.
  - More flexible than inline policies.
  - Maximum 6,144 characters per policy.

#### c) **Inline Policies**
- **Definition:** Policies directly embedded into a single user, group, or role.
- **Importance:**
  - One-to-one relationship with identity.
  - Deleted when identity is deleted.
  - Useful for specific, unique permissions.
- **Key Points:**
  - Not reusable.
  - Harder to manage at scale.
  - Use sparingly for exceptional cases.

### 2. **Resource-based Policies**
- **Definition:** Policies attached directly to AWS resources (like S3 buckets, SQS queues).
- **Importance:**
  - Enable cross-account access.
  - Define who can access specific resources.
  - Complement identity-based policies.
- **Key Points:**
  - Always inline policies.
  - Specify principal in the policy.
  - Not all services support them.
  - Examples: S3 bucket policies, Lambda function policies.

---

## Important Best Practices and Considerations

### 1. **Security Best Practices**
- Enable MFA for all human users.
- Use roles instead of sharing credentials.
- Grant least privilege access.
- Regularly audit and remove unused credentials.
- Use policy conditions for additional security.

### 2. **Policy Evaluation Logic**
- **Explicit Deny > Explicit Allow > Implicit Deny**
- Both identity-based and resource-based policies are evaluated.
- Permission boundaries can limit maximum permissions.

### 3. **Common Use Cases**
- **Cross-account access:** Use roles with trust relationships.
- **EC2 instance access:** Use instance profiles with roles.
- **Temporary access:** Use STS to generate temporary credentials.
- **Federation:** Integrate with SAML, OIDC, or Active Directory.

### 4. **Monitoring and Compliance**
- Use CloudTrail for API logging.
- Enable Access Analyzer for policy validation.
- Review IAM credential reports regularly.
- Use AWS Config for compliance checking.

### 5. **Limitations to Remember**
- IAM is eventually consistent (changes may take time to propagate).
- Some services have service-specific authorization (like S3 ACLs).
- IAM is a global service (not region-specific).
- Policy size limits exist (be concise in policy documents).





## 1. 👤 IAM Users

### What are IAM Users?
**IAM Users** represent **individual people or applications** that need access to your AWS account.

### Key Characteristics:
- **🔑 Unique Identity**: Each user has a unique name and credentials
- **📧 Permanent**: Users are long-term entities (unlike roles)
- **🚪 Multiple Access Types**: Can have console access, programmatic access, or both
- **👤 One-to-One**: One user per person (don't share user accounts)

### User Access Types:

#### **🖥️ Console Access**
- Access to AWS Web Console (browser-based)
- Requires username and password
- Can enable MFA (Multi-Factor Authentication)

#### **💻 Programmatic Access**
- Access via AWS CLI, SDKs, or APIs
- Uses Access Key ID and Secret Access Key
- No password needed

#### **🔗 Both Access Types**
- User can have both console and programmatic access
- Most flexible option for developers

### Example Use Cases:
```
👨‍💼 John (Developer) - Console + Programmatic access
👩‍💼 Sarah (Manager) - Console access only  
🤖 BackupApp - Programmatic access only
👨‍🔧 DevOps Team - Console + Programmatic access
```

## 2. 👥 IAM Groups

### What are IAM Groups?
**IAM Groups** are **collections of users** that need similar permissions. Instead of assigning permissions to each user individually, you assign them to groups.

### Benefits of Groups:
- **⚡ Efficiency**: Assign permissions once to group, not to each user
- **🔄 Easy Management**: Add/remove users from groups easily
- **📊 Organization**: Organize users by job function or department
- **🛡️ Consistency**: Ensure users with similar roles have identical permissions

### Group Examples:

#### **👨‍💻 Developers Group**
- Can create and manage EC2 instances
- Can access development S3 buckets
- Can view CloudWatch logs
- **Cannot** delete production resources

#### **👨‍💼 Managers Group**
- Can view all resources (read-only)
- Can access billing information
- Can create cost reports
- **Cannot** modify resources

#### **🔧 DevOps Group**
- Full access to infrastructure services
- Can manage IAM users and groups
- Can access production environments
- Can manage security groups

### Group Hierarchy Example:
```
🏢 Company AWS Account
├── 👥 Developers
│   ├── 👤 John (Frontend Developer)
│   ├── 👤 Alice (Backend Developer)
│   └── 👤 Bob (Full-Stack Developer)
├── 👥 DevOps
│   ├── 👤 Sarah (DevOps Engineer)
│   └── 👤 Mike (Cloud Architect)
└── 👥 Managers
    ├── 👤 Lisa (Engineering Manager)
    └── 👤 Tom (Product Manager)
```

## 3. 🎭 IAM Roles

### What are IAM Roles?
**IAM Roles** provide **temporary permissions** that can be assumed by users, applications, or AWS services.

### Key Characteristics:
- **⏰ Temporary**: Permissions are granted for a limited time
- **🔄 Assumable**: Can be assumed by different entities
- **🛡️ Secure**: No long-term credentials to manage
- **🎯 Specific Purpose**: Designed for specific tasks or services

### When to Use Roles:

#### **🤖 For AWS Services**
```
Example: EC2 instance needs to access S3
❌ Bad: Store access keys on the EC2 instance (insecure)
✅ Good: Attach IAM role to EC2 instance (secure)
```

#### **👥 For Cross-Account Access**
```
Example: Contractor needs temporary access to your account
❌ Bad: Create a permanent user account
✅ Good: Create a role they can assume temporarily
```

#### **🔄 For Applications**
```
Example: Application running on EC2 needs database access
❌ Bad: Hard-code database credentials in application
✅ Good: Use IAM role to grant temporary access
```

### Role Examples:

#### **📊 EC2-S3-Access Role**
- **Purpose**: Allow EC2 instances to read/write S3 buckets
- **Trust Policy**: EC2 service can assume this role
- **Permissions**: S3 read/write access

#### **🔄 CrossAccount-DevAccess Role**
- **Purpose**: Allow developers from another AWS account to access resources
- **Trust Policy**: Specific external AWS account
- **Permissions**: Limited development resource access

#### **🔧 Lambda-Execution Role**
- **Purpose**: Allow Lambda function to write logs and access other services
- **Trust Policy**: Lambda service can assume this role
- **Permissions**: CloudWatch Logs + specific service access

## 4. 📋 IAM Policies

### What are IAM Policies?
**IAM Policies** are **JSON documents** that define **what actions are allowed or denied** on which resources.

### Policy Structure:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*"
    }
  ]
}
```

### Policy Components:

#### **📝 Effect**
- **`Allow`**: Grant permission
- **`Deny`**: Explicitly deny permission (overrides Allow)

#### **🎯 Action**
- Specific API calls that are allowed/denied
- Examples: `s3:GetObject`, `ec2:RunInstances`, `iam:CreateUser`

#### **🎯 Resource**
- Specific AWS resources the policy applies to
- Uses ARN (Amazon Resource Name) format
- Examples: S3 buckets, EC2 instances, IAM users

#### **🌍 Condition** (Optional)
- Additional restrictions based on context
- Examples: Time-based access, IP address restrictions, MFA requirements

### Policy Types:

#### **1. 🔗 Managed Policies**
- **AWS Managed**: Created and maintained by AWS
- **Customer Managed**: Created and maintained by you
- **Reusable**: Can be attached to multiple users, groups, or roles

#### **2. 📎 Inline Policies**
- **Embedded**: Directly embedded in a single user, group, or role
- **One-to-One**: Tied to specific identity
- **Use Case**: Unique permissions for specific identity

### Common AWS Managed Policies:

#### **🔧 PowerUserAccess**
- **Description**: Full access except IAM management
- **Use Case**: Developers who need broad access but shouldn't manage users

#### **👁️ ReadOnlyAccess**
- **Description**: Read-only access to all AWS services
- **Use Case**: Managers, auditors, monitoring systems

#### **💰 Billing**
- **Description**: Access to billing and cost management
- **Use Case**: Finance team, cost management

#### **🛡️ SecurityAudit**
- **Description**: Read-only access to security-related information
- **Use Case**: Security team, compliance auditing

## 🔐 Authentication vs Authorization

### **🔑 Authentication** - "Who are you?"
- **Purpose**: Verify identity
- **Methods**: Username/password, access keys, MFA
- **Question**: Is this really John Smith?

### **🎯 Authorization** - "What can you do?"
- **Purpose**: Determine permissions
- **Methods**: IAM policies attached to users/groups/roles
- **Question**: Can John Smith create EC2 instances?

### Example Flow:
```
1. 🔑 Authentication: John logs in with username/password (verified)
2. 🎯 Authorization: Check John's policies (allowed to create EC2 instances)
3. ✅ Result: John can successfully launch an EC2 instance
```

## 🛡️ IAM Security Best Practices

### 1. 👤 **Root Account Security**
- **🔒 Secure Root Account**: Use MFA, don't use for daily tasks
- **👥 Create Admin Users**: Use IAM users for administrative tasks
- **🔑 Minimal Root Usage**: Only use root for account-level changes

### 2. 🔐 **Strong Authentication**
- **🔢 Enable MFA**: Multi-Factor Authentication for all users
- **🔄 Rotate Credentials**: Regularly rotate passwords and access keys
- **💪 Strong Passwords**: Enforce password complexity requirements

### 3. 🎯 **Least Privilege Principle**
- **📉 Minimal Permissions**: Grant only permissions needed for the job
- **⏰ Time-Limited**: Use temporary credentials when possible
- **🔍 Regular Review**: Periodically review and remove unnecessary permissions

### 4. 👥 **User Management**
- **🚫 No Shared Accounts**: One user per person
- **👥 Use Groups**: Assign permissions to groups, not individual users
- **🗑️ Remove Unused Users**: Delete accounts for people who leave

### 5. 📊 **Monitoring and Auditing**
- **📋 CloudTrail**: Log all API calls and user activities
- **🔍 Access Analyzer**: Identify unintended access to resources
- **📊 Regular Audits**: Review permissions and access patterns

## 💡 Common IAM Use Cases

### 1. 🏢 **Multi-Environment Setup**

#### **Scenario**: Company with Development, Staging, and Production environments

**Solution**:
```
👥 Groups by Environment:
├── Dev-Developers (full access to dev resources)
├── Staging-Testers (read/write staging, read-only prod)
└── Prod-Admins (full production access)

🎯 Policies by Environment:
├── Dev-FullAccess (EC2, S3, RDS in dev account)
├── Staging-TestAccess (deploy and test in staging)
└── Prod-AdminAccess (production management)
```

### 2. 🔄 **Cross-Account Access**

#### **Scenario**: Contractor needs temporary access to your AWS account

**Solution**:
```
🎭 Create IAM Role:
├── Trust Policy: Allow contractor's AWS account to assume role
├── Permission Policy: Limited access to specific resources
└── Time Limit: Set maximum session duration

🔗 Access Process:
1. Contractor assumes role from their AWS account
2. Gets temporary credentials (1-12 hours)
3. Access automatically expires
```

### 3. 🤖 **Service-to-Service Access**

#### **Scenario**: EC2 application needs to access S3 and RDS

**Solution**:
```
🎭 Create Service Role:
├── Trust Policy: Allow EC2 service to assume role
├── S3 Policy: Read/write specific S3 buckets
└── RDS Policy: Connect to specific database

📎 Attach to Instance:
├── Attach role to EC2 instance
├── Application uses instance metadata to get credentials
└── No hardcoded credentials in code
```

### 4. 👥 **Department-Based Access**

#### **Scenario**: Different departments need different AWS access levels

**Solution**:
```
🏢 Engineering Department:
├── 👥 Frontend-Developers Group
├── 👥 Backend-Developers Group
└── 👥 DevOps-Engineers Group

🏢 Business Department:
├── 👥 Product-Managers Group
├── 👥 Finance-Team Group
└── 👥 Marketing-Analytics Group

📋 Each group has tailored policies for their needs
```

## 🔧 IAM Policy Examples

### 1. 📖 **Read-Only S3 Access**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::company-reports",
        "arn:aws:s3:::company-reports/*"
      ]
    }
  ]
}
```

### 2. ⏰ **Time-Based Access**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "ec2:*",
      "Resource": "*",
      "Condition": {
        "DateGreaterThan": {
          "aws:CurrentTime": "2024-01-01T00:00:00Z"
        },
        "DateLessThan": {
          "aws:CurrentTime": "2024-12-31T23:59:59Z"
        }
      }
    }
  ]
}
```

### 3. 🌐 **IP-Based Access**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "IpAddress": {
          "aws:SourceIp": [
            "203.0.113.0/24",
            "198.51.100.0/24"
          ]
        }
      }
    }
  ]
}
```

### 4. 🔐 **MFA Required**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "ec2:TerminateInstances",
      "Resource": "*",
      "Condition": {
        "Bool": {
          "aws:MultiFactorAuthPresent": "true"
        },
        "NumericLessThan": {
          "aws:MultiFactorAuthAge": "3600"
        }
      }
    }
  ]
}
```

## 🚨 Common IAM Mistakes to Avoid

### 1. ❌ **Overly Permissive Policies**
```json
// Don't do this - gives full access to everything
{
  "Effect": "Allow",
  "Action": "*",
  "Resource": "*"
}
```

**✅ Better Approach**: Grant specific permissions needed
```json
{
  "Effect": "Allow",
  "Action": [
    "s3:GetObject",
    "s3:PutObject"
  ],
  "Resource": "arn:aws:s3:::my-app-bucket/*"
}
```

### 2. ❌ **Hardcoded Credentials**
```python
# Don't do this - credentials in code
aws_access_key = "AKIAIOSFODNN7EXAMPLE"
aws_secret_key = "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
```

**✅ Better Approach**: Use IAM roles or environment variables

### 3. ❌ **Shared User Accounts**
```
// Don't do this - multiple people sharing one account
shared-dev-account (used by 5 developers)
```

**✅ Better Approach**: Individual accounts in groups
```
john-developer (in Developers group)
alice-developer (in Developers group)
bob-developer (in Developers group)
```

### 4. ❌ **Never Rotating Credentials**
- Access keys used for years without rotation
- Same password for months/years

**✅ Better Approach**: 
- Rotate access keys every 90 days
- Enforce password rotation policy
- Use temporary credentials when possible

## 🔍 Troubleshooting IAM Issues

### 1. **"Access Denied" Errors**

#### **🔍 Check These Areas:**
1. **User Permissions**: Does the user have the required policy?
2. **Resource Permissions**: Does the policy specify the correct resource ARN?
3. **Explicit Deny**: Is there a policy with "Deny" that overrides "Allow"?
4. **Service Limits**: Has the user hit service quotas?

#### **🛠️ Debugging Tools:**
- **IAM Policy Simulator**: Test policies before applying
- **CloudTrail**: See what API calls were made and why they failed
- **Access Analyzer**: Identify permission issues

### 2. **Role Assumption Problems**

#### **Common Issues:**
1. **Trust Policy**: Role doesn't trust the entity trying to assume it
2. **Permissions**: Entity doesn't have permission to assume roles
3. **External ID**: Missing or incorrect external ID for cross-account access

#### **🔧 Solution Steps:**
1. Check the role's trust policy
2. Verify the assuming entity has `sts:AssumeRole` permission
3. Confirm any required conditions are met

### 3. **MFA Issues**

#### **Common Problems:**
1. **Token Sync**: MFA device not synchronized
2. **Policy Conditions**: MFA required but not provided
3. **Time Skew**: System time doesn't match MFA device time

## 📊 IAM Monitoring and Compliance

### 1. **📋 CloudTrail Integration**
- **Log All Actions**: Every API call is logged
- **User Attribution**: See which user performed each action
- **Security Analysis**: Detect unusual access patterns

### 2. **🔍 Access Analyzer**
- **External Access**: Identifies resources shared outside your account
- **Unused Access**: Finds permissions that are never used
- **Policy Validation**: Checks policies for security issues

### 3. **📊 Credential Reports**
- **User Status**: See all users and their credential status
- **Last Activity**: When users last accessed AWS
- **MFA Status**: Which users have MFA enabled

### 4. **🚨 Config Rules**
- **Compliance Monitoring**: Automated checks for IAM best practices
- **Root Access**: Alert when root account is used
- **MFA Compliance**: Ensure all users have MFA enabled

## ✅ Prerequisites

- **☁️ AWS Account**: Active AWS account with admin access
- **💻 Basic JSON**: Understanding of JSON format for policies
- **🔒 Security Awareness**: Understanding of security principles
- **👥 Organization Knowledge**: Understanding of your team/company structure

## 🔒 Security Reminders

⚠️ **Critical Security Guidelines:**

- **🔑 Protect Root Account**: Use MFA, don't use for daily tasks
- **👤 Individual Accounts**: One user per person, no sharing
- **🔄 Regular Rotation**: Rotate credentials regularly
- **📉 Least Privilege**: Grant minimum permissions necessary
- **📊 Monitor Activity**: Use CloudTrail and review access regularly
- **🔐 MFA Everywhere**: Enable MFA for all users
- **🗑️ Clean Up**: Remove unused users, roles, and policies

## 🎯 Next Steps

After understanding IAM fundamentals:

1. **🛠️ Hands-On Practice**: Create users, groups, and roles in your account
2. **📋 Policy Creation**: Write custom policies for your specific needs
3. **🔄 Automation**: Learn about IAM automation with CloudFormation/Terraform
4. **🔍 Advanced Topics**: Explore IAM Identity Center, SAML federation
5. **📊 Monitoring Setup**: Implement comprehensive IAM monitoring
6. **🎓 Certification**: Consider AWS security-focused certifications
7. **📚 Advanced Learning**: Explore cross-account access patterns and enterprise IAM strategies 