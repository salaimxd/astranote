---
title: 開発ガイド
description: 環境構築と開発の進め方
sidebar:
  badge: Template
---

:::danger[このドキュメントについて]
このセクションでは開発プロセスとガイドラインを定義します。
ブランチ戦略、進捗管理、テスト戦略、CI/CDパイプラインを明確にしてください。
:::

プロジェクトの**開発環境セットアップと開発進捗**を管理するセクションです。ブランチ戦略(Git Flow)、開発進捗(タスク管理・マイルストーン)、テスト戦略(ユニット・統合・E2E)、CI/CDパイプライン(ビルド・テスト・デプロイ自動化)、必要な環境、コーディング規約を含みます。

## コンテンツ

### 開発プロセスとガイド

| ドキュメント | 目的 | 主な内容 |
|------------|------|---------|
| [ブランチ戦略](/04-development/01-branch-strategy/) | Git運用ルールを定義 | ブランチモデル、命名規則、マージフロー、コミット規約 |
| [開発進捗](/04-development/02-progress/) | タスク管理方法を定義 | マイルストーン、スプリント計画、Issue管理、進捗トラッキング |
| [テスト戦略](/04-development/03-testing/) | テスト方針を定義 | テストピラミッド、カバレッジ目標、テストツール、自動化戦略 |
| [CI/CDパイプライン](/04-development/04-cicd-pipeline/) | 自動化フローを定義 | ビルド、テスト、デプロイの自動化、品質ゲート |

---

## 必要な環境

| ツール | バージョン | 用途 |
|--------|-----------|------|
| Node.js | 20+ | フロントエンド・バックエンド |
| npm / yarn / pnpm | 最新 | パッケージマネージャー |
| Docker | 24+ | コンテナ実行 |
| Git | 2.40+ | バージョン管理 |

---

## クイックスタート

### 1. リポジトリのクローン

```bash
git clone https://github.com/your-org/your-project.git
cd your-project
```

### 2. 依存関係のインストール

```bash
npm install
# または
yarn install
# または
pnpm install
```

### 3. 環境変数の設定

```bash
cp .env.example .env
# .envファイルを編集
```

### 4. Dockerで依存サービスを起動

```bash
docker compose up -d
```

### 5. 開発サーバーの起動

```bash
npm run dev
```

ブラウザで `http://localhost:3000` にアクセス

---

## ディレクトリ構成

```
/your-project
├── src/                # ソースコード
│   ├── components/     # コンポーネント
│   ├── pages/          # ページ
│   ├── services/       # API通信
│   ├── stores/         # 状態管理
│   └── utils/          # ユーティリティ
├── public/             # 静的ファイル
├── tests/              # テストコード
├── docs/               # ドキュメント
├── .env.example        # 環境変数テンプレート
├── docker-compose.yml  # Docker設定
└── package.json        # 依存関係
```

---

## 開発ワークフロー

### 1. Issue の作成

GitLab/GitHub で Issue を作成し、タスクを明確化

### 2. ブランチの作成

```bash
git checkout -b feature/#<issue>-description
```

### 3. 開発

- コードを書く
- テストを書く
- コミット

### 4. プルリクエスト / マージリクエスト

- `develop` ブランチへMR/PRを作成
- 説明文に `Closes #<issue>` を記載
- レビューを依頼

### 5. レビュー・マージ

- レビューを受ける
- 修正が必要な場合は対応
- 承認されたらマージ

---

## コーディング規約

### 一般的なルール

- **命名規則**: camelCase（変数・関数）、PascalCase（クラス・コンポーネント）
- **インデント**: スペース2つ
- **セミコロン**: 使用する
- **引用符**: シングルクォート

### Linter / Formatter

```bash
# コードフォーマット
npm run format

# Lintチェック
npm run lint

# Lint自動修正
npm run lint:fix
```

### コミットメッセージ

[Conventional Commits](https://www.conventionalcommits.org/) に従う

```
feat(component): 新機能の追加 (#123)
fix(api): バグ修正 (#124)
docs(readme): ドキュメント更新
style(button): スタイル調整
refactor(auth): リファクタリング
test(user): テスト追加
chore(deps): 依存関係更新
```

---

## テスト

### ユニットテスト

```bash
npm run test
```

### E2Eテスト

```bash
npm run test:e2e
```

### カバレッジ

```bash
npm run test:coverage
```

---

## デバッグ

### ブラウザデバッグ

- Chrome DevTools
- React DevTools / Vue DevTools

### VSCode デバッグ設定

`.vscode/launch.json` でデバッグ設定を追加

---

## トラブルシューティング

### 1. 依存関係の競合

**症状**: `npm install` が失敗する

**原因**:
- package-lock.jsonの不整合
- Node.jsバージョンの不一致

**解決方法**:
```bash
# Node.jsバージョン確認
node -v  # 必要: 20.x

# クリーンインストール
rm -rf node_modules package-lock.json
npm install

# または volta/nvm でバージョン切り替え
volta install node@20
```

---

### 2. データベース接続エラー

**症状**: `Error: connect ECONNREFUSED 127.0.0.1:5432`

**原因**: PostgreSQLコンテナが起動していない

**解決方法**:
```bash
# コンテナ状態確認
docker compose ps

# コンテナ起動
docker compose up -d postgres

# ログ確認
docker compose logs postgres

# 接続テスト
psql -h localhost -U myuser -d mydb
```

---

### 3. ポート競合

**症状**: `Error: listen EADDRINUSE: address already in use :::3000`

**解決方法**:
```bash
# プロセス確認（Mac/Linux）
lsof -i :3000

# プロセス終了
kill -9 <PID>

# または別のポートで起動
PORT=3001 npm run dev
```

---

### 4. Dockerディスク容量不足

**症状**: `no space left on device`

**解決方法**:
```bash
# 未使用リソース削除
docker system prune -a --volumes

# ディスク使用量確認
docker system df
```

---

### 5. 環境変数が読み込まれない

**症状**: アプリケーションが環境変数を認識しない

**解決方法**:
```bash
# .env ファイルの存在確認
ls -la .env

# .env.example から .env をコピー
cp .env.example .env

# .env ファイルを編集
vim .env  # または code .env

# 開発サーバーを再起動
npm run dev
```

---

## デバッグテクニック

### Node.js アプリケーション

**VSCode launch.json**:
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Debug Server",
      "program": "${workspaceFolder}/src/index.ts",
      "preLaunchTask": "tsc: build",
      "outFiles": ["${workspaceFolder}/dist/**/*.js"],
      "envFile": "${workspaceFolder}/.env"
    }
  ]
}
```

### ブラウザデバッグ

**Chrome DevTools**:
1. F12でDevToolsを開く
2. Sources タブでブレークポイント設定
3. Network タブでAPI通信を確認
4. Console タブでログ確認

---

## 推奨VSCode拡張機能

- ESLint
- Prettier
- GitLens
- Docker
- Thunder Client（APIテスト）
