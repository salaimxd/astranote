# ドキュメント作成ガイドライン

このドキュメントは、プロダクト仕様書・ガイドを執筆する際のルールと方針を定めたものです。

## 基本方針

- 特に指示がない限り、必ず**日本語**で回答・作成する
- すべてのドキュメントも**日本語で作成する**
- これらのドキュメントをプロジェクトの**正解（Single Source of Truth）**として管理する
- ドキュメントは常に更新されること
- ドキュメント更新の際は、必要に応じてディレクトリの追加・変更や、ドキュメント自体の追加・削除・構成変更を柔軟に行うこと

## テンプレート管理

### ディレクトリの役割

| ディレクトリ | 役割 | 編集 |
|---|---|---|
| `docs/_template/` | テンプレート原本（Git Submodule） | 直接編集しない |
| `docs/docs/framework/` | プロダクト固有のドキュメント | ここに記入する |

### オンデマンドテンプレートコピー

各スキル（`/discovery-persona`、`/validation-mvp` 等）を実行すると、対象ファイルが `docs/docs/framework/` に存在しない場合、スキルが `docs/_template/` から自動的にコピーして記入を開始する。一括コピーのスクリプトは不要。

### テンプレート更新の取り込み

テンプレートリポジトリに更新があった場合：

```bash
git submodule update --remote docs/_template
git add docs/_template
git commit -m "chore: テンプレートを最新に更新"
```

## ドキュメント配置ルール

ドキュメントは `docs/docs/framework/` ディレクトリ内の各フェーズディレクトリに作成する：

- `docs/docs/framework/01_discovery/` - **Discovery（発見）**：ユーザーニーズの理解と課題の明確化
  - ユーザーリサーチ、競合分析、ペルソナ、共感マップ、ジャーニーマップ、HMW（How Might We）
- `docs/docs/framework/02_validation/` - **Validation（検証）**：アイデアの実現可能性とビジネスモデルの検証
  - リーンキャンバス、MVP（Minimum Viable Product）
- `docs/docs/framework/03_refinement/` - **Refinement（深化）**：プロダクト体験の詳細設計
  - ユーザーストーリー、ストーリーマッピング、テストケース
- `docs/docs/framework/04_strategy/` - **Strategy（戦略）**：ビジネスモデルと成長指標の定義
  - 価値提案（Value Proposition）、BMC（Business Model Canvas）、North Star Metric
- `docs/docs/framework/05_specification/` - **Specification（仕様）**：実装可能な仕様への変換
  - PRD（要件定義書）、UI Flows（画面遷移図）、Design System（デザインシステム）
- `docs/docs/framework/06_engineering/` - **Engineering（技術）**：技術設計と実装
  - Tech Stack、Architecture、API Docs、ADR（Architecture Decision Records）、データ基盤
- `docs/docs/framework/07_go_to_market/` - **Go-to-Market（市場投入）**：市場投入戦略
  - 4P分析、Distribution Channels、Launch Checklist、ASO/SEO Strategy
- `docs/docs/framework/08_metrics/` - **Metrics（計測）**：成果の測定と改善
  - KPI、Funnel Analysis、Cohort Analysis

## Docs-as-Code の原則

### 基本的な考え方

- **ドキュメントを「コード」として扱う**：文書を Git などのバージョン管理システムで管理し、プログラムのコードと同じワークフロー（作成、レビュー、自動デプロイ）に組み込みます
- **開発プロセスへの統合**：ドキュメントの更新タイミングをソフトウェアの開発サイクル（SDLC）と同期させ、コードの変更と同時にドキュメントを更新するようにします

### 仕様駆動開発（ドキュメント側）

- **仕様ファースト**：実装コードを書く前に、必ず仕様書（ドキュメント）を作成・更新すること
- **仕様と実装の同期**：仕様書を唯一の信頼できる情報源（SSOT）とし、実装との乖離を防ぐため、常に最新の状態に保つこと

## ドキュメント構造

### フレームワークの8つのフェーズ

プロダクト開発は以下の8つのフェーズで構成され、各フェーズに対応したドキュメントを作成する：

1. **🔍 Discovery（発見）** - ユーザーのニーズを深く理解し、解決すべき課題を明確化
2. **✅ Validation（検証）** - アイデアの実現可能性とビジネスモデルを検証
3. **🎯 Refinement（深化）** - プロダクトの体験を詳細に設計し、開発に向けて準備
4. **🧭 Strategy（戦略）** - ビジネスモデルと成長指標を定義
5. **📦 Specification（仕様）** - 何を作るのかを具体的に定義
6. **⚙️ Engineering（技術）** - どう作るのかを技術的に設計
7. **🚀 Go-to-Market（市場投入）** - どう届けるのかを戦略的に設計
8. **📈 Metrics（計測）** - うまくいっているかを数値で測定

### フロントマターガイドライン

#### 基本方針

- すべてのMarkdownファイルにはYAML形式のフロントマターを記述する
- Rspress標準フィールド（title, description, pageType）を優先的に使用する
- カスタムフィールドは明確な命名規則に従う

#### 必須フィールド

| フィールド    | 説明                                       | 例                                              |
| ------------- | ------------------------------------------ | ----------------------------------------------- |
| `title`       | ドキュメントのタイトル（日本語・英語併記） | `ユーザーリサーチ（User Research）`             |
| `description` | 1-2行の要約（SEO最適化用）                 | `ユーザーの生の声を収集する手法とテンプレート`  |
| `docType`     | ドキュメントの種別                         | `template` / `guide` / `reference` / `overview` |

#### 推奨フィールド

| フィールド | 説明                     | 例                                                                                                                    |
| ---------- | ------------------------ | --------------------------------------------------------------------------------------------------------------------- |
| `phase`    | 該当するフェーズ         | `discovery` / `validation` / `refinement` / `strategy` / `specification` / `engineering` / `go-to-market` / `metrics` |
| `status`   | ドキュメントのステータス | `draft` / `in-progress` / `review` / `approved` / `published`                                                         |
| `pageType` | Rspressページタイプ      | `doc` / `home` / `custom`                                                                                             |

#### ドキュメントタイプ（docType）

| 値          | 説明                     | 使用例                             |
| ----------- | ------------------------ | ---------------------------------- |
| `template`  | 記入して使うテンプレート | `_template.md`, `MVP.md`, `PRD.md` |
| `guide`     | 手法やプロセスの説明     | `user_research.md`, `adr.md`       |
| `reference` | 技術的な参照情報         | `tech_stack.md`, `api_docs.md`     |
| `overview`  | フェーズの概要           | `index.md`                         |

#### ステータス（status）

| 値            | 説明                                     |
| ------------- | ---------------------------------------- |
| `draft`       | 作成中（初期ドラフト）                   |
| `in-progress` | 記入中（テンプレートへの記入作業中）     |
| `review`      | レビュー待ち（PRでレビュー依頼中）       |
| `approved`    | 承認済み（レビュー完了、適用可能）       |
| `published`   | 公開済み（テンプレートやガイドの最終版） |

#### ドキュメントタイプ別テンプレート

##### テンプレートファイル（\_template.md）

```yaml
---
title: [日本語タイトル]（[English Title]）
description: [目的を1行で]
pageType: doc
docType: template
phase: [フェーズ名]
status: published
---
```

##### ガイドドキュメント

```yaml
---
title: [タイトル]
description: [要約]
pageType: doc
docType: guide
phase: [フェーズ名]
status: [ステータス]
---
```

##### 概要ページ（index.md）

```yaml
---
title: 概要（Overview）
description: [フェーズの要約]
pageType: doc
docType: overview
phase: [フェーズ名]
status: published
---
```

#### 本文内メタデータとの使い分け

| 情報の種類                   | 管理場所       | 例                                |
| ---------------------------- | -------------- | --------------------------------- |
| ドキュメント自体のメタデータ | フロントマター | 分類（docType）、ステータス、タグ |
| プロジェクト固有の情報       | 本文           | プロジェクト名、担当者、調査期間  |
| 時系列情報                   | Git履歴        | 作成日、更新日、作成者            |

**原則**: フロントマターには「ドキュメント自体」のメタデータを記載し、「プロジェクト固有」の情報は本文に記載する。例えば、MVP.mdの「プロジェクト名」「調査期間」「調査者」は本文の「プロジェクト情報」ブロックに残す。

### ファイル命名規則

- 各フェーズディレクトリ内のファイルは、フレームワーク名をベースに命名される
- ファイル名は小文字とアンダースコアを使用する（例：`persona.md`、`empathy_map.md`、`journey_map.md`）
- 各フェーズディレクトリには `index.md` と `_meta.json` が存在する
- `_meta.json` にはナビゲーション順序を定義する

### Index.md の役割

- 各フェーズディレクトリには `index.md` が存在する
- フェーズの概要、目的、含まれるフレームワークの一覧と使い分けガイドを含む
- 新規メンバーのオンボーディングとナビゲーションの起点となる

### ドキュメント間の相互参照

- 関連ドキュメントへのリンクは相対パスまたはルートパスを使用する
- フェーズ内：`[ペルソナ](./persona)` または `[ペルソナ](/framework/discovery/persona)`
- フェーズ間：`[MVP](/framework/validation/mvp)` のようにルートパス指定
- すべての主要ドキュメントは関連フレームワークへのリンクセクションを含むこと

## ドキュメントレビューガイドライン

- ドキュメントの追加・更新時は、関連する他のドキュメントとの整合性を確認する
- 相互参照リンクが正しく機能するか確認する
- 用語の一貫性を保つ（用語集がある場合はそれに従う）
- 読者の視点で、理解しやすく、実践しやすい内容になっているか確認する

## Frontmatter MCP を使ったドキュメント管理

このプロジェクトでは、フロントマターの効率的な管理と検索のために **frontmatter MCP サーバー** を活用しています。

### 利用可能な機能

#### 1. フロントマター検索・分析

**基本的なSQLクエリ**

フロントマターをDuckDB SQLでクエリできます。`files` テーブルには全ドキュメントのフロントマター情報が含まれます。

```sql
-- フェーズごとのドキュメント数をカウント
SELECT phase, COUNT(*) as doc_count
FROM files
WHERE phase IS NOT NULL
GROUP BY phase
ORDER BY phase

-- 特定のステータスのドキュメントを検索
SELECT path, title, status
FROM files
WHERE status = 'draft' OR status = 'in-progress'

-- docTypeごとの分布を確認
SELECT docType, COUNT(*) as count
FROM files
GROUP BY docType
ORDER BY count DESC

-- キーワードで検索（配列型フィールド）
SELECT path, title, keywords
FROM files
WHERE list_contains(keywords, 'プロダクト開発')
```

#### 2. セマンティック検索

意味的な類似性に基づいてドキュメントを検索できます。256次元のベクトル埋め込みを使用して、類似したドキュメントを発見します。

```sql
-- 「ユーザー調査」に関連するドキュメントを検索
SELECT
  path,
  title,
  array_cosine_similarity(embedding, embed('ユーザー調査')) as similarity
FROM files
WHERE embedding IS NOT NULL
ORDER BY similarity DESC
LIMIT 10

-- 「データ分析」に関連するドキュメントを検索
SELECT
  path,
  title,
  phase,
  array_cosine_similarity(embedding, embed('データ分析')) as similarity
FROM files
WHERE embedding IS NOT NULL
ORDER BY similarity DESC
LIMIT 10

-- 特定フェーズ内でセマンティック検索
SELECT
  path,
  title,
  array_cosine_similarity(embedding, embed('顧客理解')) as similarity
FROM files
WHERE embedding IS NOT NULL AND phase = 'discovery'
ORDER BY similarity DESC
LIMIT 5
```

#### 3. フロントマターの一括更新

**単一ファイルの更新**

```bash
# ステータスを更新（CLIの場合）
# または mcp__frontmatter__update ツールを使用

# 例：ステータスをpublishedに更新
update(
  path="framework/01_discovery/persona.md",
  set={"status": "published"}
)

# 複数フィールドを同時更新
update(
  path="framework/01_discovery/persona.md",
  set={"status": "published", "docType": "guide"}
)

# フィールドを削除
update(
  path="framework/01_discovery/persona.md",
  unset=["oldField", "deprecatedField"]
)
```

**パターンマッチで一括更新**

```bash
# discovery フェーズの全ドキュメントのステータスを更新
batch_update(
  glob="framework/01_discovery/**/*.md",
  set={"status": "published"}
)

# 特定のフィールドを全ドキュメントから削除
batch_update(
  glob="framework/**/*.md",
  unset=["oldField"]
)

# 複数フィールドを同時に設定
batch_update(
  glob="framework/08_metrics/**/*.md",
  set={"phase": "metrics", "status": "published"}
)
```

#### 4. キーワード配列の管理

配列型のフィールド（`keywords`、`tags`など）を効率的に管理できます。

```bash
# 全ドキュメントに新しいキーワードを追加
batch_array_add(
  glob="framework/**/*.md",
  property="keywords",
  value="プロダクト開発",
  allow_duplicates=False
)

# 古いキーワードを削除
batch_array_remove(
  glob="framework/**/*.md",
  property="keywords",
  value="deprecated-tag"
)

# キーワードを置換
batch_array_replace(
  glob="framework/**/*.md",
  property="keywords",
  old_value="旧タグ",
  new_value="新タグ"
)

# キーワードをアルファベット順にソート
batch_array_sort(
  glob="framework/**/*.md",
  property="keywords",
  reverse=False
)

# 重複キーワードを削除
batch_array_unique(
  glob="framework/**/*.md",
  property="keywords"
)
```

#### 5. 進捗状況の確認

フレームワーク全体の完成度を可視化するクエリ例：

```sql
-- フェーズごとのステータス集計
SELECT
  phase,
  status,
  COUNT(*) as count
FROM files
WHERE phase IS NOT NULL
GROUP BY phase, status
ORDER BY phase, status

-- 完成度レポート
SELECT
  phase,
  COUNT(*) as total,
  SUM(CASE WHEN status = 'published' THEN 1 ELSE 0 END) as published,
  SUM(CASE WHEN status = 'draft' OR status = 'in-progress' THEN 1 ELSE 0 END) as in_progress,
  ROUND(100.0 * SUM(CASE WHEN status = 'published' THEN 1 ELSE 0 END) / COUNT(*), 1) as completion_rate
FROM files
WHERE phase IS NOT NULL
GROUP BY phase
ORDER BY phase

-- docTypeごとの完成度
SELECT
  docType,
  COUNT(*) as total,
  SUM(CASE WHEN status = 'published' THEN 1 ELSE 0 END) as published,
  ROUND(100.0 * SUM(CASE WHEN status = 'published' THEN 1 ELSE 0 END) / COUNT(*), 1) as completion_rate
FROM files
WHERE docType IS NOT NULL
GROUP BY docType
ORDER BY total DESC

-- 未完了ドキュメントの一覧（優先度付き）
SELECT
  path,
  title,
  phase,
  status,
  docType
FROM files
WHERE status != 'published' AND status IS NOT NULL
ORDER BY phase, docType
```

#### 6. インデックス管理

セマンティック検索のためのインデックス管理：

```bash
# インデックスの状態を確認
index_status()
# 戻り値: {"state": "ready"} または {"state": "indexing"} または {"state": "idle"}

# インデックスを更新（差分更新）
index_refresh()
# 初回実行時: 全ファイルをインデックス
# 2回目以降: 変更されたファイルのみ更新

# インデックス完了を待機
index_wait(timeout=120)
# タイムアウト前に完了: {"success": True, "state": "ready"}
# タイムアウト: {"success": False, "state": "indexing"}
```

### ワークフロー例

#### ドキュメント作成時

1. テンプレートからドキュメントを作成
2. フロントマターに `status: draft` を設定
3. 執筆完了後、`status: review` に更新
4. レビュー承認後、`status: published` に更新

```bash
# 1. ドキュメント作成後、statusをdraftに設定
update(
  path="framework/01_discovery/new_doc.md",
  set={"status": "draft", "phase": "discovery", "docType": "guide"}
)

# 2. 執筆完了後、reviewに変更
update(
  path="framework/01_discovery/new_doc.md",
  set={"status": "review"}
)

# 3. レビュー承認後、publishedに変更
update(
  path="framework/01_discovery/new_doc.md",
  set={"status": "published"}
)
```

#### リファクタリング時

1. セマンティック検索で関連ドキュメントを特定
2. 一括更新でフロントマターを統一
3. SQLクエリで整合性を確認

```sql
-- 1. 「ペルソナ」関連のドキュメントを検索
SELECT
  path,
  title,
  array_cosine_similarity(embedding, embed('ペルソナ')) as similarity
FROM files
WHERE embedding IS NOT NULL
ORDER BY similarity DESC
LIMIT 10

-- 2. 関連ドキュメントにキーワードを追加（batch_array_add使用）

-- 3. キーワードの整合性を確認
SELECT path, title, keywords
FROM files
WHERE list_contains(keywords, 'ペルソナ')
```

#### 進捗確認時

1. フェーズごとの完成度をクエリ
2. 未完了ドキュメントをリストアップ
3. 優先順位を決めて作業を進める

```sql
-- 1. フェーズごとの完成度
SELECT
  phase,
  COUNT(*) as total,
  SUM(CASE WHEN status = 'published' THEN 1 ELSE 0 END) as published,
  ROUND(100.0 * SUM(CASE WHEN status = 'published' THEN 1 ELSE 0 END) / COUNT(*), 1) as completion_rate
FROM files
WHERE phase IS NOT NULL
GROUP BY phase
ORDER BY completion_rate ASC

-- 2. 未完了ドキュメント（完成度の低いフェーズから）
SELECT path, title, phase, status
FROM files
WHERE status != 'published' AND phase IN (
  SELECT phase FROM files WHERE phase IS NOT NULL
  GROUP BY phase
  ORDER BY SUM(CASE WHEN status = 'published' THEN 1 ELSE 0 END) / COUNT(*) ASC
  LIMIT 3
)
ORDER BY phase, status
```

### 実用的なクエリ集

#### ドキュメントの関連性分析

```sql
-- relatedDocs フィールドを持つドキュメントを検索
SELECT path, title, phase
FROM files
WHERE relatedDocs IS NOT NULL

-- summary が長いドキュメント（詳細な説明がある）
SELECT path, title, LENGTH(summary) as summary_length
FROM files
WHERE summary IS NOT NULL
ORDER BY summary_length DESC
LIMIT 10

-- 特定のキーワードを含むドキュメント
SELECT path, title, keywords
FROM files
WHERE list_contains(keywords, 'KPI') OR list_contains(keywords, 'メトリクス')
```

#### 品質チェック

```sql
-- フロントマターが不完全なドキュメント
SELECT path, title, phase, docType, status
FROM files
WHERE title IS NULL OR phase IS NULL OR docType IS NULL

-- descriptionが短すぎるドキュメント（SEO対策）
SELECT path, title, LENGTH(description) as desc_length
FROM files
WHERE description IS NOT NULL AND LENGTH(description) < 50
ORDER BY desc_length ASC

-- summaryがないドキュメント（概要不足）
SELECT path, title, phase, docType
FROM files
WHERE summary IS NULL AND docType IN ('guide', 'template')
ORDER BY phase
```

### ツールリファレンス

#### 利用可能な MCP ツール

| ツール名                                | 説明                             | 主な用途                       |
| --------------------------------------- | -------------------------------- | ------------------------------ |
| `mcp__frontmatter__query`               | SQLクエリでフロントマターを検索  | 分析、レポート作成             |
| `mcp__frontmatter__query_inspect`       | フロントマタースキーマを取得     | 利用可能なフィールドの確認     |
| `mcp__frontmatter__update`              | 単一ファイルのフロントマター更新 | 個別ドキュメントの編集         |
| `mcp__frontmatter__batch_update`        | 複数ファイルの一括更新           | リファクタリング、一括変更     |
| `mcp__frontmatter__batch_array_add`     | 配列フィールドへの値追加         | キーワード追加                 |
| `mcp__frontmatter__batch_array_remove`  | 配列フィールドから値削除         | キーワード削除                 |
| `mcp__frontmatter__batch_array_replace` | 配列フィールドの値置換           | キーワード統一                 |
| `mcp__frontmatter__batch_array_sort`    | 配列フィールドのソート           | キーワード整理                 |
| `mcp__frontmatter__batch_array_unique`  | 配列フィールドの重複削除         | データクリーンアップ           |
| `mcp__frontmatter__index_status`        | インデックス状態の確認           | セマンティック検索の可用性確認 |
| `mcp__frontmatter__index_refresh`       | インデックスの更新               | 新規ドキュメント追加後         |
| `mcp__frontmatter__index_wait`          | インデックス完了待機             | セマンティック検索前の確認     |

### トラブルシューティング

#### インデックスが更新されない

```bash
# インデックス状態を確認
index_status()

# 手動でインデックスを更新
index_refresh()

# 完了を待機
index_wait(timeout=120)
```

#### クエリが遅い

- インデックスが完了しているか確認: `index_status()`
- `WHERE` 句でフィルタリングを活用
- `LIMIT` で結果数を制限
- セマンティック検索は `embedding IS NOT NULL` で事前フィルタ

#### フロントマターの形式エラー

- YAML形式が正しいか確認（インデント、コロン、ダッシュ）
- `query_inspect()` でスキーマを確認し、期待される型と一致しているか確認
- 配列フィールドは `[]` または `- item1` の形式で記述
