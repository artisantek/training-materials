# EC2 Instance Creation with Custom AMI

This guide covers creating custom Amazon Machine Images (AMI) and using them to launch new EC2 instances with pre-configured software and settings.

## What is an AMI and Why Do We Need It?

Think of an **AMI (Amazon Machine Image)** as a **template or blueprint** for creating EC2 instances. You use an AMI to create **identical servers** quickly and efficiently.

## Why We Need AMIs:

- **🎯 Consistency**: All instances launched from the same AMI are **identical**
- **⚡ Speed**: Launch instances in **minutes** instead of hours of manual setup  
- **📈 Scalability**: Quickly spin up **multiple identical servers**
- **🔄 Disaster Recovery**: Recreate your exact server configuration **instantly**

## How AMI Relates to EC2 Instances

The relationship is simple:

- **AMI** = The recipe/template 📋
- **EC2 Instance** = The actual running server created from that recipe 🖥️



### Practical Example:
1. You have a **web server** with Apache, PHP, and your application installed
2. You **create an AMI** from this configured server  
3. Now you can **launch 10 identical web servers** instantly using this AMI

## Types of AMIs

There are three main types of AMIs you can use:

1. **Public AMIs**
   - **What:** Available to everyone on AWS
   - **Examples:** Amazon Linux 2, Ubuntu 20.04, Windows Server 2019
   - **Use Case:** Great starting point for new projects or general-purpose servers

2. **Private AMIs**
   - **What:** Only visible and usable within your AWS account
   - **Examples:** Your own custom application servers, internal tools, or pre-configured environments
   - **Use Case:** Use for proprietary configurations, internal company images, or secure environments

3. **AWS Marketplace AMIs**
   - **What:** Provided by third parties, often with software pre-installed
   - **Examples:** WordPress, MongoDB, Nginx Plus, security appliances
   - **Use Case:** Quick deployment of popular software stacks or commercial solutions  
   - **Cost:** May include additional licensing or subscription fees

**Example Scenario:**  
Suppose you want to launch a WordPress site:

- **Option A:** Start with a Public AMI (e.g., Amazon Linux 2) and manually install WordPress (takes ~2 hours)
- **Option B:** Use a Marketplace AMI with WordPress pre-installed (ready in ~5 minutes, but may cost extra)


## Phase 1: Create Base Instance (Same as Basic EC2 Creation)

```mermaid
flowchart TD
    A[Start: AWS Console Login] -----> B[Navigate to EC2 Dashboard]
    B -----> C[Click Launch Instance]
    C -----> D[Choose Base AMI]
    D -----> E[Choose Instance Type]
    E -----> F[Create Key Pair]
    F -----> G[Download Key Pair]
    G -----> H[Configure Security Group]
    H -----> I[Review and Launch]
    I -----> J[Base Instance Running]
    J -----> K[Connect and Install Software]
    K -----> L[Stop Instance for AMI Creation]
    L -----> M[Navigate to Images and Templates]
    M -----> N[Create Custom AMI]
    N -----> O[Check AMI Status/State]
    O -----> P[Launch New Instance from Custom AMI]
    
    D@{ img: "https://raw.githubusercontent.com/artisantek/training-materials/master/AWS/02-ec2(elastic-compute-cloud)/images/ami-selection.png", h: 430, w: 1270, pos: "t"}
    E@{ img: "https://raw.githubusercontent.com/artisantek/training-materials/master/AWS/02-ec2(elastic-compute-cloud)/images/instance-type.png", h: 446, w: 1544, pos: "t"}
    F@{ img: "https://raw.githubusercontent.com/artisantek/training-materials/master/AWS/02-ec2(elastic-compute-cloud)/images/create-keypair.png", h: 510, w: 1646, pos: "t"}
    G@{ img: "https://raw.githubusercontent.com/artisantek/training-materials/master/AWS/02-ec2(elastic-compute-cloud)/images/download-keypair.png", h: 1198, w: 1240, pos: "t"}
    H@{ img: "https://raw.githubusercontent.com/artisantek/training-materials/master/AWS/02-ec2(elastic-compute-cloud)/images/security-group.png", h: 892, w: 1490, pos: "t"}
    J@{ img: "https://raw.githubusercontent.com/artisantek/training-materials/master/AWS/02-ec2(elastic-compute-cloud)/images/running-instance.png", h: 602, w: 2748, pos: "t"}
    L@{ img: "https://raw.githubusercontent.com/artisantek/training-materials/master/AWS/02-ec2(elastic-compute-cloud)/images/AMI-Pre-selection.png", h: 698, w: 2494, pos: "t"}
    M@{ img: "https://raw.githubusercontent.com/artisantek/training-materials/master/AWS/02-ec2(elastic-compute-cloud)/images/AMI-Wizard.png", h: 828, w: 2140, pos: "t"}
    N@{ img: "https://raw.githubusercontent.com/artisantek/training-materials/master/AWS/02-ec2(elastic-compute-cloud)/images/Create-AMI.png", h: 1212, w: 2070, pos: "t"}
    O@{ img: "https://raw.githubusercontent.com/artisantek/training-materials/master/AWS/02-ec2(elastic-compute-cloud)/images/AMI-state.png", h: 1136, w: 2612, pos: "t"}
    P@{ img: "https://raw.githubusercontent.com/artisantek/training-materials/master/AWS/02-ec2(elastic-compute-cloud)/images/Launch-instance-using-ami.png", h: 964, w: 1784, pos: "t"}
    
    style A fill:#e1f5fe,font-size:30px
    style D fill:#c8e6c9,font-size:30px
    style E fill:#c8e6c9,font-size:30px
    style F fill:#c8e6c9,font-size:30px
    style G fill:#c8e6c9,font-size:30px
    style H fill:#c8e6c9,font-size:30px
    style J fill:#c8e6c9,font-size:30px
    style K fill:#fff3e0,font-size:30px
    style L fill:#fff3e0,font-size:30px
    style M fill:#fff3e0,font-size:30px
    style N fill:#fff3e0,font-size:30px
    style O fill:#fff3e0,font-size:30px
    style P fill:#fff3e0,font-size:30px
    style B font-size:30px
    style C font-size:30px
    style I font-size:30px
    
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
    linkStyle 12 stroke-width:10px
    linkStyle 13 stroke-width:10px
    linkStyle 14 stroke-width:10px
```

## Few Screenshot Explanations
**Reboot Instance Option:**
- **Enable Reboot (default):** Briefly shuts down your instance during AMI creation to ensure a consistent and reliable image. This causes a few minutes of downtime.
- **No Reboot:** Creates the image while the instance is running, avoiding downtime, but may result in an inconsistent or corrupted AMI if data is not fully written to disk.

**Delete on Termination:**
- This setting controls whether the EBS volume (virtual hard drive) is deleted when the instance is terminated.
    - **Checked:** The volume is deleted with the instance (recommended for OS disks to avoid extra costs).
    - **Unchecked:** The volume is preserved after termination (important for keeping critical data).

---

**What is a Snapshot?**  
*A backup of a single EBS volume (hard drive).*

- **Purpose:** To save the data on that drive. If the drive fails or files are deleted, you can restore from the snapshot.
- **Contains:** Only the data from that one drive.
- **Use Case:** Data safety and recovery.  
  > *Note: You cannot launch a full server from just a snapshot—you need more information.*


- **Purpose:** To launch new, identical servers quickly. For example, after configuring a web server, you can create an AMI and launch many identical servers in minutes.
- **Contains:**
    - **Snapshots:** Backups of the hard drive(s).
    - **Server Settings:** Size, memory, security group, etc.
    - **Permissions:** Who can use the AMI.
- **Use Case:** Cloning servers and creating standardized starting points for new instances.



## 📋 Process Flow

### 🔧 Phase 1: Creating Custom AMI
1. **AWS Console Login** - Access your AWS account
2. **Navigate to EC2** - Go to EC2 Dashboard  
3. **Launch Base Instance** - Create instance for customization
4. **Choose Base AMI** - Select base operating system 🔑 ✅
5. **Configure Instance** - Set up instance type and settings 🔑 ✅
6. **Connect to Instance** - SSH into the instance
7. **Install & Configure** - Set up software and configurations 🔑 ✅
8. **Stop Instance** - Prepare for AMI creation 🔑 ✅
9. **Navigate to Images** - Go to Images and Templates section 🔑 ✅
10. **Create AMI** - Generate custom image 🔑 ✅
11. **Check AMI Status** - Wait for AMI to be available 🔑 ✅

### 🚀 Phase 2: Using Custom AMI
12. **Launch from Custom AMI** - Start new instance creation 🔑 ✅
13. **Configure New Instance** - Set instance details 
14. **Launch Instance** - Deploy the new instance
15. **Verify Setup** - Confirm pre-configured software is working ✅

## 💡 Common Custom AMI Use Cases

### 1. 🌐 Simple Web Server AMI

**Use Case**: Create a ready-to-use web server for hosting websites

**Basic Apache Setup:**
```bash
#!/bin/bash
# Update system packages
yum update -y

# Install Apache web server
yum install -y httpd

# Start Apache and enable on boot
systemctl start httpd
systemctl enable httpd

# Create a simple welcome page
echo "<h1>Welcome to My Custom Web Server!</h1>" > /var/www/html/index.html
echo "<p>This server was created from a custom AMI.</p>" >> /var/www/html/index.html

echo "✅ Web server setup complete!"
```

**Result**: Launch multiple identical web servers instantly!

### 2. 💻 Development Tools AMI

**Use Case**: Pre-configured development environment for coding projects

**Basic Development Setup:**
```bash
#!/bin/bash
# Update system
yum update -y

# Install basic development tools
yum install -y git
yum install -y nodejs npm
yum install -y python3 python3-pip

# Install text editors
yum install -y nano vim

# Create a welcome message
echo "Development environment ready!" > /home/ec2-user/README.txt
echo "Git, Node.js, and Python3 are installed." >> /home/ec2-user/README.txt

echo "✅ Development tools setup complete!"
```

**Result**: Instant development environment for your team!

### 3. 🐳 Docker Host AMI

**Use Case**: Container-ready server for running Docker applications

**Simple Docker Installation:**
```bash
#!/bin/bash
# Update system
yum update -y

# Install Docker
yum install -y docker

# Start Docker service
systemctl start docker
systemctl enable docker

# Add ec2-user to docker group
usermod -a -G docker ec2-user

# Verify installation
echo "Docker installed successfully" > /home/ec2-user/docker-status.txt

echo "✅ Docker setup complete!"
```

**Result**: Ready-to-use Docker host for containerized applications!

## 🏆 AMI Best Practices

### 1. 📋 **Preparation Before Creating AMI**
- **🔒 Security**: Remove sensitive data and credentials
- **🧹 Cleanup**: Clear logs and temporary files  
- **📦 Updates**: Update system packages to latest versions
- **🔑 Keys**: Remove SSH keys and user-specific data
- **📝 History**: Clear command history and logs

### 2. 🔐 **AMI Security Considerations**

**Important**: Always run this cleanup script before creating your AMI!
```bash
# Clean up script before AMI creation
#!/bin/bash
# Remove SSH host keys (will be regenerated)
rm -f /etc/ssh/ssh_host_*

# Clear bash history
history -c
rm -f ~/.bash_history

# Remove temporary files
rm -rf /tmp/*
rm -rf /var/tmp/*

# Clear log files
find /var/log -type f -exec truncate -s 0 {} \;

# Remove cloud-init logs
rm -rf /var/lib/cloud/instances/*
```

### 3. **AMI Naming Convention**
- Use descriptive names: `company-app-version-date`
- Example: `mycompany-webserver-lamp-2024-01-15`
- Include version tags for tracking

### 4. **AMI Management**
- Regularly update base AMIs
- Implement automated AMI creation
- Set up AMI lifecycle policies
- Monitor AMI usage and costs

## AMI Creation Steps (Detailed)

### 1. Prepare Instance for AMI Creation
```bash
# Connect to your instance and run cleanup
sudo yum update -y
sudo package-cleanup --oldkernels --count=1

# Remove unnecessary packages
sudo yum autoremove -y

# Clear package cache
sudo yum clean all

# Run the cleanup script above
```

### 2. Stop Instance (Required for EBS-backed AMIs)
- Navigate to EC2 Dashboard
- Select your instance
- Actions → Instance State → Stop
- Wait for instance to be in "stopped" state

### 3. Create AMI
- Right-click on stopped instance
- Select "Create Image"
- Provide Image name and description
- Configure additional EBS volumes if needed
- Click "Create Image"

### 4. Wait for AMI Creation
- Monitor AMI creation progress in AMIs section
- Status will change from "pending" to "available"
- This process can take several minutes

## Using Custom AMI

### 1. Launch Instance from Custom AMI
- Navigate to AMIs in EC2 Dashboard
- Select your custom AMI
- Click "Launch Instance from AMI"
- Follow standard EC2 launch process

### 2. Verify Custom Configuration
```bash
# SSH into new instance and verify
# Check if your software is installed
httpd -v  # For web server AMI
docker --version  # For Docker AMI
node --version  # For development AMI

# Verify services are running
systemctl status httpd
systemctl status docker
```


## Troubleshooting AMI Issues

### 1. **AMI Creation Fails**
- Ensure instance is stopped (for EBS-backed AMIs)
- Check available storage space
- Verify IAM permissions for AMI creation

### 2. **Instance Won't Boot from Custom AMI**
- Check if required drivers are installed
- Verify boot configuration
- Review CloudWatch logs for boot errors

### 3. **Missing Software/Configuration**
- Verify software was properly installed before AMI creation
- Check if services are enabled for auto-start
- Review installation logs

## ✅ Prerequisites

- **☁️ AWS Access**: AWS account with EC2 permissions
- **🖥️ EC2 Knowledge**: Understanding of EC2 instance management  
- **💻 System Admin**: Basic Linux/Windows administration skills
- **⚙️ Installation Skills**: Knowledge of software installation and configuration

## 🔒 Security Notes

⚠️ **Critical Security Reminders:**

- **🧹 Clean Data**: Remove all sensitive data before creating AMIs
- **🔐 Encryption**: Use encrypted EBS volumes for sensitive data
- **👥 Access Control**: Implement proper access controls for AMI sharing
- **🔄 Updates**: Regular security updates for base AMIs
- **🚫 No Credentials**: Never include credentials or keys in AMIs

## 🎯 Next Steps

After successful AMI creation and usage:

1. **📝 Documentation**: Document AMI contents and configuration
2. **🤖 Automation**: Set up automated AMI creation pipeline
3. **🧪 Testing**: Implement AMI testing procedures
4. **👥 Sharing**: Create AMI sharing policies if needed
5. **📊 Monitoring**: Monitor AMI usage and performance
6. **🔄 Maintenance**: Plan for regular AMI updates and maintenance 