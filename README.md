# Docker Infrastructure Documentation

AWS and Docker infrastructure documentation for application modernization team.

## Author / 作成者

**Masanori Imokawa** (TCS)

## Contents / 目次

| Document | Description |
|----------|-------------|
| [Docker Setup Guide](./READE_docker.md) | Docker configuration and multi-user setup guide |
| [AWS Deployment Guide](./aws-deployment-guide.md) | EC2 configuration recommendations for 14-member team |
| [EC2 vs Terminal Services](./ec2-vs-terminal-services.md) | Comparison between EC2 and Windows RDS |
| [EC2 Docker Compatibility](./ec2-docker-compatibility.md) | Docker compatibility on EC2 instances |
| [Windows EC2 Docker Guide](./windows-ec2-docker-guide.md) | Docker setup for Windows EC2 |
| [Architecture](./architecture.md) | System architecture documentation |
| [Claude Sonnet Pricing](./claude-sonnet-pricing.md) | AI service pricing information |

## Directory Structure / ディレクトリ構造

```
dockers/
├── README.md                      # This file
├── READE_docker.md               # Docker setup guide (Japanese)
├── architecture.md               # System architecture
├── aws-deployment-guide.md       # AWS EC2 deployment guide
├── claude-sonnet-pricing.md      # Pricing documentation
├── ec2-docker-compatibility.md   # EC2 Docker compatibility
├── ec2-vs-terminal-services.md   # EC2 vs RDS comparison
├── windows-ec2-docker-guide.md   # Windows Docker guide
├── infrastructure/               # Infrastructure configurations
└── scripts/                      # Deployment scripts
```

## Purpose / 目的

This repository contains documentation and scripts for:

- Multi-user Docker container setup
- AWS EC2 instance configuration for development teams
- Comparison of cloud infrastructure options
- Deployment automation

## License

Copyright (c) 2024 Masanori Imokawa. All rights reserved.
