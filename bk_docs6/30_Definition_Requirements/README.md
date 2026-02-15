---
title: 30_Definition_Requirements - Index
type: requirements
status: draft
role: PdM
tags: [index, requirements, prd]
---

# 30_Definition_Requirements - Index

## カテゴリ概要

プロダクトの機能要件を定義し、リリース計画を管理するドキュメント群です。
開発チームが実装する内容を明確化し、全チームが共通理解を持つための文書を管理します。

---

## 対象職種

- **主要**: PdM, EM, Designer
- **関係者**: Engineer, PMM, QA

## 更新頻度

- **PRD Index**: 週次または月次
- **PRD**: 機能ごと（開発開始前に完成）
- **Release Notes**: リリースごと

## ドキュメント一覧

### PRD Index（要件定義一覧）

すべてのPRDを一元管理し、進行状況を可視化します。

| ファイル                                   | 説明                 | ステータス |
| ------------------------------------------ | -------------------- | ---------- |
| [31.00_PRD_README.md](31.00_PRD_README.md) | 全機能の要件定義一覧 | アクティブ |

**使用タイミング**: 常時参照・更新

### PRD（Product Requirements Document）

個別機能の詳細な要件を定義します。

| ファイル                                                             | 説明                       | ステータス |
| -------------------------------------------------------------------- | -------------------------- | ---------- |
| [31.01_PRD_SSO_Login.md](31.01_PRD_SSO_Login.md)                     | SSOログイン要件定義        | 開発中     |
| [31.02_PRD_Analytics_Dashboard.md](31.02_PRD_Analytics_Dashboard.md) | 分析ダッシュボード要件定義 | Draft      |

**使用タイミング**: 新機能開発時、仕様変更時

### Release Notes（リリースノート）

各バージョンのリリース内容をユーザー向けに分かりやすく伝えます。

| ファイル                                                               | 説明                 | ステータス |
| ---------------------------------------------------------------------- | -------------------- | ---------- |
| [32.01_Release_Notes_Draft_v2.0.md](32.01_Release_Notes_Draft_v2.0.md) | v2.0リリースノート案 | Draft      |

**使用タイミング**: リリース前・リリース時

## ドキュメント間の関係

```
Roadmap (年間計画)
    ↓
PRD Index (全体管理)
    ↓
PRD (個別機能) → Tech Spec → Design Handoff
    ↓
Release Notes (リリース)
```

## 使い分けガイド

### PRD Index vs PRD

- **PRD Index**: 全機能の一覧と進捗状況を管理。週次/月次で更新。
- **PRD**: 個別機能の詳細要件。1機能につき1ファイル。

### PRD vs Tech Spec

- **PRD**: **何を**実現するか（ビジネス要件、ユーザー要件）
- **Tech Spec**: **どのように**実装するか（技術的設計、アーキテクチャ）

### Release Notes vs PRD

- **PRD**: 内部向け、詳細な仕様
- **Release Notes**: ユーザー向け、価値と使い方を簡潔に

## 新規作成ガイド

### PRDを作成する場合

1. テンプレート: このカテゴリ内のファイルをテンプレートとして使用
2. 前提条件:
   - User Interviewsまたは顧客要望が明確
   - Roadmapに含まれている
3. 必須セクション:
   - 概要、背景、目標・KPI
   - ユーザーストーリー
   - 機能要件（Must/Should/Could）
   - UI/UX要件
   - 承認
4. レビュープロセス:
   - Draft作成 → EM/Designerレビュー → 承認 → 開発開始

### PRD Indexを更新する場合

1. 新規PRD作成時にエントリー追加
2. ステータス変更時に更新（Draft → Review → Approved → In Development → Released）
3. 週次/月次で全体レビュー

### Release Notesを作成する場合

1. テンプレート: このカテゴリ内のファイルをテンプレートとして使用
2. リリース1週間前に作成開始
3. 含める内容:
   - 新機能（ユーザー価値を強調）
   - 改善点（Before/After）
   - バグ修正（影響範囲を明記）
   - 既知の問題
4. レビュー: PdM → PMM → 公開

## PRD作成のベストプラクティス

### 良いPRDの特徴

- **明確なゴール**: 「なぜこの機能が必要か」が明確
- **測定可能なKPI**: 成功の定義が定量的
- **優先順位付け**: Must/Should/Couldで明確に分類
- **具体的なユーザーストーリー**: ペルソナベースの実例
- **エッジケースの考慮**: エラーハンドリング、空状態など

### よくある失敗パターン

- ❌ **How（実装方法）を書きすぎる** → エンジニアの裁量を狭める
- ❌ **曖昧な表現が多い** → 「柔軟に」「適切に」などは避ける
- ❌ **成功指標がない** → リリース後の評価ができない
- ❌ **ステークホルダーの合意がない** → 後から大幅な変更が発生

## よくある質問

### Q: PRDはどこまで詳細に書くべきですか？

**A**: エンジニアが実装を開始できるレベルの詳細度が目安です。ただし、実装方法（How）は書かず、要件（What）に集中してください。

### Q: PRDの承認者は誰ですか？

**A**: 最低限、PdM、EM、Design Leadの3名。PMM（マーケ要素が強い場合）、TechLead（技術的に複雑な場合）も含めることがあります。

### Q: PRDは途中で変更できますか？

**A**: はい。ただし、大きな変更の場合は再度承認プロセスを経てください。軽微な変更は更新履歴に記録して関係者に共有。

### Q: Release Notesはいつ公開しますか？

**A**: リリース当日またはリリース直後。Draft版は1週間前から内部共有し、CSやSalesが準備できるようにしてください。

---

## 関連ドキュメント

- [10_Strategy_Planning](../10_Strategy_Planning/README.md) - Roadmapとの連携
- [20_Discovery_Research](../20_Discovery_Research/README.md) - ユーザーニーズの確認
- [40_Design_UX](../40_Design_UX/README.md) - デザイン仕様
- [50_Engineering_Tech](../50_Engineering_Tech/README.md) - 技術仕様
- [60_Marketing_GTM](../60_Marketing_GTM/README.md) - ローンチ計画
