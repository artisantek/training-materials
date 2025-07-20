# Authentication and Authorization Fundamentals

## **Authentication (AuthN)**

Authentication is the process of verifying your identity. It answers the question: **"Who are you?"**

You prove your identity by providing credentials that only you should know or possess, such as:

- A username and password
- An access key
- A multi-factor authentication (MFA) code from your phone
- A biometric scan (fingerprint, face ID)

Authentication is the **first step**—the "front door" of security. If you fail authentication, you cannot access the system at all.

## **Authorization (AuthZ)**

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

## **Part 1: The Login Process (Top Section)**

- **User & Login:**  
  The process starts when a user attempts to log in by providing credentials (such as a username and password) to prove their identity.

- **AWS IAM: The Security System:**  
  The login request is sent to the central AWS IAM service. IAM performs two key functions:
  1. **Authentication:** IAM first checks if the user's credentials are correct.
  2. **Authorization:** IAM is also responsible for checking what permissions the user has (explained in the next step).

- **Authentication Arrow:**  
  The arrow labeled "Authentication" leaving the IAM box shows that authentication was successful. The user's identity is now confirmed.

## **Part 2: The Authenticated User and Their Actions (Bottom Section)**

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

## Understanding IAM: A Real-World Analogy

Think of **AWS IAM (Identity and Access Management)** as the master security system for a high-tech corporate headquarters. It's not just a lock on the front door; it's the intelligent system that controls access to every room, file cabinet, and piece of equipment inside.

- **🏢 The Corporate Building** is your **AWS Account**—the entire environment containing your valuable applications and data.
- **👤 Employees** are your **IAM Users**. They are the individuals (like developers or administrators) who need long-term, persistent access to do their jobs.
- **🗂️ Employee Departments** are **IAM Groups**. Just like you'd grant everyone in the 'Marketing' department access to the same file shares, you can put IAM Users into groups (e.g., `Developers`, `Testers`, `Admins`) to manage their permissions collectively.
- **🎭 Visitor & Contractor Badges** are **IAM Roles**. You wouldn't give a visitor a permanent employee keycard. A role is a temporary set of permissions that anyone (or any application) can assume to perform a specific task and then give up. This is perfect for granting temporary or cross-account access.
- **📜 The Rules on the Badge** are **IAM Policies**. A policy is a document that explicitly defines what doors a badge can open. It answers the question, "Is this person *authorized* to do this?"

## Why IAM is Essential for Your AWS Environment

IAM is not just a feature; it's a foundational pillar of a well-architected AWS cloud. Here's why it's so critical:

- **🔐 Enforce a Strong Security Posture**  
  IAM operates on a "deny by default" model. This means no one can do anything until you explicitly grant them permission, creating a secure foundation for your entire account.

- **🎯 Implement the Principle of Least Privilege**  
  Don't just give a developer access to all of your databases. IAM lets you grant the *minimum* permissions necessary for a user to perform their job—down to a single action on a specific resource. This granular control dramatically reduces your security risk.

- **👥 Simplify Access Management at Scale**  
  Instead of manually assigning permissions to 20 engineers one by one, place them in a `Developers` group and apply policies to the group. When a new developer joins, just add them to the group to grant them the same permissions instantly.

- **🔄 Securely Grant Temporary Credentials**  
  Hardcoding access keys in applications is a major security risk. With IAM Roles, your applications can request temporary credentials that automatically expire, eliminating the danger of leaked long-term keys.

- **📊 Enable Auditing and Compliance**  
  IAM integrates with AWS CloudTrail to log every single action taken in your account. This provides a detailed audit trail of "who did what, and when," which is critical for security analysis, troubleshooting, and meeting regulatory compliance requirements.

- **💰 Prevent Accidental and Malicious Spending**  
  By restricting who has permission to create or modify resources, you prevent costly configuration mistakes and unauthorized usage that can lead to unexpected and significant bills.

---

# Core IAM Components

<img src="images/IAM-types.png" alt="IAM Types" width="600"/>

## Principals ("Who")

Principals are entities that can make requests to AWS services.

## 1. 👤 IAM Users: The Foundation of Identity in AWS

An **IAM User** is an entity you create in AWS to represent a person (like an administrator or developer) or an application that needs long-term, persistent access to your AWS resources. Each user has a unique identity and its own set of security credentials.

### Key Characteristics of an IAM User

- **Unique Identity:** Every IAM user has a unique name within an AWS account and is assigned its own set of security credentials.
- **Permanent Credentials:** Unlike temporary roles, users have long-term credentials that remain valid until they are manually changed or revoked.
- **Permissions Model:** Permissions can be assigned to a user in two ways:
  1. Attaching a policy directly to the user.
  2. Adding the user to one or more IAM Groups that already have policies attached. (This is the recommended best practice).
- **Principle of One-to-One:** It is a security best practice to create one IAM user per person or application. **Credentials should never be shared.**
- **Service Quota:** You can create up to 5,000 IAM users per AWS account.

### User Access Types

An IAM user can be granted one or both of the following types of access:

#### **🖥️ Console Access (for Humans)**

This allows a user to sign in to the AWS Management Console through a web browser.

- **Credentials:** Requires a unique **Username** and **Password**.
- **Security:** Multi-Factor Authentication (MFA) should always be enabled for enhanced security.

#### **💻 Programmatic Access (for Applications & Tools)**

This allows a user to interact with AWS services through the Command Line Interface (CLI), SDKs, or direct API calls.

- **Credentials:** Uses an **Access Key ID** and a **Secret Access Key**.
- **Usage:** Ideal for applications, scripts, or developer tools that need to automate tasks in AWS.

A user can be configured with console access, programmatic access, or both, depending on their role and needs.

### Security Best Practices for IAM Users

- **Enable MFA:** Always enforce Multi-Factor Authentication for all users with console access.
- **Apply Least Privilege:** Grant users only the minimum permissions they need to perform their job. Avoid giving broad permissions like `*:*`.
- **Use Groups for Permissions:** Instead of attaching policies directly to users, organize users into groups based on their job function and apply policies to the groups.
- **Rotate Access Keys:** Regularly rotate programmatic access keys to limit the risk posed by a potential leak.
- **Avoid Using the Root User:** The account's root user should not be used for daily tasks. Create dedicated IAM users for all administrative work.

### Example Use Cases

| User / Application | Access Type Needed | Justification |
|-------------------|-------------------|---------------|
| **John (Developer)** | Console + Programmatic | Needs to browse the console and run CLI commands. |
| **Sarah (Manager)** | Console Only | Needs to view dashboards and reports, not code. |
| **🤖 BackupApp** | Programmatic Only | An automated script that needs API access to S3. |

## 2. 👥 IAM Groups: Managing Permissions at Scale

An **IAM Group** is a collection of IAM users. Instead of attaching permission policies directly to individual users, you can attach them to a group. Any user placed in that group automatically inherits the permissions assigned to it, providing a powerful and efficient way to manage access for multiple users at once.

### How Groups Work: Key Characteristics

Understanding these rules is essential for using groups effectively:

- **A User Can Belong to Multiple Groups:** A single user can be a member of up to 10 different groups, inheriting the combined permissions of all of them.
- **Groups Cannot Be Nested:** You cannot create a group inside another group. The hierarchy is flat.
- **Groups Are for Users Only:** You can only place IAM users in a group. You cannot place other groups or IAM roles in a group.
- **Groups Are Not True Identities:** A group itself cannot be identified as a principal in a policy. This means you cannot write a policy that grants access *to a group* (e.g., in an S3 bucket policy). The permissions are always evaluated based on the user who inherits them from the group.
- **Service Quotas:** An AWS account can have a maximum of 300 IAM groups by default.

### The Benefits of Using IAM Groups

- **⚡ Simplified and Efficient Administration:** Manage permissions for dozens or hundreds of users by modifying a single group policy instead of editing each user's permissions individually.
- **🛡️ Consistent and Standardized Permissions:** Ensure that all users with the same job function (e.g., all developers) have the exact same set of permissions, reducing the risk of human error and security gaps.
- **🔄 Streamlined Onboarding and Offboarding:** When a new employee joins, simply add them to the appropriate group(s) to grant them access instantly. When they leave, removing them from the groups revokes all their inherited permissions in one step.
- **📊 Clear Organizational Structure:** Organize users based on job roles, departments, or project teams, making your AWS account easier to understand and audit.

### Practical Group Examples

#### **👨‍💻 Developers Group**

Aimed at developers who need to build and test applications.

- **Can:** Create and manage development EC2 instances, access development S3 buckets, and view CloudWatch logs.
- **Cannot:** Modify IAM policies or delete production resources.

#### **👨‍💼 Managers Group**

For managers or stakeholders who need visibility without modification rights.

- **Can:** View all resources (read-only access), access billing dashboards, and create cost reports.
- **Cannot:** Create, modify, or delete any resources.

#### **🔧 DevOps Group**

For engineers responsible for infrastructure and deployment.

- **Can:** Have full access to infrastructure services (EC2, VPC, etc.), manage IAM users and groups, and deploy applications to production environments.

### Example Hierarchy in an AWS Account

This shows how individual users are organized into functional groups.

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

## 3. 🎭 IAM Roles: Secure, Temporary Access in AWS

An **IAM Role** is a secure way to grant temporary permissions to entities that you trust. Unlike an IAM user, a role does not have its own permanent credentials (like a password or access keys). Instead, when an entity assumes a role, it receives temporary security credentials for that session.

Roles are the preferred security mechanism for nearly all programmatic access in AWS because they eliminate the need to manage and store long-term keys.

### Key Characteristics of an IAM Role

- **🛡️ Secure and Temporary:** Permissions are granted for a limited, configurable duration (from 15 minutes to 12 hours). Once the session expires, the credentials become invalid, drastically reducing the risk of a leaked key.
- **🔄 Assumable:** Roles are designed to be assumed by trusted entities (principals), such as IAM users, applications running on AWS services, or external users you've federated.
- **🎯 Specific Purpose:** Roles are typically created for a specific task, adhering to the principle of least privilege.
- **Powered by AWS STS:** The **AWS Security Token Service (STS)** is the service that provides the temporary credentials when a role is assumed.

A role is defined by two critical policies:

1. **Trust Policy:** Defines *who* is allowed to assume the role (e.g., the EC2 service, a specific AWS account, or a federated user).
2. **Permissions Policy:** Defines *what* actions the entity can perform and on which resources once it has assumed the role.

### When to Use an IAM Role (Key Use Cases)

Using roles is a security best practice. Here are the most common scenarios:

#### **1. For AWS Services Accessing Other AWS Services**

This is the most common use case. An AWS service (like an EC2 instance or a Lambda function) needs permission to interact with other services (like S3 or DynamoDB).

- **❌ Bad:** Hard-coding an IAM user's access keys directly on the EC2 instance. This is a major security risk.
- **✅ Good:** Attaching an IAM Role to the EC2 instance. The application running on the instance can automatically retrieve temporary credentials from the role, without any keys being stored on disk.

#### **2. For Cross-Account Access**

When you need to grant users or services from another AWS account access to resources in your account.

- **❌ Bad:** Creating a permanent IAM user in your account for an external contractor or a partner company. You then have to manage their credentials.
- **✅ Good:** Creating a role that trusts the external AWS account. Users in that account can then assume the role to gain temporary access to your specified resources.

#### **3. For Identity Federation**

When you want to grant access to users from an external identity provider (IdP).

- **Corporate Identity (e.g., Active Directory):** Employees can sign in with their existing corporate credentials and assume a role in AWS, without needing a separate IAM user.
- **Web Identity (e.g., Login with Google, Facebook):** Users of a mobile or web app can sign in with a public identity provider and be mapped to an IAM role that grants them temporary access to your AWS backend.

### Practical Role Examples

#### **📊 EC2-S3-Access-Role**

- **Purpose:** Allow an EC2 instance to read objects from and write objects to a specific S3 bucket.
- **Trust Policy:** Trusts the EC2 service (`ec2.amazonaws.com`) to assume this role.
- **Permissions Policy:** Grants actions like `s3:GetObject` and `s3:PutObject` on the target bucket.

#### **🔄 CrossAccount-ReadOnly-Role**

- **Purpose:** Allow developers from a partner's AWS account to view (but not modify) your development resources.
- **Trust Policy:** Trusts a specific external AWS Account ID.
- **Permissions Policy:** Grants read-only actions (e.g., `ec2:DescribeInstances`, `s3:ListBucket`).

#### **🔧 Lambda-Execution-Role**

- **Purpose:** Allow a Lambda function to perform its tasks, such as writing logs and accessing a DynamoDB table.
- **Trust Policy:** Trusts the Lambda service (`lambda.amazonaws.com`) to assume this role.
- **Permissions Policy:** Grants permissions to create log streams in CloudWatch Logs and perform `GetItem`/`PutItem` actions on a specific DynamoDB table.

## 4. 📋 IAM Policies: The Language of Permissions

**IAM Policies** are the heart of authorization in AWS. They are formal documents, written in JSON, that explicitly define permissions. Every time a user or service makes a request in AWS, their permissions are evaluated based on the policies attached to their identity and the resource they are trying to access.

### The Anatomy of a Policy Document

All IAM policies are composed of one or more **statements**, each containing the following key elements:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowS3ReadAccess",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::my-example-bucket",
        "arn:aws:s3:::my-example-bucket/*"
      ],
      "Condition": {
        "IpAddress": {"aws:SourceIp": "203.0.113.0/24"}
      }
    }
  ]
}
```

- **`Effect`**: The outcome of the statement. This can be either `Allow` or `Deny`. An explicit `Deny` always overrides an `Allow`.
- **`Action`**: The specific API call(s) that are being permitted or denied (e.g., `ec2:StartInstances`, `s3:GetObject`).
- **`Resource`**: The specific AWS resource(s) that the action applies to, identified by their **Amazon Resource Name (ARN)**.
- **`Condition` (Optional)**: Specifies circumstances under which the policy is in effect. This adds a powerful layer of control, such as restricting access based on an IP address range, the time of day, or whether MFA was used.
- **`Sid` (Statement ID)**: A useful (but optional) identifier for the statement, making policies easier to read and debug.
- **`Version`**: The policy language version, which should always be set to `2012-10-17`.

### Types of Policies

There are two primary categories of IAM policies: Identity-based and Resource-based.

#### 1. Identity-Based Policies

These policies are attached directly to an IAM **identity** (a user, group, or role). They define what that identity is allowed to do. There are three kinds:

| Type | Definition | Key Characteristics & Use Cases |
|------|------------|--------------------------------|
| **AWS Managed Policies** | Pre-built policies created and managed by AWS for common use cases. | **Pros:** Easy to use, automatically updated by AWS. <br> **Cons:** Cannot be modified. <br> **Examples:** `AdministratorAccess`, `ReadOnlyAccess`, `PowerUserAccess`. |
| **Customer Managed Policies** | Custom policies that you create and manage in your AWS account. | **Pros:** Reusable, version-controlled, and flexible. The recommended approach for custom permissions. <br> **Cons:** You are responsible for maintenance. |
| **Inline Policies** | Policies embedded directly into a single user, group, or role. | **Pros:** Strict one-to-one relationship with an identity. <br> **Cons:** Not reusable, harder to manage at scale. Use sparingly for truly exceptional cases. |

#### 2. Resource-Based Policies

These policies are attached directly to an **AWS resource**, such as an S3 bucket, an SQS queue, or a Lambda function. They define who has permission to access *that specific resource*.

- **Key Function:** Their primary purpose is to enable secure, direct **cross-account access** without needing to create complex IAM roles.
- **`Principal` Element:** Unlike identity-based policies, a resource-based policy must specify a `Principal`—the account, user, role, or service that is allowed or denied access.
- **Inline Nature:** They are always inline policies, embedded directly into the resource's configuration.

### Common AWS Managed Policies and Their Use Cases

| Policy Name | Description | Best For |
|-------------|-------------|----------|
| **`AdministratorAccess`** | Grants full access to all AWS services and resources. | Your primary administrative users (not the root account). |
| **`PowerUserAccess`** | Grants full access *except* for the management of IAM and Organizations. | Developers who need broad access but should not be able to manage users. |
| **`ReadOnlyAccess`** | Grants read-only access to view all AWS services and resources. | Managers, auditors, and monitoring systems. |
| **`SecurityAudit`** | Grants read-only access to security-related services and configurations. | Your internal security team or external compliance auditors. |
| **`Billing`** | Grants permissions to view and manage billing and cost data. | Members of your finance or cost management team. |

---

# 🚨 Common IAM Mistakes to Avoid

## 1. ❌ **Overly Permissive Policies**

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

## 2. ❌ **Hardcoded Credentials**

```python
# Don't do this - credentials in code
aws_access_key = "AKIAIOSFODNN7EXAMPLE"
aws_secret_key = "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
```

**✅ Better Approach**: Use IAM roles or environment variables

## 3. ❌ **Shared User Accounts**

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

## 4. ❌ **Never Rotating Credentials**

- Access keys used for years without rotation
- Same password for months/years

**✅ Better Approach**:

- Rotate access keys every 90 days
- Enforce password rotation policy
- Use temporary credentials when possible

---

# 🔍 Troubleshooting IAM Issues

## 1. **"Access Denied" Errors**

### **🔍 Check These Areas:**

1. **User Permissions**: Does the user have the required policy?
2. **Resource Permissions**: Does the policy specify the correct resource ARN?
3. **Explicit Deny**: Is there a policy with "Deny" that overrides "Allow"?
4. **Service Limits**: Has the user hit service quotas?

### **🛠️ Debugging Tools:**

- **IAM Policy Simulator**: Test policies before applying
- **CloudTrail**: See what API calls were made and why they failed
- **Access Analyzer**: Identify permission issues

## 2. **Role Assumption Problems**

### **Common Issues:**

1. **Trust Policy**: Role doesn't trust the entity trying to assume it
2. **Permissions**: Entity doesn't have permission to assume roles
3. **External ID**: Missing or incorrect external ID for cross-account access

### **🔧 Solution Steps:**

1. Check the role's trust policy
2. Verify the assuming entity has `sts:AssumeRole` permission
3. Confirm any required conditions are met

---

# 🎯 Next Steps

After understanding IAM fundamentals:

1. **🛠️ Hands-On Practice**: Create users, groups, and roles in your account
2. **📋 Policy Creation**: Write custom policies for your specific needs
3. **🔄 Automation**: Learn about IAM automation with CloudFormation/Terraform
4. **🔍 Advanced Topics**: Explore IAM Identity Center, SAML federation
5. **📊 Monitoring Setup**: Implement comprehensive IAM monitoring
6. **🎓 Certification**: Consider AWS security-focused certifications
7. **📚 Advanced Learning**: Explore cross-account access patterns and enterprise IAM strategies

---

# 🛡️ IAM Security Best Practices

Securing your AWS environment starts with a robust Identity and Access Management (IAM) strategy. Following these best practices is essential for protecting your resources, controlling costs, and meeting compliance requirements.

## Foundational Security Principles

These are the core tenets that should guide all of your IAM configurations.

### 1. Secure Your AWS Account Root User

The root user has unrestricted access to your entire account. It must be protected above all else.

- **Enable MFA:** Immediately enable Multi-Factor Authentication (MFA) on the root user.
- **Create an Admin User:** For daily administrative tasks, create a separate IAM user with administrative privileges and use that instead.
- **Lock Away Root Credentials:** Do not use the root user for any routine work. Only use it for specific tasks that absolutely require it (e.g., changing account settings, closing the account).

### 2. Enforce Strong Authentication for All Users

- **Mandate MFA:** Require Multi-Factor Authentication for all human users to prevent unauthorized access from compromised passwords.
- **Enforce Strong Password Policies:** Configure a password policy that requires complexity (uppercase, lowercase, numbers, symbols) and a minimum length.
- **Rotate Credentials Regularly:** Regularly rotate passwords and programmatic access keys to limit the window of opportunity for a compromised credential.

### 3. Embrace the Principle of Least Privilege

Grant only the minimum permissions necessary for a user or service to perform its job.

- **Start with Minimal Permissions:** Begin with a restrictive policy and grant additional permissions only as needed, rather than starting with broad permissions and trying to pare them down.
- **Use Temporary Credentials:** Whenever possible, use IAM Roles to grant temporary, time-limited access instead of creating users with long-term keys.
- **Review and Refine Permissions:** Periodically review IAM policies and remove any permissions that are no longer required.

### 4. Organize Users and Permissions Effectively

- **Use Groups for Permissions:** Assign permissions to IAM Groups based on job functions (e.g., `Developers`, `Admins`, `Auditors`), and then place users into those groups. This is vastly more scalable than attaching policies to individual users.
- **One User per Person:** Never share IAM user accounts. Each individual should have their own unique user to ensure clear accountability and auditing.
- **Remove Unused Credentials:** Promptly disable or delete users and credentials for employees who have left the company or for applications that have been decommissioned.

## Monitoring, Auditing, and Validation

Continuously monitor your IAM configurations to ensure they remain secure.

- **Log All Activity with AWS CloudTrail:** Enable CloudTrail in all regions to capture a detailed log of every API call made in your account. This provides a crucial audit trail of "who did what, and when."
- **Identify Unintended Access with IAM Access Analyzer:** Use Access Analyzer to identify resources (like S3 buckets or IAM roles) that are shared with an external entity. It helps you discover and remediate overly permissive access.
- **Audit Your Security Posture Regularly:** Use IAM Credential Reports to get a list of all users and the status of their credentials (MFA, password age, access key age).
- **Check for Compliance with AWS Config:** Use AWS Config rules to continuously check whether your IAM policies and configurations comply with your organization's security standards.

## Understanding IAM Mechanics and Patterns

### Policy Evaluation Logic

Understanding how permissions are evaluated is critical. The logic always follows this order:

1. **Explicit Deny:** If any policy contains an explicit `Deny`, the request is denied. This always overrides any `Allow`.
2. **Explicit Allow:** If there is no `Deny`, and at least one policy contains an `Allow` for the action, the request is allowed.
3. **Default Deny (Implicit Deny):** If there is no `Deny` and no `Allow`, the request is denied by default.

### Common Architectural Patterns

- **EC2 Instance Access:** Use **IAM Roles for EC2** (via instance profiles) to grant applications running on an instance temporary, secure access to other AWS services.
- **Cross-Account Access:** Use **IAM Roles** with a trust policy that specifies another AWS account. This allows users from the trusted account to assume the role and access your resources temporarily.
- **Federation:** Integrate IAM with an external identity provider (like Active Directory via SAML, or Google/Facebook via OIDC) to allow users to sign in with their existing credentials and assume a role in AWS.

## Important Considerations and Limitations

- **Eventual Consistency:** IAM is a global service with high availability. Changes you make are replicated across AWS regions and are subject to eventual consistency, meaning they may take a short time to propagate everywhere.
- **IAM is a Global Service:** IAM users, groups, and roles are created globally; you do not create them in a specific region.
- **Service-Specific Authorization:** Some services, like Amazon S3, have their own resource-based authorization mechanisms (e.g., Bucket Policies, Access Control Lists) that work in conjunction with IAM policies.
- **Policy Size Limits:** Be aware that IAM entities (users, groups, roles) and policies have size limits. Keep policies concise and well-structured.







