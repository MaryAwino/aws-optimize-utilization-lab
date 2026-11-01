# AWS Optimize Utilization Lab – Step-by-Step Walkthrough

## Introduction

In this lab, I optimized an AWS Café application environment by removing an unnecessary local database installation and downsizing an EC2 instance.

I also used AWS Pricing Calculator to compare the estimated monthly infrastructure costs before and after optimization.

**AWS Region:** US West (Oregon) – `us-west-2`

---

# PART 1: Optimize the EC2 Instance

## Step 1: Connect to the Café EC2 Instance

I opened the AWS Management Console and navigated to:

**EC2 → Instances → CafeInstance**

I downloaded the instance's PuTTY private key (`.ppk`) and used PuTTY to establish an SSH connection.

This gave me terminal access to the Linux EC2 instance.

## Step 2: Stop the Local MariaDB Service

The Café application uses Amazon RDS for MariaDB. Therefore, the locally installed MariaDB server on the EC2 instance was unnecessary for this lab.

I stopped the local database service:

```bash
sudo systemctl stop mariadb
```

**Explanation:**
- `sudo` runs the command with administrative privileges.
- `systemctl` manages Linux system services.
- `stop` stops the service.
- `mariadb` identifies the database service.

## Step 3: Uninstall MariaDB Server

I removed the local MariaDB server package:

```bash
sudo yum -y remove mariadb-server
```

**Explanation:**
- `yum` is the package manager used by this Linux environment.
- `remove` uninstalls the specified package.
- `-y` automatically confirms the removal.

The removal completed successfully.

## Step 4: Connect to the CLI Host

I closed the Café instance terminal and opened another PuTTY session to connect to the CLI Host.

The CLI Host allowed me to manage AWS resources using the AWS Command Line Interface (AWS CLI).

## Step 5: Identify the AWS Region

I ran:

```bash
curl http://169.254.169.254/latest/dynamic/instance-identity/document | grep region
```

**Explanation:**
- `curl` retrieves information from a URL.
- The URL accesses EC2 instance metadata.
- `grep region` filters the output to display the AWS Region.

The output showed:

```text
"region" : "us-west-2"
```

## Step 6: Configure AWS CLI

I configured the AWS CLI:

```bash
aws configure
```

I supplied the temporary lab credentials and configured:

| Setting | Value |
|---|---|
| Default region | `us-west-2` |
| Output format | `json` |

The access key and secret key were entered directly into the terminal and were not included in this documentation.

**Important:** Never publish AWS access keys, secret keys, passwords, or private keys in a public repository.

## Step 7: Find the Café EC2 Instance ID

I used the following command:

```bash
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=CafeInstance" \
  --query "Reservations[*].Instances[*].InstanceId"
```

**Explanation:**
- `describe-instances` retrieves EC2 instance information.
- `--filters` searches using the instance's Name tag.
- `--query` extracts only the instance ID.

The command returned the Café instance ID.

## Step 8: Stop the EC2 Instance

Before changing the instance type, I stopped the EC2 instance:

```bash
aws ec2 stop-instances \
  --instance-ids <instance-id>
```

I replaced `<instance-id>` with the actual ID returned in the previous step.

I then checked its state:

```bash
aws ec2 describe-instances \
  --instance-ids <instance-id> \
  --query "Reservations[*].Instances[*].State.Name"
```

The instance state was confirmed as:

```text
stopped
```

## Step 9: Modify the EC2 Instance Type

The original instance type was `t3.small`.

I changed it to `t3.micro`:

```bash
aws ec2 modify-instance-attribute \
  --instance-id <instance-id> \
  --instance-type "{\"Value\": \"t3.micro\"}"
```

**Explanation:**
- `modify-instance-attribute` changes an EC2 instance attribute.
- `--instance-id` identifies the instance.
- `--instance-type` specifies the new instance type.

The command completed without an error.

## Step 10: Start the EC2 Instance

After modifying the instance type, I started it again:

```bash
aws ec2 start-instances \
  --instance-ids <instance-id>
```

I checked its configuration and status:

```bash
aws ec2 describe-instances \
  --instance-ids <instance-id> \
  --query "Reservations[*].Instances[*].[InstanceType,PublicDnsName,PublicIpAddress,State.Name]"
```

The output confirmed:

- Instance type: `t3.micro`
- State: `running`

## Step 11: Test the Café Website

I copied the public DNS name from the command output and opened the website in a browser:

```text
http://<public-dns>/cafe
```

The Café website loaded successfully.

This confirmed that the application remained accessible after the EC2 optimization.

---

# PART 2: AWS Pricing Calculator

## Step 12: Open AWS Pricing Calculator

I visited:

https://calculator.aws

I selected **Create estimate** and added Amazon EC2.

## Step 13: Configure the Original EC2 Estimate

I configured the original EC2 estimate with the following values:

| Setting | Value |
|---|---|
| Region | US West (Oregon) |
| Tenancy | Shared Instances |
| Operating system | Linux |
| Workload | Constant |
| Number of instances | 1 |
| Instance type | `t3.small` |
| Pricing strategy | On-Demand |
| Utilization | 100% |
| EBS storage | 40 GB |
| Monitoring | Disabled |

I saved and added the EC2 service to my estimate.

## Step 14: Add Amazon RDS for MariaDB

I added Amazon RDS for MariaDB to the estimate.

I configured it as follows:

| Setting | Value |
|---|---|
| Region | US West (Oregon) |
| Database engine | MariaDB |
| Instance type | `db.t3.micro` |
| Number of nodes | 1 |
| Deployment | Single-AZ |
| Pricing | On-Demand |
| Utilization | 100% |
| Storage | 20 GB |
| Storage type | General Purpose SSD (gp2) |

I saved and added the RDS service.

## Step 15: Review the Original Estimate

The original monthly estimate was:

| Service | Monthly cost |
|---|---:|
| Amazon EC2 | $19.18 |
| Amazon RDS for MariaDB | $54.86 |
| **Total** | **$74.04** |

I removed duplicate EC2 entries and an unrelated `t4g.nano` Savings Plan entry to ensure the comparison contained only the intended resources.

## Step 16: Optimize the EC2 Estimate

I edited the EC2 configuration and changed:

| Setting | Original | Optimized |
|---|---|---|
| EC2 instance | `t3.small` | `t3.micro` |
| EBS storage | 40 GB | 20 GB |

I kept the other EC2 settings unchanged.

I also kept the RDS configuration unchanged to make the comparison fair.

## Step 17: Review the Optimized Estimate

The updated monthly estimate was:

| Item | Monthly cost |
|---|---:|
| Original estimate | $74.04 |
| Optimized estimate | $64.45 |
| **Monthly savings** | **$9.59** |

Annual estimated savings:

```text
$9.59 × 12 = $115.08
```

The estimated monthly cost decreased by approximately 12.95%.

## Step 18: Save and Export the Estimate

I saved the updated estimate and exported it as a PDF for documentation.

---

# Final Results

| Resource | Before | After |
|---|---|---|
| EC2 instance | `t3.small` | `t3.micro` |
| EBS storage | 40 GB | 20 GB |
| RDS instance | `db.t3.micro` | `db.t3.micro` |
| RDS storage | 20 GB | 20 GB |
| Café website | Accessible | Accessible |

## Key Lessons Learned

1. **EC2 Right-Sizing:** Selecting an instance type that matches application requirements helps avoid unnecessary compute costs.
2. **Resource Utilization:** Unnecessary services running on an instance can increase resource consumption.
3. **Managed Databases:** Amazon RDS provides a managed database service, reducing the need to maintain a separate local database installation.
4. **AWS CLI:** I practised retrieving instance information, stopping and starting instances, and modifying EC2 attributes.
5. **Cost Optimization:** AWS Pricing Calculator helps estimate infrastructure costs and compare alternative configurations.
6. **Application Verification:** Testing the website after infrastructure changes is important to confirm application availability.

## Conclusion

This lab gave me practical experience in AWS infrastructure optimization, EC2 instance management, Linux service administration, AWS CLI operations, and cloud cost estimation.

It demonstrated how right-sizing resources and removing unnecessary services can help reduce infrastructure costs while maintaining application availability.
