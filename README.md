# AWS Scalable Web Application

## Project Overview

This project demonstrates the design and implementation of a highly available, scalable, and secure web application on AWS.

The architecture is designed to handle varying traffic loads, maintain application availability, and provide reliable database services.

### Key Objectives

- High Availability
- Scalability
- Security
- Performance Optimization
- Fault Tolerance

## Architecture Diagram

![AWS Architecture Diagram](./aws-architecture-diagram.png)

## AWS Services

| Service | Purpose |
|---|---|
| Amazon EC2 | Hosts the web application |
| Application Load Balancer (ALB) | Distributes incoming traffic |
| Auto Scaling Group | Automatically adjusts EC2 capacity |
| Amazon RDS Multi-AZ | Provides a highly available database |
| Amazon CloudFront | Delivers content with low latency |
| AWS IAM | Manages access and permissions |
| Amazon VPC | Provides network isolation |
| Security Groups | Controls inbound and outbound traffic |
## How It Works

1. Users access the application through Amazon CloudFront.
2. CloudFront forwards dynamic requests to the Application Load Balancer.
3. The Application Load Balancer distributes incoming traffic across EC2 instances.
4. EC2 instances run inside an Auto Scaling Group across multiple Availability Zones.
5. The application communicates with Amazon RDS to store and retrieve data.
6. RDS Multi-AZ provides database availability through standby replication.

## Security

- Network isolation using Amazon VPC.
- Traffic control using Security Groups.
- Access management using AWS IAM.
- Private subnets for application and database resources.

