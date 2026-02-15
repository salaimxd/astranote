# ドキュメント作成ガイドライン

このドキュメントは、プロダクト仕様書・ガイドを執筆する際のルールと方針を定めたものです。

## 基本方針

- 特に指示がない限り、必ず**日本語**で回答・作成する
- すべてのドキュメントも**日本語で作成する**
- これらのドキュメントをプロジェクトの**正解（Single Source of Truth）**として管理する
- ドキュメントは常に更新されること
- ドキュメント更新の際は、必要に応じてディレクトリの追加・変更や、ドキュメント自体の追加・削除・構成変更を柔軟に行うこと

## ドキュメント配置ルール

ドキュメントは `docs/docs/framework/` ディレクトリ内の各フェーズディレクトリに作成する：

- `docs/docs/framework/discovery/` - **Discovery（発見）**：ユーザーニーズの理解と課題の明確化
  - ペルソナ、共感マップ、HMW（How Might We）
- `docs/docs/framework/validation/` - **Validation（検証）**：アイデアの実現可能性とビジネスモデルの検証
  - リーンキャンバス、MVP（Minimum Viable Product）
- `docs/docs/framework/refinement/` - **Refinement（深化）**：プロダクト体験の詳細設計
  - ジャーニーマップ、ユーザーストーリー、ストーリーマッピング、テストケース
- `docs/docs/framework/scalability/` - **Scalability（拡大）**：成長と組織拡大の仕組み構築
  - BMC（Business Model Canvas）、4P分析、North Star Metric、KPI、データ基盤、ADR（Architecture Decision Records）

## Docs-as-Code の原則

### 基本的な考え方

- **ドキュメントを「コード」として扱う**：文書を Git などのバージョン管理システムで管理し、プログラムのコードと同じワークフロー（作成、レビュー、自動デプロイ）に組み込みます
- **開発プロセスへの統合**：ドキュメントの更新タイミングをソフトウェアの開発サイクル（SDLC）と同期させ、コードの変更と同時にドキュメントを更新するようにします

### 仕様駆動開発（ドキュメント側）

- **仕様ファースト**：実装コードを書く前に、必ず仕様書（ドキュメント）を作成・更新すること
- **仕様と実装の同期**：仕様書を唯一の信頼できる情報源（SSOT）とし、実装との乖離を防ぐため、常に最新の状態に保つこと

## ドキュメント構造

### フレームワークの4つのフェーズ

プロダクト開発は以下の4つのフェーズで構成され、各フェーズに対応したドキュメントを作成する：

1. **🔍 Discovery（発見）** - ユーザーのニーズを深く理解し、解決すべき課題を明確化
2. **✅ Validation（検証）** - アイデアの実現可能性とビジネスモデルを検証
3. **🎯 Refinement（深化）** - プロダクトの体験を詳細に設計し、開発に向けて準備
4. **📈 Scalability（拡大）** - プロダクトの成長と組織の拡大を支える仕組みを構築

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

## Frontmatter 管理ツール

このプロジェクトでは **frontmatter MCP サーバー** を使ってドキュメントのメタデータを管理しています。

### 主な機能

1. **検索**: SQLクエリでフロントマターを検索・集計
2. **セマンティック検索**: 意味的な類似性でドキュメントを発見
3. **一括更新**: 複数ドキュメントのフロントマターを一括変更
4. **進捗確認**: フェーズごとの完成度を可視化

### よく使うクエリ例

```sql
-- フェーズごとのドキュメント数
SELECT phase, COUNT(*) as count
FROM files
WHERE phase IS NOT NULL
GROUP BY phase

-- 未完了ドキュメントの一覧
SELECT path, title, status
FROM files
WHERE status != 'published' AND status IS NOT NULL

-- セマンティック検索（例：ユーザー調査関連）
SELECT
  path,
  title,
  array_cosine_similarity(embedding, embed('ユーザー調査')) as score
FROM files
WHERE embedding IS NOT NULL
ORDER BY score DESC
LIMIT 10

-- 完成度レポート
SELECT
  phase,
  COUNT(*) as total,
  SUM(CASE WHEN status = 'published' THEN 1 ELSE 0 END) as published,
  ROUND(100.0 * SUM(CASE WHEN status = 'published' THEN 1 ELSE 0 END) / COUNT(*), 1) as completion_rate
FROM files
WHERE phase IS NOT NULL
GROUP BY phase
ORDER BY phase
```

### 一括更新の例

```python
# ステータスを一括更新
batch_update(
  glob="framework/01_discovery/**/*.md",
  set={"status": "published"}
)

# キーワードを追加
batch_array_add(
  glob="framework/**/*.md",
  property="keywords",
  value="プロダクト開発"
)
```

詳細は `docs/CLAUDE.md` の「Frontmatter MCP を使ったドキュメント管理」セクションを参照してください。
