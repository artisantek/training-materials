# Creating Access Keys for CLI Access - Step by Step Guide

This comprehensive guide walks you through creating access keys for AWS CLI access for existing IAM users. You'll learn how to generate access keys, configure CLI credentials, and securely manage programmatic access for command-line operations.

## What You'll Learn

- How to navigate to existing user's security credentials
- Step-by-step access key creation process
- How to configure AWS CLI with new credentials
- Best practices for CLI access key management
- Security considerations for programmatic access

## Prerequisites

- AWS account with administrative access
- Understanding of [IAM fundamentals](00-iam-fundamentals.md)
- Existing IAM user account
- AWS CLI installed on your local machine
- Browser access to AWS Management Console

---

## 📋 Complete CLI Access Key Creation Process

```mermaid
flowchart TD
    A[Start: AWS Console Login] -----> B[Navigate to Existing User]
    B -----> C[Access Security Credentials Tab]
    C -----> D[Add any tag and click on the Create access key]
    D -----> E[Download and Secure Access Keys]
    E -----> F[Configure AWS CLI]
    F -----> G[✅ CLI Access Setup Complete]
    
    B@{ img: "https://raw.githubusercontent.com/artisantek/training-materials/master/AWS/03-iam(identity-and-access-management)/images/cli-1.png", h: 1152, w: 2554, pos: "t"}
    C@{ img: "https://raw.githubusercontent.com/artisantek/training-materials/master/AWS/03-iam(identity-and-access-management)/images/cli-2.png", h: 1074, w: 2022, pos: "t"}
    E@{ img: "https://raw.githubusercontent.com/artisantek/training-materials/master/AWS/03-iam(identity-and-access-management)/images/cli-4.png", h: 790, w: 1706, pos: "t"}
    F@{ img: "https://raw.githubusercontent.com/artisantek/training-materials/master/AWS/03-iam(identity-and-access-management)/images/cli-5.png", h: 196, w: 1810, pos: "t"}
    G@{ img: "https://raw.githubusercontent.com/artisantek/training-materials/master/AWS/03-iam(identity-and-access-management)/images/cli-6.png", h: 320, w: 2146, pos: "t"}
    
    style A fill:#e1f5fe,font-size:30px
    style B fill:#c8e6c9,font-size:30px
    style C fill:#c8e6c9,font-size:30px
    style D fill:#fff9c4,font-size:30px
    style E fill:#c8e6c9,font-size:30px
    style F fill:#c8e6c9,font-size:30px
    style G fill:#e8f5e8,font-size:30px
    
    %% Arrow styling - thick arrows
    linkStyle default stroke-width:10px
    linkStyle 0 stroke-width:10px
    linkStyle 1 stroke-width:10px
    linkStyle 2 stroke-width:10px
    linkStyle 3 stroke-width:10px
    linkStyle 4 stroke-width:10px
    linkStyle 5 stroke-width:10px
```

## 📋 Process Flow

### 🔧 Step-by-Step CLI Access Key Creation
1. **AWS Console Login** - Access your AWS account
2. **Navigate to Existing User** - Find the user who needs CLI access 🔑 ✅
3. **Access Security Credentials Tab** - Open user's security settings 🔑 ✅
4. **Add any tag and click on the Create access key** - Configure and create access key 🔑 ✅
5. **Download and Secure Access Keys** - Save credentials securely 🔑 ✅
6. **Configure AWS CLI** - Set up local CLI with new credentials 🔑 ✅

---

# 🛠️ Detailed Step Explanations

## Step 1: AWS Console Login
**Purpose**: Access your AWS account to begin the CLI access key creation process

**Actions**:
- Open web browser and navigate to AWS Management Console
- Enter your AWS account credentials (root user or IAM admin user)
- Ensure you're using an account with IAM user management permissions

**Key Points**:
- Use administrative credentials for IAM operations
- Ensure secure connection (HTTPS)
- Consider enabling MFA for enhanced security

---

## Step 2: Navigate to Existing User
**Purpose**: Locate the IAM user who needs CLI access

**Actions**:
- Search for "IAM" in the AWS services search bar
- Click on IAM to open the dashboard
- Click "Users" in the left navigation panel
- Find and click on the specific user who needs CLI access

**Key Points**:
- The user must already exist (this guide assumes user creation is complete)
- Note the user's current permissions and attached policies
- Ensure this user should have programmatic access

**Important Considerations**:
- Verify the user's identity and purpose
- Check if the user already has access keys (AWS allows maximum 2 access keys per user)
- Confirm the user needs CLI access for their role

---

## Step 3: Access Security Credentials Tab
**Purpose**: Navigate to the user's credential management section

**Actions**:
- In the user details page, click on the "Security credentials" tab
- Review existing access keys (if any)
- Check MFA device status
- Note console access settings

**Security Review**:
- **Existing Access Keys**: Check if user already has active keys
- **MFA Status**: Verify if MFA is enabled for the user
- **Password Settings**: Review console access configuration
- **Last Activity**: Check when the user last accessed AWS

**Best Practice Check**:
- Maximum 2 access keys per user allowed
- Deactivate unused or old access keys
- Consider access key rotation schedule

---

## Step 4: Add Tags and Create Access Key
**Purpose**: Configure access key with proper tagging and create the credentials

**Actions**:
- Click "Create access key" button
- Choose "Command Line Interface (CLI)" as the use case
- Add descriptive tags for the access key:
  - **Purpose**: CLI-Access, Development, Production, etc.
  - **Owner**: Username or team name
  - **Environment**: Dev, Staging, Prod
  - **Created**: Date of creation
- Click "Create access key" to generate credentials

**Tagging Best Practices**:
- **Purpose**: Clearly indicate what the key is used for
- **Environment**: Specify the environment (dev/staging/prod)
- **Owner**: Identify who is responsible for this key
- **Project**: Associate with specific project or application
- **Expiry**: Planned rotation date (if applicable)

**Security Considerations**:
- Tags help with access key management and auditing
- Use consistent tagging strategy across all access keys
- Tags are visible in CloudTrail logs for better tracking

---

## Step 5: Download and Secure Access Keys
**Purpose**: Obtain and securely store the generated access credentials

**Critical Actions**:
1. **Download Credentials**: Click "Download .csv file" immediately
2. **Copy Credentials**: Note the Access Key ID and Secret Access Key
3. **Secure Storage**: Store in password manager or secure location
4. **Backup**: Create secure backup of credentials

**🚨 Security Warnings**:
- **This is the ONLY time you can view the Secret Access Key**
- Never share credentials via email or unsecured channels
- Store credentials in encrypted password managers
- Do not commit credentials to version control systems
- Consider using AWS Secrets Manager for application credentials

**Immediate Security Actions**:
- [ ] Download CSV file immediately
- [ ] Store credentials in secure password manager
- [ ] Delete credentials from Downloads folder after secure storage
- [ ] Never share credentials via email or chat
- [ ] Document the key's purpose and owner

---

## Step 6: Configure AWS CLI
**Purpose**: Set up AWS CLI on your local machine with the new credentials

**Prerequisites**:
- AWS CLI installed on your local machine
- Terminal or command prompt access
- Downloaded access key credentials

**Configuration Process**:

### Option 1: Using `aws configure` command
```bash
# Run AWS CLI configuration
aws configure

# You'll be prompted to enter:
# AWS Access Key ID: [Enter your Access Key ID]
# AWS Secret Access Key: [Enter your Secret Access Key]  
# Default region name: [Enter your preferred region, e.g., us-east-1]
# Default output format: [Enter json, yaml, text, or table]
```


### Option 2: Using environment variables
```bash
# Set environment variables (Linux/Mac)
export AWS_ACCESS_KEY_ID="your-access-key-id"
export AWS_SECRET_ACCESS_KEY="your-secret-access-key"
export AWS_DEFAULT_REGION="us-east-1"

# Set environment variables (Windows)
set AWS_ACCESS_KEY_ID=your-access-key-id
set AWS_SECRET_ACCESS_KEY=your-secret-access-key
set AWS_DEFAULT_REGION=us-east-1
```

**Verification Commands**:
```bash
# Test CLI configuration
aws sts get-caller-identity

# List S3 buckets (if user has permission)
aws s3 ls

# Check IAM user details
aws iam get-user
```

**Configuration Best Practices**:
- Set appropriate default region for your use case
- Choose output format that suits your workflow
- Test configuration with simple, safe commands first

---

# 🔧 Post-Configuration Tasks

## Immediate Verification
- [ ] Test AWS CLI with `aws sts get-caller-identity`
- [ ] Verify user permissions with safe read commands
- [ ] Confirm region settings are correct
- [ ] Test specific services the user needs to access
- [ ] Document the CLI setup for future reference

## Security Hardening
- [ ] Set up credential rotation schedule (every 90 days)
- [ ] Set up environment-specific configurations
- [ ] Enable CloudTrail monitoring for API calls
- [ ] Document approved CLI usage patterns

---

# 🔒 Security Best Practices for CLI Access

## Access Key Management

### ✅ **Do's**
- **Rotate access keys regularly** (every 90 days minimum)
- **Store keys securely** using credential management tools
- **Monitor key usage** through CloudTrail logs
- **Use environment variables** in CI/CD pipelines instead of hardcoded keys
- **Apply least privilege** permissions to CLI users

### ❌ **Don'ts**
- **Never hardcode** access keys in scripts or applications
- **Don't share** access keys between users or systems
- **Avoid storing keys** in plain text files
- **Don't commit** keys to version control systems
- **Never expose** keys in logs or error messages
- **Don't use root user** access keys for CLI access

## CLI Configuration Security

### Multiple AWS Accounts
```bash
# Use profiles for different AWS accounts
aws configure --profile personal-account
aws configure --profile work-account
aws configure --profile staging-account

# Use specific profile for commands
aws s3 ls --profile work-account
```

### Credential File Security
```bash
# Secure credential files (Linux/Mac)
chmod 600 ~/.aws/credentials
chmod 600 ~/.aws/config

# Location of AWS credential files:
# Linux/Mac: ~/.aws/credentials and ~/.aws/config
# Windows: %USERPROFILE%\.aws\credentials and %USERPROFILE%\.aws\config
```

---

# 🚨 Troubleshooting Common CLI Issues

## Authentication Problems
**Symptoms**:
- "Unable to locate credentials" errors
- "The security token included in the request is invalid" errors
- AWS CLI commands return access denied

**Solutions**:
1. **Check credential configuration**:
   ```bash
   aws configure list
   ```
2. **Verify access key status** in IAM console
3. **Test with caller identity**:
   ```bash
   aws sts get-caller-identity
   ```
4. **Check if keys are active** and not expired
5. **Verify region settings** match your resources

## Permission Issues
**Symptoms**:
- "Access Denied" for specific AWS services
- Some CLI commands work, others fail
- Inconsistent permission behavior

**Solutions**:
1. **Check IAM policies** attached to the user
2. **Test permissions** with IAM Policy Simulator
3. **Verify resource ARNs** in policies
4. **Check for explicit deny** statements
5. **Review resource-based policies** (S3 bucket policies, etc.)

## Configuration Problems
**Symptoms**:
- CLI commands use wrong region
- Output format issues
- Profile not found errors

**Solutions**:
1. **Check AWS configuration**:
   ```bash
   aws configure list
   aws configure list --profile profile-name
   ```
2. **Verify profile names** and file syntax
3. **Check environment variables** that might override settings
4. **Validate credential file format**
5. **Reset configuration** if necessary:
   ```bash
   aws configure --profile profile-name
   ```

---

# 📚 Related Resources

## AWS Documentation
- [AWS CLI User Guide](https://docs.aws.amazon.com/cli/latest/userguide/)
- [AWS CLI Configuration](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html)
- [IAM Access Keys Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html)

## CLI Commands Reference
- [AWS CLI Command Reference](https://docs.aws.amazon.com/cli/latest/reference/)
- [STS Commands](https://docs.aws.amazon.com/cli/latest/reference/sts/)
- [IAM Commands](https://docs.aws.amazon.com/cli/latest/reference/iam/)

## Security Guides
- [IAM Security Best Practices](00-iam-fundamentals.md#iam-security-best-practices)
- [AWS CLI Security Best Practices](https://docs.aws.amazon.com/cli/latest/userguide/cli-security.html)
- [Credential Management](https://docs.aws.amazon.com/cli/latest/userguide/cli-security-credentials.html)

## Related IAM Guides
- [IAM Fundamentals](00-iam-fundamentals.md)
- [IAM User Creation](01-iam-user-creation.md)
- [IAM Groups Creation](02-iam-groups-creation.md)

---

# ✅ Summary

You have successfully created CLI access keys and configured AWS CLI! Here's what you accomplished:

## ✅ **Completed Tasks**
- ✅ Located existing IAM user in AWS console
- ✅ Navigated to user's security credentials section
- ✅ Created access key with proper tagging
- ✅ Downloaded and secured access key credentials
- ✅ Configured AWS CLI with new credentials
- ✅ Verified CLI access with test commands

## 🎯 **Next Steps**
1. **Test CLI permissions** with your required AWS services
2. **Set up credential rotation** schedule (every 90 days)
3. **Configure additional profiles** if working with multiple accounts
4. **Set up monitoring** for CLI usage through CloudTrail
5. **Document CLI usage patterns** for your team


Your AWS CLI is now ready for secure programmatic access to AWS services!
