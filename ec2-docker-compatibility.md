# EC2 における Docker 互換性ガイド

## 概要

このドキュメントは、AWS EC2 インスタンス上での Docker の互換性についてまとめたものです。

> **⚠️ 重要な結論**
>
> **EC2 で Docker を使用する場合は、Linux インスタンスを使用してください。**
>
> Windows EC2 インスタンスでの Docker は動作しません。

---

## 互換性サマリー

| プラットフォーム | EC2 対応状況 | テスト済みインスタンス | 備考 |
|-----------------|-------------|---------------------|------|
| **Linux Docker** | ✅ 動作する | 一般的なインスタンス | 推奨 |
| **Windows Docker** | ❌ 動作しない | t3, c5 | 使用不可 |

---

## Windows Docker が EC2 で動作しない理由

### 検証結果

以下のインスタンスタイプで Windows Docker の動作を検証しましたが、いずれも正常に動作しませんでした：

- **t3 シリーズ**: 動作しない ❌
- **c5 シリーズ**: 動作しない ❌

### 技術的な理由

#### 1. ネスト仮想化の非対応

EC2 インスタンスはデフォルトで**ネスト仮想化（Nested Virtualization）をサポートしていません**。

Docker Desktop は以下のいずれかを必要としますが、これらは EC2 環境では利用できません：
- **WSL2 (Windows Subsystem for Linux 2)** - 仮想化機能が必要
- **Hyper-V** - ネスト仮想化が必要

#### 2. 「Virtualization support not detected」エラー

Docker Desktop を起動すると、以下のエラーが表示されます：

> **Virtualization support not detected**
>
> Docker Desktop failed to start because virtualisation support wasn't detected.

これは EC2 の仮想化レイヤーが、ゲスト OS 内でさらに仮想化を行うことを許可していないためです。

#### 3. Docker Engine (Windows Server) も制限あり

Windows Server 向けの Docker Engine（Containers 機能を使用）も、EC2 環境では制限があり、期待通りに動作しない場合があります。

---

## 推奨: Linux EC2 で Docker を使用する

Linux EC2 インスタンスでは Docker が問題なく動作します。

### Linux Docker のメリット

- ✅ ネスト仮想化が不要
- ✅ ネイティブコンテナサポート
- ✅ 安定した動作
- ✅ コミュニティサポートが充実
- ✅ 豊富な Docker イメージが利用可能

### Linux EC2 での Docker インストール手順

#### Amazon Linux 2023 / Amazon Linux 2

```bash
# パッケージを更新
sudo yum update -y

# Docker をインストール
sudo yum install -y docker

# Docker サービスを起動
sudo systemctl start docker

# 自動起動を有効化
sudo systemctl enable docker

# ec2-user を docker グループに追加（sudo なしで docker コマンドを実行可能に）
sudo usermod -aG docker ec2-user

# グループ変更を反映（再ログインまたは以下を実行）
newgrp docker
```

#### Ubuntu

```bash
# パッケージを更新
sudo apt update

# 必要なパッケージをインストール
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common

# Docker の公式 GPG キーを追加
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Docker リポジトリを追加
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Docker をインストール
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io

# ユーザーを docker グループに追加
sudo usermod -aG docker $USER

# グループ変更を反映
newgrp docker
```

### Docker の動作確認

```bash
# バージョン確認
docker --version

# 動作テスト
docker run hello-world

# コンテナ一覧
docker ps -a
```

---

## どうしても Windows コンテナが必要な場合

Windows コンテナが必要な特殊なケースでは、以下の選択肢を検討してください：

### 1. ベアメタルインスタンスを使用する

ネスト仮想化をサポートするベアメタルインスタンスを使用：
- `i3.metal`
- `m5.metal`
- `c5.metal`
- その他の `.metal` インスタンス

> **⚠️ 注意**: ベアメタルインスタンスは非常に高コストです。

### 2. オンプレミス環境を使用する

物理サーバーで Hyper-V を有効にした Windows Server 環境であれば、Docker が正常に動作します。

### 3. Azure を検討する

Microsoft Azure は Windows コンテナのサポートが充実しており、Windows Docker ワークロードに適している場合があります。

---

## まとめ

| 項目 | 推奨 |
|------|------|
| EC2 で Docker を使う場合 | **Linux インスタンスを使用** |
| Windows EC2 で Docker | **使用不可**（t3, c5 で検証済み） |
| Windows コンテナが必須 | ベアメタル or オンプレミス or Azure を検討 |

---

## 関連ドキュメント

- [Docker 公式ドキュメント](https://docs.docker.com/)
- [Amazon Linux での Docker インストール](https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/developerguide/create-container-image.html)
- [AWS EC2 インスタンスタイプ](https://aws.amazon.com/jp/ec2/instance-types/)
