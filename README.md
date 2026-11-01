# AWS Activity – Optimize Utilization

## Project Overview

This lab focused on optimizing AWS resource utilization and reducing infrastructure costs. I worked with an Amazon EC2 instance hosting a Café website and used the AWS Pricing Calculator to compare infrastructure costs before and after optimization.

## Objectives

- Remove an unnecessary local MariaDB installation from an EC2 instance.
- Right-size an EC2 instance to reduce compute costs.
- Verify that the hosted website remains accessible after optimization.
- Use AWS Pricing Calculator to estimate and compare monthly infrastructure costs.

## AWS Services and Tools Used

- Amazon EC2
- Amazon RDS for MariaDB
- Amazon Elastic Block Store (EBS)
- AWS CLI
- AWS Pricing Calculator
- PuTTY (SSH connection)

## Task 1: Optimize the EC2 Instance

### 1. Remove the Local MariaDB Installation

Connected to the Café EC2 instance through SSH using PuTTY.

Stopped the MariaDB service:

```bash
sudo systemctl stop mariadb
```

Uninstalled the MariaDB server:

```bash
sudo yum -y remove mariadb-server
```

This removed the locally installed database server because the lab architecture uses Amazon RDS for MariaDB.

### 2. Identify the EC2 Instance

Configured the AWS CLI with the lab credentials and the `us-west-2` region.

Retrieved the Café instance ID:

```bash
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=CafeInstance" \
  --query "Reservations[*].Instances[*].InstanceId"
```

### 3. Stop the EC2 Instance

```bash
aws ec2 stop-instances \
  --instance-ids <instance-id>
```

### 4. Change the Instance Type

Changed the instance type from `t3.small` to `t3.micro`:

```bash
aws ec2 modify-instance-attribute \
  --instance-id <instance-id> \
  --instance-type "{\"Value\": \"t3.micro\"}"
```

### 5. Restart the Instance

```bash
aws ec2 start-instances \
  --instance-ids <instance-id>
```

Verified the instance type and status:

```bash
aws ec2 describe-instances \
  --instance-ids <instance-id> \
  --query "Reservations[*].Instances[*].[InstanceType,PublicDnsName,PublicIpAddress,State.Name]"
```

The instance was running with the `t3.micro` configuration.

### 6. Test the Café Website

Opened the Café website using the EC2 public DNS:

```text
http://<public-dns>/cafe
```

The website loaded successfully after the optimization.

## Task 2: Estimate Costs Using AWS Pricing Calculator

Created an estimate using AWS Pricing Calculator.

### Before Optimization

| Resource | Configuration |
|---|---|
| EC2 instance | t3.small |
| EC2 storage | 40 GB gp2 |
| Operating system | Linux |
| Pricing | On-Demand |
| Usage | Constant |
| RDS engine | MariaDB |
| RDS instance | db.t3.micro |
| RDS storage | 20 GB gp2 |
| Deployment | Single-AZ |

### After Optimization

| Resource | Configuration |
|---|---|
| EC2 instance | t3.micro |
| EC2 storage | 20 GB gp2 |
| Operating system | Linux |
| Pricing | On-Demand |
| Usage | Constant |
| RDS engine | MariaDB |
| RDS instance | db.t3.micro |
| RDS storage | 20 GB gp2 |
| Deployment | Single-AZ |

The RDS configuration remained unchanged.

## Cost Comparison

| Cost | Before | After |
|---|---:|---:|
| Estimated monthly total | $74.04 | $64.45 |

**Estimated monthly savings:** $9.59

**Estimated annual savings:** $115.08

The optimized estimate showed an approximately 12.95% reduction in monthly costs.

*Note: These are the estimates displayed by AWS Pricing Calculator during this lab.*

## Key Lessons Learned

- Right-sizing EC2 instances can help reduce unnecessary compute costs.
- Removing redundant services can simplify infrastructure.
- Amazon RDS can provide managed database services without requiring a local database installation on the application server.
- AWS Pricing Calculator helps estimate infrastructure costs before implementing changes.
- Infrastructure optimization should include testing to ensure that applications remain available after changes.

## Conclusion

This lab provided practical experience in AWS resource optimization, EC2 instance management, database service separation, and cloud cost estimation.

It demonstrated how infrastructure decisions can improve resource utilization while maintaining application availability.

## Screenshots

Screenshots will be added to the `screenshots/` folder to document the configuration changes, website verification, and cost comparison.
