---
title: 認証・認可設計
description: 認証方式、セッション管理
sidebar:
  badge: Template
---

:::danger[このドキュメントについて]
このドキュメントは認証・認可の仕組みを定義するためのテンプレートです。
認証方式、トークン管理、権限設計を明確にしてください。
:::

JWT認証方式、トークンの種類と構造、認証フロー(ログイン・トークンリフレッシュ・ログアウト)、RBAC権限設計、トークン保存戦略、セキュリティ対策(CSRF・XSS)、ソーシャルログイン(OAuth 2.0)、パスワードポリシーを定義します。

## 認証方式

### JWT（JSON Web Token）

ステートレスな認証を実現するためにJWTを使用します。

#### トークンの種類

| トークン | 有効期限 | 用途 |
|---------|---------|------|
| アクセストークン | 15分 | API認証 |
| リフレッシュトークン | 7日 | アクセストークン再発行 |

#### トークン構造

```
Header.Payload.Signature
```

**Payload例**:
```json
{
  "sub": "user_123",
  "email": "user@example.com",
  "role": "user",
  "iat": 1704067200,
  "exp": 1704068100
}
```

---

## 認証フロー

### ログイン

```
1. ユーザーがメール/パスワードを送信
   ↓
2. サーバーが認証情報を検証
   ↓
3. アクセストークン・リフレッシュトークンを発行
   ↓
4. クライアントがトークンを保存
```

### トークンリフレッシュ

```
1. アクセストークンの有効期限切れ
   ↓
2. リフレッシュトークンを使用して新しいアクセストークンを取得
   ↓
3. 新しいアクセストークンで再リクエスト
```

### ログアウト

```
1. クライアントがトークンを削除
   ↓
2. リフレッシュトークンをサーバー側で無効化（オプション）
```

---

## 認可（Authorization）

### RBAC（Role-Based Access Control）

#### ロール定義

| ロール | 権限 |
|--------|------|
| `admin` | すべての操作 |
| `user` | 自分のリソースの操作 |
| `guest` | 読み取りのみ |

#### 権限チェック

```typescript
// ミドルウェア例
function requireRole(allowedRoles: string[]) {
  return (req, res, next) => {
    const userRole = req.user.role;
    if (allowedRoles.includes(userRole)) {
      next();
    } else {
      res.status(403).json({ error: 'Forbidden' });
    }
  };
}

// 使用例
app.delete('/users/:id', requireRole(['admin']), deleteUser);
```

---

## トークン保存

### フロントエンド

#### オプション1: LocalStorage

**メリット**:
- 実装が簡単
- JavaScriptから自由にアクセス可能

**デメリット**:
- XSS攻撃に脆弱

#### オプション2: HTTPOnly Cookie

**メリット**:
- XSS攻撃から保護
- ブラウザが自動的に送信

**デメリット**:
- CSRF対策が必要

**推奨**: HTTPOnly Cookie + CSRF トークン

---

## セキュリティ対策

### CSRF対策

- CSRF トークンの使用
- SameSite Cookie 属性の設定

### XSS対策

- HTTPOnly Cookie の使用
- Content Security Policy (CSP) の設定
- 入力値のサニタイゼーション

### トークン漏洩対策

- HTTPS の強制
- トークンの短い有効期限
- リフレッシュトークンのローテーション

---

## ソーシャルログイン

### 対応プロバイダー

- Google
- GitHub
- Twitter/X

### OAuth 2.0 フロー

```
1. ユーザーがソーシャルログインボタンをクリック
   ↓
2. プロバイダーの認証ページにリダイレクト
   ↓
3. ユーザーが認証を許可
   ↓
4. コールバックURLにリダイレクト
   ↓
5. 認可コードをアクセストークンに交換
   ↓
6. ユーザー情報を取得してアカウントを作成/更新
   ↓
7. 自社のJWTトークンを発行
```

---

## パスワードポリシー

### 要件

- 最小長: 8文字
- 大文字・小文字・数字を含む
- 特殊文字を含む（推奨）

### ハッシュ化

- **アルゴリズム**: bcrypt / Argon2
- **ソルトラウンド**: 10（bcrypt の場合）

```typescript
// パスワードハッシュ化例
const hashedPassword = await bcrypt.hash(password, 10);

// パスワード検証例
const isValid = await bcrypt.compare(password, hashedPassword);
```
