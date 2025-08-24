# CloudLaunch AWS Infrastructure

## Project Overview

This project implements the CloudLaunch platform infrastructure on AWS, utilizing core services including S3, IAM, and VPC to create a secure, scalable web hosting environment with proper network segmentation and access controls.

## Architecture Overview

```
CloudLaunch Platform Architecture:

Internet
    |
    v
CloudFront (Optional CDN)
    |
    v
S3 Static Website (cloudlaunch-site-bucket)
    |
    v
VPC (10.0.0.0/16)
    |
    +-- Public Subnet (10.0.1.0/24)
    |   └── Future Load Balancers/Bastion Hosts
    |
    +-- App Subnet (10.0.2.0/24) [Private]
    |   └── Future Application Servers
    |
    +-- DB Subnet (10.0.3.0/28) [Private]
        └── Future Database Servers

Additional S3 Storage:
- cloudlaunch-private-bucket (Secure file storage)
- cloudlaunch-visible-only-bucket (List-only access)
```

## Task 1: Static Website Hosting with S3 and IAM

### What Was Implemented

#### S3 Bucket Configuration
- **cloudlaunch-site-24082025**: Public bucket configured for static website hosting
  - Enabled static website hosting with `index.html` as default document
  - Applied bucket policy for public read access to website content
  - Serves as the main website frontend accessible via HTTP

- **cloudlaunch-private-24082025**: Private secure storage bucket
  - All public access blocked via bucket policy
  - Used for storing sensitive application data and user uploads
  - Accessible only via IAM credentials

- **cloudlaunch-visible-only-24082025**: List-only access bucket
  - Private bucket with restricted access
  - Users can list contents but cannot download files
  - Demonstrates granular permission control

#### IAM Security Implementation
- **Created IAM User**: `cloudlaunch-user` with programmatic access
- **Custom IAM Policy**: Implemented principle of least privilege with specific permissions:
  - **S3 ListBucket**: Access to list contents of all three buckets
  - **S3 GetObject**: Read access to site bucket objects only
  - **S3 GetObject + PutObject**: Read/write access to private bucket objects
  - **EC2 VPC Read-Only**: Limited VPC resource inspection capabilities
  - **Regional Restrictions**: VPC access restricted to us-east-1 region only

#### CloudFront Distribution (Bonus)
- Configured global CDN for improved performance
- HTTPS redirect for secure content delivery
- Custom origin pointing to S3 static website endpoint

### Key Security Features
✅ **Principle of Least Privilege**: IAM user has only necessary permissions  
✅ **No Delete Permissions**: Prevents accidental data loss  
✅ **Resource-Specific Access**: Permissions tied to specific bucket ARNs  
✅ **Regional Restrictions**: VPC access limited to designated region  
✅ **Public Access Controls**: Proper separation of public vs private content  

## Task 2: VPC Network Design

### What Was Implemented

#### VPC Foundation
- **VPC CIDR**: 10.0.0.0/16 providing 65,536 IP addresses
- **Region**: us-east-1 for optimal performance and availability
- **DNS Resolution**: Enabled for internal service discovery

#### Subnet Architecture
- **Public Subnet** (10.0.1.0/24):
  - 254 available IP addresses
  - Availability Zone: us-east-1a
  - Internet access via Internet Gateway
  - Future home for load balancers and bastion hosts

- **Application Subnet** (10.0.2.0/24) - Private:
  - 254 available IP addresses  
  - Availability Zone: us-east-1b
  - No direct internet access
  - Isolated environment for application servers

- **Database Subnet** (10.0.3.0/28) - Private:
  - 14 available IP addresses
  - Availability Zone: us-east-1c
  - Maximum security isolation for database servers
  - Smaller subnet size for cost optimization

#### Network Routing
- **Internet Gateway**: Provides internet access for public subnet
- **Route Tables**:
  - Public route table: Routes 0.0.0.0/0 traffic to Internet Gateway
  - Application route table: Internal VPC routing only
  - Database route table: Most restrictive routing

#### Security Groups
- **Application Security Group** (`cloudlaunch-app-sg`):
  - Inbound: HTTP (port 80) from VPC CIDR (10.0.0.0/16)
  - Outbound: All traffic (default)
  - Allows load balancer to application server communication

- **Database Security Group** (`cloudlaunch-db-sg`):
  - Inbound: MySQL (port 3306) from Application Subnet only (10.0.2.0/24)
  - Outbound: All traffic (default)
  - Maximum security - database only accessible from app tier

### Network Security Features
✅ **Network Segmentation**: Three-tier architecture with proper isolation  
✅ **Defense in Depth**: Multiple security layers (NACLs, Security Groups, Routing)  
✅ **Principle of Least Privilege**: Database access restricted to application subnet only  
✅ **No Internet Access**: Private subnets isolated from internet  
✅ **Port Restrictions**: Only necessary ports opened in security groups  

## Deployment Summary

### Resources Created
- **3 S3 Buckets** with different access patterns
- **1 IAM User** with custom policy
- **1 VPC** with complete network infrastructure
- **3 Subnets** across different availability zones
- **1 Internet Gateway** for public internet access
- **3 Route Tables** for traffic management
- **2 Security Groups** with restrictive rules
- **1 CloudFront Distribution** for global content delivery

### Testing and Verification
All components were tested to ensure:
- Static website accessible via S3 endpoint and CloudFront
- IAM user can perform allowed operations and blocked from restricted ones
- VPC resources properly isolated with correct routing
- Security groups enforcing access controls
- Network connectivity working as designed

## Cost Optimization

### Implemented Cost-Saving Measures
- **Right-sized Subnets**: Database subnet uses /28 instead of /24
- **Regional Deployment**: Single region deployment reduces data transfer costs
- **S3 Standard Storage**: Appropriate for frequently accessed website content
- **No NAT Gateway**: Private subnets don't require internet access

## Security Best Practices Implemented

### Data Protection
- Sensitive data stored in private S3 buckets
- Public bucket contains only website assets
- No database credentials in code or configuration files

### Access Control
- IAM policies follow least privilege principle
- Multi-factor authentication recommended for root account
- Regular access key rotation scheduled

### Network Security
- Private subnets have no internet gateway routes
- Security groups act as virtual firewalls
- Database tier isolated from public internet

## Submission Instructions

### Console Access Details
- **AWS Console URL**: `https://062823296349.signin.aws.amazon.com/console`
- **Account ID**: `062823296349`

### IAM User Credentials
- **Username**: `cloudlaunch-user`
- **Temporary Password**: `TempPassword123!`
- **Access Key ID**: `AKIAQ5IEPNVO5VENJPSZ`
- **Secret Access Key**: `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`
- **Password Reset Required**: ✅ Yes - User must change password on first console login

### Login Instructions
1. Navigate to the AWS Console URL above
2. Enter Account ID (if using generic console URL)
3. Username: `cloudlaunch-user`
4. Password: `TempPassword123!`
5. System will prompt to create new password on first login
6. Follow AWS password policy requirements for new password

## S3 STATIC LINK
http://cloudlaunch-site-24082025.s3-website-eu-west-1.amazonaws.com/

## CLOUDFRONT URL
du301f9xgsmix.cloudfront.net

## FORMATTED JSON POLICY
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "ListAllBuckets",
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket"
            ],
            "Resource": [
               "arn:aws:s3:::cloudlaunch-site-24082025",
                "arn:aws:s3:::cloudlaunch-private-24082025",
                "arn:aws:s3:::cloudlaunch-visible-only-24082025"
            ]
        },
        {
            "Sid": "GetObjectSiteBucket",
            "Effect": "Allow",
            "Action": [
                "s3:GetObject"
            ],
            "Resource": "arn:aws:s3:::cloudlaunch-site-24082025/*"
        },
        {
            "Sid": "GetPutObjectPrivateBucket",
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": "arn:aws:s3:::cloudlaunch-private-24082025/*"
        },
        {
            "Sid": "VPCReadOnlyAccess",
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeVpcs",
                "ec2:DescribeSubnets",
                "ec2:DescribeRouteTables",
                "ec2:DescribeSecurityGroups",
                "ec2:DescribeInternetGateways",
                "ec2:DescribeNetworkAcls"
            ],
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "ec2:Region": "eu-west-1"
                }
            }
        }
    ]
}



**Documentation**: This README serves as the primary reference for the CloudLaunch AWS infrastructure implementation
