# Claude Sonnet 4.5 Pricing Guide / Claude Sonnet 4.5 料金ガイド

## English

### Overview

Claude Sonnet 4.5 is Anthropic's most intelligent model, available through:
- Anthropic API (Direct)
- Amazon Bedrock
- Claude Code (CLI/IDE Agent)
- Kiro IDE (AWS AI Development Environment)

### Pricing Table

| Tier | Input Tokens | Output Tokens |
|------|--------------|---------------|
| **Standard** | $3 / 1M tokens | $15 / 1M tokens |
| **Extended Context (200K+)** | ~$6 / 1M tokens | ~$22.50 / 1M tokens |
| **Prompt Caching (read)** | $0.30 / 1M tokens | - |
| **Prompt Caching (5-min write)** | $3.75 / 1M tokens | - |
| **Prompt Caching (1-hour write)** | $6 / 1M tokens | - |
| **Batch Processing** | $1.50 / 1M tokens | $7.50 / 1M tokens |

### Claude Code Subscription Plans

| Plan | Monthly Cost | Features |
|------|-------------|----------|
| **Pro** | $20/month ($17 annual) | ~40-80 hours Sonnet 4/week, Claude Code included |
| **Max 5x** | $100/month | 5x higher limits than Pro, Opus 4.5 access |
| **Max 20x** | $200/month | 20x higher limits, unrestricted access |

### Kiro IDE Pricing (AWS)

| Plan | Monthly Cost | Credits Included |
|------|-------------|------------------|
| **Free** | $0 | 50 credits/month |
| **Pro** | $20/month | 1,000 credits/month |
| **Pro+** | $40/month | 3,000 credits/month |
| **Power** | $200/month | Unlimited credits |

*Additional credits: $0.04 each. First-time users receive 500 bonus credits.*

### Cost Savings Options

- **Prompt Caching**: Up to 90% savings on repeated context
- **Batch Processing**: 50% cost reduction for non-real-time requests

### Cost Estimate: 14 Developers × 2 Months

#### Usage Assumptions (per developer per day)

| Parameter | Conservative | Moderate | Heavy |
|-----------|-------------|----------|-------|
| Input tokens | 50K | 150K | 500K |
| Output tokens | 20K | 60K | 200K |
| Working days | 44 | 44 | 44 |

#### Total Cost Estimates (14 developers × 44 days)

| Usage Level | Input Cost | Output Cost | **Total** |
|-------------|------------|-------------|-----------|
| **Conservative** | $92.40 | $184.80 | **$277** |
| **Moderate** | $277.20 | $554.40 | **$832** |
| **Heavy** | $924.00 | $1,848.00 | **$2,772** |

#### With Optimization

| Strategy | Potential Savings | Optimized Cost (Heavy) |
|----------|-------------------|------------------------|
| Prompt Caching (90%) | ~$2,495 | **~$277** |
| Batch Processing (50%) | ~$1,386 | **~$1,386** |

### Summary

For **14 developers over 2 months**:
- Light usage: ~$300/month ($600 total)
- Moderate usage: ~$400-800/month ($800-$1,600 total)
- Heavy usage: ~$1,400/month ($2,800 total)
- With optimization: $300-$1,400 total

---

## 日本語

### 概要

Claude Sonnet 4.5はAnthropicの最も高性能なモデルで、以下のプラットフォームで利用可能です：
- Anthropic API（直接）
- Amazon Bedrock
- Claude Code（CLI/IDEエージェント）
- Kiro IDE（AWS AI開発環境）

### 料金表

| プラン | 入力トークン | 出力トークン |
|--------|-------------|-------------|
| **標準** | $3 / 100万トークン | $15 / 100万トークン |
| **拡張コンテキスト (200K以上)** | 約$6 / 100万トークン | 約$22.50 / 100万トークン |
| **プロンプトキャッシュ（読み取り）** | $0.30 / 100万トークン | - |
| **プロンプトキャッシュ（5分書き込み）** | $3.75 / 100万トークン | - |
| **プロンプトキャッシュ（1時間書き込み）** | $6 / 100万トークン | - |
| **バッチ処理** | $1.50 / 100万トークン | $7.50 / 100万トークン |

### Claude Code サブスクリプションプラン

| プラン | 月額料金 | 特徴 |
|--------|---------|------|
| **Pro** | $20/月（年払い$17） | Sonnet 4週40-80時間相当、Claude Code含む |
| **Max 5x** | $100/月 | Proの5倍の使用量、Opus 4.5アクセス |
| **Max 20x** | $200/月 | 20倍の使用量、無制限アクセス |

### Kiro IDE 料金（AWS）

| プラン | 月額料金 | クレジット |
|--------|---------|-----------|
| **Free** | $0 | 50クレジット/月 |
| **Pro** | $20/月 | 1,000クレジット/月 |
| **Pro+** | $40/月 | 3,000クレジット/月 |
| **Power** | $200/月 | 無制限 |

*追加クレジット: $0.04/個。初回登録で500ボーナスクレジット付与。*

### コスト削減オプション

- **プロンプトキャッシュ**: 繰り返しコンテキストで最大90%削減
- **バッチ処理**: 非リアルタイムリクエストで50%削減

### コスト見積もり: 14人の開発者 × 2ヶ月

#### 使用量の前提（開発者1人あたり1日）

| パラメータ | 控えめ | 中程度 | 多め |
|-----------|--------|--------|------|
| 入力トークン | 5万 | 15万 | 50万 |
| 出力トークン | 2万 | 6万 | 20万 |
| 稼働日数 | 44日 | 44日 | 44日 |

#### 総コスト見積もり（14人の開発者 × 44日）

| 使用レベル | 入力コスト | 出力コスト | **合計** |
|------------|------------|------------|----------|
| **控えめ** | $92.40 | $184.80 | **$277** |
| **中程度** | $277.20 | $554.40 | **$832** |
| **多め** | $924.00 | $1,848.00 | **$2,772** |

#### 最適化適用時

| 戦略 | 削減可能額 | 最適化後コスト（多め） |
|------|-----------|----------------------|
| プロンプトキャッシュ (90%) | 約$2,495 | **約$277** |
| バッチ処理 (50%) | 約$1,386 | **約$1,386** |

### まとめ

**14人の開発者が2ヶ月間使用する場合**:
- 控えめな使用: 約$300/月（合計$600）
- 中程度の使用: 約$400-800/月（合計$800-$1,600）
- 多めの使用: 約$1,400/月（合計$2,800）
- 最適化適用時: 合計$300-$1,400

---

## References / 参考リンク

- [Anthropic Pricing](https://www.anthropic.com/pricing)
- [Claude Code](https://www.anthropic.com/claude-code)
- [Claude Subscription Plans](https://claude.com/pricing)
- [Amazon Bedrock Pricing](https://aws.amazon.com/bedrock/pricing/)
- [Kiro IDE](https://kiro.dev/)
- [Kiro Pricing](https://kiro.dev/pricing/)
- [Claude Documentation](https://docs.anthropic.com/)

---

*Last updated / 最終更新: 2025-12-21*
