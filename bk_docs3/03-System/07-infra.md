---
title: インフラ構成
description: クラウド環境、デプロイ、監視
sidebar:
  badge: Template
---

:::danger[このドキュメントについて]
このドキュメントはインフラ構成を定義するためのテンプレートです。
クラウド環境、デプロイ戦略、監視体制を明確にしてください。
:::

クラウドプロバイダー選定、環境別インフラ構成(開発・ステージング・本番)、Infrastructure as Code(Terraform/Pulumi)、コンテナ化(Docker/Kubernetes)、CI/CDパイプライン、デプロイ戦略(ブルーグリーン・カナリア)、監視・ロギング、バックアップ・DR、セキュリティ、コスト最適化を定義します。

## クラウドプロバイダー

### 選定: AWS / GCP / Azure

**選定理由**: （理由を記載）

---

## インフラ構成

### 開発環境

```
┌─ Developer Machine ──────────────┐
│  Docker Compose                  │
│  ├─ App Container                │
│  ├─ Database Container           │
│  └─ Cache Container              │
└──────────────────────────────────┘
```

### ステージング環境

```
┌─ Load Balancer ──────────────────┐
│  ALB / Cloud Load Balancing      │
└──────────────────────────────────┘
              ↓
┌─ Application Servers ────────────┐
│  ECS / Cloud Run / App Service   │
│  (Auto Scaling)                  │
└──────────────────────────────────┘
              ↓
┌─ Database ───────────────────────┐
│  RDS / Cloud SQL / Azure DB      │
└──────────────────────────────────┘
              ↓
┌─ Cache ──────────────────────────┐
│  ElastiCache / Memorystore       │
└──────────────────────────────────┘
```

### 本番環境

```
┌─ CDN ────────────────────────────┐
│  CloudFront / Cloud CDN          │
└──────────────────────────────────┘
              ↓
┌─ Load Balancer ──────────────────┐
│  ALB / Cloud Load Balancing      │
└──────────────────────────────────┘
              ↓
┌─ Application Servers ────────────┐
│  Multi-AZ / Multi-Region         │
│  (Auto Scaling)                  │
└──────────────────────────────────┘
              ↓
┌─ Database ───────────────────────┐
│  RDS Multi-AZ / Cloud SQL HA     │
│  (Read Replicas)                 │
└──────────────────────────────────┘
```

---

## Infrastructure as Code（IaC）

### 採用ツール

| ツール | 用途 |
|--------|------|
| Terraform / Pulumi | インフラプロビジョニング |
| Ansible / Chef | 設定管理 |
| Docker Compose | ローカル開発環境 |
| Kubernetes Manifests / Helm | コンテナオーケストレーション |

### ディレクトリ構成

```
/infrastructure
├── terraform/
│   ├── modules/           # 再利用可能なモジュール
│   │   ├── vpc/
│   │   ├── rds/
│   │   └── ecs/
│   ├── environments/      # 環境別設定
│   │   ├── dev/
│   │   ├── staging/
│   │   └── production/
│   └── backend.tf         # Terraformステート管理
├── ansible/               # 設定管理
└── docker/                # Docker設定
```

### Terraform例

```hcl
# environments/production/main.tf
module "vpc" {
  source = "../../modules/vpc"

  environment = "production"
  cidr_block  = "10.0.0.0/16"
}

module "rds" {
  source = "../../modules/rds"

  environment    = "production"
  instance_class = "db.t3.large"
  vpc_id         = module.vpc.vpc_id
}
```

### IaCベストプラクティス

- **バージョン管理**: すべてのインフラコードをGitで管理
- **ステート管理**: Terraformステートはリモートバックエンド（S3, GCS）に保存
- **環境分離**: 環境ごとに独立したステートファイル
- **モジュール化**: 再利用可能なモジュールを作成
- **変更レビュー**: `terraform plan` の結果をPRでレビュー

---

## コンテナ化

### Docker

**Dockerfile例**:
```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .

EXPOSE 3000
CMD ["npm", "start"]
```

### オーケストレーション

- **Kubernetes** (EKS / GKE / AKS)
- **Docker Compose** (開発環境)
- **ECS / Cloud Run** (マネージドコンテナ)

---

## CI/CD

### パイプライン

```
1. コミット・プッシュ
   ↓
2. ビルド
   ↓
3. テスト実行
   ↓
4. コンテナイメージ作成
   ↓
5. レジストリにプッシュ
   ↓
6. デプロイ（ステージング/本番）
```

### ツール

- **CI**: GitHub Actions / GitLab CI / CircleCI
- **CD**: ArgoCD / Flux / AWS CodeDeploy

### デプロイ戦略

#### ブルーグリーンデプロイ

- 新バージョン（Green）を並行稼働
- トラフィックを切り替え
- 問題があれば即座にロールバック

#### カナリアリリース

- 一部のユーザーに新バージョンを配信
- 段階的にトラフィックを増やす
- メトリクスを監視しながら展開

---

## 監視・ロギング

### メトリクス監視

- **ツール**: Prometheus / CloudWatch / Stackdriver
- **監視項目**:
  - CPU使用率
  - メモリ使用率
  - リクエスト数
  - レスポンスタイム
  - エラー率

### ログ管理

- **ツール**: ELK Stack / CloudWatch Logs / Cloud Logging
- **ログレベル**: ERROR, WARN, INFO, DEBUG
- **保持期間**: 30日（本番）

### アラート

| アラート | 閾値 | 通知先 |
|---------|------|--------|
| CPU使用率 | > 80% | Slack / PagerDuty |
| エラー率 | > 5% | Slack / Email |
| レスポンスタイム | > 1000ms | Slack |
| ディスク使用率 | > 85% | Email |

### APM（Application Performance Monitoring）

- **ツール**: New Relic / Datadog / AWS X-Ray
- **監視内容**:
  - トランザクショントレース
  - データベースクエリ
  - 外部API呼び出し

---

## バックアップ・リカバリ

### データベースバックアップ

- **頻度**: 毎日自動バックアップ
- **保持期間**: 7日間
- **スナップショット**: 週次で取得
- **リストア手順**: ドキュメント化

### ディザスタリカバリ（DR）

#### RTO / RPO 目標

| 環境 | RTO | RPO |
|------|-----|-----|
| Production | < 1時間 | < 15分 |
| Staging | < 4時間 | < 1時間 |

#### バックアップ戦略

- **データベース**: 自動バックアップ（日次）+ PITRリカバリ
- **ストレージ**: クロスリージョンレプリケーション
- **インフラ**: IaCによる再構築

#### DR訓練

- **頻度**: 四半期ごと
- **内容**: フルシステムリカバリのシミュレーション

---

## セキュリティ

### ネットワークセキュリティ

- VPC / Virtual Network の分離
- セキュリティグループ / ファイアウォールルール
- プライベートサブネット配置

### アクセス制御

- IAM ロールの最小権限化
- MFA（多要素認証）の有効化
- アクセスキーのローテーション

### データ暗号化

- **保存時**: KMS / Cloud KMS での暗号化
- **通信時**: TLS 1.3
- **バックアップ**: 暗号化済み

---

## コスト最適化

### リソース最適化

- 適切なインスタンスサイズの選択
- オートスケーリングの活用
- 予約インスタンス / Savings Plans

### モニタリング

- コストエクスプローラー / Cost Management
- 予算アラートの設定
- 未使用リソースの定期的なクリーンアップ
