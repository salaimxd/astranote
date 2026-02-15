# プロジェクト共通ガイドライン

このドキュメントは、プロジェクト全体の共通方針を定めています。

**重要：サブディレクトリ内で作業する場合は、必ずそのディレクトリの CLAUDE.md を Read ツールで読み込んでから作業を開始すること。** ルートの共通方針に加え、各ディレクトリ固有のルール・規約が定義されています。

## 📋 サブディレクトリ別ガイドライン

| 作業対象 | 読むべきファイル | 内容 |
|----------|------------------|------|
| `docs/` 配下 | **[docs/CLAUDE.md](./docs/CLAUDE.md)** | ドキュメント執筆ルール、フレームワーク構造、frontmatter管理 |
| `app/` 配下 | **[app/CLAUDE.md](./app/CLAUDE.md)** | コーディング規約、ブランチ戦略、品質チェック |
| `mvp/` 配下 | **[mvp/CLAUDE.md](./mvp/CLAUDE.md)** | MVP開発原則、スコープ管理、検証フロー |

## 全体方針

### 基本原則

- 特に指示がない限り、必ず**日本語**で回答・作成する
- **仕様駆動開発（Specification-Driven Development）**を実践する
- `docs/` ディレクトリの仕様書を**唯一の信頼できる情報源（SSOT）**とする

### プロジェクト構成

```
プロジェクトルート/
├── docs/                  # ドキュメントサイト（プロダクト仕様・ガイド）
│   ├── CLAUDE.md          # ドキュメント執筆者向けガイドライン
│   ├── _template/         # フレームワークテンプレート（Git Submodule）
│   │   └── 01_discovery/ ... 08_metrics/
│   └── docs/framework/    # プロダクト固有のドキュメント（テンプレートからコピー）
│       ├── 01_discovery/        # Discovery（発見）
│       ├── 02_validation/       # Validation（検証）
│       ├── 03_refinement/       # Refinement（深化）
│       ├── 04_strategy/         # Strategy（戦略）
│       ├── 05_specification/    # Specification（仕様）
│       ├── 06_engineering/      # Engineering（設計）
│       ├── 07_go_to_market/     # Go-to-Market（市場投入）
│       └── 08_metrics/          # Metrics（計測）
│
├── app/                   # アプリケーションコード（本番用）
│   └── CLAUDE.md          # 開発者向けガイドライン
│
└── mvp/                   # MVP検証用アプリ
    └── CLAUDE.md          # MVP開発者向けガイドライン
```

### 役割分担

| ディレクトリ | 役割                         | 主な内容                                                 |
| ------------ | ---------------------------- | -------------------------------------------------------- |
| `docs/`      | プロダクト仕様・ガイド       | ペルソナ、ジャーニーマップ、ユーザーストーリー、仕様書等 |
| `app/`       | アプリケーション実装（本番用） | ソースコード、テスト、設定ファイル等                     |
| `mvp/`       | MVP検証用アプリ              | 仮説検証用の軽量実装、計測コード等                       |

### Docs-as-Code の採用

- **ドキュメントを「コード」として扱う**：文書を Git で管理し、コードと同じワークフロー（作成、レビュー、自動デプロイ）に組み込む
- **開発プロセスへの統合**：ドキュメントの更新をソフトウェア開発サイクル（SDLC）と同期させる

### 仕様駆動開発の実践

1. **仕様ファースト**：コードを書く前に、必ず `docs/` 内の仕様書を作成・更新する
2. **仕様と実装の同期**：仕様書を SSOT とし、実装との乖離を防ぐため常に最新の状態に保つ
3. **相互参照**：仕様書と実装コードを相互に参照できるようにする

## サブディレクトリガイドラインの読み込みルール

**作業対象のサブディレクトリに `CLAUDE.md` がある場合、作業開始前に必ず Read ツールで読み込むこと。**

例：
- `docs/` 配下のドキュメントを作成・編集する → まず `docs/CLAUDE.md` を読む
- `app/` 配下のコードを書く → まず `app/CLAUDE.md` を読む
- `mvp/` 配下で開発する → まず `mvp/CLAUDE.md` を読む

## ドキュメント管理ツール

### Frontmatter MCP サーバー

このプロジェクトでは、ドキュメントのメタデータ管理に **frontmatter MCP サーバー** を活用しています。

**主な用途**：

- フロントマターの検索・集計（SQLクエリ）
- セマンティック検索（意味的類似性による検索）
- フロントマターの一括更新
- フレームワーク全体の進捗可視化

**設定場所**: [.mcp.json](./.mcp.json)

**詳細な使い方**: [docs/CLAUDE.md](./docs/CLAUDE.md#frontmatter-mcp-を使ったドキュメント管理)

**クイックサンプル**：

```sql
-- フェーズごとのドキュメント数
SELECT phase, COUNT(*) as count FROM files GROUP BY phase

-- セマンティック検索
SELECT path, title,
       array_cosine_similarity(embedding, embed('ユーザー調査')) as similarity
FROM files ORDER BY similarity DESC LIMIT 10
```

## テンプレート管理（Git Submodule）

`docs/_template/` は独立リポジトリ [product-framework](https://github.com/salaimxd/product-framework) を Git Submodule として取り込んでいます。

### 初回クローン時

```bash
git clone --recursive https://github.com/salaimxd/product-project.git
# または clone 後に
git submodule update --init
```

### テンプレート更新の取り込み

```bash
git submodule update --remote docs/_template
git add docs/_template
git commit -m "chore: テンプレートを最新に更新"
```

### オンデマンドテンプレートコピー

各スキル（`/discovery-persona`、`/validation-mvp` 等）を実行すると、対象ファイルが `docs/docs/framework/` に存在しない場合、`docs/_template/` から自動的にコピーされる。一括コピーのスクリプトは不要。

### 注意事項

- `docs/_template/` 内のファイルは直接編集しない（upstream リポで管理）
- テンプレートの変更は `product-framework` リポで行い、submodule update で取り込む
- `docs/docs/framework/` はプロダクト固有の内容を記入する作業ディレクトリ

## クイックスタート

### ドキュメント執筆者

1. `docs/CLAUDE.md` を読む
2. 該当スキルを実行（例: `/discovery-persona`）→ テンプレートが自動コピーされる
3. `docs/docs/framework/` 配下のコピーされたファイルにプロダクト固有の内容を記入

### アプリケーション開発者

1. `app/CLAUDE.md` を読む
2. `.env.example` をコピーして `.env` を作成
3. `docs/docs/framework/` 配下の仕様書を確認
4. `feature/#<issue>-*` ブランチを作成して開発開始

### MVP開発者

1. `mvp/CLAUDE.md` を読む
2. `docs/docs/framework/02_validation/mvp.md` で仮説・スコープ・成功基準を確認
3. `.env.example` をコピーして `.env` を作成
4. Must Have の機能のみを最小限で実装
5. 計測の仕組みを組み込み、リリースして検証開始
