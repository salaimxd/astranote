---
title: CI/CDパイプライン
description: 継続的インテグレーション・デプロイメント戦略
sidebar:
  badge: Template
---

:::danger[このドキュメントについて]
このドキュメントはCI/CDパイプラインを定義するためのテンプレートです。
ビルド、テスト、デプロイの自動化フローを明確にしてください。
:::

パイプライン全体フロー、環境戦略(Development・Staging・Production)、GitHub Actions設定例、デプロイメント戦略(Blue-Green・カナリア)、ロールバック戦略、モニタリング・アラート、セキュリティスキャン、パフォーマンステスト、コスト最適化を定義します。

## パイプライン概要

```
┌─ Push to Feature Branch ────────────────────────┐
│  1. Lint & Format Check                         │
│  2. Unit Tests                                  │
│  3. Build                                       │
└─────────────────────────────────────────────────┘
                    ↓
┌─ Pull Request to develop ───────────────────────┐
│  1. 上記すべて                                   │
│  2. Integration Tests                           │
│  3. E2E Tests                                   │
│  4. Security Scan                               │
│  5. Code Coverage Check                         │
└─────────────────────────────────────────────────┘
                    ↓
┌─ Merge to develop ──────────────────────────────┐
│  1. Auto Deploy to Development Environment      │
│  2. Smoke Tests                                 │
└─────────────────────────────────────────────────┘
                    ↓
┌─ Merge to release/* ────────────────────────────┐
│  1. Deploy to Staging Environment               │
│  2. Full E2E Tests                              │
│  3. Performance Tests                           │
│  4. Manual QA                                   │
└─────────────────────────────────────────────────┘
                    ↓
┌─ Merge to main ─────────────────────────────────┐
│  1. Create Release Tag                          │
│  2. Deploy to Production                        │
│  3. Smoke Tests                                 │
│  4. Health Check                                │
└─────────────────────────────────────────────────┘
```

---

## 環境戦略

### 環境構成

| 環境 | ブランチ | URL | 用途 |
|------|---------|-----|------|
| Development | develop | dev.example.com | 開発統合環境 |
| Staging | release/* | staging.example.com | 本番前検証 |
| Production | main | example.com | 本番環境 |

### 環境変数管理

**ツール**: [AWS Secrets Manager / GCP Secret Manager / HashiCorp Vault]

**構成**:
```
.env.development
.env.staging
.env.production
```

**セキュリティルール**:
- 秘匿情報は Git にコミットしない
- Secret Manager で一元管理
- 環境ごとにアクセス権限を分離

---

## GitHub Actions 設定例

### PR チェック

```yaml
# .github/workflows/pr-check.yml
name: PR Check

on:
  pull_request:
    branches: [develop]

jobs:
  lint-and-test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Lint
        run: npm run lint

      - name: Type check
        run: npm run type-check

      - name: Unit tests
        run: npm run test:coverage

      - name: Build
        run: npm run build

      - name: E2E tests
        run: npm run test:e2e
```

### デプロイメント

```yaml
# .github/workflows/deploy-production.yml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm ci

      - name: Build
        run: npm run build
        env:
          NODE_ENV: production

      - name: Deploy
        run: npm run deploy:production

      - name: Health check
        run: |
          response=$(curl -s -o /dev/null -w "%{http_code}" https://example.com/health)
          if [ "$response" != "200" ]; then
            echo "Health check failed"
            exit 1
          fi
```

---

## デプロイメント戦略

### Blue-Green デプロイメント

**フロー**:
1. 新バージョン（Green）を並行環境にデプロイ
2. Green環境でスモークテスト実行
3. ロードバランサーのトラフィックをGreenに切り替え
4. 問題なければBlue環境を削除、問題あればBlueに戻す

**メリット**:
- ダウンタイムゼロ
- 即座にロールバック可能

### カナリアデプロイメント

**フロー**:
1. 新バージョンを一部ユーザー（5%）に展開
2. メトリクスを監視（エラー率、レスポンスタイム）
3. 問題なければ段階的に拡大（10% → 50% → 100%）
4. 問題があれば即座にロールバック

---

## ロールバック戦略

### 自動ロールバック条件

以下の条件で自動的にロールバック:
- デプロイ後のヘルスチェック失敗
- エラー率が閾値を超過
- レスポンスタイムが閾値を超過

### 手動ロールバック手順

```bash
# 1. 前のバージョンタグを確認
git tag -l

# 2. 前のバージョンにロールバック
git checkout v1.2.3

# 3. デプロイ
npm run deploy:production

# 4. ヘルスチェック
npm run test:smoke -- --env=production
```

---

## モニタリング・アラート

### デプロイメントメトリクス

| メトリクス | 目標値 | アラート条件 |
|-----------|--------|-------------|
| デプロイ成功率 | > 95% | < 90% |
| デプロイ時間 | < 10分 | > 15分 |
| ロールバック率 | < 5% | > 10% |
| MTTR（平均復旧時間） | < 30分 | > 1時間 |

### 本番環境モニタリング

**ツール**: [Datadog / New Relic / CloudWatch]

**監視項目**:
- エラー率
- レスポンスタイム（P50, P95, P99）
- スループット
- CPU / メモリ使用率
- データベース接続数

---

## セキュリティ

### SAST（静的解析）

**ツール**: [SonarQube / Snyk / GitHub Code Scanning]

```yaml
- name: SAST
  run: npm run security:sast
```

### 依存関係スキャン

```yaml
- name: Dependency check
  run: npm audit --audit-level=high
```

---

## パフォーマンステスト

### 負荷テスト

リリース前にステージング環境で実施:

```bash
k6 run --vus 1000 --duration 5m tests/load/api.js
```

**合格基準**:
- P95レスポンスタイム < 200ms
- エラー率 < 0.1%
- スループット > 1000 req/sec

---

## コスト最適化

### CI時間短縮

- **並列実行**: ジョブを並列化
- **キャッシュ活用**: 依存関係・ビルドキャッシュ
- **選択的テスト**: 変更ファイルに関連するテストのみ実行

### インフラコスト削減

- Development環境: 夜間・週末停止
- Staging環境: 必要時のみ起動
- Spot Instances活用
