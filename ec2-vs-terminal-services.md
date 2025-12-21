# EC2 vs Terminal Services: Comprehensive Comparison
# EC2 vs ターミナルサービス：包括的比較

## Executive Summary / エグゼクティブサマリー

This document compares **individual EC2 instances (1 per user)** versus **Windows Terminal Services (RDS)** for a 14-member development team.

このドキュメントでは、14名の開発チーム向けに**個別EC2インスタンス（1ユーザー1台）**と**Windowsターミナルサービス（RDS）**を比較します。

### Quick Decision Matrix / 判断マトリックス

| Factor / 要素 | EC2 (1 per user) | Terminal Services |
|--------------|------------------|-------------------|
| **Initial Cost / 初期コスト** | Lower / 低い | Higher / 高い |
| **Monthly Cost / 月額コスト** | ~$1,838/mo | ~$2,800-3,500/mo |
| **Licensing / ライセンス** | None for Linux / Linuxは不要 | Required CALs + Server / CAL + サーバー必要 |
| **Isolation / 分離性** | Complete / 完全 | Shared environment / 共有環境 |
| **Maintainability / 保守性** | Per-instance / インスタンス毎 | Centralized / 集中管理 |
| **Flexibility / 柔軟性** | High / 高い | Limited / 限定的 |
| **Best For / 最適な用途** | Development teams / 開発チーム | Office workers / オフィスワーカー |

---

## Why EC2 (1 per user) is Better for Development Teams
## なぜ開発チームにはEC2（1ユーザー1台）が優れているか

### 1. No Windows Licensing Costs for Linux
### 1. Linuxは Windowsライセンスコスト不要

For development work (Next.js, Java, Docker), Linux is ideal:
開発作業（Next.js、Java、Docker）にはLinuxが最適：

| Item | EC2 Linux | Terminal Services |
|------|-----------|-------------------|
| OS License | **$0** (included in EC2) | Required for Windows Server |
| CAL License | **$0** | Required per user |
| RDS CAL | **$0** | Required per user |

### 2. Complete Isolation
### 2. 完全な分離

```
EC2 Model:                          Terminal Services Model:
┌─────────────────────────────┐     ┌─────────────────────────────┐
│  User 1    User 2    User 3 │     │        Single Server         │
│  ┌─────┐   ┌─────┐   ┌─────┐│     │  ┌─────────────────────────┐ │
│  │ EC2 │   │ EC2 │   │ EC2 ││     │  │   Shared Resources      │ │
│  │ #1  │   │ #2  │   │ #3  ││     │  │  User1 User2 User3 ...  │ │
│  └─────┘   └─────┘   └─────┘│     │  │  (Competing for CPU,    │ │
│                              │     │  │   RAM, Disk I/O)        │ │
│  ✓ Independent resources    │     │  └─────────────────────────┘ │
│  ✓ No resource contention   │     │  ✗ Resource contention      │
│  ✓ Crash isolation          │     │  ✗ One crash affects all    │
└─────────────────────────────┘     └─────────────────────────────┘
```

### 3. Developer Flexibility
### 3. 開発者の柔軟性

| Capability | EC2 (individual) | Terminal Services |
|------------|------------------|-------------------|
| Install any software | ✓ Admin access | ✗ Restricted |
| Run Docker containers | ✓ Full support | ⚠ Complex setup |
| Reboot anytime | ✓ Independent | ✗ Affects others |
| Custom environment | ✓ Per-user | ✗ Shared config |
| Root/Admin access | ✓ Full control | ✗ Limited |

### 4. Better for Modern Development
### 4. モダン開発に最適

Development workloads benefit from isolation:
開発ワークロードは分離から恩恵を受ける：

- **Docker builds**: Require significant resources, shouldn't compete / 大量リソースを必要とし、競合すべきでない
- **npm/yarn install**: Heavy I/O operations / 重いI/O操作
- **Compilation**: CPU-intensive, shouldn't slow others / CPU集約型、他者を遅くすべきでない
- **IDE/Editor**: Memory-hungry applications / メモリを大量消費するアプリケーション

---

## Terminal Services Licensing Costs (Detailed)
## ターミナルサービスライセンスコスト（詳細）

### Required Licenses for Terminal Services
### ターミナルサービスに必要なライセンス

To run a Windows Terminal Services environment, you need:
Windowsターミナルサービス環境を運用するには以下が必要：

#### 1. Windows Server License
#### 1. Windows Serverライセンス

| Edition | Licensing Model | Approximate Cost |
|---------|-----------------|------------------|
| Windows Server 2022 Standard | Per 16 cores | ~$1,069 |
| Windows Server 2022 Datacenter | Per 16 cores | ~$6,155 |

**Note**: AWS EC2 Windows includes the OS license in the hourly rate, but at a premium (~$0.08/hr additional).
**注意**: AWS EC2 WindowsはOSライセンスが時間料金に含まれますが、追加料金（~$0.08/hr）がかかります。

#### 2. Remote Desktop Services (RDS) CALs
#### 2. リモートデスクトップサービス（RDS）CAL

**Per-User CAL (Recommended for development teams):**
**ユーザー単位CAL（開発チーム向け推奨）：**

| CAL Type | Cost per User | For 14 Users |
|----------|---------------|--------------|
| RDS User CAL | ~$154 | **$2,156** |

**Per-Device CAL (Alternative):**
**デバイス単位CAL（代替）：**

| CAL Type | Cost per Device | For 14 Devices |
|----------|-----------------|----------------|
| RDS Device CAL | ~$154 | **$2,156** |

#### 3. Windows Server CALs (Also Required)
#### 3. Windows Server CAL（これも必要）

| CAL Type | Cost per User | For 14 Users |
|----------|---------------|--------------|
| Windows Server User CAL | ~$48 | **$672** |

### Total Terminal Services Licensing Cost (One-time)
### ターミナルサービスライセンス総コスト（一時費用）

| Component | Cost |
|-----------|-----:|
| Windows Server 2022 Standard (if self-managed) | $1,069 |
| RDS CALs (14 users) | $2,156 |
| Windows Server CALs (14 users) | $672 |
| **Total One-Time License Cost** | **$3,897** |
| **日本円換算** | **約¥584,550** |

---

## Comprehensive Cost Comparison
## 包括的コスト比較

### Scenario: 14-Member Development Team, 2-Month Project
### シナリオ: 14名開発チーム、2ヶ月プロジェクト

#### Option 1: EC2 Individual Instances (Linux-based)
#### オプション1: EC2個別インスタンス（Linuxベース）

| Item | Monthly | 2-Month |
|------|--------:|--------:|
| EC2 Linux (11 × t3.large) | $1,053.14 | $2,106.28 |
| EC2 Windows (3 × t3.xlarge for .NET) | $785.67 | $1,571.34 |
| EBS Storage (50GB × 14) | $70.00 | $140.00 |
| Data Transfer | $14.00 | $28.00 |
| **Total** | **$1,922.81** | **$3,845.62** |
| **日本円換算** | **¥288,421** | **¥576,843** |

#### Option 2: Windows Terminal Services on EC2
#### オプション2: EC2上のWindowsターミナルサービス

| Item | Monthly | 2-Month |
|------|--------:|--------:|
| EC2 Windows (1 × m5.4xlarge, 16 vCPU, 64GB) | $1,344.00 | $2,688.00 |
| RDS CALs (14 users, amortized) | $179.67* | $359.33 |
| Windows Server CALs (amortized) | $56.00* | $112.00 |
| EBS Storage (500GB for shared) | $50.00 | $100.00 |
| Backup/Redundancy instance | $672.00 | $1,344.00 |
| **Total** | **$2,301.67** | **$4,603.33** |
| **日本円換算** | **¥345,250** | **¥690,500** |

*CALs amortized over 12 months / CALは12ヶ月で償却

#### Option 3: AWS WorkSpaces (Alternative)
#### オプション3: AWS WorkSpaces（代替案）

| Item | Monthly | 2-Month |
|------|--------:|--------:|
| WorkSpaces Standard (14 × $35/mo) | $490.00 | $980.00 |
| WorkSpaces Performance (14 × $60/mo) | $840.00 | $1,680.00 |
| **Note**: Limited for development / 開発には制限あり | - | - |

### Cost Comparison Summary
### コスト比較サマリー

```
┌────────────────────────────────────────────────────────────────────┐
│              2-Month Project Cost Comparison                        │
│                2ヶ月プロジェクトコスト比較                            │
├────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  EC2 Individual (Linux)     ████████████████░░░░░░  $3,845         │
│                                                                     │
│  Terminal Services          ████████████████████████  $4,603       │
│                                                                     │
│  AWS WorkSpaces (Perf)      ████████░░░░░░░░░░░░░░░  $1,680*       │
│                             *Limited for dev work                   │
│                                                                     │
│  Savings with EC2 Individual: $758 (16% less)                      │
│  EC2個別での節約: $758 (16%削減)                                     │
│                                                                     │
└────────────────────────────────────────────────────────────────────┘
```

---

## Maintainability Comparison
## 保守性比較

### EC2 Individual Instances
### EC2個別インスタンス

| Aspect | Pros | Cons |
|--------|------|------|
| Updates | Per-instance control | Multiple instances to update |
| Monitoring | Per-instance CloudWatch | More dashboards |
| Backup | Individual snapshots | More backups to manage |
| Security | Isolated attack surface | More security groups |
| Scaling | Add/remove instances | Requires automation |

**Automation Solutions / 自動化ソリューション:**
- AWS Systems Manager for patch management / パッチ管理
- Launch Templates for consistency / 一貫性のため
- Auto Scaling Groups for scaling / スケーリングのため
- Ansible/Terraform for IaC / Infrastructure as Code

### Terminal Services
### ターミナルサービス

| Aspect | Pros | Cons |
|--------|------|------|
| Updates | Single point of update | Requires maintenance window |
| Monitoring | Centralized | Complex multi-user monitoring |
| Backup | Single backup | Single point of failure |
| Security | Central management | Shared vulnerabilities |
| Scaling | Vertical only | Expensive scaling |

---

## AWS Deployment Scripts
## AWSデプロイメントスクリプト

### Terraform Configuration for 14-User EC2 Deployment
### 14ユーザーEC2デプロイメント用Terraform設定

Create `infrastructure/main.tf`:

```hcl
# main.tf - EC2 Infrastructure for 14-member team

terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "ap-northeast-1"  # Tokyo
}

# Variables
variable "environment" {
  default = "development"
}

variable "team_members" {
  type = map(object({
    name          = string
    role          = string
    instance_type = string
    os            = string
  }))
  default = {
    "frontend-1" = { name = "frontend-1", role = "frontend", instance_type = "t3.large", os = "linux" }
    "frontend-2" = { name = "frontend-2", role = "frontend", instance_type = "t3.large", os = "linux" }
    "frontend-3" = { name = "frontend-3", role = "frontend", instance_type = "t3.large", os = "linux" }
    "frontend-4" = { name = "frontend-4", role = "frontend", instance_type = "t3.large", os = "linux" }
    "frontend-5" = { name = "frontend-5", role = "frontend", instance_type = "t3.large", os = "linux" }
    "java-1"     = { name = "java-1", role = "backend", instance_type = "t3.large", os = "linux" }
    "java-2"     = { name = "java-2", role = "backend", instance_type = "t3.large", os = "linux" }
    "java-3"     = { name = "java-3", role = "backend", instance_type = "t3.large", os = "linux" }
    "java-4"     = { name = "java-4", role = "backend", instance_type = "t3.large", os = "linux" }
    "dotnet-1"   = { name = "dotnet-1", role = "dotnet", instance_type = "t3.xlarge", os = "windows" }
    "dotnet-2"   = { name = "dotnet-2", role = "dotnet", instance_type = "t3.xlarge", os = "windows" }
    "dotnet-3"   = { name = "dotnet-3", role = "dotnet", instance_type = "t3.xlarge", os = "windows" }
    "devops-1"   = { name = "devops-1", role = "devops", instance_type = "t3.large", os = "linux" }
    "devops-2"   = { name = "devops-2", role = "devops", instance_type = "t3.large", os = "linux" }
  }
}

# VPC
resource "aws_vpc" "dev_vpc" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name        = "dev-team-vpc"
    Environment = var.environment
  }
}

# Internet Gateway
resource "aws_internet_gateway" "dev_igw" {
  vpc_id = aws_vpc.dev_vpc.id

  tags = {
    Name = "dev-team-igw"
  }
}

# Public Subnet
resource "aws_subnet" "dev_public" {
  vpc_id                  = aws_vpc.dev_vpc.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "ap-northeast-1a"
  map_public_ip_on_launch = true

  tags = {
    Name = "dev-team-public-subnet"
  }
}

# Route Table
resource "aws_route_table" "dev_public_rt" {
  vpc_id = aws_vpc.dev_vpc.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.dev_igw.id
  }

  tags = {
    Name = "dev-team-public-rt"
  }
}

resource "aws_route_table_association" "dev_public_rta" {
  subnet_id      = aws_subnet.dev_public.id
  route_table_id = aws_route_table.dev_public_rt.id
}

# Security Group for Linux instances
resource "aws_security_group" "linux_sg" {
  name        = "dev-linux-sg"
  description = "Security group for Linux development instances"
  vpc_id      = aws_vpc.dev_vpc.id

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]  # Restrict to your IP in production
    description = "SSH access"
  }

  ingress {
    from_port   = 3000
    to_port     = 3100
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
    description = "Development ports"
  }

  ingress {
    from_port   = 8080
    to_port     = 8090
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
    description = "Application ports"
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "dev-linux-sg"
  }
}

# Security Group for Windows instances
resource "aws_security_group" "windows_sg" {
  name        = "dev-windows-sg"
  description = "Security group for Windows development instances"
  vpc_id      = aws_vpc.dev_vpc.id

  ingress {
    from_port   = 3389
    to_port     = 3389
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]  # Restrict to your IP in production
    description = "RDP access"
  }

  ingress {
    from_port   = 5985
    to_port     = 5986
    protocol    = "tcp"
    cidr_blocks = ["10.0.0.0/16"]
    description = "WinRM"
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "dev-windows-sg"
  }
}

# Data source for Ubuntu AMI
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]  # Canonical

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}

# Data source for Windows Server AMI
data "aws_ami" "windows" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["Windows_Server-2022-English-Full-Base-*"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}

# Key Pair (you should create this separately)
resource "aws_key_pair" "dev_key" {
  key_name   = "dev-team-key"
  public_key = file("~/.ssh/id_rsa.pub")  # Update this path
}

# EC2 Instances
resource "aws_instance" "dev_instances" {
  for_each = var.team_members

  ami           = each.value.os == "linux" ? data.aws_ami.ubuntu.id : data.aws_ami.windows.id
  instance_type = each.value.instance_type
  subnet_id     = aws_subnet.dev_public.id
  key_name      = aws_key_pair.dev_key.key_name

  vpc_security_group_ids = each.value.os == "linux" ? [aws_security_group.linux_sg.id] : [aws_security_group.windows_sg.id]

  root_block_device {
    volume_size = 50
    volume_type = "gp3"
    encrypted   = true
  }

  user_data = each.value.os == "linux" ? local.linux_user_data : local.windows_user_data

  tags = {
    Name        = "dev-${each.value.name}"
    Role        = each.value.role
    Environment = var.environment
    ManagedBy   = "terraform"
  }
}

# User data scripts
locals {
  linux_user_data = <<-EOF
    #!/bin/bash
    set -e

    # Update system
    apt-get update && apt-get upgrade -y

    # Install Docker
    apt-get install -y apt-transport-https ca-certificates curl gnupg lsb-release
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
    echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
    apt-get update
    apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

    # Install Node.js
    curl -fsSL https://deb.nodesource.com/setup_20.x | bash -
    apt-get install -y nodejs

    # Install Java
    apt-get install -y openjdk-17-jdk maven

    # Install Git
    apt-get install -y git

    # Add ubuntu user to docker group
    usermod -aG docker ubuntu

    # Install AWS CLI
    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
    apt-get install -y unzip
    unzip awscliv2.zip
    ./aws/install

    # Cleanup
    rm -rf awscliv2.zip aws

    echo "Setup complete!"
  EOF

  windows_user_data = <<-EOF
    <powershell>
    # Install Chocolatey
    Set-ExecutionPolicy Bypass -Scope Process -Force
    [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072
    iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))

    # Install development tools
    choco install -y git nodejs visualstudio2022community dotnet-sdk awscli

    # Install Docker (Windows Containers)
    Install-WindowsFeature -Name Containers
    Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
    Install-Package -Name docker -ProviderName DockerMsftProvider -Force

    # Enable WinRM for remote management
    Enable-PSRemoting -Force

    Write-Host "Setup complete! Please restart for Docker to work."
    </powershell>
  EOF
}

# Elastic IPs for each instance (optional, but useful for static access)
resource "aws_eip" "dev_eips" {
  for_each = var.team_members
  instance = aws_instance.dev_instances[each.key].id
  domain   = "vpc"

  tags = {
    Name = "eip-${each.value.name}"
  }
}

# Outputs
output "instance_details" {
  value = {
    for key, instance in aws_instance.dev_instances : key => {
      id         = instance.id
      public_ip  = aws_eip.dev_eips[key].public_ip
      private_ip = instance.private_ip
      type       = instance.instance_type
    }
  }
}

output "ssh_commands" {
  value = {
    for key, instance in aws_instance.dev_instances : key =>
    var.team_members[key].os == "linux" ?
    "ssh -i ~/.ssh/id_rsa ubuntu@${aws_eip.dev_eips[key].public_ip}" :
    "mstsc /v:${aws_eip.dev_eips[key].public_ip}"
  }
}
```

### AWS CLI Deployment Script (Alternative)
### AWS CLIデプロイメントスクリプト（代替）

Create `scripts/deploy-ec2-team.sh`:

```bash
#!/bin/bash
# deploy-ec2-team.sh - Deploy EC2 instances for development team
# EC2インスタンスを開発チーム向けにデプロイ

set -e

# Configuration / 設定
REGION="ap-northeast-1"
VPC_CIDR="10.0.0.0/16"
SUBNET_CIDR="10.0.1.0/24"
KEY_NAME="dev-team-key"
ENVIRONMENT="development"

# Colors for output / 出力用カラー
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

log_info() {
    echo -e "${GREEN}[INFO]${NC} $1"
}

log_warn() {
    echo -e "${YELLOW}[WARN]${NC} $1"
}

log_error() {
    echo -e "${RED}[ERROR]${NC} $1"
}

# Check AWS CLI is configured / AWS CLIが設定されているか確認
check_aws_cli() {
    log_info "Checking AWS CLI configuration..."
    if ! aws sts get-caller-identity &>/dev/null; then
        log_error "AWS CLI is not configured. Please run 'aws configure' first."
        exit 1
    fi
    log_info "AWS CLI configured successfully"
}

# Create VPC / VPCを作成
create_vpc() {
    log_info "Creating VPC..."
    VPC_ID=$(aws ec2 create-vpc \
        --cidr-block $VPC_CIDR \
        --region $REGION \
        --tag-specifications "ResourceType=vpc,Tags=[{Key=Name,Value=dev-team-vpc},{Key=Environment,Value=$ENVIRONMENT}]" \
        --query 'Vpc.VpcId' \
        --output text)

    aws ec2 modify-vpc-attribute --vpc-id $VPC_ID --enable-dns-hostnames '{"Value": true}' --region $REGION
    aws ec2 modify-vpc-attribute --vpc-id $VPC_ID --enable-dns-support '{"Value": true}' --region $REGION

    log_info "VPC created: $VPC_ID"
    echo $VPC_ID
}

# Create Internet Gateway / インターネットゲートウェイを作成
create_internet_gateway() {
    local vpc_id=$1
    log_info "Creating Internet Gateway..."

    IGW_ID=$(aws ec2 create-internet-gateway \
        --region $REGION \
        --tag-specifications "ResourceType=internet-gateway,Tags=[{Key=Name,Value=dev-team-igw}]" \
        --query 'InternetGateway.InternetGatewayId' \
        --output text)

    aws ec2 attach-internet-gateway --internet-gateway-id $IGW_ID --vpc-id $vpc_id --region $REGION

    log_info "Internet Gateway created: $IGW_ID"
    echo $IGW_ID
}

# Create Subnet / サブネットを作成
create_subnet() {
    local vpc_id=$1
    log_info "Creating Subnet..."

    SUBNET_ID=$(aws ec2 create-subnet \
        --vpc-id $vpc_id \
        --cidr-block $SUBNET_CIDR \
        --availability-zone "${REGION}a" \
        --region $REGION \
        --tag-specifications "ResourceType=subnet,Tags=[{Key=Name,Value=dev-team-public-subnet}]" \
        --query 'Subnet.SubnetId' \
        --output text)

    aws ec2 modify-subnet-attribute --subnet-id $SUBNET_ID --map-public-ip-on-launch --region $REGION

    log_info "Subnet created: $SUBNET_ID"
    echo $SUBNET_ID
}

# Create Route Table / ルートテーブルを作成
create_route_table() {
    local vpc_id=$1
    local igw_id=$2
    local subnet_id=$3

    log_info "Creating Route Table..."

    RT_ID=$(aws ec2 create-route-table \
        --vpc-id $vpc_id \
        --region $REGION \
        --tag-specifications "ResourceType=route-table,Tags=[{Key=Name,Value=dev-team-public-rt}]" \
        --query 'RouteTable.RouteTableId' \
        --output text)

    aws ec2 create-route --route-table-id $RT_ID --destination-cidr-block 0.0.0.0/0 --gateway-id $igw_id --region $REGION
    aws ec2 associate-route-table --route-table-id $RT_ID --subnet-id $subnet_id --region $REGION

    log_info "Route Table created: $RT_ID"
}

# Create Security Groups / セキュリティグループを作成
create_security_groups() {
    local vpc_id=$1
    log_info "Creating Security Groups..."

    # Linux Security Group
    LINUX_SG=$(aws ec2 create-security-group \
        --group-name dev-linux-sg \
        --description "Security group for Linux development instances" \
        --vpc-id $vpc_id \
        --region $REGION \
        --query 'GroupId' \
        --output text)

    aws ec2 authorize-security-group-ingress --group-id $LINUX_SG --protocol tcp --port 22 --cidr 0.0.0.0/0 --region $REGION
    aws ec2 authorize-security-group-ingress --group-id $LINUX_SG --protocol tcp --port 3000-3100 --cidr 0.0.0.0/0 --region $REGION
    aws ec2 authorize-security-group-ingress --group-id $LINUX_SG --protocol tcp --port 8080-8090 --cidr 0.0.0.0/0 --region $REGION

    log_info "Linux Security Group created: $LINUX_SG"

    # Windows Security Group
    WINDOWS_SG=$(aws ec2 create-security-group \
        --group-name dev-windows-sg \
        --description "Security group for Windows development instances" \
        --vpc-id $vpc_id \
        --region $REGION \
        --query 'GroupId' \
        --output text)

    aws ec2 authorize-security-group-ingress --group-id $WINDOWS_SG --protocol tcp --port 3389 --cidr 0.0.0.0/0 --region $REGION
    aws ec2 authorize-security-group-ingress --group-id $WINDOWS_SG --protocol tcp --port 5985-5986 --cidr 10.0.0.0/16 --region $REGION

    log_info "Windows Security Group created: $WINDOWS_SG"

    echo "$LINUX_SG $WINDOWS_SG"
}

# Get latest AMIs / 最新AMIを取得
get_amis() {
    log_info "Getting latest AMIs..."

    UBUNTU_AMI=$(aws ec2 describe-images \
        --owners 099720109477 \
        --filters "Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*" \
        --query 'sort_by(Images, &CreationDate)[-1].ImageId' \
        --region $REGION \
        --output text)

    WINDOWS_AMI=$(aws ec2 describe-images \
        --owners amazon \
        --filters "Name=name,Values=Windows_Server-2022-English-Full-Base-*" \
        --query 'sort_by(Images, &CreationDate)[-1].ImageId' \
        --region $REGION \
        --output text)

    log_info "Ubuntu AMI: $UBUNTU_AMI"
    log_info "Windows AMI: $WINDOWS_AMI"

    echo "$UBUNTU_AMI $WINDOWS_AMI"
}

# Launch instance / インスタンスを起動
launch_instance() {
    local name=$1
    local role=$2
    local instance_type=$3
    local os=$4
    local subnet_id=$5
    local sg_id=$6
    local ami_id=$7

    log_info "Launching instance: $name ($instance_type, $os)"

    INSTANCE_ID=$(aws ec2 run-instances \
        --image-id $ami_id \
        --instance-type $instance_type \
        --key-name $KEY_NAME \
        --subnet-id $subnet_id \
        --security-group-ids $sg_id \
        --block-device-mappings "DeviceName=/dev/sda1,Ebs={VolumeSize=50,VolumeType=gp3,Encrypted=true}" \
        --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=dev-$name},{Key=Role,Value=$role},{Key=Environment,Value=$ENVIRONMENT}]" \
        --region $REGION \
        --query 'Instances[0].InstanceId' \
        --output text)

    log_info "Instance launched: $INSTANCE_ID"
    echo $INSTANCE_ID
}

# Main deployment / メインデプロイメント
main() {
    log_info "Starting deployment for 14-member development team..."
    log_info "開発チーム14名向けのデプロイメントを開始します..."

    check_aws_cli

    # Create infrastructure / インフラを作成
    VPC_ID=$(create_vpc)
    IGW_ID=$(create_internet_gateway $VPC_ID)
    SUBNET_ID=$(create_subnet $VPC_ID)
    create_route_table $VPC_ID $IGW_ID $SUBNET_ID

    SG_IDS=$(create_security_groups $VPC_ID)
    LINUX_SG=$(echo $SG_IDS | cut -d' ' -f1)
    WINDOWS_SG=$(echo $SG_IDS | cut -d' ' -f2)

    AMI_IDS=$(get_amis)
    UBUNTU_AMI=$(echo $AMI_IDS | cut -d' ' -f1)
    WINDOWS_AMI=$(echo $AMI_IDS | cut -d' ' -f2)

    # Define team members / チームメンバーを定義
    # Format: name:role:instance_type:os
    TEAM_MEMBERS=(
        "frontend-1:frontend:t3.large:linux"
        "frontend-2:frontend:t3.large:linux"
        "frontend-3:frontend:t3.large:linux"
        "frontend-4:frontend:t3.large:linux"
        "frontend-5:frontend:t3.large:linux"
        "java-1:backend:t3.large:linux"
        "java-2:backend:t3.large:linux"
        "java-3:backend:t3.large:linux"
        "java-4:backend:t3.large:linux"
        "dotnet-1:dotnet:t3.xlarge:windows"
        "dotnet-2:dotnet:t3.xlarge:windows"
        "dotnet-3:dotnet:t3.xlarge:windows"
        "devops-1:devops:t3.large:linux"
        "devops-2:devops:t3.large:linux"
    )

    # Launch instances / インスタンスを起動
    INSTANCE_IDS=()
    for member in "${TEAM_MEMBERS[@]}"; do
        IFS=':' read -r name role instance_type os <<< "$member"

        if [ "$os" == "linux" ]; then
            sg_id=$LINUX_SG
            ami_id=$UBUNTU_AMI
        else
            sg_id=$WINDOWS_SG
            ami_id=$WINDOWS_AMI
        fi

        instance_id=$(launch_instance $name $role $instance_type $os $SUBNET_ID $sg_id $ami_id)
        INSTANCE_IDS+=($instance_id)
    done

    log_info "Waiting for instances to be running..."
    aws ec2 wait instance-running --instance-ids ${INSTANCE_IDS[@]} --region $REGION

    log_info "Deployment complete! / デプロイメント完了！"
    log_info "=========================================="

    # Show instance details / インスタンス詳細を表示
    for instance_id in "${INSTANCE_IDS[@]}"; do
        DETAILS=$(aws ec2 describe-instances \
            --instance-ids $instance_id \
            --region $REGION \
            --query 'Reservations[0].Instances[0].[Tags[?Key==`Name`].Value|[0],PublicIpAddress,InstanceType]' \
            --output text)
        log_info "$DETAILS"
    done
}

# Run main function
main "$@"
```

---

## Alternative Solutions
## 代替ソリューション

### 1. Amazon WorkSpaces
### 1. Amazon WorkSpaces

| Feature | Details |
|---------|---------|
| **Best for** | Standard office work, not heavy development |
| **Pricing** | $35-80/user/month |
| **Pros** | Managed service, auto-patching |
| **Cons** | Limited customization, not ideal for Docker |

### 2. Amazon AppStream 2.0
### 2. Amazon AppStream 2.0

| Feature | Details |
|---------|---------|
| **Best for** | Streaming specific applications |
| **Pricing** | Pay per session hour |
| **Pros** | No need to manage full VMs |
| **Cons** | Application streaming only |

### 3. AWS Nimble Studio
### 3. AWS Nimble Studio

| Feature | Details |
|---------|---------|
| **Best for** | Creative/media workloads |
| **Pricing** | Variable |
| **Pros** | High-performance graphics |
| **Cons** | Specific use case |

### 4. GitHub Codespaces / GitPod
### 4. GitHub Codespaces / GitPod

| Feature | Details |
|---------|---------|
| **Best for** | Cloud-native development |
| **Pricing** | $18-36/user/month |
| **Pros** | Pre-configured environments, instant start |
| **Cons** | Dependent on browser, limited for heavy workloads |

### 5. Hybrid Approach
### 5. ハイブリッドアプローチ

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Hybrid Development Environment                    │
│                     ハイブリッド開発環境                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────────────────┐    ┌──────────────────────┐               │
│  │  GitHub Codespaces   │    │   EC2 Instances      │               │
│  │                      │    │                      │               │
│  │  • Quick code edits  │    │  • Heavy builds      │               │
│  │  • Code reviews      │    │  • Docker work       │               │
│  │  • Simple tasks      │    │  • Full development  │               │
│  │                      │    │                      │               │
│  │  Cost: ~$18/user/mo  │    │  Cost: ~$96/user/mo  │               │
│  └──────────────────────┘    └──────────────────────┘               │
│                                                                      │
│  Use Case Strategy / ユースケース戦略:                               │
│  • Use Codespaces for 80% of quick tasks                            │
│  • Use EC2 for 20% of heavy workloads                               │
│  • Potential savings: 40-50%                                        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Recommendation Summary
## 推奨サマリー

### For Your 14-Member Development Team:
### 14名開発チーム向け：

1. **Primary Choice: EC2 Individual Instances (Linux)**
   - Lower cost (~$1,839/month for Option A)
   - No licensing complexity
   - Full isolation and flexibility
   - Better for Docker/Node.js/Java development

2. **For .NET Developers: EC2 Windows (3 instances)**
   - Windows included in EC2 pricing
   - No separate CAL purchases needed
   - Full Visual Studio support

3. **Avoid Terminal Services for Development:**
   - Higher licensing costs
   - Shared resource contention
   - Limited developer flexibility
   - Complex maintenance for development workloads

### Cost Savings Achieved:
### 実現されるコスト削減:

| Approach | 2-Month Cost | Savings |
|----------|--------------|---------|
| Terminal Services | ~$4,603 | Baseline |
| EC2 Individual | ~$3,845 | **$758 (16%)** |
| Hybrid (Codespaces + EC2) | ~$2,800 | **$1,803 (39%)** |

---

## References / 参考リンク

- [AWS EC2 Pricing](https://aws.amazon.com/ec2/pricing/)
- [Microsoft RDS CAL Pricing](https://www.microsoft.com/en-us/licensing/product-licensing/client-access-license)
- [AWS WorkSpaces Pricing](https://aws.amazon.com/workspaces/pricing/)
- [GitHub Codespaces Pricing](https://github.com/features/codespaces)

---

*Last updated / 最終更新: 2025-12-21*
