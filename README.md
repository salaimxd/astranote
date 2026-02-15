# Product Template

プロダクト開発のためのテンプレートプロジェクト。Discovery（発見）から Metrics（計測）まで、8つのフェーズのフレームワークに沿ってドキュメント駆動でプロダクトを開発する。

## セットアップ

### クローン

```bash
git clone --recursive https://github.com/salaimxd/product-project.git my-project
cd my-project
```

> `--recursive` を付けることで、テンプレート（`docs/_template`）サブモジュールも同時に取得されます。

もし `--recursive` を付け忘れた場合：

```bash
git submodule update --init
```

### 新プロジェクトとして使う

#### 方法A: リモートを切り替える

```bash
git remote remove origin
git remote add origin https://github.com/<your-org>/<new-project>.git
git push -u origin main
```

#### 方法B: 履歴をリセットして始める

```bash
rm -rf .git
git init
rm -rf docs/_template
git submodule add https://github.com/salaimxd/product-framework.git docs/_template
git add -A
git commit -m "Initial commit"
git remote add origin https://github.com/<your-org>/<new-project>.git
git push -u origin main
```

> **注意**: `rm -rf .git` でリポジトリを初期化すると、サブモジュール `docs/_template` の `.git` 参照が壊れ、通常のディレクトリとして残ります。`rm -rf docs/_template` で一度削除してから `git submodule add` で再登録してください。

## テンプレートの更新

テンプレートリポジトリ（[product-framework](https://github.com/salaimxd/product-framework)）に更新があった場合：

```bash
git submodule update --remote docs/_template
git add docs/_template
git commit -m "chore: テンプレートを最新に更新"
```

## ディレクトリ構成

```
.
├── docs/                  # ドキュメントサイト（Rspress）
│   ├── _template/         # フレームワークテンプレート（Git Submodule）
│   └── docs/framework/    # プロダクト固有のドキュメント（作業ディレクトリ）
├── app/                   # アプリケーションコード（本番用）
└── mvp/                   # MVP検証用アプリ
```

| ディレクトリ | 役割 |
|---|---|
| `docs/_template/` | テンプレート原本（直接編集しない） |
| `docs/docs/framework/` | プロダクト固有のドキュメント（ここに記入する） |
| `app/` | 本番アプリケーション |
| `mvp/` | MVP検証用の軽量実装 |

## ドキュメントサイトの起動

```bash
cd docs
bun install
bun run dev
```

詳細は [docs/README.md](./docs/README.md) を参照。
