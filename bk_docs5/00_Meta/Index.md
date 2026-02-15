# 統合プロダクトドキュメント - 全体Index

> **このドキュメントについて**
> プロダクトドキュメント全体の最上位エントリーポイントです。
> 新規メンバーのオンボーディング、ドキュメント検索、全体構造の把握に活用してください。

---

## 📚 全カテゴリ一覧

| カテゴリ | 説明 | 主な利用職種 |
| --- | --- | --- |
| **[00_Meta](#00_meta-このカテゴリ)** | 運用ルール、用語集、テンプレート | 全員 |
| **[10_Strategy_Planning](../10_Strategy_Planning/Index.md)** | ビジョン、ロードマップ、OKRs | PdM, EM, PMM |
| **[20_Discovery_Research](../20_Discovery_Research/Index.md)** | ユーザー調査、市場分析 | PdM, Designer, PMM |
| **[30_Definition_Requirements](../30_Definition_Requirements/Index.md)** | PRD、リリースノート | PdM, EM, Designer |
| **[40_Design_UX](../40_Design_UX/Index.md)** | デザイン仕様、ガイドライン | Designer, Engineer |
| **[50_Engineering_Tech](../50_Engineering_Tech/Index.md)** | 技術仕様、API、ADR | Engineer, TechLead |
| **[60_Marketing_GTM](../60_Marketing_GTM/Index.md)** | マーケティング、GTM戦略 | PMM, Sales |
| **[70_Customer_Enablement](../70_Customer_Enablement/Index.md)** | 営業資料、ヘルプ、FAQ | Sales, CS |

---

## 00_Meta（このカテゴリ）

### カテゴリ概要
プロダクトドキュメント全体の運用ルール、チーム体制、用語集など、メタ情報を管理します。

### 対象職種
- **主要**: 全員
- **管理者**: Librarian（ドキュメント管理者）

### 更新頻度
- **Team Charter**: 組織変更時、半年〜1年ごとに見直し
- **Glossary**: 新規用語追加時に随時更新

---

## 00_Meta内のドキュメント一覧

### チーム憲章
チームの役割分担と意思決定フローを定義します。

| ファイル | 説明 | ステータス |
| --- | --- | --- |
| [00.01_Team_Charter.md](00.01_Team_Charter.md) | チームの役割分担と承認フロー | アクティブ |

**使用タイミング**: チーム編成時、意思決定プロセス確認時

---

### 用語集
社内用語・略語を統一管理します。

| ファイル | 説明 | ステータス |
| --- | --- | --- |
| [00.02_Glossary.md](00.02_Glossary.md) | 社内用語集・略語集 | アクティブ |

**使用タイミング**: 新メンバーのオンボーディング、用語の確認時

---


## ドキュメント構造マップ

```
00_Meta (このカテゴリ)
├── Index, Team Charter, Glossary

10_Strategy_Planning
├── Vision/Mission, Roadmap, OKRs
    ↓
20_Discovery_Research
├── User Interviews, Persona, Market Trends, Churn Analysis
    ↓
30_Definition_Requirements
├── PRD Index, PRD, Release Notes
    ↓ ↓
40_Design_UX         50_Engineering_Tech
├── Design Handoff   ├── Tech Spec
├── Design System    ├── API Spec
└── Tone and Voice   ├── ADR
                     └── Post-Mortem
    ↓
60_Marketing_GTM
├── Messaging House, FAB Matrix
├── Launch Plan, Battlecard
    ↓
70_Customer_Enablement
├── Sales Pitch, Help Article, FAQ
```

---

## 新規メンバーのオンボーディング

### 初日にやること
1. [Index.md](Index.md) を読む（全体構造の理解）
2. [00.01_Team_Charter.md](00.01_Team_Charter.md) を読む（役割と責任の理解）
3. [00.02_Glossary.md](00.02_Glossary.md) を読む（用語の理解）
4. 自分の職種に関連するカテゴリのIndex.mdを読む

### 初週にやること
- 自分の職種で主に使うテンプレートを確認
- 既存のドキュメントをいくつか読んでフォーマットを理解

---

## ドキュメント管理のルール

### 命名規則
```
[カテゴリ番号].[サブカテゴリ番号]_[ドキュメント名].md

例:
- 31.01_PRD_SSO_Login.md
- 52.01_ADR_Database_Choice.md
```

### ファイル配置ルール
- 各カテゴリディレクトリに適切に配置
- Index.mdを更新してリンクを追加
- 関連ドキュメントへの相互リンクを設定

### プレースホルダーの扱い
ドキュメント内のプレースホルダーは、実際の情報に置き換えて使用してください：
- **`<Function_Name>`、`<Persona_Name>`等**: ファイル名のプレースホルダー。コピー時に実際の名前に変更
- **`[Name]`**: 担当者名を記入（例: "山田太郎"）
- **`[Link]`、`(link)`**: 実際のURLまたは相対パスに置き換え（例: `../30_Definition_Requirements/31.01_PRD_SSO.md`）
- **`MM/DD`、`YYYY/MM/DD`**: 実際の日付を記入（例: 2025/01/19）
- **`<!-- コメント -->`**: 記入ガイド。記入後はコメントを削除

### バージョン管理
- Gitでバージョン管理
- コミットメッセージは明確に（例: "PRD-031: SSO要件を追加"）
- 更新履歴はGitの履歴で管理（ドキュメント内の更新履歴セクションは不要）

### レビュープロセス
1. Draft作成
2. 関係者レビュー（各テンプレートの「承認」セクション参照）
3. 承認
4. マージ・公開
5. 関係者への共有

---

## よくある質問

### Q: どのドキュメントから読めばいいですか？
**A**: 職種別の推奨読書順序：

**PdM:**
1. Vision/Mission → Roadmap → OKRs
2. User Interviews、Persona
3. PRD Index、PRDテンプレート
4. Messaging House

**Engineer:**
1. Roadmap → PRD Index
2. Tech Spec、API Spec、ADRテンプレート
3. Design Handoff、Design System

**Designer:**
1. Vision/Mission → Persona
2. Design System、Tone and Voice
3. Design Handoffテンプレート

**PMM:**
1. Vision/Mission → Persona
2. Messaging House、FAB Matrix
3. Launch Plan、Battlecard

**Sales/CS:**
1. Messaging House、FAB Matrix
2. Sales Pitch、Help Article、FAQ
3. Battlecard

### Q: ドキュメントが見つからない時は？
**A**:
1. 各カテゴリのIndex.mdを確認
2. [00.00_Index.md](00.00_Index.md) の検索ガイドを参照
3. Gitで検索（ファイル名、内容）
4. Librarian（ドキュメント管理者）に相談

### Q: ドキュメントを新規作成する時は？
**A**:
1. 該当カテゴリ内の既存ファイルをテンプレートとして使用
2. ファイルをコピーしてプレースホルダーを実際の内容に置き換え
3. プレースホルダーを埋める
4. カテゴリのIndex.mdを更新
5. レビュー・承認プロセスを経る

### Q: 既存ドキュメントを更新する時は？
**A**:
1. 該当ドキュメントを直接編集
2. 「更新履歴」セクションに変更内容を記録
3. 大きな変更の場合は関係者にレビュー依頼
4. Gitでコミット

---

## Librarian（ドキュメント管理者）の役割

### 主な責任
- ドキュメント構造の維持
- テンプレートの更新・追加
- 定期的なドキュメントレビュー（古い情報の削除）
- 新メンバーのオンボーディング支援
- ベストプラクティスの共有

### 定期タスク
- **月次**: 各カテゴリのIndex.mdが最新か確認
- **四半期**: テンプレートの改善レビュー
- **半年**: ドキュメント全体の棚卸し

---

## サポート・問い合わせ

### ドキュメント管理に関する質問
- **Librarian**: [Name]
- **Slack**: #product-docs
- **問い合わせフォーム**: [Link]

### ツール・システムの問い合わせ
- **Git/GitHub**: [Link to Wiki]
- **Markdown記法**: [Link to Guide]

---

## 関連ドキュメント
- [10_Strategy_Planning](../10_Strategy_Planning/Index.md)
- [20_Discovery_Research](../20_Discovery_Research/Index.md)
- [30_Definition_Requirements](../30_Definition_Requirements/Index.md)
- [40_Design_UX](../40_Design_UX/Index.md)
- [50_Engineering_Tech](../50_Engineering_Tech/Index.md)
- [60_Marketing_GTM](../60_Marketing_GTM/Index.md)
- [70_Customer_Enablement](../70_Customer_Enablement/Index.md)
