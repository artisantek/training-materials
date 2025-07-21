# IAM User Creation with Access Keys - Step by Step Guide

This comprehensive guide walks you through creating an IAM user with programmatic access keys in AWS. You'll learn how to set up a new user, configure permissions, and generate secure access credentials for API/CLI access.

## What You'll Learn

- How to navigate the IAM dashboard
- Step-by-step user creation process
- How to configure access types and permissions
- Best practices for access key management
- Security considerations for programmatic access

## Prerequisites

- AWS account with administrative access
- Understanding of [IAM fundamentals](00-iam-fundamentals.md)
- Browser access to AWS Management Console

---

## 📋 Complete IAM User Creation Process

```mermaid
flowchart TD
    A[Start: AWS Console Login] -----> B[Navigate to IAM Dashboard]
    B -----> C[Access Users Section]
    C -----> D[Set Permissions/Policies]
    D -----> E[Review and Create User]
    E -----> F[Review User Configuration]
    F -----> G[Add the Credentials]
    G -----> H[✅ IAM User Creation Complete]
    
    B@{ img: "https://raw.githubusercontent.com/artisantek/training-materials/master/AWS/03-iam(identity-and-access-management)/images/user-1.png", h: 936, w: 2086, pos: "t"}
    C@{ img: "https://raw.githubusercontent.com/artisantek/training-materials/master/AWS/03-iam(identity-and-access-management)/images/user-2.png", h: 1088, w: 2300, pos: "t"}
    D@{ img: "https://raw.githubusercontent.com/artisantek/training-materials/master/AWS/03-iam(identity-and-access-management)/images/user-3.png", h: 1274, w: 2008, pos: "t"}
    E@{ img: "https://raw.githubusercontent.com/artisantek/training-materials/master/AWS/03-iam(identity-and-access-management)/images/user-4.png", h: 708, w: 2002, pos: "t"}
    F@{ img: "https://raw.githubusercontent.com/artisantek/training-materials/master/AWS/03-iam(identity-and-access-management)/images/user-5.png", h: 678, w: 2080, pos: "t"}
    G@{ img: "https://raw.githubusercontent.com/artisantek/training-materials/master/AWS/03-iam(identity-and-access-management)/images/user-6.png", h: 1014, w: 1538, pos: "t"}
    
    style A fill:#e1f5fe,font-size:30px
    style B fill:#c8e6c9,font-size:30px
    style C fill:#c8e6c9,font-size:30px
    style D fill:#c8e6c9,font-size:30px
    style E fill:#c8e6c9,font-size:30px
    style F fill:#c8e6c9,font-size:30px
    style G fill:#c8e6c9,font-size:30px
    style H fill:#e8f5e8,font-size:30px
    
    %% Arrow styling - thick arrows
    linkStyle default stroke-width:10px
    linkStyle 0 stroke-width:10px
    linkStyle 1 stroke-width:10px
    linkStyle 2 stroke-width:10px
    linkStyle 3 stroke-width:10px
    linkStyle 4 stroke-width:10px
    linkStyle 5 stroke-width:10px
    linkStyle 6 stroke-width:10px
```

## 📋 Process Flow

### 🔧 Step-by-Step IAM User Creation
1. **AWS Console Login** - Access your AWS account
2. **Navigate to IAM Dashboard** - Go to IAM service 🔑 ✅
3. **Access Users Section** - Click on Users in left navigation 🔑 ✅
4. **Set Permissions/Policies** - Configure user permissions 🔑 ✅
5. **Review and Create User** - Verify settings and create 🔑 ✅
6. **Review User Configuration** - Verify all settings 🔑 ✅
7. **Add the Credentials** - Create and download access keys 🔑 ✅

---

# 🛠️ Detailed Step Explanations

## Step 1: AWS Console Login
**Purpose**: Access your AWS account to begin the IAM user creation process

**Actions**:
- Open web browser and navigate to AWS Management Console
- Enter your AWS account credentials (root user or IAM admin user)
- Ensure you're using an account with IAM user creation permissions

**Key Points**:
- Use administrative credentials for IAM operations
- Ensure secure connection (HTTPS)

---

## Step 2: Navigate to IAM Dashboard
**Purpose**: Access the AWS Identity and Access Management service

**Actions**:
- Search for "IAM" in the AWS services search bar
- Click on IAM to open the dashboard

**Key Points**:
- Ensure you have administrative permissions
- Note the current user count in your account
- Review any existing users, groups, roles, and policies

---

## Step 3: Access Users Section
**Purpose**: Navigate to the user management area

**Actions**:
- Click "Users" in the left navigation panel
- Review existing users (if any)
- Check for naming conflicts with your planned username
- Prepare to create a new user

**Important**:
- Each user should have a unique, descriptive name
- Review current users to understand naming patterns
- Plan the new user's purpose and permissions

---

## Step 4: Set Permissions/Policies
**Purpose**: Configure user permissions and assign appropriate policies

**Key Configurations**:
- **Username**: Use descriptive names (e.g., `john-developer`, `api-service-user`)
- **Permission Assignment**: Add to groups, attach policies, or copy from existing user
- **Access Type**: Choose between console access, programmatic access, or both
- **Policy Selection**: Select appropriate AWS managed or custom policies

**Best Practices**:
- Apply least privilege principle
- Prefer group-based permissions over direct policy attachments
- Use AWS managed policies when possible
- Document the user's intended purpose

---

## Step 5: Review and Create User
**Purpose**: Verify all settings and create the user

**Review Checklist**:
- ✅ Username follows naming conventions
- ✅ Permissions follow least privilege principle
- ✅ Access type matches intended usage
- ✅ Policies are correctly assigned
- ✅ Tags are added for organization

**Final Actions**:
- Double-check all configurations
- Add descriptive tags if needed
- Click "Create user" to finalize the creation

---

## Step 6: Review User Configuration
**Purpose**: Confirm user was created successfully and review final settings

**Actions**:
- Verify user appears in the users list
- Review assigned permissions and policies
- Check user ARN and creation details
- Note any additional configuration needed

**Verification Points**:
- User status is active
- Permissions are correctly applied
- No errors or warnings present

---

## Step 7: Add the Credentials
**Purpose**: Generate and securely store the user's access credentials

**Critical Actions**:
2. **Download Credentials**: Click "Download .csv" immediately
3. **Secure Storage**: Store the secrets in secure location in your laptop.
4. **Test Access**: Verify credentials work before distributing

**🚨 Security Warnings**:
- **This is the ONLY time you can view the Secret Access Key**
- Never share credentials via email or unsecured channels
- Store credentials securely using proper credential management tools
- Test credentials before distributing to ensure they work

---

## For Human Users
### Initial Setup Checklist
- [ ] Provide secure access to credentials
- [ ] Send setup instructions for AWS CLI
- [ ] Require MFA setup for console access
- [ ] Provide relevant documentation and training
- [ ] Test access with user to ensure functionality

### Ongoing Management
- [ ] Schedule regular access reviews
- [ ] Monitor usage patterns
- [ ] Update permissions as role changes
- [ ] Ensure compliance with security policies

## For Application Users
### Integration Setup
- [ ] Configure application with new credentials
- [ ] Test all required AWS service interactions
- [ ] Implement proper error handling
- [ ] Set up monitoring and alerting
- [ ] Document the integration

### Security Hardening
- [ ] Use environment variables for credentials
- [ ] Implement credential rotation mechanism
- [ ] Monitor API usage patterns
- [ ] Set up alerts for unusual activity
- [ ] Regular security assessments

---

# 📚 Related Resources

## AWS Documentation
- [IAM User Guide](https://docs.aws.amazon.com/IAM/latest/UserGuide/)
- [AWS CLI Configuration](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html)
- [IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)

## Security Guides
- [IAM Security Best Practices](00-iam-fundamentals.md#iam-security-best-practices)
- [Access Key Management](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html)
- [MFA Setup Guide](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_mfa.html)

## Tools and Utilities
- [AWS CLI](https://aws.amazon.com/cli/)
- [AWS SDKs](https://aws.amazon.com/tools/)
- [IAM Policy Simulator](https://policysim.aws.amazon.com/)
- [AWS CloudTrail](https://aws.amazon.com/cloudtrail/)

---

# ✅ Summary

You have successfully created an IAM user with programmatic access! Here's what you accomplished:

## ✅ **Completed Tasks**
- ✅ Navigated to IAM dashboard
- ✅ Created new IAM user with appropriate name
- ✅ Configured access type and permissions
- ✅ Generated secure access keys
- ✅ Downloaded and secured credentials
- ✅ Applied security best practices

## 🎯 **Next Steps**
1. **Test the user's permissions** with sample commands
2. **Implement access key rotation** schedule
3. **Monitor usage** through CloudTrail
4. **Review and adjust permissions** as needed

## 🔒 **Security Reminders**
- Store access keys securely and never share them
- Rotate credentials regularly (every 90 days)
- Monitor usage for unusual activity
- Apply least privilege principle consistently
- Use IAM roles instead of access keys when possible
