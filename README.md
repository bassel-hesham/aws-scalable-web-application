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

