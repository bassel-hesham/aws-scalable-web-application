# Scalable Web Application with ALB and Auto Scaling (EC2-Based)
Manara Tech · AWS Solutions Architect Associate · Region: us-east-1

![Architecture](aws-architecture-diagram.png)

Editable source: `aws-architecture-diagram.svg` (opens in draw.io, Inkscape, Figma).

## Architecture summary
- **Network:** Production VPC `10.0.0.0/16`, two AZs, six subnets (Public `10.0.1.0/24`, `10.0.2.0/24`; Private App `10.0.11.0/24`, `10.0.12.0/24`; Private DB `10.0.21.0/24`, `10.0.22.0/24`). The Internet Gateway is attached to the VPC. Each public subnet holds its own NAT Gateway; each private app subnet routes `0.0.0.0/0` to the NAT Gateway in its own AZ. DB subnets have local routes only.
- **Entry path:** Internet Users → Internet Gateway → ALB → HTTPS :443 listener → target group → healthy EC2 instances. The Route 53 Alias A record points directly to the ALB. AWS WAF is a Web ACL (OWASP Top 10 rules) associated with the ALB, not a hop in the path.
- **Compute:** one Auto Scaling Group across both private app subnets. The Launch Template and Target Tracking policy sit outside the AZ containers and are attached to that one ASG. Instances are private with no public IPs.
- **Data:** RDS MySQL Multi-AZ in the private DB subnets: primary in one AZ, synchronous standby in the other, automated failover, not publicly accessible.
- **Static content:** CloudFront serves static assets only, from its own distribution domain or a separate DNS record. Its origin request goes to the ALB (assumed origin, via the IGW). Dynamic traffic does not go through CloudFront, and the main Route 53 Alias still targets the ALB.
- **Security:** SG-ALB allows HTTPS 443 from intended clients and, for the CloudFront origin path, only from the CloudFront origin-facing managed prefix list (not open to all sources); SG-ALB → SG-EC2 (app port from SG-ALB only) → SG-RDS (DB port from SG-EC2 only); NACLs on every subnet tier.
- **Operations:** CloudWatch collects ALB/EC2/ASG metrics and logs → dashboards and alarms → SNS → administrator. Systems Manager Session Manager gives shell access to private instances with no bastion or SSH. Route 53 health checks monitor the ALB endpoint.

## Assumptions
- MySQL chosen (brief allows MySQL or PostgreSQL).
- The ALB is the assumed CloudFront origin; the brief does not define one. No extra component was added.
- SG-ALB client sources are labelled "intended clients" because the brief does not fix them.
- Systems Manager needs an IAM Instance Profile on the EC2 instances and outbound connectivity via NAT Gateway or VPC Endpoints (noted on the diagram).
- Icon tiles use AWS category colours. Swap in the official AWS Architecture Icons if your evaluator requires them.

## Validation checklist
| # | Requirement | Status |
|---|---|---|
| 1 | Exactly two AZs | Covered |
| 2 | Six subnets in the correct VPC and AZ, with CIDRs | Covered |
| 3 | NAT Gateway A / B in Public Subnet A / B | Covered |
| 4 | IGW attached to the VPC | Covered |
| 5 | Internet-facing ALB spans both public subnets | Covered |
| 6 | All EC2 in private app subnets, no public IPs | Covered |
| 7 | One ASG spanning both AZs | Covered |
| 8 | Launch Template and Target Tracking shown outside the AZs, attached to the one ASG | Covered |
| 9 | RDS Multi-AZ in private DB subnets, not public | Covered |
| 10 | Standby labelled as RDS Multi-AZ standby | Covered |
| 11 | WAF Web ACL with OWASP Top 10 associated with the ALB (not in the path) | Covered |
| 12 | CloudFront for static caching, origin path to ALB labelled | Covered |
| 13 | Route 53 Alias → ALB, plus health checks | Covered |
| 14 | SGs as resource controls, NACLs per subnet, route tables labelled | Covered |
| 15 | CloudWatch (dashboards, alarms), SNS, Systems Manager outside the VPC | Covered |
| 16 | Solid = traffic, dashed = monitoring/management, legend | Covered |
