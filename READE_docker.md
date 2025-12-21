# Docker セットアップガイド

このリポジトリには、アプリケーションを実行するための Docker 設定が含まれています。

*Last updated: 2025-12-21*

## アーキテクチャ概要

このプロジェクトは **マルチユーザー Docker 構成** を使用しています。

### 重要な設計原則

**各ユーザーは独立したコンテナを持ちます:**
- User1 → `user1-app` コンテナ（ポート 3001）
- User2 → `user2-app` コンテナ（ポート 3002）
- User3 → `user3-app` コンテナ（ポート 3003）

**この構成の利点:**
1. **独立性**: 各ユーザーのアプリは完全に分離されています
2. **個別更新**: 1つのユーザーアプリを更新しても他のユーザーに影響しません
3. **データ分離**: 各ユーザーは専用のデータボリュームを持ちます
4. **スケーラビリティ**: 新しいユーザーを簡単に追加できます

**必ず守るべきルール:**
- アプリを更新する際は、必ずこの構成を維持してください
- 各ユーザーのコンテナ名とポートを変更しないでください
- `docker-compose.yml` の構造を保持してください

## 前提条件

### Docker のインストール

Windows に Docker Desktop をインストールする必要があります。

#### Chocolatey を使用したインストール

```bash
choco install docker-desktop -y
```

インストール後、環境変数を更新するために新しいターミナルを開くか、以下を実行:

```powershell
Import-Module $env:ChocolateyInstall\helpers\chocolateyProfile.psm1
refreshenv
```

#### 手動インストール

1. [Docker Desktop for Windows](https://www.docker.com/products/docker-desktop/) をダウンロード
2. インストーラーを実行
3. コンピューターを再起動

### 仮想化サポートの有効化

Docker Desktop を実行するには、システムで仮想化が有効になっている必要があります。

**重要: Windows EC2 での Docker Desktop インストール失敗**

**結論: Windows EC2 では Docker Desktop は使用できません**

AWS EC2 Windows インスタンスでは、以下の理由により Docker Desktop のインストールと実行が失敗します:

1. **WSL 2 のインストール失敗**
   - EC2 Windows インスタンスでは WSL 2 が正常にインストールできません
   - Docker Desktop は WSL 2 または Hyper-V を必要とします

2. **ネストされた仮想化の非サポート**
   - EC2 はネストされた仮想化をサポートしていません
   - Hyper-V や WSL 2 バックエンドが動作しません

**発生するエラー:**

```
Virtualization support not detected
Docker Desktop failed to start because virtualisation support wasn't detected.
```

**WSL インストール試行時のエラー:**

```
wsl --install
# EC2 では失敗します
```

**EC2 での解決策:**

1. **物理マシンまたはネストされた仮想化をサポートする環境を使用**
   - ローカルの物理 Windows マシン
   - ネストされた仮想化をサポートする VM

2. **または Linux ベースの Docker を使用**
   - EC2 Linux インスタンス（Ubuntu、Amazon Linux など）を使用
   - Docker Engine を直接インストール（Docker Desktop は不要）
   ```bash
   # Ubuntu の例
   sudo apt-get update
   sudo apt-get install docker.io docker-compose -y
   sudo systemctl start docker
   sudo systemctl enable docker
   ```

3. **Windows Server コンテナを使用**（限定的なサポート）
   - Windows Server 2019 以降の EC2 インスタンス
   - Windows コンテナのみサポート（Linux コンテナは不可）

**物理マシンでの仮想化有効化:**

1. **BIOS で仮想化を有効にする**:
   - コンピューターを再起動
   - 起動時に BIOS に入る（通常は F2、F10、Del、または Esc キーを押す）
   - "Virtualization Technology"、"Intel VT-x"、または "AMD-V" を探す
   - 有効にする
   - 保存して終了

2. **WSL 2 を使用する** (Windows 10/11 の場合):
   ```bash
   wsl --install
   ```
   再起動後、Docker Desktop を WSL 2 バックエンドを使用するように設定

## Docker の確認

Docker が正しくインストールされているか確認:

```bash
docker --version
docker compose version
```

Docker Desktop が実行中か確認:

```bash
docker ps
```

## 使用方法

### 複数ユーザーアプリの管理

このセットアップでは、複数のユーザーが個別のアプリケーションインスタンスを実行できます。

#### 管理スクリプトの使用

```bash
# スクリプトに実行権限を付与
chmod +x manage-apps.sh

# 特定のユーザーアプリを起動
./manage-apps.sh start user1

# 特定のユーザーアプリを停止
./manage-apps.sh stop user2

# 特定のユーザーアプリを更新（再ビルド）
./manage-apps.sh update user3

# 特定のユーザーアプリのログを表示
./manage-apps.sh logs user1

# すべてのアプリの状態を確認
./manage-apps.sh status

# すべてのユーザーアプリを起動
./manage-apps.sh start-all

# すべてのユーザーアプリを停止
./manage-apps.sh stop-all

# すべてのユーザーアプリを更新
./manage-apps.sh update-all
```

#### Docker Compose コマンドを直接使用

```bash
# 特定のユーザーアプリを起動
docker compose up -d user1-app

# 特定のユーザーアプリを停止
docker compose stop user2-app

# 特定のユーザーアプリを再起動
docker compose restart user3-app

# 特定のユーザーアプリを更新
docker compose build user1-app
docker compose up -d user1-app

# すべてのアプリを起動
docker compose up -d

# すべてのアプリを停止
docker compose down
```

### 個別アプリの更新方法

各ユーザーのアプリを個別に更新できます:

```bash
# User1 のアプリのみを更新
./manage-apps.sh update user1

# または Docker Compose で直接
docker compose build user1-app
docker compose up -d user1-app

# 他のユーザーアプリは影響を受けません
```

### コンテナのビルドと起動

```bash
# ビルドしてバックグラウンドで起動
docker compose up --build -d

# ログを確認
docker compose logs -f

# コンテナの状態を確認
docker ps
```

### コンテナの停止

```bash
docker compose down
```

### コンテナの再起動

```bash
docker compose restart
```

## ファイル構成

- `Dockerfile` - Docker イメージのビルド設定
- `docker-compose.yml` - Docker Compose の設定（複数ユーザー対応）
- `.dockerignore` - Docker ビルドから除外するファイル
- `manage-apps.sh` - 個別アプリ管理スクリプト

## トラブルシューティング

### "docker: command not found" エラー

新しいターミナルを開くか、環境変数を更新:

```powershell
Import-Module $env:ChocolateyInstall\helpers\chocolateyProfile.psm1
refreshenv
```

### "Virtualization support not detected" エラー

**Windows EC2 インスタンスの場合:**

このエラーは **Windows EC2 では解決できません**。理由:
- WSL 2 のインストールが失敗する
- ネストされた仮想化が利用できない
- Hyper-V が動作しない

**解決策:**
1. **Linux EC2 インスタンスを使用**（推奨）
2. **ローカルの物理 Windows マシンを使用**

**物理マシンの場合:**
- BIOS で仮想化を有効にしてください（上記参照）
- WSL 2 をインストール: `wsl --install`

### Docker Desktop が起動しない

1. 仮想化が有効になっているか確認
2. Docker Desktop を再起動
3. コンピューターを再起動
4. Docker Desktop を再インストール

### EC2 で Docker を使用する推奨方法

```bash
# Linux EC2 インスタンスを起動（Ubuntu 22.04 など）
# SSH で接続後:

# Docker のインストール
sudo apt-get update
sudo apt-get install docker.io docker-compose -y

# Docker サービスの開始
sudo systemctl start docker
sudo systemctl enable docker

# 現在のユーザーを docker グループに追加（sudo なしで実行可能）
sudo usermod -aG docker $USER

# ログアウトして再ログイン後、確認
docker --version
docker compose version

# このリポジトリをクローン
git clone https://github.com/tcs-imokawa-masanori/dockers.git
cd dockers

# コンテナを起動
docker compose up --build -d
```

## ポート

各ユーザーアプリは異なるポートで実行されます:

- User1 アプリケーション: `http://13.158.26.153:3001`
- User2 アプリケーション: `http://13.158.26.153:3002`
- User3 アプリケーション: `http://13.158.26.153:3003`

追加のユーザーが必要な場合は、`docker-compose.yml` に新しいサービスを追加してください。

### アプリケーションのテスト

#### フロントエンドのテスト

ブラウザで以下の URL にアクセス:
- User1: http://13.158.26.153:3001
- User2: http://13.158.26.153:3002
- User3: http://13.158.26.153:3003

#### バックエンド API のテスト

```bash
# User1 の API をテスト
curl http://13.158.26.153:3001/api/hello

# User2 の API をテスト
curl http://13.158.26.153:3002/api/hello

# User3 の API をテスト
curl http://13.158.26.153:3003/api/hello
```

#### すべてのアプリを一度にテスト

```bash
ssh -i edion.pem ubuntu@13.158.26.153 "curl -s http://localhost:3001/api/hello && curl -s http://localhost:3002/api/hello && curl -s http://localhost:3003/api/hello"
```

## 環境変数

必要に応じて `docker-compose.yml` の `environment` セクションで環境変数を設定できます。

## アプリケーションの更新手順

### 重要: 常にこの構成を使用してください

アプリケーションを更新する際は、**必ず以下の手順に従ってください**。この構成により、各ユーザーが独立したコンテナを維持できます。

### 個別ユーザーの更新（推奨）

特定のユーザーのアプリのみを更新する場合:

```bash
# User1 のアプリのみを更新
./manage-apps.sh update user1

# または Docker Compose で直接
sudo docker compose build user1-app
sudo docker compose up -d user1-app

# 確認
./manage-apps.sh logs user1
```

**利点:**
- 他のユーザーのアプリは影響を受けません
- ダウンタイムが最小限
- 問題が発生した場合、1つのユーザーのみに影響

### すべてのユーザーを一度に更新

すべてのユーザーアプリを同時に更新する場合:

```bash
# すべてのアプリを更新
./manage-apps.sh update-all

# または Docker Compose で直接
sudo docker compose down
sudo docker compose up --build -d

# 確認
./manage-apps.sh status
```

### コードの変更後の更新手順

1. **コードを変更**
   ```bash
   # ローカルでコードを編集
   vim app/page.tsx
   ```

2. **Git にプッシュ**
   ```bash
   git add .
   git commit -m "Update application code"
   git push origin main
   ```

3. **EC2 で更新を取得**
   ```bash
   ssh -i edion.pem ubuntu@13.158.26.153
   cd dockers
   git pull
   ```

4. **特定のユーザーを更新**
   ```bash
   # User1 のみ更新
   ./manage-apps.sh update user1

   # または複数のユーザーを更新
   ./manage-apps.sh update user1
   ./manage-apps.sh update user2
   ```

5. **動作確認**
   ```bash
   # API をテスト
   curl http://localhost:3001/api/hello

   # ログを確認
   ./manage-apps.sh logs user1
   ```

### 新しいユーザーの追加

新しいユーザーを追加する場合:

1. **docker-compose.yml を編集**
   ```yaml
   # User 4 Application を追加
   user4-app:
     build: .
     container_name: user4-app
     ports:
       - "3004:3000"
     environment:
       - NODE_ENV=development
       - USER_ID=user4
       - APP_NAME=user4-app
     volumes:
       - user4-data:/app/data
     restart: unless-stopped

   volumes:
     user1-data:
     user2-data:
     user3-data:
     user4-data:  # 追加
   ```

2. **新しいユーザーを起動**
   ```bash
   sudo docker compose up -d user4-app
   ```

### トラブルシューティング: 更新が反映されない場合

```bash
# キャッシュをクリアして再ビルド
sudo docker compose build --no-cache user1-app
sudo docker compose up -d user1-app

# または完全にクリーンアップ
sudo docker compose down
sudo docker system prune -f
sudo docker compose up --build -d
```

### ベストプラクティス

1. **常に個別更新を優先**: 1つのユーザーのみに影響を与える変更の場合
2. **更新前にバックアップ**: 重要なデータがある場合
3. **ログを確認**: 更新後は必ずログで動作を確認
4. **段階的ロールアウト**: User1 → User2 → User3 の順に更新
5. **この構成を維持**: `docker-compose.yml` の構造を変更しない

## サポート

問題が発生した場合は、[Docker のドキュメント](https://docs.docker.com/)を参照してください。

## 設定ファイルリファレンス

### docker-compose.yml の構造

この構成は **必ず維持** してください:

```yaml
version: '3.8'

services:
  # 各ユーザーは独立したサービス
  user1-app:
    build: .                    # 同じ Dockerfile を使用
    container_name: user1-app   # 一意のコンテナ名
    ports:
      - "3001:3000"             # 外部ポート:内部ポート
    environment:
      - USER_ID=user1           # ユーザー識別子
      - APP_NAME=user1-app      # アプリ名
    volumes:
      - user1-data:/app/data    # 専用データボリューム
    restart: unless-stopped

  user2-app:
    # 同じ構造を繰り返し

volumes:
  user1-data:  # 各ユーザーの永続データ
  user2-data:
  user3-data:
```

### 重要な設定項目

| 項目 | 説明 | 変更可否 |
|------|------|----------|
| `container_name` | コンテナの一意な名前 | 変更しない |
| `ports` | ポートマッピング | 慎重に変更 |
| `USER_ID` | ユーザー識別子 | 必要に応じて変更可 |
| `APP_NAME` | アプリケーション名 | 必要に応じて変更可 |
| `volumes` | データの永続化 | 削除しない |

### 環境変数の使用

各コンテナは以下の環境変数を使用します:

```javascript
// Next.js アプリ内で使用
const userId = process.env.USER_ID;      // "user1", "user2", etc.
const appName = process.env.APP_NAME;    // "user1-app", "user2-app", etc.
```

これにより、各ユーザーのアプリが自分の識別情報を認識できます。

## クイックリファレンス

### よく使うコマンド

```bash
# 状態確認
./manage-apps.sh status

# 個別起動
./manage-apps.sh start user1

# 個別停止
./manage-apps.sh stop user1

# 個別更新（最も重要）
./manage-apps.sh update user1

# ログ確認
./manage-apps.sh logs user1

# すべて起動
./manage-apps.sh start-all

# すべて停止
./manage-apps.sh stop-all
```

### 緊急時の対応

```bash
# すべてのコンテナを停止
sudo docker compose down

# すべてのコンテナを再起動
sudo docker compose restart

# 特定のコンテナを強制再起動
sudo docker restart user1-app

# コンテナの状態を確認
sudo docker ps -a

# リソース使用状況を確認
sudo docker stats
```
