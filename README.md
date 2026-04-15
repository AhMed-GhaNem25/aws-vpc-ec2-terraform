# Terraform AWS Infrastructure Setup

A Terraform project that provisions a complete AWS infrastructure environment — including VPC, networking, security group, and an EC2 instance — using reusable variables and dynamic AMI lookup.

---

## Infrastructure Overview

| Resource | Name Pattern | Description |
|---|---|---|
| VPC | `{env}-vpc` | Isolated network for the environment |
| Subnet | `{env}-subnet` | Public subnet within the VPC |
| Internet Gateway | `{env}-igw` | Enables internet access for the VPC |
| Route Table | `{env}-rt` | Routes all outbound traffic through the IGW |
| Security Group | `{env}-sec-grp` | Allows SSH (22) and app traffic (8080) |
| EC2 Instance | `{env}-terraform-ec2` | Amazon Linux 2 instance with public IP |
| Key Pair | `tfkey55` | SSH key pair for EC2 access |

---

## Prerequisites

- [Terraform](https://developer.hashicorp.com/terraform/install) >= 1.0
- AWS CLI configured with valid credentials (`aws configure`)
- An existing SSH key pair at `~/.ssh/id_rsa.pub`
- A `userdata.sh` script in the project root

---

## Project Structure

```
.
├── main.tf             # Main infrastructure configuration
├── terraform.tfvars    # Variable values (not committed to git)
└── userdata.sh         # EC2 startup script
```

---

## Variables

Create a `terraform.tfvars` file in the project root:

```hcl
vpc_cidr_block    = "10.0.0.0/16"
subnet_cidr_block = "10.0.0.0/24"
env               = "dev"
avail_zone        = "us-east-1a"
my-ip             = "YOUR_IP"
instance_type     = "t2.micro"
```

> ⚠️ **Never commit `terraform.tfvars` to Git** — it contains your IP. Add it to `.gitignore`.

---

## Usage

```bash
# Initialize
terraform init

# Preview changes
terraform plan -var-file=terraform.tfvars

# Apply
terraform apply -var-file=terraform.tfvars

# Destroy when done
terraform destroy -var-file=terraform.tfvars
```

---

## Outputs

| Output | Description |
|---|---|
| `aws_ami_id` | The AMI ID used for the EC2 instance |
| `ec2-public-ip` | The public IP of the EC2 instance |

---

## Connect to EC2

```bash
ssh -i ~/.ssh/id_rsa ec2-user@<ec2-public-ip>
```

---

## Notes

- AMI is dynamically fetched — always uses the latest Amazon Linux 2 HVM image for the selected region
- All resources are tagged with the `env` variable
- Inbound SSH is restricted to your IP only; port 8080 is open to the internet
