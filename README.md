# Securely Deploying Resources in a VPC using CloudFormation - Lab Project

## Overview

This project demonstrates the creation and deployment of a secure VPC infrastructure using AWS CloudFormation. The lab focuses on implementing networking best practices including public and private subnet architecture, NAT gateways for secure internet access, and Systems Manager Session Manager for secure instance management.

## Project Objectives

- Create a well-architected VPC spanning multiple Availability Zones
- Deploy resources securely within public and private subnets
- Implement proper routing and security group configurations
- Demonstrate secure instance management using AWS Systems Manager Session Manager
- Validate network connectivity and internet access patterns

## Architecture Overview

### Network Design
- **VPC**: 10.0.0.0/16 CIDR block across 2 Availability Zones
- **Public Subnet**: 10.0.1.0/24 (AZ-1) with direct internet access
- **Private Subnet**: 10.0.2.0/24 (AZ-2) with NAT gateway for outbound internet access
- **Internet Gateway**: For public subnet internet connectivity
- **NAT Gateway**: For private subnet outbound internet access

### Infrastructure Components
- **Web Server**: Apache HTTP server in public subnet
- **Private Instance**: Amazon Linux instance in private subnet
- **Security Groups**: Configured for HTTP/HTTPS and ICMP traffic
- **VPC Endpoints**: For Systems Manager communication
- **IAM Roles**: For Systems Manager instance management

## CloudFormation Template Analysis

### Key Resources Created

#### 1. VPC Infrastructure (`cfn-week2-lab.yaml`)

**VPC and Subnets:**
```yaml
VPC:
  Type: AWS::EC2::VPC
  Properties:
    CidrBlock: 10.0.0.0/16
    EnableDnsSupport: true
    EnableDnsHostnames: true

PublicSubnet:
  Type: AWS::EC2::Subnet
  Properties:
    CidrBlock: 10.0.1.0/24
    AvailabilityZone: !Select [0, !GetAZs '']
    MapPublicIpOnLaunch: true

PrivateSubnet:
  Type: AWS::EC2::Subnet
  Properties:
    CidrBlock: 10.0.2.0/24
    AvailabilityZone: !Select [1, !GetAZs '']
    MapPublicIpOnLaunch: false
```

#### 2. Networking Components

**Internet Gateway and NAT Gateway:**
- Internet Gateway provides direct internet access for public subnet
- NAT Gateway enables outbound internet connectivity for private subnet
- Proper route table associations for traffic routing

**Route Tables:**
- Public Route Table: Routes 0.0.0.0/0 to Internet Gateway
- Private Route Table: Routes 0.0.0.0/0 to NAT Gateway

#### 3. Security Groups

**Public Security Group:**
- Inbound: HTTP (80), HTTPS (443), ICMP from private subnet
- Outbound: All traffic (0.0.0.0/0)

**Private Security Group:**
- Inbound: ICMP from public security group
- Outbound: All traffic (0.0.0.0/0)

#### 4. VPC Endpoints for Systems Manager
- SSM Endpoint: `com.amazonaws.region.ssm`
- SSM Messages Endpoint: `com.amazonaws.region.ssmmessages`
- EC2 Messages Endpoint: `com.amazonaws.region.ec2messages`

#### 5. EC2 Instances

**Public Web Server:**
- Instance Type: t3.micro
- AMI: Latest Amazon Linux 2
- User Data: Installs Apache HTTP server and SSM agent
- Custom HTML content displaying full name

**Private Instance:**
- Instance Type: t3.micro
- AMI: Latest Amazon Linux 2
- User Data: Installs and configures SSM agent

## Lab Results and Validation

### 1. Web Server Accessibility
**Test**: Access Apache web server from public internet
- **URL**: `ec2-18-159-106-40.eu-central-1.compute.amazonaws.com`
- **Content**: "Hello from my CloudFormation Web Server"
- **Name Display**: "Ibrahim Anyars Safianu"
- **Status**: ✅ Successfully accessible
- **Screenshot**: `Ibrahim_Anyars_Safianu_Wk2_lab_Image_1.png`

### 2. Private Instance Connectivity from Web Server
**Test**: Ping private instance from public web server using Session Manager
- **Command**: `ping 10.0.2.71`
- **Source**: Public Web Server (10.0.1.233)
- **Target**: Private Instance (10.0.2.71)
- **Result**: 14 packets transmitted, 14 received, 0% packet loss
- **Average RTT**: 0.567ms
- **Status**: ✅ Internal communication successful
- **Screenshot**: `Ibrahim_Anyars_Safianu_Wk2_lab_Image_2.png`

### 3. Internet Connectivity from Web Server
**Test**: Ping google.com from public web server
- **Command**: `ping google.com`
- **Target IP**: 142.250.184.206
- **Result**: 13 packets transmitted, 13 received, 0% packet loss
- **Average RTT**: 0.870ms
- **Status**: ✅ Direct internet access confirmed
- **Screenshot**: `Ibrahim_Anyars_Safianu_Wk2_lab_Image_3.png`

### 4. Web Server Connectivity from Private Instance
**Test**: Ping public web server from private instance
- **Command**: `ping 10.0.1.233`
- **Source**: Private Instance (10.0.2.71)
- **Target**: Public Web Server (10.0.1.233)
- **Result**: 14 packets transmitted, 14 received, 0% packet loss
- **Average RTT**: 0.598ms
- **Status**: ✅ Bi-directional internal communication confirmed
- **Screenshot**: `Ibrahim_Anyars_Safianu_Wk2_lab_Image_4.png`

### 5. Internet Connectivity from Private Instance
**Test**: Ping amazon.com from private instance via NAT Gateway
- **Command**: `ping amazon.com`
- **Target IP**: 52.94.236.248
- **Result**: 14 packets transmitted, 14 received, 0% packet loss
- **Average RTT**: 91.626ms
- **Status**: ✅ Outbound internet access via NAT Gateway confirmed
- **Screenshot**: `Ibrahim_Anyars_Safianu_Wk2_lab_Image_5.png`

## Security Features Implemented

### 1. Network Segmentation
- **Public Subnet**: Hosts internet-facing resources
- **Private Subnet**: Hosts backend resources without direct internet access
- **Security Groups**: Act as virtual firewalls controlling traffic flow

### 2. Secure Remote Access
- **No SSH Keys Required**: Using Systems Manager Session Manager
- **IAM-based Access**: Instance access controlled via IAM roles
- **Encrypted Communication**: All Session Manager traffic encrypted in transit

### 3. Controlled Internet Access
- **Public Subnet**: Direct internet access via Internet Gateway
- **Private Subnet**: Outbound-only internet access via NAT Gateway
- **No Inbound Internet Access**: Private instances protected from direct internet access

### 4. VPC Endpoints
- **Private Communication**: Systems Manager traffic stays within AWS network
- **No Internet Gateway Dependency**: Private instances can use SSM without internet access
- **Enhanced Security**: Reduces attack surface by avoiding internet routing

## Best Practices Demonstrated

### 1. Infrastructure as Code (IaC)
- **Version Control**: CloudFormation template stored in version control
- **Reproducibility**: Infrastructure can be recreated consistently
- **Documentation**: Template serves as infrastructure documentation

### 2. Least Privilege Access
- **IAM Roles**: Instances have only necessary permissions for SSM
- **Security Groups**: Minimal required ports and protocols opened
- **Network ACLs**: Default deny with specific allow rules

### 3. High Availability Design
- **Multi-AZ Deployment**: Resources spread across different availability zones
- **Redundancy**: NAT Gateway provides reliable outbound connectivity
- **Scalability**: Architecture supports horizontal scaling

### 4. Monitoring and Management
- **Systems Manager Integration**: Centralized instance management
- **CloudFormation Stack**: Organized resource management and updates
- **Tagging Strategy**: Consistent resource tagging for organization

## Technical Implementation Details

### CloudFormation Stack Parameters
- **Template Version**: 2010-09-09
- **Description**: CloudFormation template for a VPC with public and private subnets
- **Stack Name**: Used as prefix for all resource names

### User Data Scripts
**Public Web Server:**
```bash
#!/bin/bash
yum update -y
yum install -y https://s3.amazonaws.com/ec2-downloads-windows/SSMAgent/latest/linux_amd64/amazon-ssm-agent.rpm
systemctl start amazon-ssm-agent
systemctl enable amazon-ssm-agent
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>Hello from my CloudFormation Web Server</h1><br>My full name is: Ibrahim Anyars Safianu" > /var/www/html/index.html
```

**Private Instance:**
```bash
#!/bin/bash
yum update -y
yum install -y https://s3.amazonaws.com/ec2-downloads-windows/SSMAgent/latest/linux_amd64/amazon-ssm-agent.rpm
systemctl start amazon-ssm-agent
systemctl enable amazon-ssm-agent
```


## Files Included

### Infrastructure Code
1. `cfn-week2-lab.yaml` - Complete CloudFormation template
2. `Securely deploying resources in a VPC using cfn LAB.docx` - Lab requirements document

### Validation Screenshots
1. `Ibrahim_Anyars_Safianu_Wk2_lab_Image_1.png` - Web server accessibility test
2. `Ibrahim_Anyars_Safianu_Wk2_lab_Image_2.png` - Private instance ping from web server
3. `Ibrahim_Anyars_Safianu_Wk2_lab_Image_3.png` - Internet connectivity from web server
4. `Ibrahim_Anyars_Safianu_Wk2_lab_Image_4.png` - Web server ping from private instance
5. `Ibrahim_Anyars_Safianu_Wk2_lab_Image_5.png` - Internet connectivity from private instance

## Deployment Instructions

### Prerequisites
- AWS CLI configured with appropriate permissions
- CloudFormation access permissions
- Systems Manager permissions for session management

### Deployment Steps
1. **Deploy CloudFormation Stack**:
   ```bash
   aws cloudformation create-stack \
     --stack-name vpc-lab-stack \
     --template-body file://cfn-week2-lab.yaml \
     --capabilities CAPABILITY_IAM
   ```

2. **Verify Stack Creation**:
   ```bash
   aws cloudformation describe-stacks --stack-name vpc-lab-stack
   ```

3. **Test Web Server**:
   - Get public IP from CloudFormation outputs
   - Access via web browser

4. **Test Session Manager**:
   - Connect to instances via AWS Console or CLI
   - Perform connectivity tests

## Learning Outcomes

This lab successfully demonstrates:

1. **VPC Architecture Design**: Understanding of public/private subnet patterns
2. **CloudFormation Proficiency**: Infrastructure as Code implementation
3. **Security Best Practices**: Principle of least privilege and defense in depth
4. **Network Routing**: Internet Gateway and NAT Gateway configuration
5. **Systems Management**: Secure instance access without traditional SSH
6. **Connectivity Testing**: Network validation and troubleshooting skills

## Future Enhancements

Potential improvements for production environments:

1. **Multi-AZ NAT Gateways**: Redundancy for high availability
2. **Application Load Balancer**: For web server high availability
3. **Auto Scaling Groups**: Dynamic scaling based on demand
4. **CloudWatch Monitoring**: Comprehensive logging and alerting
5. **AWS Config Rules**: Automated compliance checking
6. **VPC Flow Logs**: Network traffic analysis and security monitoring

## Conclusion

This CloudFormation-based VPC lab successfully demonstrates secure infrastructure deployment using AWS best practices. All lab requirements were met, including proper network segmentation, secure instance management, and comprehensive connectivity validation.

## Author

**Ibrahim Anyars Safianu**
- Lab Completion Date: August 2025
- AWS Region: eu-central-1
- CloudFormation Stack: Successfully deployed and tested

---

*This project demonstrates practical application of AWS VPC concepts, CloudFormation Infrastructure as Code principles, and secure cloud architecture design patterns.*
