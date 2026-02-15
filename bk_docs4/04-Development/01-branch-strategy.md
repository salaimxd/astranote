---
title: ブランチ戦略
description: Git運用ルール
sidebar:
  badge: Template
---

:::danger[このドキュメントについて]
このドキュメントはGit運用ルールを定義するためのテンプレートです。
ブランチモデル、命名規則、マージフローを明確にしてください。
:::

Git Flow ベースのブランチ戦略、ブランチ種類(main/develop/release/feature/fix/hotfix)と命名規則、ブランチ作成フロー、マージルール、Conventional Commitsによるコミットメッセージ規約、リリースフロー、Issue連携、ブランチ保護設定を定義します。

## ブランチモデル

### Git Flow ベース

プロジェクトでは Git Flow をベースとしたブランチ戦略を採用します。

```
main (本番)
  ↑
release/v* (リリース準備)
  ↑
develop (開発統合)
  ↑
feature/#<issue>-* (機能開発)
fix/#<issue>-* (バグ修正)
```

---

## ブランチ種類

### main ブランチ

- **用途**: 本番環境にデプロイされる安定版
- **マージ元**: `release/*` ブランチのみ
- **保護**: 直接コミット禁止、レビュー必須

### develop ブランチ

- **用途**: 開発中の機能を統合
- **マージ元**: `feature/*`, `fix/*` ブランチ
- **保護**: 直接コミット禁止、レビュー必須

### release/* ブランチ

- **用途**: リリース準備とバグ修正
- **命名規則**: `release/v1.0.0`
- **作成タイミング**: マイルストーン完了時
- **マージ先**: `main` と `develop`

### feature/* ブランチ

- **用途**: 新機能開発
- **命名規則**: `feature/#<issue>-short-description`
- **作成元**: `develop`
- **マージ先**: `develop`

**例**:
```
feature/#12-user-authentication
feature/#23-payment-integration
```

### fix/* ブランチ

- **用途**: バグ修正
- **命名規則**: `fix/#<issue>-short-description`
- **作成元**: `develop`
- **マージ先**: `develop`

**例**:
```
fix/#45-login-error
fix/#67-memory-leak
```

### hotfix/* ブランチ

- **用途**: 本番環境の緊急修正
- **命名規則**: `hotfix/#<issue>-short-description`
- **作成元**: `main`
- **マージ先**: `main` と `develop`

**例**:
```
hotfix/#89-critical-security-patch
```

---

## ブランチ作成フロー

### 機能開発

```bash
# 1. develop ブランチを最新化
git checkout develop
git pull origin develop

# 2. feature ブランチを作成
git checkout -b feature/#12-user-auth

# 3. 開発・コミット
git add .
git commit -m "feat(auth): ユーザー認証機能を追加 (#12)"

# 4. リモートにプッシュ
git push origin feature/#12-user-auth

# 5. PR/MR を作成（GitHub/GitLab上で）
```

### バグ修正

```bash
# 1. develop ブランチを最新化
git checkout develop
git pull origin develop

# 2. fix ブランチを作成
git checkout -b fix/#45-login-error

# 3. 修正・コミット
git add .
git commit -m "fix(auth): ログインエラーを修正 (#45)"

# 4. リモートにプッシュ
git push origin fix/#45-login-error

# 5. PR/MR を作成
```

---

## マージルール

### PR/MR 要件

- [ ] コードレビュー完了（最低1名の承認）
- [ ] CI/CDパイプラインが成功
- [ ] コンフリクトが解決済み
- [ ] Issue番号が説明文に記載（`Closes #<issue>`）

### マージ方法

#### Squash and Merge（推奨）

- 複数のコミットを1つにまとめる
- コミット履歴がクリーンに保たれる

#### Merge Commit

- すべてのコミット履歴を保持
- 詳細な履歴が必要な場合

#### Rebase and Merge

- 履歴を線形に保つ
- コンフリクトが少ない場合

---

## コミットメッセージ規約

### Conventional Commits

```
<type>(<scope>): <subject> (#<issue>)

<body>

<footer>
```

### Type 一覧

| Type | 説明 |
|------|------|
| `feat` | 新機能 |
| `fix` | バグ修正 |
| `docs` | ドキュメント |
| `style` | コードスタイル（空白、フォーマット） |
| `refactor` | リファクタリング |
| `test` | テスト追加・修正 |
| `chore` | ビルド・補助ツール |
| `perf` | パフォーマンス改善 |
| `ci` | CI設定 |

### 例

```bash
feat(auth): JWT認証機能を追加 (#12)

- JWTトークンの生成・検証
- ミドルウェアの実装
- ログイン/ログアウトAPI

Closes #12

---

fix(api): ユーザー取得APIのバグを修正 (#45)

nullチェックが不足していたため、存在しないユーザーIDで
500エラーが発生していた問題を修正

Closes #45
```

---

## リリースフロー

### 1. Release ブランチの作成

```bash
git checkout develop
git pull origin develop
git checkout -b release/v1.0.0
```

### 2. バージョン番号の更新

```bash
# package.json のバージョンを更新
npm version 1.0.0

git add package.json
git commit -m "chore(release): v1.0.0"
```

### 3. リリース準備

- 最終テスト
- ドキュメント更新
- CHANGELOG.md 更新

### 4. main へマージ

```bash
git checkout main
git merge --no-ff release/v1.0.0
git tag -a v1.0.0 -m "Release v1.0.0"
git push origin main --tags
```

### 5. develop へマージ

```bash
git checkout develop
git merge --no-ff release/v1.0.0
git push origin develop
```

### 6. Release ブランチの削除

```bash
git branch -d release/v1.0.0
git push origin --delete release/v1.0.0
```

---

## Issue連携

### Issue番号の記載

MR/PR の説明文に以下を記載すると、マージ時に自動でIssueがクローズされます：

```markdown
Closes #12
Closes #23, #45
Fixes #67
Resolves #89
```

### コミットメッセージでの参照

```bash
git commit -m "feat(auth): ログイン機能を追加 (#12)"
```

---

## ブランチ保護設定

### GitHub/GitLab 設定

#### main ブランチ

- Require pull request reviews before merging (1+ approvals)
- Require status checks to pass before merging
- Require branches to be up to date before merging
- Include administrators

#### develop ブランチ

- Require pull request reviews before merging (1+ approvals)
- Require status checks to pass before merging

---

## 運用ルール

### ブランチの寿命

- **Feature/Fix ブランチ**: マージ後すぐ削除
- **Release ブランチ**: リリース後削除
- **Hotfix ブランチ**: マージ後すぐ削除

### コンフリクト解決

```bash
# 1. develop を最新化
git checkout develop
git pull origin develop

# 2. feature ブランチで rebase
git checkout feature/#12-user-auth
git rebase develop

# 3. コンフリクトを解決
# ファイルを編集

# 4. 続行
git add .
git rebase --continue

# 5. 強制プッシュ（注意）
git push origin feature/#12-user-auth --force-with-lease
```
