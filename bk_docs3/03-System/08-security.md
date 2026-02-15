---
title: セキュリティ要件
description: OWASP対策、入力バリデーション
sidebar:
  badge: Template
---

:::danger[このドキュメントについて]
このドキュメントはセキュリティ要件を定義するためのテンプレートです。
OWASP対策、データ保護、脆弱性対策を明確にしてください。
:::

OWASP Top 10対策、入力バリデーション・サニタイゼーション、認証・認可のセキュリティ、APIセキュリティ(レート制限・CORS・セキュリティヘッダー)、データ保護(PII・GDPR対応)、セキュリティテスト(SAST・DAST・脆弱性スキャン)、インシデント対応フロー、コンプライアンスを定義します。

## OWASP Top 10 対策

### A01: アクセス制御の不備

**対策**:
- 最小権限の原則
- デフォルトで拒否
- セッション管理の適切な実装

### A02: 暗号化の失敗

**対策**:
- HTTPS の強制
- 保存データの暗号化
- 強力な暗号化アルゴリズムの使用（AES-256）

### A03: インジェクション

**対策**:
- プリペアドステートメントの使用
- ORM の活用
- 入力バリデーション

### A04: 安全でない設計

**対策**:
- セキュアな設計パターンの採用
- 脅威モデリングの実施
- コードレビューの徹底

### A05: セキュリティの設定ミス

**対策**:
- デフォルト認証情報の変更
- 不要なサービスの無効化
- セキュリティヘッダーの設定

### A06: 脆弱で古いコンポーネント

**対策**:
- 依存関係の定期的な更新
- 脆弱性スキャンの自動化（Dependabot / Snyk）
- バージョン管理

### A07: 識別と認証の失敗

**対策**:
- 多要素認証（MFA）
- 強力なパスワードポリシー
- セッションタイムアウト

### A08: ソフトウェアとデータの整合性の不備

**対策**:
- 署名検証
- コード署名
- CI/CDパイプラインのセキュア化

### A09: セキュリティログとモニタリングの不備

**対策**:
- 包括的なロギング
- リアルタイムアラート
- ログの改ざん防止

### A10: サーバーサイドリクエストフォージェリ（SSRF）

**対策**:
- URLホワイトリスト
- ネットワーク分離
- 入力検証

---

## 入力バリデーション

### フロントエンド

```typescript
// Zod を使用した例
import { z } from 'zod';

const userSchema = z.object({
  email: z.string().email('有効なメールアドレスを入力してください'),
  password: z.string().min(8, 'パスワードは8文字以上必要です'),
  name: z.string().min(1, '名前を入力してください').max(50),
});

// バリデーション実行
const result = userSchema.safeParse(formData);
if (!result.success) {
  console.error(result.error);
}
```

### バックエンド

```typescript
// express-validator を使用した例
import { body, validationResult } from 'express-validator';

app.post('/users',
  body('email').isEmail(),
  body('password').isLength({ min: 8 }),
  body('name').isLength({ min: 1, max: 50 }),
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }
    // 処理続行
  }
);
```

### サニタイゼーション

- HTMLタグの除去
- SQLインジェクション対策
- XSS対策

---

## 認証・認可のセキュリティ

### パスワードセキュリティ

- **ハッシュ化**: bcrypt / Argon2
- **ソルト**: ランダムソルトの使用
- **ペッパー**: アプリケーション共通の秘密鍵

### トークンセキュリティ

- **JWT署名**: HS256 / RS256
- **有効期限**: 短い有効期限（15分）
- **リフレッシュトークン**: ローテーション

### セッションセキュリティ

- **HTTPOnly Cookie**: XSS対策
- **Secure フラグ**: HTTPS限定
- **SameSite**: CSRF対策

---

## APIセキュリティ

### レート制限

```typescript
import rateLimit from 'express-rate-limit';

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15分
  max: 100, // 最大100リクエスト
  message: 'リクエストが多すぎます。しばらくしてから再試行してください。',
});

app.use('/api/', limiter);
```

### CORS設定

```typescript
import cors from 'cors';

app.use(cors({
  origin: 'https://example.com',
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
}));
```

### セキュリティヘッダー

```typescript
import helmet from 'helmet';

app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "'unsafe-inline'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", 'data:', 'https:'],
    },
  },
}));
```

---

## データ保護

### 個人情報（PII）

- **暗号化**: 保存時・通信時
- **アクセス制御**: 必要最小限
- **データマスキング**: ログ出力時

### GDPR / プライバシー法対応

- **データ削除**: ユーザーからの削除要求に対応
- **データエクスポート**: データポータビリティ
- **同意管理**: Cookie同意バナー

---

## セキュリティテスト

### 静的解析（SAST）

- **ツール**: SonarQube / ESLint Security Plugin
- **実行タイミング**: コミット時・PR時

### 動的解析（DAST）

- **ツール**: OWASP ZAP / Burp Suite
- **実行タイミング**: ステージング環境

### 脆弱性スキャン

- **依存関係**: npm audit / Snyk / Dependabot
- **コンテナ**: Trivy / Clair
- **頻度**: 毎日自動実行

### ペネトレーションテスト

- **頻度**: 年1回
- **対象**: 本番環境相当の環境
- **実施者**: 第三者機関

---

## インシデント対応

### セキュリティインシデント対応フロー

```
1. 検知・報告
   ↓
2. 初期対応（影響範囲の特定）
   ↓
3. 封じ込め（攻撃の遮断）
   ↓
4. 根絶（脆弱性の修正）
   ↓
5. 復旧（サービスの再開）
   ↓
6. 事後分析（再発防止策の策定）
```

### 連絡体制

- **セキュリティチーム**: security@example.com
- **緊急連絡先**: オンコール担当者
- **エスカレーション**: CTO / セキュリティ責任者

---

## コンプライアンス

### セキュリティ基準

- ISO 27001
- SOC 2 Type II
- PCI DSS（決済機能がある場合）

### 定期的なレビュー

- セキュリティポリシーの年次レビュー
- アクセス権限の四半期レビュー
- インシデント対応手順の半期レビュー
