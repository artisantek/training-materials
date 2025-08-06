# AWS CloudFormation Tutorial: VPC and EC2 Instance

This document provides a conceptual overview of AWS CloudFormation and a practical example of creating a VPC with multiple subnets and launching an EC2 instance within it.

## Core CloudFormation Concepts

Before diving into the template, it's important to understand the fundamental building blocks of AWS CloudFormation.

### 1. What is CloudFormation?
AWS CloudFormation is an infrastructure as code (IaC) service that allows you to model, provision, and manage AWS and third-party-resources by writing templates. It helps you automate the process of setting up your cloud environment in a repeatable and predictable way.

For more details, see the [Official AWS CloudFormation Documentation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html).

### 2. Templates
A CloudFormation template is a JSON or YAML formatted text file. This file describes all the AWS resources you want to create and configure. You can think of it as a blueprint for your AWS environment.

Learn more about [Template Formats](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-formats.html).

### 3. Stacks
When you use CloudFormation to provision resources, it creates them in a single unit called a **stack**. You manage all the resources in a stack as a single unit. You can create, update, and delete a collection of resources by creating, updating, and deleting stacks.

Read about [Working with Stacks](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/stacks.html).

### 4. Change Sets
A change set is a summary of proposed changes to your stack that you can review before implementing them. Change sets allow you to see how your changes might impact your running resources before you execute them, which is particularly useful for production environments.

Find out more about [Using Change Sets](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-updating-stacks-changesets.html).

### 5. Resources
This is a required section of the template where you declare the AWS resources that you want to include in the stack, such as an Amazon EC2 instance or an Amazon S3 bucket.

Refer to the [Resource and Property Types Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-template-resource-type-ref.html).

### 6. Parameters
Parameters allow you to input custom values to your template each time you create or update a stack. They make your templates more reusable and flexible. For example, you could have a parameter for the EC2 instance type.

More information on [Parameters](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/parameters-section-structure.html).

### 7. Mappings
Mappings are a way to "map" a key to a corresponding set of named values. They are useful for specifying conditional parameter values, similar to a lookup table. For instance, you could map an AWS region to a specific Amazon Machine Image (AMI) ID.

Read about [Mappings](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/mappings-section-structure.html).

### 8. Outputs
The `Outputs` section declares output values that you can import into other stacks, view on the AWS CloudFormation console, or query using the AWS CLI. For example, you can output the Instance ID or the Public IP address of your EC2 instance.

Learn about [Outputs](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/outputs-section-structure.html).

### 9. Intrinsic Functions
CloudFormation provides several built-in functions, known as intrinsic functions, to help you manage your stacks. These functions can be used in your templates to assign values to properties that are not available until runtime. Some common functions are `!Ref`, `!GetAtt`, `!Sub`, and `!Join`.

See the [Intrinsic Function Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/intrinsic-function-reference.html).

---

## Building CloudFormation Templates: From Simple to Complex

Below are a series of templates that start with a simple resource and gradually add more CloudFormation features to build a complete environment.

### Example 1: A Basic EC2 Instance

This is the simplest possible template to create an EC2 instance. It has only the `Resources` section with a single resource.

*   It deploys a `t2.micro` instance.
*   It uses a hardcoded AMI ID for Amazon Linux 2 in the `us-east-1` region. This will fail if you run it in another region.
*   It will launch in your **Default VPC** and use the **Default Security Group**.

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: >-
  A very simple CloudFormation template to demonstrate the basic structure.
  This creates a single EC2 instance in your default VPC.
  Note: This template uses a specific AMI ID for Amazon Linux 2 in us-east-1.
  It also uses the default security group for the VPC.
Resources:
  SimpleEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: 'ami-05ffe3c48a9991133' # Amazon Linux 2 AMI ID for us-east-1
      InstanceType: t2.micro
      Tags:
        - Key: Name
          Value: MyFirstCFN-Instance
```

### Example 2: Adding Outputs and Intrinsic Functions

Now, let's make the template more useful. We will:
*   Create a new Security Group for our instance instead of using the default.
*   Use the `!Ref` and `!GetAtt` intrinsic functions to link the resources together.
*   Add an `Outputs` section to display information about the created resources, like the Instance ID and Public IP.

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: >
  An EC2 instance with a Security Group, Outputs, and Intrinsic Functions.
Resources:
  MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: 'ami-05ffe3c48a9991133' # Amazon Linux 2 AMI ID for us-east-1
      InstanceType: t2.micro
      # Here we use !Ref to refer to the security group created below.
      SecurityGroupIds:
        - !Ref MySecurityGroup
      Tags:
        - Key: Name
          Value: EC2-With-Outputs

  MySecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Enable SSH access via port 22
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0

Outputs:
  InstanceId:
    Description: The Instance ID of the EC2 instance.
    Value: !Ref MyEC2Instance
  PublicIP:
    Description: The Public IP address of the EC2 instance.
    Value: !GetAtt MyEC2Instance.PublicIp
  SecurityGroupId:
    Description: The ID of the security group.
    Value: !Ref MySecurityGroup
```

### Example 3: Adding Parameters for Reusability

Hardcoding values like the `InstanceType` and `ImageId` is not ideal. Let's add a `Parameters` section to make the template reusable.
*   We add a parameter `InstanceType` that allows the user to select an instance size during deployment.
*   For the AMI ID, we use a public SSM parameter. This is a best practice as it automatically fetches the latest Amazon Linux 2 AMI ID for the region where you are deploying the stack.

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: >
  An EC2 instance with Parameters to make the template reusable.

Parameters:
  InstanceType:
    Description: EC2 instance type.
    Type: String
    Default: t2.micro
    AllowedValues:
      - t2.micro
      - t2.small
      - t3.micro
      - t3.small
    ConstraintDescription: Must be a valid EC2 instance type.

  LatestAmiId:
    Type: 'AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>'
    Default: '/aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2'
    Description: The latest Amazon Linux 2 AMI ID.

Resources:
  MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      # We use !Ref to get the values from the Parameters section.
      InstanceType: !Ref InstanceType
      ImageId: !Ref LatestAmiId
      SecurityGroupIds:
        - !Ref MySecurityGroup
      Tags:
        - Key: Name
          Value: EC2-With-Parameters

  MySecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Enable SSH access via port 22
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0

Outputs:
  InstanceId:
    Description: The Instance ID of the EC2 instance.
    Value: !Ref MyEC2Instance
  PublicIP:
    Description: The Public IP address of the EC2 instance.
    Value: !GetAtt MyEC2Instance.PublicIp
```

### Example 4: Advanced Template - VPC with an EC2 Instance

Finally, let's create a full networking environment for our EC2 instance. This template creates a custom VPC, which is a best practice for production environments. This template demonstrates a complete, real-world scenario.

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: >
  This template deploys a VPC with two public subnets, an Internet Gateway,
  a route table, a security group, and an EC2 instance.

Parameters:
  InstanceType:
    Description: EC2 instance type
    Type: String
    Default: t2.micro
    AllowedValues:
      - t2.micro
      - t2.small
      - t3.micro
      - t3.small
    ConstraintDescription: must be a valid EC2 instance type.

  LatestAmiId:
    Type: 'AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>'
    Default: '/aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2'
    Description: The latest Amazon Linux 2 AMI ID.

Resources:
  # VPC
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsSupport: true
      EnableDnsHostnames: true
      Tags:
        - Key: Name
          Value: MyVPC

  # Subnets
  PublicSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.1.0/24
      AvailabilityZone: !Select [ 0, !GetAZs '' ]
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: PublicSubnet1

  PublicSubnet2:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.2.0/24
      AvailabilityZone: !Select [ 1, !GetAZs '' ]
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: PublicSubnet2

  # Internet Gateway
  InternetGateway:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
        - Key: Name
          Value: MyInternetGateway

  VPCGatewayAttachment:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref VPC
      InternetGatewayId: !Ref InternetGateway

  # Route Table
  PublicRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: PublicRouteTable

  PublicRoute:
    Type: AWS::EC2::Route
    DependsOn: VPCGatewayAttachment
    Properties:
      RouteTableId: !Ref PublicRouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref InternetGateway

  # Route Table Associations
  PublicSubnet1RouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PublicSubnet1
      RouteTableId: !Ref PublicRouteTable

  PublicSubnet2RouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PublicSubnet2
      RouteTableId: !Ref PublicRouteTable

  # Security Group
  EC2SecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Enable SSH access via port 22 and HTTP on port 80
      VpcId: !Ref VPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
      Tags:
        - Key: Name
          Value: EC2SecurityGroup

  # EC2 Instance
  EC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !Ref InstanceType
      SecurityGroupIds:
        - !Ref EC2SecurityGroup
      SubnetId: !Ref PublicSubnet1
      ImageId: !Ref LatestAmiId
      Tags:
        - Key: Name
          Value: MyEC2Instance

Outputs:
  VPCId:
    Description: The ID of the created VPC
    Value: !Ref VPC
  PublicSubnet1Id:
    Description: The ID of the first public subnet
    Value: !Ref PublicSubnet1
  PublicSubnet2Id:
    Description: The ID of the second public subnet
    Value: !Ref PublicSubnet2
  EC2InstanceId:
    Description: The ID of the created EC2 instance
    Value: !Ref EC2Instance
  EC2PublicIp:
    Description: The Public IP of the EC2 Instance
    Value: !GetAtt EC2Instance.PublicIp
```

### How to Deploy a Template

You can deploy any of these CloudFormation templates using either the AWS Management Console or the AWS Command Line Interface (CLI).

**Using the AWS Management Console:**

1.  Copy one of the YAML examples above and save it into a file (e.g., `cfn-template.yaml`).
2.  Navigate to the AWS CloudFormation console.
3.  Click on **Create stack** > **With new resources (standard)**.
4.  Choose **Upload a template file** and select the `.yaml` file you just saved.
5.  Click **Next**.
6.  Enter a **Stack name** (e.g., `My-CFN-Stack`).
7.  If the template has parameters, you can review and change them.
8.  Click **Next**, and on the next page, click **Next** again.
9.  Review the details, acknowledge that AWS CloudFormation might create IAM resources, and click **Create stack**.

CloudFormation will now start provisioning the resources. You can monitor the progress in the **Events** tab. Once the status is `CREATE_COMPLETE`, all your resources are ready. Check the **Outputs** tab for any values you exported from the template.

**Using the AWS CLI:**

1.  Make sure you have the AWS CLI installed and configured.
2.  Save one of the templates to a file (e.g., `cfn-template.yaml`).
3.  Run the following command in your terminal:

```bash
aws cloudformation create-stack \
  --stack-name My-CFN-Stack \
  --template-body file://cfn-template.yaml
```
If the template has parameters, you can specify them like this:
```bash
aws cloudformation create-stack \
  --stack-name My-CFN-Stack \
  --template-body file://cfn-template.yaml \
  --parameters ParameterKey=InstanceType,ParameterValue=t2.small
```

This will start the stack creation process. You can check the status with `aws cloudformation describe-stacks --stack-name My-CFN-Stack`. 
