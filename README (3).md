# Scalable Web Application with ALB and Auto Scaling

**AWS Solutions Architect -- Associate Graduation Project \| Manara
Tech**\
**Architecture type:** EC2-based, two Availability Zones\
**AWS Region:** `us-east-1`\
**Project status:** Architecture design and documentation only. The
architecture shown here is a proposed design; it has not been deployed
or validated in a live AWS environment.

## 1. Project Overview

This project presents a highly available and scalable web application
architecture on AWS. It uses an Application Load Balancer (ALB) to
distribute application traffic across EC2 instances managed by an Auto
Scaling Group (ASG). The application instances run in private
application subnets across two Availability Zones, while an Amazon RDS
for MySQL database uses a Multi-AZ deployment in private database
subnets.

Amazon CloudFront is used to cache and deliver static assets. Amazon
Route 53 provides the application DNS record, pointing directly to the
ALB for dynamic application requests. AWS WAF is associated with the ALB
to inspect and filter incoming HTTP(S) requests.

The design also includes network isolation, managed instance access
through AWS Systems Manager Session Manager, and monitoring and
notifications through Amazon CloudWatch and Amazon SNS.

## 2. Architecture Diagram

The architecture diagram is available in the `architecture/` directory.

- `architecture-diagram (1).png` — diagram preview.
- `architecture-diagram (2).drawio` — editable source file created with diagrams.net (draw.io).

[Scalable Web Application Architecture](https://github.com/bassel-hesham/aws-scalable-web-application/blob/main/architecture/architecture-diagram%20(1).png) ([image](https://github.com/bassel-hesham/aws-scalable-web-application/raw/main/architecture/architecture-diagram%20(1).png))
## 3. Request Flow

### Dynamic application requests

1.  A user accesses the application domain.
2.  Amazon Route 53 resolves the application DNS name using an Alias
    record pointing to the Application Load Balancer.
3.  The request reaches the public ALB over HTTPS on port `443`.
4.  AWS WAF, associated with the ALB, evaluates the incoming request
    according to the configured Web ACL rules.
5.  The ALB listener forwards accepted requests to the Target Group.
6.  The Target Group distributes requests to healthy EC2 instances in
    the private application subnets.
7.  The application instances communicate with the RDS for MySQL primary
    database over TCP port `3306`.

HTTP requests on port `80` are redirected to HTTPS on port `443`.

### Static asset delivery

1.  Users request static assets through the CloudFront distribution
    domain.
2.  CloudFront checks its cache for the requested object.
3.  On a cache hit, CloudFront serves the cached asset.
4.  On a cache miss, CloudFront fetches the object from the ALB origin
    over HTTPS and can cache the response according to the
    distribution's cache behavior.

CloudFront is intended for static asset delivery only in this design.
Dynamic application requests use the Route 53 Alias record that points
directly to the ALB; they are not routed through CloudFront.

## 4. Network Design

The architecture uses one VPC across two Availability Zones, with
separate public, private application, and private database subnets.

### VPC and subnet CIDR allocation

| Network tier | Availability Zone A (`us-east-1a`) | Availability Zone B (`us-east-1b`) |
|---|---:|---:|
| VPC | `10.0.0.0/16` | `10.0.0.0/16` |
| Public subnet | `10.0.1.0/24` | `10.0.2.0/24` |
| Private application subnet | `10.0.11.0/24` | `10.0.12.0/24` |
| Private database subnet | `10.0.21.0/24` | `10.0.22.0/24` |

### Public subnets

The public subnets host the internet-facing Application Load Balancer
and one NAT Gateway per Availability Zone. Their route tables provide a
default route (`0.0.0.0/0`) to the Internet Gateway.

### Private application subnets

The EC2 instances managed by the Auto Scaling Group run in the private
application subnets. Each application subnet uses the NAT Gateway in the
same Availability Zone for permitted outbound internet connectivity.

The instances do not require inbound SSH access. Administrative access
is provided through AWS Systems Manager Session Manager.

### Private database subnets

The RDS database is placed in private database subnets across the two
Availability Zones. These subnets do not have a default route to the
internet.

## 5. Compute and Load Balancing

### Application Load Balancer

The internet-facing Application Load Balancer operates at Layer 7 and is
deployed across both public subnets.

-   HTTPS listener: TCP port `443`.
-   HTTP listener: TCP port `80`, configured to redirect requests to
    HTTPS.
-   An ACM SSL/TLS certificate is attached to the ALB HTTPS listener.
-   Accepted requests are forwarded to the application Target Group.
-   The Target Group uses HTTP port `80` for application traffic and
    health checks.

### EC2 Auto Scaling Group

The Auto Scaling Group spans the two private application subnets and
uses a Launch Template to define the EC2 instances.

| Setting | Value |
|---|---:|
| Minimum capacity | 2 instances |
| Desired capacity | 2 instances |
| Maximum capacity | 4 instances |
| Scaling policy | Target Tracking |
| Target metric | Average CPU utilization |
| Target value | 50% |

The ALB Target Group health checks provide health information about the
application instances. The design assumes that the Auto Scaling Group is
configured to use Elastic Load Balancing health checks so that unhealthy
instances can be replaced.

## 6. Database Design

Amazon RDS for MySQL is deployed using a Multi-AZ DB instance
configuration.

-   The primary DB instance handles application database operations.
-   A standby DB instance is maintained in a different Availability
    Zone.
-   Synchronous replication and automatic failover support database
    availability.
-   The standby is intended for failover; it is not used as a
    read-scaling endpoint in this design.
-   The database is located in private database subnets and is not
    publicly accessible.

The application connects to MySQL over TCP port `3306`.

## 7. Security

### AWS WAF

An AWS WAF Web ACL is associated with the Application Load Balancer. AWS
Managed Rules are used to help inspect and protect incoming web
requests, including common web attack patterns.

### Security Groups

The diagram represents the following security-group traffic
relationships:

| Security Group | Inbound traffic |
|---|---|
| ALB-SG | TCP `443` from the internet; TCP `80` from the internet for redirect to HTTPS |
| App-SG | TCP `80` from ALB-SG |
| DB-SG | TCP `3306` from App-SG |

Security Groups are stateful. The application instances are not exposed
directly to inbound internet traffic, and no inbound SSH rule is part of
this design.

### Network ACLs

Network ACLs can provide an additional subnet-level network control
layer. Because NACLs are stateless, their inbound and outbound rules
must account for the required application traffic and ephemeral response
ports. Detailed NACL rule numbers are not specified in this architecture
diagram.

### Instance access

AWS Systems Manager Session Manager is used for administrative access to
EC2 instances without opening inbound SSH (port `22`). The EC2 instances
use an IAM instance profile with the `AmazonSSMManagedInstanceCore`
permissions policy. In this design, the instances reach Systems Manager
through outbound connectivity provided by the NAT Gateways.

## 8. DNS and Content Delivery

### Amazon Route 53

Route 53 provides an Alias record that points the application domain
directly to the ALB. This supports the dynamic application request path.
Route 53 health evaluation for an ALB Alias is distinct from the ALB
Target Group health checks.

### Amazon CloudFront

CloudFront is configured to deliver and cache static assets only. Its
distribution uses the ALB as its origin, and the origin fetch occurs
over HTTPS when an object is not available in the CloudFront cache.

The application domain's dynamic request path remains Route 53 to ALB,
as shown in the diagram.

## 9. Monitoring and Notifications

Amazon CloudWatch is used for dashboards, metrics, and alarms related to
the architecture. Amazon SNS is used as the notification destination for
configured alarms, such as email or SMS notifications.

The diagram shows the monitoring and alarm flow from AWS resources to
CloudWatch and then to an SNS topic. Exact alarm thresholds and
notification subscriptions are configuration details and are not defined
in the diagram.

## 10. Availability and Resilience

The architecture improves availability through multiple design choices:

-   The ALB is deployed across two Availability Zones.
-   The Auto Scaling Group maintains a minimum and desired capacity of
    two EC2 instances across the two application subnets.
-   Target Tracking can increase or decrease the number of EC2 instances
    based on average CPU utilization.
-   One NAT Gateway is deployed in each Availability Zone, and each
    application subnet uses its same-AZ NAT Gateway.
-   RDS Multi-AZ maintains a standby database instance in a separate
    Availability Zone for automatic failover.

A NAT Gateway in one Availability Zone is not used as the default
outbound path for the application subnet in the other Availability Zone.

## 11. Design Assumptions and Scope

-   This repository documents an architecture design; it does not claim
    that AWS resources have been deployed, tested, or benchmarked.
-   The workload region shown is `us-east-1`.
-   The architecture uses two Availability Zones: `us-east-1a` and
    `us-east-1b`.
-   The ALB is internet-facing and spans both public subnets.
-   The ALB uses an ACM certificate for HTTPS.
-   CloudFront is used for static assets only; dynamic application
    requests go directly from Route 53 to the ALB.
-   CloudFront uses the ALB as its HTTPS origin for static asset cache
    misses.
-   The RDS engine is MySQL, and the application-to-database connection
    uses TCP port `3306`.
-   The application Target Group uses HTTP port `80`.
-   Each private application subnet uses its same-AZ NAT Gateway for
    outbound connectivity.
-   The private database subnets have no internet default route.
-   EC2 administrative access uses Systems Manager Session Manager, not
    SSH.
-   Exact instance sizes, database instance class, domain name,
    certificate ARN, application implementation, NACL rule numbers,
    alarm thresholds, and WAF rule-group selections are
    deployment-specific details and are intentionally not invented here.

## 12. Repository Deliverables

The project submission is intended to include:

1.  A solution architecture diagram.
2.  The editable diagram source file.
3.  This README documenting the proposed architecture and design
    decisions.

## 13. Disclaimer

This is an architecture design project prepared for learning and
documentation purposes. The diagram and README describe the intended AWS
solution. They should not be interpreted as evidence of a live
deployment, successful testing, production readiness, or measured
performance.
