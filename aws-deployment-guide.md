# AWS Deployment Guide for Application Modernization Team / AWSデプロイメントガイド（アプリケーションモダナイゼーションチーム向け）

## English

### Overview

This guide provides AWS EC2 configuration recommendations for a **14-member development team** working on application modernization projects:
- **.NET to Next.js/React** migration
- **Java environment** development

**Design Philosophy**: Each developer gets their own dedicated VM for maximum independence, flexibility, and isolation rather than sharing high-spec machines.

---

### Team Structure & VM Allocation

| Role | Count | Primary Work | Recommended OS |
|------|-------|--------------|----------------|
| Frontend Developers (Next.js/React) | 5 | React/Next.js development | Linux (Ubuntu) |
| Backend Developers (Java) | 4 | Java/Spring Boot development | Linux (Ubuntu) |
| .NET Legacy Developers | 3 | .NET maintenance & migration | Windows Server |
| DevOps/Infrastructure | 2 | CI/CD, Docker, deployment | Linux (Ubuntu) |

---

### Recommended Instance Types

#### For Linux Developers (Frontend, Backend Java, DevOps)

| Instance Type | vCPU | Memory | Storage | Use Case |
|---------------|------|--------|---------|----------|
| **t3.medium** | 2 | 4 GB | EBS | Light development, testing |
| **t3.large** (Recommended) | 2 | 8 GB | EBS | Standard development |
| **t3.xlarge** | 4 | 16 GB | EBS | Heavy compilation, Docker builds |
| **m5.large** | 2 | 8 GB | EBS | Steady workload, no burst |
| **m5.xlarge** | 4 | 16 GB | EBS | Heavy workload, multiple containers |

#### For Windows Developers (.NET)

| Instance Type | vCPU | Memory | Storage | Use Case |
|---------------|------|--------|---------|----------|
| **t3.large** | 2 | 8 GB | EBS | Light .NET development |
| **t3.xlarge** (Recommended) | 4 | 16 GB | EBS | Standard .NET/Visual Studio |
| **t3.2xlarge** | 8 | 32 GB | EBS | Heavy .NET, multiple solutions |
| **m5.xlarge** | 4 | 16 GB | EBS | Steady workload |
| **m5.2xlarge** | 8 | 32 GB | EBS | Enterprise .NET development |

---

### Pricing Comparison (Tokyo Region - ap-northeast-1)

#### On-Demand Pricing (Per Hour)

| Instance Type | Linux (USD/hr) | Windows (USD/hr) | Linux (JPY/hr)* | Windows (JPY/hr)* |
|---------------|----------------|------------------|-----------------|-------------------|
| t3.medium | $0.0544 | $0.0744 | ¥8.16 | ¥11.16 |
| t3.large | $0.1088 | $0.1488 | ¥16.32 | ¥22.32 |
| t3.xlarge | $0.2176 | $0.2976 | ¥32.64 | ¥44.64 |
| t3.2xlarge | $0.4352 | $0.5952 | ¥65.28 | ¥89.28 |
| m5.large | $0.124 | $0.192 | ¥18.60 | ¥28.80 |
| m5.xlarge | $0.248 | $0.384 | ¥37.20 | ¥57.60 |
| m5.2xlarge | $0.496 | $0.768 | ¥74.40 | ¥115.20 |

*JPY calculated at 150 JPY/USD

---

### Recommended Configuration for 14 Members

#### Option A: Cost-Optimized (Recommended)

| Role | Count | Instance Type | OS | Hourly Cost | Monthly Cost (8hr/day × 22 days) |
|------|-------|---------------|----|-----------:|--------------------------------:|
| Frontend Dev | 5 | t3.large | Linux | $0.1088 | $95.74 each |
| Java Backend Dev | 4 | t3.large | Linux | $0.1088 | $95.74 each |
| .NET Dev | 3 | t3.xlarge | Windows | $0.2976 | $261.89 each |
| DevOps | 2 | t3.large | Linux | $0.1088 | $95.74 each |

**Total Monthly Cost:**
- Linux (11 instances): 11 × $95.74 = **$1,053.14**
- Windows (3 instances): 3 × $261.89 = **$785.67**
- **Grand Total: $1,838.81/month (~¥275,821)**

#### Option B: Performance-Optimized

| Role | Count | Instance Type | OS | Hourly Cost | Monthly Cost (8hr/day × 22 days) |
|------|-------|---------------|----|-----------:|--------------------------------:|
| Frontend Dev | 5 | t3.xlarge | Linux | $0.2176 | $191.49 each |
| Java Backend Dev | 4 | m5.xlarge | Linux | $0.248 | $218.24 each |
| .NET Dev | 3 | t3.2xlarge | Windows | $0.5952 | $523.78 each |
| DevOps | 2 | m5.xlarge | Linux | $0.248 | $218.24 each |

**Total Monthly Cost:**
- Frontend (5): 5 × $191.49 = **$957.45**
- Java (4): 4 × $218.24 = **$872.96**
- .NET (3): 3 × $523.78 = **$1,571.34**
- DevOps (2): 2 × $218.24 = **$436.48**
- **Grand Total: $3,838.23/month (~¥575,734)**

---

### 2-Month Project Cost Estimate

| Configuration | Monthly Cost | 2-Month Total | 2-Month Total (JPY) |
|---------------|--------------|---------------|---------------------|
| Option A (Cost-Optimized) | $1,838.81 | **$3,677.62** | **¥551,643** |
| Option B (Performance) | $3,838.23 | **$7,676.46** | **¥1,151,469** |

---

### Additional Costs to Consider

| Item | Estimated Monthly Cost |
|------|----------------------:|
| EBS Storage (50GB × 14) | ~$70 |
| Data Transfer (100GB out) | ~$14 |
| Elastic IP (14 addresses) | $0 (if attached) |
| S3 Storage (shared artifacts) | ~$25 |
| Load Balancer (if needed) | ~$20 |

**Additional Monthly Total: ~$129 (~¥19,350)**

---

### Cost Saving Strategies

#### 1. Reserved Instances (1-year commitment)
- **Savings**: Up to 40% off On-Demand pricing
- **Best for**: Long-term projects (6+ months)

#### 2. Spot Instances (for non-critical workloads)
- **Savings**: Up to 90% off On-Demand pricing
- **Use case**: CI/CD runners, batch processing, testing

#### 3. Scheduled Start/Stop
- Run instances only during work hours (8 hours × 22 days)
- **Savings**: ~75% vs 24/7 operation

#### 4. Right-sizing
- Start with t3.large, upgrade if needed
- Monitor CloudWatch metrics for CPU/Memory usage

---

### Recommended Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                         AWS VPC (10.0.0.0/16)                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │              Public Subnet (10.0.1.0/24)                      │   │
│  │                                                               │   │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐             │   │
│  │  │ Frontend-1  │ │ Frontend-2  │ │ Frontend-3  │  ...        │   │
│  │  │ (t3.large)  │ │ (t3.large)  │ │ (t3.large)  │             │   │
│  │  │   Linux     │ │   Linux     │ │   Linux     │             │   │
│  │  └─────────────┘ └─────────────┘ └─────────────┘             │   │
│  │                                                               │   │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐             │   │
│  │  │  Java-1     │ │  Java-2     │ │  Java-3     │  ...        │   │
│  │  │ (t3.large)  │ │ (t3.large)  │ │ (t3.large)  │             │   │
│  │  │   Linux     │ │   Linux     │ │   Linux     │             │   │
│  │  └─────────────┘ └─────────────┘ └─────────────┘             │   │
│  │                                                               │   │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐             │   │
│  │  │  .NET-1     │ │  .NET-2     │ │  .NET-3     │             │   │
│  │  │ (t3.xlarge) │ │ (t3.xlarge) │ │ (t3.xlarge) │             │   │
│  │  │  Windows    │ │  Windows    │ │  Windows    │             │   │
│  │  └─────────────┘ └─────────────┘ └─────────────┘             │   │
│  │                                                               │   │
│  │  ┌─────────────┐ ┌─────────────┐                              │   │
│  │  │  DevOps-1   │ │  DevOps-2   │                              │   │
│  │  │ (t3.large)  │ │ (t3.large)  │                              │   │
│  │  │   Linux     │ │   Linux     │                              │   │
│  │  └─────────────┘ └─────────────┘                              │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │              Shared Services                                  │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐      │   │
│  │  │   RDS    │  │   S3     │  │  ECR     │  │ CodeBuild│      │   │
│  │  │ (if req) │  │ Artifacts│  │  Docker  │  │  CI/CD   │      │   │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘      │   │
│  └──────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

---

### Setup Checklist

#### Initial Setup
- [ ] Create VPC with appropriate subnets
- [ ] Configure Security Groups (SSH/RDP access)
- [ ] Create IAM roles for EC2 instances
- [ ] Set up key pairs for each team member

#### Per Developer Instance
- [ ] Launch EC2 instance with appropriate AMI
- [ ] Attach 50GB EBS volume
- [ ] Assign Elastic IP (optional)
- [ ] Install development tools:
  - **Linux**: Docker, Node.js, Java, Git
  - **Windows**: Visual Studio, .NET SDK, Git

#### Shared Resources
- [ ] Set up S3 bucket for artifacts
- [ ] Configure ECR for Docker images
- [ ] Set up CodeBuild/CodePipeline (optional)

---

## 日本語

### 概要

このガイドは、アプリケーションモダナイゼーションプロジェクトに取り組む**14名の開発チーム**向けのAWS EC2構成を提供します：
- **.NET から Next.js/React** への移行
- **Java環境**開発

**設計思想**: 高スペックのマシンを共有するのではなく、各開発者が独自の専用VMを持つことで、最大限の独立性、柔軟性、分離を実現します。

---

### チーム構成とVM割り当て

| 役割 | 人数 | 主な作業 | 推奨OS |
|------|------|----------|--------|
| フロントエンド開発者（Next.js/React） | 5名 | React/Next.js開発 | Linux (Ubuntu) |
| バックエンド開発者（Java） | 4名 | Java/Spring Boot開発 | Linux (Ubuntu) |
| .NETレガシー開発者 | 3名 | .NETメンテナンス＆移行 | Windows Server |
| DevOps/インフラ | 2名 | CI/CD、Docker、デプロイ | Linux (Ubuntu) |

---

### 推奨インスタンスタイプ

#### Linux開発者向け（フロントエンド、Javaバックエンド、DevOps）

| インスタンスタイプ | vCPU | メモリ | ストレージ | 用途 |
|-------------------|------|--------|-----------|------|
| **t3.medium** | 2 | 4 GB | EBS | 軽量開発、テスト |
| **t3.large**（推奨） | 2 | 8 GB | EBS | 標準開発 |
| **t3.xlarge** | 4 | 16 GB | EBS | 重いコンパイル、Dockerビルド |
| **m5.large** | 2 | 8 GB | EBS | 安定したワークロード |
| **m5.xlarge** | 4 | 16 GB | EBS | 重いワークロード、複数コンテナ |

#### Windows開発者向け（.NET）

| インスタンスタイプ | vCPU | メモリ | ストレージ | 用途 |
|-------------------|------|--------|-----------|------|
| **t3.large** | 2 | 8 GB | EBS | 軽量.NET開発 |
| **t3.xlarge**（推奨） | 4 | 16 GB | EBS | 標準.NET/Visual Studio |
| **t3.2xlarge** | 8 | 32 GB | EBS | 重い.NET、複数ソリューション |
| **m5.xlarge** | 4 | 16 GB | EBS | 安定したワークロード |
| **m5.2xlarge** | 8 | 32 GB | EBS | エンタープライズ.NET開発 |

---

### 料金比較（東京リージョン - ap-northeast-1）

#### オンデマンド料金（1時間あたり）

| インスタンスタイプ | Linux (USD/時) | Windows (USD/時) | Linux (円/時)* | Windows (円/時)* |
|-------------------|----------------|------------------|----------------|------------------|
| t3.medium | $0.0544 | $0.0744 | ¥8.16 | ¥11.16 |
| t3.large | $0.1088 | $0.1488 | ¥16.32 | ¥22.32 |
| t3.xlarge | $0.2176 | $0.2976 | ¥32.64 | ¥44.64 |
| t3.2xlarge | $0.4352 | $0.5952 | ¥65.28 | ¥89.28 |
| m5.large | $0.124 | $0.192 | ¥18.60 | ¥28.80 |
| m5.xlarge | $0.248 | $0.384 | ¥37.20 | ¥57.60 |
| m5.2xlarge | $0.496 | $0.768 | ¥74.40 | ¥115.20 |

*1ドル=150円で計算

---

### 14名向け推奨構成

#### オプションA: コスト最適化（推奨）

| 役割 | 人数 | インスタンスタイプ | OS | 時間単価 | 月額（8時間/日 × 22日） |
|------|------|-------------------|----|---------:|------------------------:|
| フロントエンド開発 | 5 | t3.large | Linux | $0.1088 | $95.74（各） |
| Javaバックエンド開発 | 4 | t3.large | Linux | $0.1088 | $95.74（各） |
| .NET開発 | 3 | t3.xlarge | Windows | $0.2976 | $261.89（各） |
| DevOps | 2 | t3.large | Linux | $0.1088 | $95.74（各） |

**月額合計:**
- Linux（11インスタンス）: 11 × $95.74 = **$1,053.14**（約¥157,971）
- Windows（3インスタンス）: 3 × $261.89 = **$785.67**（約¥117,850）
- **総合計: $1,838.81/月（約¥275,821）**

#### オプションB: パフォーマンス最適化

| 役割 | 人数 | インスタンスタイプ | OS | 時間単価 | 月額（8時間/日 × 22日） |
|------|------|-------------------|----|---------:|------------------------:|
| フロントエンド開発 | 5 | t3.xlarge | Linux | $0.2176 | $191.49（各） |
| Javaバックエンド開発 | 4 | m5.xlarge | Linux | $0.248 | $218.24（各） |
| .NET開発 | 3 | t3.2xlarge | Windows | $0.5952 | $523.78（各） |
| DevOps | 2 | m5.xlarge | Linux | $0.248 | $218.24（各） |

**月額合計:**
- フロントエンド（5名）: 5 × $191.49 = **$957.45**
- Java（4名）: 4 × $218.24 = **$872.96**
- .NET（3名）: 3 × $523.78 = **$1,571.34**
- DevOps（2名）: 2 × $218.24 = **$436.48**
- **総合計: $3,838.23/月（約¥575,734）**

---

### 2ヶ月プロジェクトコスト見積もり

| 構成 | 月額コスト | 2ヶ月合計 | 2ヶ月合計（円） |
|------|-----------|----------|----------------|
| オプションA（コスト最適化） | $1,838.81 | **$3,677.62** | **約¥551,643** |
| オプションB（パフォーマンス） | $3,838.23 | **$7,676.46** | **約¥1,151,469** |

---

### 追加コストの検討

| 項目 | 月額推定コスト |
|------|---------------:|
| EBSストレージ（50GB × 14） | 約$70（約¥10,500） |
| データ転送（100GB送信） | 約$14（約¥2,100） |
| Elastic IP（14アドレス） | $0（アタッチ時） |
| S3ストレージ（共有アーティファクト） | 約$25（約¥3,750） |
| ロードバランサー（必要な場合） | 約$20（約¥3,000） |

**追加月額合計: 約$129（約¥19,350）**

---

### コスト削減戦略

#### 1. リザーブドインスタンス（1年コミットメント）
- **削減率**: オンデマンド料金から最大40%オフ
- **最適な用途**: 長期プロジェクト（6ヶ月以上）

#### 2. スポットインスタンス（非クリティカルワークロード向け）
- **削減率**: オンデマンド料金から最大90%オフ
- **用途**: CI/CDランナー、バッチ処理、テスト

#### 3. スケジュールされた起動/停止
- 作業時間のみインスタンスを実行（8時間 × 22日）
- **削減率**: 24時間365日運用と比較して約75%削減

#### 4. 適切なサイジング
- t3.largeから開始し、必要に応じてアップグレード
- CloudWatchメトリクスでCPU/メモリ使用率を監視

---

### 推奨アーキテクチャ

```
┌─────────────────────────────────────────────────────────────────────┐
│                         AWS VPC (10.0.0.0/16)                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │              パブリックサブネット (10.0.1.0/24)                │   │
│  │                                                               │   │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐             │   │
│  │  │フロントエンド1│ │フロントエンド2│ │フロントエンド3│  ...        │   │
│  │  │ (t3.large)  │ │ (t3.large)  │ │ (t3.large)  │             │   │
│  │  │   Linux     │ │   Linux     │ │   Linux     │             │   │
│  │  └─────────────┘ └─────────────┘ └─────────────┘             │   │
│  │                                                               │   │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐             │   │
│  │  │  Java-1     │ │  Java-2     │ │  Java-3     │  ...        │   │
│  │  │ (t3.large)  │ │ (t3.large)  │ │ (t3.large)  │             │   │
│  │  │   Linux     │ │   Linux     │ │   Linux     │             │   │
│  │  └─────────────┘ └─────────────┘ └─────────────┘             │   │
│  │                                                               │   │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐             │   │
│  │  │  .NET-1     │ │  .NET-2     │ │  .NET-3     │             │   │
│  │  │ (t3.xlarge) │ │ (t3.xlarge) │ │ (t3.xlarge) │             │   │
│  │  │  Windows    │ │  Windows    │ │  Windows    │             │   │
│  │  └─────────────┘ └─────────────┘ └─────────────┘             │   │
│  │                                                               │   │
│  │  ┌─────────────┐ ┌─────────────┐                              │   │
│  │  │  DevOps-1   │ │  DevOps-2   │                              │   │
│  │  │ (t3.large)  │ │ (t3.large)  │                              │   │
│  │  │   Linux     │ │   Linux     │                              │   │
│  │  └─────────────┘ └─────────────┘                              │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │              共有サービス                                     │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐      │   │
│  │  │   RDS    │  │   S3     │  │  ECR     │  │ CodeBuild│      │   │
│  │  │（必要時） │  │アーティファクト│ │  Docker  │  │  CI/CD   │      │   │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘      │   │
│  └──────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

---

### セットアップチェックリスト

#### 初期セットアップ
- [ ] 適切なサブネットを持つVPCを作成
- [ ] セキュリティグループを設定（SSH/RDPアクセス）
- [ ] EC2インスタンス用のIAMロールを作成
- [ ] 各チームメンバー用のキーペアを設定

#### 開発者ごとのインスタンス
- [ ] 適切なAMIでEC2インスタンスを起動
- [ ] 50GB EBSボリュームをアタッチ
- [ ] Elastic IPを割り当て（オプション）
- [ ] 開発ツールをインストール:
  - **Linux**: Docker、Node.js、Java、Git
  - **Windows**: Visual Studio、.NET SDK、Git

#### 共有リソース
- [ ] アーティファクト用S3バケットを設定
- [ ] DockerイメージパブリケーションS3バケットのECRを設定
- [ ] CodeBuild/CodePipelineを設定（オプション）

---

### Quick Start コマンド

#### Linux インスタンスのセットアップ

```bash
# Docker のインストール
sudo apt-get update
sudo apt-get install -y docker.io docker-compose git

# Node.js のインストール (Next.js/React 開発用)
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs

# Java のインストール (Java 開発用)
sudo apt-get install -y openjdk-17-jdk maven

# Docker サービスの開始
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker $USER
```

#### Windows インスタンスのセットアップ

```powershell
# Chocolatey のインストール
Set-ExecutionPolicy Bypass -Scope Process -Force
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072
iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))

# 開発ツールのインストール
choco install -y git visualstudio2022community dotnet-sdk nodejs
```

---

### 総合コストサマリー

#### オプションA（コスト最適化）- 2ヶ月プロジェクト

| 項目 | 月額 | 2ヶ月合計 |
|------|-----:|----------:|
| EC2インスタンス（14台） | $1,838.81 | $3,677.62 |
| EBS/S3/データ転送 | $129.00 | $258.00 |
| **総合計** | **$1,967.81** | **$3,935.62** |
| **日本円換算** | **約¥295,171** | **約¥590,343** |

#### オプションB（パフォーマンス最適化）- 2ヶ月プロジェクト

| 項目 | 月額 | 2ヶ月合計 |
|------|-----:|----------:|
| EC2インスタンス（14台） | $3,838.23 | $7,676.46 |
| EBS/S3/データ転送 | $129.00 | $258.00 |
| **総合計** | **$3,967.23** | **$7,934.46** |
| **日本円換算** | **約¥595,084** | **約¥1,190,169** |

---

## References / 参考リンク

- [AWS EC2 Pricing](https://aws.amazon.com/ec2/pricing/)
- [AWS Pricing Calculator](https://calculator.aws/)
- [EC2 Instance Types](https://aws.amazon.com/ec2/instance-types/)
- [AWS Tokyo Region](https://aws.amazon.com/local/tokyo-region/)

---

*Last updated / 最終更新: 2025-12-21*
