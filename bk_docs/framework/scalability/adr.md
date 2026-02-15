---
title: アーキテクチャ決定記録（ADR）
---

> このドキュメントは、ソフトウェアアーキテクチャに関する重要な決定を記録・共有し、チーム全体でコンテキストを共有するためのテンプレートです。

## なぜADRが必要か

**従来の課題:**
- なぜその技術を選んだのか、半年後には誰も覚えていない
- 新メンバーが過去の決定の背景を理解できない
- 同じ議論を何度も繰り返す
- 技術的負債が蓄積し、変更が困難になる
- アーキテクチャドキュメントが長すぎて読まれない

**ADRが解決すること:**
- 重要な決定とその理由を記録
- 意思決定の背景とトレードオフを明確化
- チーム全体でコンテキストを共有
- 将来の見直し・変更の判断材料
- 軽量で継続しやすいドキュメンテーション

## ADRの基本構造

ADRは以下の要素で構成されます（Michael Nygardのテンプレート）。

### 1. タイトル（Title）

**決定事項を簡潔に表現**

- ADR番号を付ける（ADR-001、ADR-002...）
- 決定内容が分かるタイトル

### 2. ステータス（Status）

**決定の現在の状態**

- **Proposed（提案中）**: 検討中、まだ決定していない
- **Accepted（承認）**: 決定され、実装中または実装済み
- **Deprecated（非推奨）**: 新しい決定に置き換えられた
- **Superseded（置換）**: 別のADRに置き換えられた（ADR番号を記載）

### 3. コンテキスト（Context）

**なぜこの決定が必要なのか**

- 背景と課題
- 現在の状況
- 技術的・ビジネス的な制約
- 考慮した代替案

**ポイント:**
- 事実を記述（意見や推測ではなく）
- 将来の読者が状況を理解できるように
- 簡潔に、しかし必要な情報は省略しない

### 4. 決定（Decision）

**何を選択したか**

- 明確な決定事項
- 「〜する」という形で記述
- 具体的に

### 5. 結果（Consequences）

**この決定による影響**

- ポジティブな結果（メリット）
- ネガティブな結果（デメリット、トレードオフ）
- リスクと緩和策
- 今後の課題

**ポイント:**
- 正直に記述（良いことだけでなく、悪いことも）
- 具体的な影響を記載
- 対処すべきリスクを明確にする

## ADRの作成プロセス

### 1. 決定が必要なタイミング

**ADRを書くべき決定:**
- 技術スタックの選択（言語、フレームワーク、ライブラリ）
- アーキテクチャパターンの選択（モノリス、マイクロサービス、サーバーレス等）
- データベースの選択（RDB、NoSQL、ストレージ）
- インフラの選択（クラウドプロバイダー、Kubernetes等）
- セキュリティに関する決定（認証、認可、暗号化）
- API設計（REST、GraphQL、gRPC）
- デプロイ戦略
- 外部サービスの選択

**ADRを書かなくて良い決定:**
- 日常的な実装の詳細
- 容易に変更可能な決定
- チーム内の慣習（コーディング規約等）

### 2. 作成ステップ

**Step 1: 新しいADRファイルを作成**

ファイル名の例:
```
docs/adr/0001-use-microservices.md
docs/adr/0002-use-postgresql.md
```

**Step 2: テンプレートに従って記述**

```markdown
# ADR-001: マイクロサービスアーキテクチャの採用

## Status
Proposed

## Context
[背景と課題を記述]

## Decision
[決定事項を記述]

## Consequences
[影響を記述]
```

**Step 3: チームでレビュー**

- プルリクエストで共有
- 議論し、合意形成
- 必要に応じて修正

**Step 4: ステータスを更新**

- 決定したら `Status: Accepted`
- 実装とともにマージ

**Step 5: 定期的に見直し**

- 状況が変わったら更新
- 決定が古くなったら `Status: Deprecated` または `Superseded`

### 3. ADRの管理

**ディレクトリ構造:**
```
project/
├── docs/
│   └── adr/
│       ├── 0001-use-microservices.md
│       ├── 0002-use-postgresql.md
│       ├── 0003-use-graphql.md
│       └── README.md  # ADR一覧とガイド
```

**バージョン管理:**
- Gitで管理
- コードと一緒にバージョン管理
- プルリクエストでレビュー

**ツール:**
- [adr-tools](https://github.com/npryce/adr-tools): ADR作成を支援するCLIツール
- [log4brains](https://github.com/thomvaill/log4brains): ADRのWebビューア

## ADRテンプレート

### 基本テンプレート（Michael Nygard）

```markdown
# ADR-XXX: [タイトル]

## Status
[Proposed | Accepted | Deprecated | Superseded]

## Context
[背景と課題、考慮した代替案]

## Decision
[決定事項]

## Consequences
[影響（ポジティブ、ネガティブ、リスク）]
```

### 拡張テンプレート

```markdown
# ADR-XXX: [タイトル]

## Status
[Proposed | Accepted | Deprecated | Superseded]

## Date
YYYY-MM-DD

## Context
### 背景
[なぜこの決定が必要か]

### 課題
[解決すべき問題]

### 制約
[技術的・ビジネス的制約]

### 代替案
1. [案1]
2. [案2]
3. [案3]

## Decision
[何を選択したか、なぜそれを選んだか]

## Consequences
### ポジティブ
+ [メリット1]
+ [メリット2]

### ネガティブ
- [デメリット1]
- [デメリット2]

### リスク
- [リスク1]: [緩和策]
- [リスク2]: [緩和策]

### 今後の課題
- [TODO1]
- [TODO2]

## Links
- 関連するADR: ADR-XXX
- 参考資料: [URL]
```

## ADRのベストプラクティス

### 1. シンプルに保つ

- 長すぎるADRは読まれない
- 要点を簡潔に
- 詳細は別ドキュメントやリンクで

### 2. 事実ベースで記述

- 意見や推測ではなく、事実を記述
- 「〜かもしれない」ではなく、「〜である」
- データや根拠を示す

### 3. トレードオフを明確にする

- 良いことだけでなく、悪いことも記述
- すべての決定にはトレードオフがある
- 正直に記録することが重要

### 4. 定期的に見直す

- 状況が変わったらADRを更新
- 古くなった決定は `Deprecated` または `Superseded`
- 生きたドキュメントとして管理

### 5. チーム全体で共有

- プルリクエストでレビュー
- 決定プロセスに全員が参加
- 新メンバーのオンボーディング資料として活用

### 6. 番号を振る

- ADRに連番を付ける（ADR-001、ADR-002...）
- 時系列で管理
- 参照しやすくする

### 7. 削除しない

- 古いADRも残す（ステータスを更新）
- 過去の決定の履歴として価値がある
- なぜその決定をやめたかも重要な情報

## チェックリスト

### 作成時

- [ ] タイトルは決定事項を明確に表しているか？
- [ ] ステータスは正しいか？
- [ ] コンテキストに背景と課題が記述されているか？
- [ ] 代替案を検討したか？
- [ ] 決定事項は具体的か？
- [ ] ポジティブとネガティブな結果を両方記述したか？
- [ ] リスクと緩和策を記載したか？
- [ ] チームでレビューしたか？

### レビュー時

- [ ] 決定は合理的か？
- [ ] トレードオフは理解されているか？
- [ ] 代替案は十分に検討されたか？
- [ ] 実装可能か？
- [ ] 既存のADRと矛盾していないか？

## よくある落とし穴

**1. 完璧を求めすぎる**
- ADRは完璧である必要はない
- まずは書いて、後で更新

**2. すべてを記録しようとする**
- 重要な決定のみ記録
- 日常的な実装詳細は不要

**3. 書いて終わり**
- 定期的に見直す
- 状況が変わったら更新

**4. 一人で書く**
- チームでレビュー
- 多様な視点を取り入れる

**5. 曖昧に書く**
- 具体的に記述
- 将来の読者が理解できるように

## ツールとリソース

### ADR作成ツール

**adr-tools:**
```bash
# インストール
brew install adr-tools

# 初期化
adr init docs/adr

# 新しいADR作成
adr new "Use PostgreSQL for main database"

# ADRリスト表示
adr list
```

**log4brains:**
```bash
# インストール
npm install -g log4brains

# 初期化
log4brains init

# 新しいADR作成
log4brains adr new

# Webプレビュー
log4brains preview
```

### テンプレート

- [Michael Nygardのテンプレート](https://github.com/joelparkerhenderson/architecture-decision-record/blob/main/templates/decision-record-template-by-michael-nygard/index.md)
- [MADR（Markdown ADR）](https://adr.github.io/madr/)
- [Y-Statements](https://medium.com/olzzio/y-statements-10eb07b5a177)

## 次のステップ

ADRを導入したら、以下に進みましょう。

1. **チームでADRの文化を作る**
   - 重要な決定はADRで記録する習慣
   - プルリクエストでレビュー
   - 新メンバーのオンボーディングで活用

2. **[BMC](./bmc.md)や[4P分析](./4p_analysis.md)と連携**
   - ビジネス戦略と技術決定を整合させる
   - BMCのリソースや活動をADRで具体化

3. **技術的負債の管理**
   - ADRでトレードオフを可視化
   - 定期的に見直し、リファクタリング計画を立てる

4. **スケーラブルなアーキテクチャの構築**
   - ADRで重要な技術決定を記録
   - チーム全体で技術的コンテキストを共有

---

## 参考資料

- [Michael Nygard「Documenting Architecture Decisions」](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions)
- [ADR GitHub Organization](https://adr.github.io/)
- [ThoughtWorks Technology Radar - ADR](https://www.thoughtworks.com/radar/techniques/lightweight-architecture-decision-records)
