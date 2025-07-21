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
    C -----> D[Click Create User]
    D -----> E[Set User Details & Permissions]
    E -----> F[Configure Access Type]
    F -----> G[Review User Configuration]
    G -----> H[Generate Access Keys]
    H -----> I[✅ IAM User Creation Complete]
    
    B@{ img: "https://raw.githubusercontent.com/artisantek/training-materials/master/AWS/03-iam(identity-and-access-management)/images/user-1.png", h: 936, w: 2086, pos: "t"}
    C@{ img: "https://raw.githubusercontent.com/artisantek/training-materials/master/AWS/03-iam(identity-and-access-management)/images/user-2.png", h: 1088, w: 2300, pos: "t"}
    E@{ img: "https://raw.githubusercontent.com/artisantek/training-materials/master/AWS/03-iam(identity-and-access-management)/images/user-3.png", h: 1274, w: 2008, pos: "t"}
    F@{ img: "https://raw.githubusercontent.com/artisantek/training-materials/master/AWS/03-iam(identity-and-access-management)/images/user-4.png", h: 708, w: 2002, pos: "t"}
    G@{ img: "https://raw.githubusercontent.com/artisantek/training-materials/master/AWS/03-iam(identity-and-access-management)/images/user-5.png", h: 678, w: 2080, pos: "t"}
    H@{ img: "https://raw.githubusercontent.com/artisantek/training-materials/master/AWS/03-iam(identity-and-access-management)/images/user-6.png", h: 1014, w: 1538, pos: "t"}
    
    style A fill:#e1f5fe,font-size:30px
    style B fill:#c8e6c9,font-size:30px
    style C fill:#c8e6c9,font-size:30px
    style D fill:#c8e6c9,font-size:30px
    style E fill:#c8e6c9,font-size:30px
    style F fill:#c8e6c9,font-size:30px
    style G fill:#c8e6c9,font-size:30px
    style H fill:#c8e6c9,font-size:30px
    style I fill:#e8f5e8,font-size:30px
    
    %% Arrow styling - thick arrows
    linkStyle default stroke-width:10px
    linkStyle 0 stroke-width:10px
    linkStyle 1 stroke-width:10px
    linkStyle 2 stroke-width:10px
    linkStyle 3 stroke-width:10px
    linkStyle 4 stroke-width:10px
    linkStyle 5 stroke-width:10px
    linkStyle 6 stroke-width:10px
    linkStyle 7 stroke-width:10px
```

## 📋 Process Flow

### 🔧 Step-by-Step IAM User Creation
1. **AWS Console Login** - Access your AWS account
2. **Navigate to IAM Dashboard** - Go to IAM service 🔑 ✅
3. **Access Users Section** - Click on Users in left navigation 🔑 ✅
4. **Click Create User** - Initiate user creation process
5. **Set User Details & Permissions** - Configure user information 🔑 ✅
6. **Configure Access Type** - Set programmatic/console access 🔑 ✅
7. **Review User Configuration** - Verify all settings 🔑 ✅
8. **Generate Access Keys** - Create and download credentials 🔑 ✅

---

# 🛠️ Detailed Step Explanations

## Step 1: Navigate to IAM Dashboard
**Purpose**: Access the AWS Identity and Access Management service

**Actions**:
- Sign in to AWS Management Console
- Search for "IAM" in services
- Click on IAM to open the dashboard
- Review the IAM overview interface

**Key Points**:
- Ensure you have administrative permissions
- Familiarize yourself with the IAM dashboard layout
- Note the current user count in your account

---

## Step 2: Access Users Section
**Purpose**: Navigate to the user management area

**Actions**:
- Click "Users" in the left navigation panel
- Review existing users (if any)
- Prepare to create a new user
- Check for naming conflicts

**Important**:
- Each user should have a unique, descriptive name
- Review current users to understand naming patterns
- Plan the new user's purpose and permissions

---

## Step 3: Set User Details & Permissions
**Purpose**: Configure the basic user information and initial permission settings

**Key Configurations**:
- **Username**: Use descriptive names (e.g., `john-developer`, `api-service-user`)
- **Access Type**: Choose between console access, programmatic access, or both
- **Password Settings**: Configure password requirements if console access is enabled
- **Permission Assignment**: Add to groups, attach policies, or copy from existing user

**Best Practices**:
- Use descriptive usernames that indicate purpose
- Apply least privilege principle
- Prefer group-based permissions over direct policy attachments
- Enable password reset requirement for human users

---

## Step 4: Configure Access Type
**Purpose**: Specify the type of access the user needs

**Access Type Options**:
- **✅ Programmatic Access**: Generates access key ID and secret access key for API/CLI
- **⬜ Console Access**: Allows AWS Management Console login via web browser
- **✅ Both**: User gets both programmatic and console access

**Security Considerations**:
- Programmatic access is ideal for applications and scripts
- Console access should be limited to human users
- MFA should be mandatory for console users
- Use temporary credentials when possible

---

## Step 5: Review User Configuration
**Purpose**: Verify all settings before creating the user

**Review Checklist**:
- ✅ Username follows naming conventions
- ✅ Access type matches intended usage
- ✅ Permissions follow least privilege principle
- ✅ Security settings are appropriate
- ✅ Tags are added for organization

**Final Actions**:
- Double-check permission assignments
- Verify security settings
- Add descriptive tags
- Click "Create user" to finalize

---

## Step 6: Generate and Secure Access Keys
**Purpose**: Obtain and securely store the user's access credentials

**Critical Actions**:
1. **Download Credentials**: Click "Download .csv" immediately
2. **Note Access Keys**: Record Access Key ID and Secret Access Key
3. **Secure Storage**: Store in password manager or secure location
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

# 🚨 Troubleshooting Common Issues

## Access Denied Errors
**Symptoms**:
- API calls return "Access Denied" errors
- User cannot perform expected actions
- CLI commands fail with permission errors

**Solutions**:
1. Check IAM policies attached to user/groups
2. Verify resource-based policies (S3 bucket policies, etc.)
3. Confirm no explicit DENY statements
4. Test with IAM Policy Simulator
5. Check service-specific quotas and limits

## Invalid Credentials
**Symptoms**:
- "Invalid access key" errors
- Authentication failures
- CLI configuration issues

**Solutions**:
1. Verify access key format (correct copying)
2. Check if keys are active in IAM console
3. Confirm region settings in CLI/SDK
4. Test with temporary credentials to isolate issue
5. Regenerate access keys if necessary

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
1. **Configure AWS CLI** with the new credentials
2. **Test the user's permissions** with sample commands
3. **Implement access key rotation** schedule
4. **Monitor usage** through CloudTrail
5. **Review and adjust permissions** as needed

## 🔒 **Security Reminders**
- Store access keys securely and never share them
- Rotate credentials regularly (every 90 days)
- Monitor usage for unusual activity
- Apply least privilege principle consistently
- Use IAM roles instead of access keys when possible

Your new IAM user is ready for secure programmatic access to AWS services!
