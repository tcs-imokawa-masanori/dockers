# Windows EC2 で Docker をインストールする方法

## 前提条件

- Windows Server 2019 または 2022 の EC2 インスタンス
- 管理者権限でのアクセス
- インスタンスタイプ: t3.medium 以上を推奨（メモリ 4GB 以上）
- 動作確認済みインスタンス: c5

---

## 方法1: Docker Desktop のインストール（GUI環境向け）

> **⚠️ EC2 環境では非推奨**
>
> Docker Desktop は WSL2 または Hyper-V を必要としますが、EC2 インスタンスはネスト仮想化をサポートしていないため、「Virtualization support not detected」エラーが発生します。
>
> **EC2 環境では「方法2: Docker Engine のインストール」を使用してください。**

以下の手順はオンプレミス環境や、ベアメタルインスタンス（`i3.metal`, `m5.metal` など）を使用する場合の参考情報です。

### 1. WSL2 の有効化

PowerShell を管理者として実行し、以下のコマンドを実行：

```powershell
# WSL機能を有効化
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

# 仮想マシンプラットフォームを有効化
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart

# 再起動
Restart-Computer
```

### 2. WSL2 をデフォルトに設定

再起動後、PowerShell で以下を実行：

```powershell
wsl --set-default-version 2
```

### 3. Docker Desktop のダウンロードとインストール

```powershell
# Docker Desktop インストーラーをダウンロード
Invoke-WebRequest -Uri "https://desktop.docker.com/win/main/amd64/Docker%20Desktop%20Installer.exe" -OutFile "$env:TEMP\DockerDesktopInstaller.exe"

# インストールを実行
Start-Process -FilePath "$env:TEMP\DockerDesktopInstaller.exe" -ArgumentList "install", "--quiet" -Wait
```

---

## 方法2: Docker Engine のインストール（Windows Server向け・推奨）

Windows Server では Docker Engine を直接インストールする方法が推奨されます。

### 1. Containers 機能の有効化

```powershell
# Containers機能をインストール
Install-WindowsFeature -Name Containers

# 再起動
Restart-Computer
```

### 2. Docker のインストール

```powershell
# DockerMsftProvider をインストール
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force

# Docker パッケージをインストール
Install-Package -Name docker -ProviderName DockerMsftProvider -Force

# 再起動
Restart-Computer
```

### 3. Docker サービスの確認と起動

```powershell
# Docker サービスの状態確認
Get-Service docker

# Docker サービスを起動
Start-Service docker

# 自動起動を設定
Set-Service -Name docker -StartupType Automatic
```

---

## Docker の基本コマンド

### バージョン確認

```powershell
docker --version
docker version
```

### Docker の動作確認

```powershell
# Hello World コンテナを実行
docker run hello-world

# Windows コンテナのテスト
docker run mcr.microsoft.com/windows/nanoserver:ltsc2022 cmd /c "echo Hello from Windows Container"
```

### イメージの操作

```powershell
# イメージ一覧を表示
docker images

# イメージをダウンロード
docker pull mcr.microsoft.com/windows/servercore:ltsc2022

# イメージを削除
docker rmi <イメージID>
```

### コンテナの操作

```powershell
# 実行中のコンテナ一覧
docker ps

# すべてのコンテナ一覧（停止中含む）
docker ps -a

# コンテナを起動
docker start <コンテナID>

# コンテナを停止
docker stop <コンテナID>

# コンテナを削除
docker rm <コンテナID>
```

### コンテナの実行

```powershell
# インタラクティブモードで実行
docker run -it mcr.microsoft.com/windows/servercore:ltsc2022 powershell

# バックグラウンドで実行（デタッチモード）
docker run -d --name mycontainer mcr.microsoft.com/windows/servercore:ltsc2022

# ポートマッピング付きで実行
docker run -d -p 8080:80 --name webserver mcr.microsoft.com/windows/servercore/iis:windowsservercore-ltsc2022
```

### ログの確認

```powershell
# コンテナのログを表示
docker logs <コンテナID>

# リアルタイムでログを追跡
docker logs -f <コンテナID>
```

### コンテナに接続

```powershell
# 実行中のコンテナにシェルで接続
docker exec -it <コンテナID> powershell
```

---

## トラブルシューティング

### 「Virtualization support not detected」エラーが表示される場合

Docker Desktop を起動時に以下のエラーが表示される場合があります：

> **Virtualization support not detected**
> Docker Desktop failed to start because virtualisation support wasn't detected.

**原因：**
EC2 インスタンスはデフォルトでネスト仮想化（Nested Virtualization）をサポートしていないため、Docker Desktop が必要とする Hyper-V や WSL2 のバックエンドが動作しません。

**解決方法：**

1. **推奨: Docker Engine を使用する（方法2）**

   Windows Server では Docker Desktop ではなく、Docker Engine を直接インストールすることを推奨します。本ガイドの「方法2: Docker Engine のインストール」を参照してください。Docker Engine は仮想化機能を必要とせず、Windows コンテナをネイティブに実行できます。

2. **ベアメタルインスタンスを使用する**

   どうしても Docker Desktop を使用したい場合は、ネスト仮想化をサポートするベアメタルインスタンス（例: `i3.metal`, `m5.metal` など）を使用してください。ただし、コストが高いため、通常は方法2を推奨します。

3. **BIOS 設定の確認（オンプレミス環境の場合）**

   EC2 ではなくオンプレミス環境でこのエラーが発生した場合は、BIOS で Intel VT-x または AMD-V が有効になっているか確認してください。

---

### Docker サービスが起動しない場合

```powershell
# イベントログを確認
Get-EventLog -LogName Application -Source Docker -Newest 10

# Docker サービスを再起動
Restart-Service docker
```

### ネットワーク関連の問題

```powershell
# Docker ネットワーク一覧
docker network ls

# NAT ネットワークを再作成
docker network rm nat
docker network create -d nat nat
```

### ディスク容量の確認

```powershell
# Docker が使用しているディスク容量を確認
docker system df

# 不要なリソースをクリーンアップ
docker system prune -a
```

---

## AWS EC2 固有の設定

### セキュリティグループの設定

Docker コンテナで公開するポートに応じて、EC2 のセキュリティグループでインバウンドルールを設定してください。

例：
- HTTP (80)
- HTTPS (443)
- カスタムポート (8080 など)

### EBS ボリュームの推奨

Docker イメージとコンテナデータ用に、十分な容量の EBS ボリュームを確保してください。
推奨: 50GB 以上

---

## 参考リンク

- [Docker 公式ドキュメント](https://docs.docker.com/)
- [Microsoft Windows Containers ドキュメント](https://docs.microsoft.com/ja-jp/virtualization/windowscontainers/)
- [AWS EC2 Windows ドキュメント](https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/WindowsGuide/)
