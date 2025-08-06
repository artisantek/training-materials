# EC2 Instance Creation with User Data

This guide covers creating AWS EC2 instances with user data scripts for automated configuration and software installation.

## Overview

User data allows you to pass data to your instance when it launches, enabling automated configuration, software installation, and setup tasks without manual intervention after instance creation.

## What is User Data?

User data is a feature that allows you to:
- Run scripts automatically when an instance starts
- Install software packages
- Configure services
- Set up applications
- Perform system configurations
- Download and install files

## EC2 with User Data Creation Flow

```mermaid
flowchart TD
        A[Start: AWS Console Login] -----> B[Navigate to EC2 Dashboard]
    B -----> C[Click Launch Instance]
    C -----> D[Choose AMI]
    D -----> E[Choose Instance Type]
    E -----> H[Create Key Pair]
    H -----> I[Download Key Pair]

    I -----> J[Configure Security Group]
    J -----> F[Configure Advanced Details - User Data]
    F -----> G[Add User Data Like shown]
    G -----> K[Review and Launch]
    K -----> L[Instance Running]
    L -----> M[Connect to Instance in your terminal using this command **ssh -i keypair.pem username@public_ip**]
    
    D@{ img: "https://raw.githubusercontent.com/artisantek/training-materials/master/AWS/02-ec2(elastic-compute-cloud)/images/ami-selection.png", h: 430, w: 1270, pos: "t"}
    E@{ img: "https://raw.githubusercontent.com/artisantek/training-materials/master/AWS/02-ec2(elastic-compute-cloud)/images/instance-type.png", h: 446, w: 1544, pos: "t"}
    F@{ img: "https://raw.githubusercontent.com/artisantek/training-materials/master/AWS/02-ec2(elastic-compute-cloud)/images/advanced-detail.png", h: 332, w: 560, pos: "t"}
    G@{ img: "https://raw.githubusercontent.com/artisantek/training-materials/master/AWS/02-ec2(elastic-compute-cloud)/images/user-data.png", h: 2000, w: 1452, pos: "t"}
    H@{ img: "https://raw.githubusercontent.com/artisantek/training-materials/master/AWS/02-ec2(elastic-compute-cloud)/images/create-keypair.png", h: 510, w: 1646, pos: "t"}
    I@{ img: "https://raw.githubusercontent.com/artisantek/training-materials/master/AWS/02-ec2(elastic-compute-cloud)/images/download-keypair.png", h: 1198, w: 1240, pos: "t"}
    J@{ img: "https://raw.githubusercontent.com/artisantek/training-materials/master/AWS/02-ec2(elastic-compute-cloud)/images/security-group.png", h: 892, w: 1490, pos: "t"}
    L@{ img: "https://raw.githubusercontent.com/artisantek/training-materials/master/AWS/02-ec2(elastic-compute-cloud)/images/running-instance.png", h: 602, w: 2748, pos: "t"}
    
    style A fill:#e1f5fe,font-size:30px
    style D fill:#c8e6c9,font-size:30px
    style E fill:#c8e6c9,font-size:30px
    style F fill:#fff3e0,font-size:30px
    style G fill:#fff3e0,font-size:30px
    style H fill:#c8e6c9,font-size:30px
    style I fill:#c8e6c9,font-size:30px
    style J fill:#c8e6c9,font-size:30px
    style L fill:#c8e6c9,font-size:30px
    style B font-size:30px
    style C font-size:30px
    style K font-size:30px
    style M font-size:30px
    
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
    linkStyle 8 stroke-width:10px
    linkStyle 9 stroke-width:10px
    linkStyle 10 stroke-width:10px
    linkStyle 11 stroke-width:10px
```

## Process Flow

1. **AWS Console Login** - Access your AWS account
2. **Navigate to EC2** - Go to EC2 Dashboard  
3. **Launch Instance** - Click "Launch Instance"
4. **Choose AMI** - Choose your operating system 🔑 ✅
5. **Choose Instance Type** - Select instance size 🔑 ✅
6. **Create Key Pair** - Create SSH key pair 🔑 ✅
7. **Download Key Pair** - Save key file securely 🔑 ✅
8. **Configure Security Group** - Set up security rules 🔑 ✅
9. **Configure Advanced Details** - Navigate to advanced details 🔑 ✅
10. **Add User Data** - Write your automation script 🔑 ✅
11. **Review and Launch** - Final review and launch 📝
12. **Instance Running** - Verify instance is running 🔑 ✅
13. **Connect and Test** - SSH and verify user data execution 📝

## Common User Data Examples

### 1. Java, NPM, Maven Installation (RHEL/Amazon Linux)

```bash
#!/bin/bash

# Download and install nvm:
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash

# in lieu of restarting the shell
\. "$HOME/.nvm/nvm.sh"

# Download and install Node.js:
nvm install 24

# Verify the Node.js version:
node -v # Should print "v24.3.0".
nvm current # Should print "v24.3.0".

# Verify npm version:
npm -v # Should print "11.4.2".

#Maven Installation:
sudo yum update -y && sudo yum upgrade -y
sudo yum install java-21-openjdk wget -y
java --version
wget https://dlcdn.apache.org/maven/maven-3/3.9.10/binaries/apache-maven-3.9.10-bin.tar.gz
sudo tar xzf apache-maven-3.9.10-bin.tar.gz -C /opt
# Create and populate the maven.sh file in one step
echo 'export MAVEN_HOME=/opt/apache-maven-3.9.10' | sudo tee /etc/profile.d/maven.sh
echo 'export PATH=$MAVEN_HOME/bin:$PATH' | sudo tee -a /etc/profile.d/maven.sh
sudo chmod +x /etc/profile.d/maven.sh
source /etc/profile.d/maven.sh
mvn -version


#Install pip
sudo yum install python3-pip -y
```

### 2. Ubuntu/Debian System Setup

```bash
#!/bin/bash
apt-get update
apt-get install -y apache2
systemctl start apache2
systemctl enable apache2
echo "<h1>Ubuntu Server with User Data</h1>" > /var/www/html/index.html
```

### 3. Docker Installation and Container Setup

```bash
#!/bin/bash
yum update -y
yum install -y docker
systemctl start docker
systemctl enable docker
usermod -a -G docker ec2-user
docker run -d -p 80:80 --name webserver nginx
```

### 4. Node.js Application Setup

```bash
#!/bin/bash
yum update -y
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
nvm install node
npm install -g pm2
```


## User Data Best Practices

### 1. **Script Structure**
- Always start with shebang (`#!/bin/bash`)
- Use proper error handling
- Log outputs for debugging

### 2. **Error Handling Example**
```bash
#!/bin/bash
exec > >(tee /var/log/user-data.log|logger -t user-data -s 2>/dev/console) 2>&1
yum update -y || { echo "Failed to update packages"; exit 1; }
yum install -y httpd || { echo "Failed to install Apache"; exit 1; }
```

### 3. **Security Considerations**
- Don't include sensitive data in user data scripts
- Use IAM roles for AWS service access
- Limit permissions and access

### 4. **Testing**
- Test scripts locally before using in user data
- Use CloudWatch logs for monitoring
- Check `/var/log/user-data.log` for debugging

## User Data Limitations

1. **Size Limit**: 16 KB maximum
2. **Execution**: Runs only on first boot (cloud-init)
3. **Root Access**: Scripts run as root user
4. **No User Interaction**: Must be fully automated
5. **Network Dependency**: Internet access required for downloads

## Advanced User Data Patterns

### 1. **Multi-line User Data with Variables**
```bash
#!/bin/bash
APP_NAME="myapp"
APP_VERSION="1.0.0"
LOG_FILE="/var/log/${APP_NAME}-install.log"

exec > >(tee ${LOG_FILE}) 2>&1
echo "Starting installation of ${APP_NAME} v${APP_VERSION}"
# Installation commands here
```

### 2. **Conditional Installation**
```bash
#!/bin/bash
if [ -f /etc/redhat-release ]; then
    yum update -y
    yum install -y httpd
elif [ -f /etc/debian_version ]; then
    apt-get update
    apt-get install -y apache2
fi
```

### 3. **Service Configuration**
```bash
#!/bin/bash
# Install and configure nginx
yum update -y
yum install -y nginx

# Create custom configuration
cat > /etc/nginx/conf.d/myapp.conf << 'EOF'
server {
    listen 80;
    server_name _;
    location / {
        proxy_pass http://localhost:3000;
    }
}
EOF

systemctl start nginx
systemctl enable nginx
```

## Troubleshooting User Data

### 1. **Check Execution Logs**
```bash
# View user data execution logs
sudo cat /var/log/user-data.log
sudo cat /var/log/cloud-init-output.log
```

### 2. **Verify Script Syntax**
```bash
# Test script locally
bash -n your-script.sh
```

### 3. **Common Issues**
- Script doesn't execute: Check shebang and permissions
- Network issues: Ensure internet connectivity
- Package installation fails: Check repositories and dependencies

## Prerequisites

- AWS account with EC2 permissions
- Understanding of shell scripting
- Knowledge of target operating system commands
- SSH client for instance access

## Security Notes

- User data is visible in instance metadata
- Use IAM roles instead of hardcoded credentials
- Consider using AWS Systems Manager Parameter Store for sensitive data
- Scripts run with root privileges - use carefully

## Next Steps

After successful user data execution:
1. Connect to instance via SSH
2. Verify installed software and services
3. Check application logs
4. Test functionality
5. Create AMI if configuration is reusable 