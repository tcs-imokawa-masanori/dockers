# Multi-User Docker Architecture
# マルチユーザー Docker アーキテクチャ

## Overview / 概要

This document describes the architecture for running isolated multi-user applications on a single EC2 instance using Docker containers.

このドキュメントでは、Docker コンテナを使用して単一の EC2 インスタンス上で分離されたマルチユーザーアプリケーションを実行するためのアーキテクチャについて説明します。

### Key Principles / 主要原則

| Principle / 原則 | Description / 説明 |
|-----------|-------------|
| **Isolation / 分離** | Each user runs in a completely independent container / 各ユーザーは完全に独立したコンテナで実行される |
| **Portability / 可搬性** | Consistent environment across development and production / 開発環境と本番環境で一貫した環境 |
| **Scalability / 拡張性** | Easy to add new users by adding container definitions / コンテナ定義を追加するだけで新しいユーザーを簡単に追加可能 |
| **Maintainability / 保守性** | Individual updates without affecting other users / 他のユーザーに影響を与えずに個別の更新が可能 |

---

## System Architecture / システムアーキテクチャ

### High-Level Overview / 全体概要

```
                                    Internet
                                   インターネット
                                        │
                                        ▼
┌───────────────────────────────────────────────────────────────────────────────┐
│                           AWS EC2 Instance (Linux)                            │
│                        AWS EC2 インスタンス (Linux)                            │
│                              IP: xxx.xxx.xxx.xxx                              │
│  ┌─────────────────────────────────────────────────────────────────────────┐  │
│  │                           Docker Engine                                  │  │
│  │                         Docker エンジン                                   │  │
│  │                                                                          │  │
│  │   ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐       │  │
│  │   │   user1-app     │   │   user2-app     │   │   user3-app     │  ...  │  │
│  │   │                 │   │                 │   │                 │       │  │
│  │   │  ┌───────────┐  │   │  ┌───────────┐  │   │  ┌───────────┐  │       │  │
│  │   │  │ Next.js   │  │   │  │ Next.js   │  │   │  │ Next.js   │  │       │  │
│  │   │  │ Frontend  │  │   │  │ Frontend  │  │   │  │ Frontend  │  │       │  │
│  │   │  │フロントエンド│  │   │  │フロントエンド│  │   │  │フロントエンド│  │       │  │
│  │   │  ├───────────┤  │   │  ├───────────┤  │   │  ├───────────┤  │       │  │
│  │   │  │ API       │  │   │  │ API       │  │   │  │ API       │  │       │  │
│  │   │  │ Backend   │  │   │  │ Backend   │  │   │  │ Backend   │  │       │  │
│  │   │  │バックエンド │  │   │  │バックエンド │  │   │  │バックエンド │  │       │  │
│  │   │  └───────────┘  │   │  └───────────┘  │   │  └───────────┘  │       │  │
│  │   │                 │   │                 │   │                 │       │  │
│  │   │  Port: 3001     │   │  Port: 3002     │   │  Port: 3003     │       │  │
│  │   │  ポート: 3001    │   │  ポート: 3002    │   │  ポート: 3003    │       │  │
│  │   └────────┬────────┘   └────────┬────────┘   └────────┬────────┘       │  │
│  │            │                     │                     │                │  │
│  │            ▼                     ▼                     ▼                │  │
│  │   ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐       │  │
│  │   │  user1-data     │   │  user2-data     │   │  user3-data     │       │  │
│  │   │    (Volume)     │   │    (Volume)     │   │    (Volume)     │       │  │
│  │   │  (ボリューム)    │   │  (ボリューム)    │   │  (ボリューム)    │       │  │
│  │   └─────────────────┘   └─────────────────┘   └─────────────────┘       │  │
│  │                                                                          │  │
│  └─────────────────────────────────────────────────────────────────────────┘  │
└───────────────────────────────────────────────────────────────────────────────┘
```

### Component Details / コンポーネント詳細

| Component / コンポーネント | Role / 役割 | Technology / 技術 |
|-----------|------|------------|
| EC2 Instance / EC2インスタンス | Host machine / ホストマシン | Amazon Linux 2023 / Ubuntu |
| Docker Engine / Dockerエンジン | Container runtime / コンテナランタイム | Docker CE |
| User Container / ユーザーコンテナ | Isolated application / 分離されたアプリケーション | Next.js (Frontend + API / フロントエンド + API) |
| Data Volume / データボリューム | Persistent storage / 永続ストレージ | Docker Volume |

---

## Container Isolation / コンテナ分離

Each user container operates independently with complete isolation:

各ユーザーコンテナは完全に分離された状態で独立して動作します：

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                            Container Isolation                               │
│                              コンテナ分離                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  user1-app                 user2-app                 user3-app              │
│  ┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐    │
│  │                  │     │                  │     │                  │    │
│  │  Environment     │     │  Environment     │     │  Environment     │    │
│  │  環境変数         │     │  環境変数         │     │  環境変数         │    │
│  │  ┌────────────┐  │     │  ┌────────────┐  │     │  ┌────────────┐  │    │
│  │  │USER_ID     │  │     │  │USER_ID     │  │     │  │USER_ID     │  │    │
│  │  │ = user1    │  │     │  │ = user2    │  │     │  │ = user3    │  │    │
│  │  ├────────────┤  │     │  ├────────────┤  │     │  ├────────────┤  │    │
│  │  │APP_NAME    │  │     │  │APP_NAME    │  │     │  │APP_NAME    │  │    │
│  │  │ = user1-app│  │     │  │ = user2-app│  │     │  │ = user3-app│  │    │
│  │  └────────────┘  │     │  └────────────┘  │     │  └────────────┘  │    │
│  │                  │     │                  │     │                  │    │
│  │  Network         │     │  Network         │     │  Network         │    │
│  │  ネットワーク     │     │  ネットワーク     │     │  ネットワーク     │    │
│  │  Port: 3001      │     │  Port: 3002      │     │  Port: 3003      │    │
│  │                  │     │                  │     │                  │    │
│  │  Storage         │     │  Storage         │     │  Storage         │    │
│  │  ストレージ       │     │  ストレージ       │     │  ストレージ       │    │
│  │  /app/data ──────┼─X───┼──────────────────┼─X───┼──────────────────│    │
│  │       │          │     │       │          │     │       │          │    │
│  └───────┼──────────┘     └───────┼──────────┘     └───────┼──────────┘    │
│          │                        │                        │               │
│          ▼                        ▼                        ▼               │
│   [user1-data]             [user2-data]             [user3-data]           │
│                                                                              │
│   X = No cross-container access / コンテナ間アクセス不可                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Isolation Guarantees / 分離の保証

- **Process Isolation / プロセス分離**: Each container runs in its own namespace / 各コンテナは独自の名前空間で実行される
- **Network Isolation / ネットワーク分離**: Containers have separate network stacks / コンテナは別々のネットワークスタックを持つ
- **Filesystem Isolation / ファイルシステム分離**: Each container has its own filesystem / 各コンテナは独自のファイルシステムを持つ
- **Volume Isolation / ボリューム分離**: Data volumes are not shared between containers / データボリュームはコンテナ間で共有されない

---

## Request Flow / リクエストフロー

### API Request Lifecycle / APIリクエストのライフサイクル

```
  Client Browser
  クライアントブラウザ
        │
        │  1. HTTP Request / HTTPリクエスト
        │     GET http://xxx.xxx.xxx.xxx:3001/api/hello
        ▼
┌───────────────────────────────────────┐
│           EC2 Instance                │
│         EC2 インスタンス               │
│                                       │
│   Port 3001 ─────────────────────┐    │
│   ポート 3001                     │    │
│                                  │    │
│   ┌──────────────────────────────▼──┐ │
│   │        user1-app Container      │ │
│   │      user1-app コンテナ          │ │
│   │                                 │ │
│   │   2. Route to Next.js API       │ │
│   │      Next.js APIへルーティング    │ │
│   │            │                    │ │
│   │            ▼                    │ │
│   │   3. Read Environment Variables │ │
│   │      環境変数の読み取り           │ │
│   │      USER_ID = "user1"          │ │
│   │      APP_NAME = "user1-app"     │ │
│   │            │                    │ │
│   │            ▼                    │ │
│   │   4. Process Request            │ │
│   │      リクエストの処理             │ │
│   │            │                    │ │
│   │            ▼                    │ │
│   │   5. Generate Response          │ │
│   │      レスポンスの生成             │ │
│   │                                 │ │
│   └─────────────────────────────────┘ │
│                                       │
└───────────────────────────────────────┘
        │
        │  6. JSON Response / JSONレスポンス
        │
        ▼
  ┌─────────────────────────────────┐
  │ {                               │
  │   "message": "Hello World...",  │
  │   "userId": "user1",            │
  │   "appName": "user1-app",       │
  │   "timestamp": "2025-...",      │
  │   "status": "success"           │
  │ }                               │
  └─────────────────────────────────┘
```

---

## Update Strategies / 更新戦略

### Strategy 1: Individual Update (Recommended) / 戦略1: 個別更新（推奨）

Update a single user without affecting others:

他のユーザーに影響を与えずに単一のユーザーを更新します：

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        Individual Update Flow                                │
│                          個別更新フロー                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Step 1          Step 2          Step 3          Step 4          Result    │
│   ステップ1        ステップ2        ステップ3        ステップ4        結果      │
│                                                                              │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐   │
│  │  Code   │    │  Git    │    │  Git    │    │ Update  │    │ user1   │   │
│  │ Change  │───▶│  Push   │───▶│  Pull   │───▶│ user1   │───▶│ Updated │   │
│  │ コード   │    │         │    │ (EC2)   │    │ user1   │    │ 更新完了 │   │
│  │  変更   │    │         │    │         │    │  更新   │    │         │   │
│  └─────────┘    └─────────┘    └─────────┘    └─────────┘    └─────────┘   │
│                                                                              │
│                                                     │                        │
│                                                     ▼                        │
│                                              ┌─────────────┐                 │
│                                              │ user2, user3│                 │
│                                              │ Unaffected  │                 │
│                                              │ 影響なし     │                 │
│                                              └─────────────┘                 │
│                                                                              │
│  Command / コマンド: ./manage-apps.sh update user1                           │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Strategy 2: Full Update / 戦略2: 全体更新

Update all users simultaneously:

すべてのユーザーを同時に更新します：

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          Full Update Flow                                    │
│                          全体更新フロー                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Step 1          Step 2          Step 3          Step 4                    │
│   ステップ1        ステップ2        ステップ3        ステップ4                  │
│                                                                              │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────────────────────┐   │
│  │  Code   │    │  Git    │    │  Git    │    │     Update All          │   │
│  │ Change  │───▶│  Push   │───▶│  Pull   │───▶│     全体更新             │   │
│  │ コード   │    │         │    │ (EC2)   │    │                         │   │
│  │  変更   │    │         │    │         │    │  ┌─────┐ ┌─────┐ ┌─────┐│   │
│  └─────────┘    └─────────┘    └─────────┘    │  │user1│ │user2│ │user3││   │
│                                               │  │  ▲  │ │  ▲  │ │  ▲  ││   │
│                                               │  └──┼──┘ └──┼──┘ └──┼──┘│   │
│                                               │     └──────┼──────┘     │   │
│                                               │            │            │   │
│                                               │      All Restarted      │   │
│                                               │       全て再起動         │   │
│                                               └─────────────────────────┘   │
│                                                                              │
│  Command / コマンド: ./manage-apps.sh update-all                             │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Update Command Summary / 更新コマンドの概要

| Command / コマンド | Scope / 範囲 | Downtime / ダウンタイム | Use Case / 使用例 |
|---------|-------|----------|----------|
| `./manage-apps.sh update <user>` | Single user / 単一ユーザー | ~30 seconds for target / 対象に約30秒 | Bug fix, user-specific changes / バグ修正、ユーザー固有の変更 |
| `./manage-apps.sh update-all` | All users / 全ユーザー | ~1-2 minutes for all / 全体で約1-2分 | Major version upgrade / メジャーバージョンアップグレード |

---

## Scaling / スケーリング

### Adding a New User / 新規ユーザーの追加

```yaml
# docker-compose.yml

services:
  # Existing users / 既存ユーザー
  user1-app:
    # ... existing config / 既存の設定
  user2-app:
    # ... existing config / 既存の設定
  user3-app:
    # ... existing config / 既存の設定

  # New user / 新規ユーザー
  user4-app:
    build: .
    container_name: user4-app
    ports:
      - "3004:3000"          # New external port / 新しい外部ポート
    environment:
      - USER_ID=user4
      - APP_NAME=user4-app
    volumes:
      - user4-data:/app/data
    restart: unless-stopped

volumes:
  user1-data:
  user2-data:
  user3-data:
  user4-data:                # New volume / 新しいボリューム
```

### Scaling Considerations / スケーリングの考慮事項

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         Scaling Decision Matrix                              │
│                        スケーリング判断マトリックス                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Users      RAM Required    Recommended Instance    Est. Monthly Cost      │
│   ユーザー数  必要RAM         推奨インスタンス          月額費用概算            │
│   ─────      ────────────    ────────────────────    ─────────────────      │
│   1-3        512MB - 1GB     t3.small (2GB RAM)      ~$15-20                │
│   4-8        1GB - 2GB       t3.medium (4GB RAM)     ~$30-40                │
│   9-16       2GB - 4GB       t3.large (8GB RAM)      ~$60-80                │
│   17+        4GB+            Consider load balancer   Varies                 │
│              4GB以上          ロードバランサー検討      状況による              │
│                                                                              │
│   Note: Each container uses approximately 60-100MB RAM at idle              │
│   注：各コンテナはアイドル時に約60-100MB RAMを使用                              │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Resource Management / リソース管理

### Container Resource Usage / コンテナリソース使用量

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        Resource Monitoring                                   │
│                        リソースモニタリング                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Container      CPU %      Memory Usage       Network I/O                  │
│   コンテナ        CPU使用率   メモリ使用量        ネットワークI/O              │
│   ─────────      ─────      ────────────       ───────────                  │
│   user1-app      0.00%      63.06 MiB          3.31 kB / 7.67 kB            │
│   user2-app      0.00%      58.98 MiB          2.78 kB / 1.66 kB            │
│   user3-app      0.00%      57.21 MiB          2.59 kB / 1.66 kB            │
│                                                                              │
│   Command / コマンド: docker stats --no-stream                               │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Resource Limits Configuration (Optional) / リソース制限設定（オプション）

```yaml
# docker-compose.yml

user1-app:
  # ... other config / その他の設定
  deploy:
    resources:
      limits:
        cpus: '0.5'         # Max 50% of one CPU core / 最大CPUコア1つの50%
        memory: 512M        # Max 512MB RAM / 最大512MB RAM
      reservations:
        cpus: '0.25'        # Guaranteed 25% CPU / 保証25% CPU
        memory: 256M        # Guaranteed 256MB RAM / 保証256MB RAM
```

---

## Security Architecture / セキュリティアーキテクチャ

### Network Security / ネットワークセキュリティ

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        Network Security Model                                │
│                     ネットワークセキュリティモデル                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│                              Internet                                        │
│                            インターネット                                     │
│                                  │                                           │
│                                  ▼                                           │
│   ┌──────────────────────────────────────────────────────────────────────┐  │
│   │                    AWS Security Group                                 │  │
│   │                  AWS セキュリティグループ                               │  │
│   │                                                                       │  │
│   │   Inbound Rules / インバウンドルール:                                   │  │
│   │   ┌───────────────────────────────────────────────────────────────┐  │  │
│   │   │ Port 22   │ SSH      │ Your IP only │ Administration        │  │  │
│   │   │ ポート22  │ SSH      │ 自分のIPのみ  │ 管理用                 │  │  │
│   │   │ Port 3001 │ TCP      │ 0.0.0.0/0    │ user1 Application     │  │  │
│   │   │ ポート3001│ TCP      │ 0.0.0.0/0    │ user1 アプリケーション  │  │  │
│   │   │ Port 3002 │ TCP      │ 0.0.0.0/0    │ user2 Application     │  │  │
│   │   │ ポート3002│ TCP      │ 0.0.0.0/0    │ user2 アプリケーション  │  │  │
│   │   │ Port 3003 │ TCP      │ 0.0.0.0/0    │ user3 Application     │  │  │
│   │   │ ポート3003│ TCP      │ 0.0.0.0/0    │ user3 アプリケーション  │  │  │
│   │   └───────────────────────────────────────────────────────────────┘  │  │
│   │                                                                       │  │
│   └──────────────────────────────────────────────────────────────────────┘  │
│                                  │                                           │
│                                  ▼                                           │
│   ┌──────────────────────────────────────────────────────────────────────┐  │
│   │                      Docker Network                                   │  │
│   │                    Docker ネットワーク                                 │  │
│   │                                                                       │  │
│   │   ┌─────────┐     ┌─────────┐     ┌─────────┐                        │  │
│   │   │ user1   │     │ user2   │     │ user3   │                        │  │
│   │   │  :3001  │     │  :3002  │     │  :3003  │                        │  │
│   │   └─────────┘     └─────────┘     └─────────┘                        │  │
│   │        │               │               │                              │  │
│   │        └───────────────┼───────────────┘                              │  │
│   │                        │                                              │  │
│   │              Isolated by default                                      │  │
│   │              デフォルトで分離                                          │  │
│   │                                                                       │  │
│   └──────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Data Security / データセキュリティ

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          Data Security                                       │
│                        データセキュリティ                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   1. Volume Isolation / ボリューム分離                                        │
│      ┌─────────────────────────────────────────────────────────────────┐    │
│      │                                                                  │    │
│      │   user1-data ◄────── Only user1-app can access                  │    │
│      │                      user1-appのみアクセス可能                    │    │
│      │   user2-data ◄────── Only user2-app can access                  │    │
│      │                      user2-appのみアクセス可能                    │    │
│      │   user3-data ◄────── Only user3-app can access                  │    │
│      │                      user3-appのみアクセス可能                    │    │
│      │                                                                  │    │
│      └─────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│   2. Environment Variable Management / 環境変数管理                          │
│      ┌─────────────────────────────────────────────────────────────────┐    │
│      │                                                                  │    │
│      │   .env (not committed to git / gitにコミットしない)               │    │
│      │   ─────────────────────────────                                  │    │
│      │   USER1_API_KEY=secret_key_1                                     │    │
│      │   USER2_API_KEY=secret_key_2                                     │    │
│      │   USER3_API_KEY=secret_key_3                                     │    │
│      │                                                                  │    │
│      └─────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│   3. Secret Injection / シークレット注入                                      │
│      ┌─────────────────────────────────────────────────────────────────┐    │
│      │                                                                  │    │
│      │   docker-compose.yml                                             │    │
│      │   ──────────────────                                             │    │
│      │   user1-app:                                                     │    │
│      │     environment:                                                 │    │
│      │       - API_KEY=${USER1_API_KEY}                                │    │
│      │                                                                  │    │
│      └─────────────────────────────────────────────────────────────────┘    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Operations / 運用

### Common Commands / 基本コマンド

| Operation / 操作 | Command / コマンド | Description / 説明 |
|-----------|---------|-------------|
| Start all / 全て起動 | `docker compose up -d` | Start all containers / 全コンテナを起動 |
| Stop all / 全て停止 | `docker compose down` | Stop all containers / 全コンテナを停止 |
| Start one / 個別起動 | `docker compose up -d user1-app` | Start specific container / 特定のコンテナを起動 |
| Stop one / 個別停止 | `docker compose stop user1-app` | Stop specific container / 特定のコンテナを停止 |
| View logs / ログ確認 | `docker logs -f user1-app` | Follow container logs / コンテナログをフォロー |
| Shell access / シェルアクセス | `docker exec -it user1-app /bin/sh` | Access container shell / コンテナシェルにアクセス |
| Resource usage / リソース使用量 | `docker stats` | Real-time resource monitoring / リアルタイムリソース監視 |

### Health Check / ヘルスチェック

```bash
# Check all container status / 全コンテナのステータス確認
docker compose ps

# Check specific container health / 特定コンテナのヘルス確認
docker inspect --format='{{.State.Health.Status}}' user1-app

# Check container resource usage / コンテナリソース使用量確認
docker stats --no-stream
```

---

## Disaster Recovery / 災害復旧

### Backup Strategy / バックアップ戦略

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          Backup Strategy                                     │
│                          バックアップ戦略                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Component           Backup Method            Frequency                     │
│   コンポーネント        バックアップ方法          頻度                          │
│   ─────────           ─────────────            ─────────                     │
│   Application Code    Git Repository           On every push                 │
│   アプリコード         Gitリポジトリ            プッシュ毎                      │
│   Docker Config       Git Repository           On every change               │
│   Docker設定          Gitリポジトリ            変更毎                          │
│   User Data Volumes   docker volume backup     Daily / Weekly                │
│   ユーザーデータ       dockerボリュームバックアップ 日次/週次                    │
│   EC2 Instance        AMI Snapshot             Weekly                        │
│   EC2インスタンス      AMIスナップショット       週次                           │
│                                                                              │
│   Volume Backup Command / ボリュームバックアップコマンド:                       │
│   docker run --rm -v user1-data:/data -v $(pwd):/backup \                   │
│     alpine tar czf /backup/user1-data-backup.tar.gz /data                   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Recovery Procedures / 復旧手順

```bash
# 1. Restore from Git / Gitから復元
git clone <repository-url>
cd <project>

# 2. Restore volumes (if backed up) / ボリュームを復元（バックアップがある場合）
docker run --rm -v user1-data:/data -v $(pwd):/backup \
  alpine tar xzf /backup/user1-data-backup.tar.gz -C /

# 3. Start containers / コンテナを起動
docker compose up -d
```

---

## Architecture Benefits / アーキテクチャのメリット

| Benefit / メリット | Description / 説明 |
|---------|-------------|
| **Isolation / 分離** | Each user operates in a completely independent environment / 各ユーザーは完全に独立した環境で動作 |
| **Individual Updates / 個別更新** | Update one user without affecting others / 他のユーザーに影響を与えずに1人のユーザーを更新 |
| **Fault Containment / 障害封じ込め** | Issues in one container don't impact others / 1つのコンテナの問題が他に影響しない |
| **Easy Scaling / 簡単なスケーリング** | Add new users by simply adding container definitions / コンテナ定義を追加するだけで新規ユーザー追加 |
| **Resource Efficiency / リソース効率** | Share host resources while maintaining isolation / 分離を維持しながらホストリソースを共有 |
| **Consistent Environment / 一貫した環境** | Same container image ensures consistency / 同じコンテナイメージで一貫性を確保 |

---

## Related Documentation / 関連ドキュメント

- [EC2 Docker Compatibility Guide](./ec2-docker-compatibility.md) - Platform compatibility information / プラットフォーム互換性情報
- [Windows EC2 Docker Guide](./windows-ec2-docker-guide.md) - Windows-specific instructions / Windows固有の手順
