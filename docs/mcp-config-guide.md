# MCP サーバー設定ガイド

このドキュメントでは、プロジェクトで使用している MCP（Model Context Protocol）サーバーの設定と使い方を説明します。

## 設定ファイル

MCP サーバーの設定は、プロジェクトルートの [`.mcp.json`](../.mcp.json) で管理されています。

## 利用可能なサーバー

### 1. Frontmatter サーバー

ドキュメントのフロントマター（メタデータ）を効率的に管理・検索するためのサーバー。

#### 設定内容

```json
{
  "frontmatter": {
    "type": "stdio",
    "command": "uvx",
    "args": ["--from", "frontmatter-mcp[semantic]", "frontmatter-mcp"],
    "env": {
      "FRONTMATTER_BASE_DIR": "./docs/docs",
      "FRONTMATTER_ENABLE_SEMANTIC": "true",
      "MCP_TIMEOUT": "300000"
    }
  }
}
```

#### 設定パラメータ

| パラメータ | 値 | 説明 |
|-----------|-----|------|
| `FRONTMATTER_BASE_DIR` | `./docs/docs` | ドキュメントのルートディレクトリ |
| `FRONTMATTER_ENABLE_SEMANTIC` | `true` | セマンティック検索を有効化 |
| `MCP_TIMEOUT` | `300000` | タイムアウト時間（ミリ秒）= 5分 |

#### 主な機能

**1. SQLクエリによる検索**

DuckDB SQLを使って、フロントマターを柔軟に検索・集計できます。

```sql
-- フェーズごとのドキュメント数
SELECT phase, COUNT(*) as count
FROM files
WHERE phase IS NOT NULL
GROUP BY phase

-- 未完了ドキュメント
SELECT path, title, status
FROM files
WHERE status != 'published'
```

**2. セマンティック検索**

256次元のベクトル埋め込みを使った意味的類似性検索。

```sql
SELECT path, title,
       array_cosine_similarity(embedding, embed('ユーザー調査')) as similarity
FROM files
WHERE embedding IS NOT NULL
ORDER BY similarity DESC
LIMIT 10
```

**3. フロントマターの一括更新**

複数ドキュメントのメタデータを効率的に更新。

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

#### インデックス管理

セマンティック検索を使用するには、インデックスの作成が必要です。

```python
# インデックスの状態を確認
index_status()
# 戻り値: {"state": "ready"} | {"state": "indexing"} | {"state": "idle"}

# インデックスを更新（差分更新）
index_refresh()
# 初回: 全ファイルをインデックス
# 2回目以降: 変更されたファイルのみ

# インデックス完了を待機
index_wait(timeout=120)
# success: {"success": True, "state": "ready"}
# timeout: {"success": False, "state": "indexing"}
```

**インデックス更新のタイミング**：
- 新しいドキュメントを追加した後
- ドキュメントの内容を大幅に変更した後
- セマンティック検索の結果が古いと感じた時

#### トラブルシューティング

**問題: セマンティック検索が動作しない**

```python
# 1. インデックス状態を確認
status = index_status()
print(status)

# 2. インデックスが "idle" または古い場合、更新
if status["state"] != "ready":
    index_refresh()
    index_wait(timeout=180)
```

**問題: クエリが遅い**

- `WHERE` 句でフィルタリングを活用
- `LIMIT` で結果数を制限
- セマンティック検索は `embedding IS NOT NULL` で事前フィルタ

**問題: タイムアウトエラー**

`.mcp.json` の `MCP_TIMEOUT` 値を増やす：

```json
"env": {
  "MCP_TIMEOUT": "600000"  // 10分に延長
}
```

### 2. Astro Docs サーバー

Astro フレームワークの公式ドキュメントを検索するためのサーバー。

#### 設定内容

```json
{
  "astro-docs": {
    "type": "http",
    "url": "https://mcp.docs.astro.build/mcp"
  }
}
```

#### 使い方

```python
# Astro ドキュメントを検索
search_astro_docs(query="コンポーネント")
```

### 3. Context7 サーバー

ライブラリドキュメントの検索と参照を提供するサーバー。

#### 設定内容

```json
{
  "context7": {
    "type": "stdio",
    "command": "npx",
    "args": ["-y", "@upstash/context7-mcp"],
    "env": {}
  }
}
```

#### 使い方

```python
# ライブラリIDを検索
resolve_library_id(
  libraryName="react",
  query="状態管理の方法"
)

# ドキュメントを取得
query_docs(
  libraryId="/facebook/react",
  query="useStateフックの使い方"
)
```

### 4. Sequential Thinking サーバー

段階的な思考プロセスをサポートするサーバー。

#### 設定内容

```json
{
  "sequential-thinking": {
    "type": "stdio",
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-sequential-thinking"],
    "env": {}
  }
}
```

### 5. App Insight MCP サーバー

App Store と Google Play のアプリ情報を取得するサーバー。

#### 設定内容

```json
{
  "app-insight-mcp": {
    "command": "npx",
    "args": ["-y", "@jeromyfu/app-insight-mcp"]
  }
}
```

#### 使い方

```python
# App Store でアプリを検索
app_store_search(term="メモアプリ", country="jp")

# アプリの詳細を取得
app_store_details(id=123456789, country="jp")

# レビューを取得
app_store_reviews(id=123456789, country="jp", page=1)
```

## 設定のカスタマイズ

### タイムアウトの調整

大量のファイルを扱う場合、タイムアウト値を増やすことを推奨します。

```json
"env": {
  "MCP_TIMEOUT": "600000"  // 10分
}
```

### ベースディレクトリの変更

Frontmatter サーバーのベースディレクトリを変更する場合：

```json
"env": {
  "FRONTMATTER_BASE_DIR": "./path/to/docs"
}
```

**注意**: パスはプロジェクトルートからの相対パスで指定してください。

## よくある質問

### Q1: MCP サーバーはいつ起動する？

A: Claude Code や対応するクライアントが `.mcp.json` を読み込んだ際に自動的に起動します。

### Q2: 複数の MCP サーバーを同時に使える？

A: はい、`.mcp.json` に定義されたすべてのサーバーが同時に利用可能です。

### Q3: Frontmatter サーバーのインデックスはいつ更新される？

A: 自動では更新されません。`index_refresh()` を手動で実行する必要があります。

### Q4: セマンティック検索を無効化したい

A: `.mcp.json` の設定を変更：

```json
"env": {
  "FRONTMATTER_ENABLE_SEMANTIC": "false"
}
```

ただし、この場合は `embedding` 列と `embed()` 関数が使用できなくなります。

## 参考リンク

- **Frontmatter MCP**: [GitHub](https://github.com/yourusername/frontmatter-mcp)
- **MCP 仕様**: [Model Context Protocol](https://modelcontextprotocol.io/)
- **詳細な使い方**: [docs/CLAUDE.md](./CLAUDE.md#frontmatter-mcp-を使ったドキュメント管理)
