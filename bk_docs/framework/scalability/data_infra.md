---
title: データ基盤（Data Infrastructure）
---

> このドキュメントは、NSM/KPIをリアルタイムで追跡し、データドリブンな意思決定を可能にするデータ基盤の設計ガイドです。

## なぜデータ基盤が必要か

**課題:**
- NSM/KPIは定義したが、どう計測するか分からない
- データが散在していて、全体像が見えない
- 意思決定に時間がかかる（データ取得に数日）
- 施策の効果が測定できない

**データ基盤が解決すること:**
- リアルタイムでのKPI追跡
- データに基づく迅速な意思決定
- 施策の効果測定と最適化
- チーム全体でのデータ共有

## データ基盤の全体像

<div style="overflow-x: auto; border: 2px solid #333; display: inline-block; width: 100%;">
<table style="width: 100%; border-collapse: collapse; border-radius: 0;">
  <tr>
    <td style="border: 1px solid #ddd; padding: 15px; width: 20%; vertical-align: top; height: 100px;">
      <strong>1. 計測</strong><br/>
      <strong>Instrumentation</strong><br/><br/>
      ユーザー行動やシステムイベントを記録
    </td>
    <td style="border: 1px solid #ddd; padding: 15px; width: 20%; vertical-align: top; height: 100px;">
      <strong>2. 収集</strong><br/>
      <strong>Collection</strong><br/><br/>
      データを集約し、整形
    </td>
    <td style="border: 1px solid #ddd; padding: 15px; width: 20%; vertical-align: top; height: 100px;">
      <strong>3. 保存</strong><br/>
      <strong>Storage</strong><br/><br/>
      データを安全に保管
    </td>
    <td style="border: 1px solid #ddd; padding: 15px; width: 20%; vertical-align: top; height: 100px;">
      <strong>4. 分析</strong><br/>
      <strong>Analysis</strong><br/><br/>
      メトリクスを計算
    </td>
    <td style="border: 1px solid #ddd; padding: 15px; width: 20%; vertical-align: top; height: 100px;">
      <strong>5. 可視化</strong><br/>
      <strong>Visualization</strong><br/><br/>
      ダッシュボードで表示
    </td>
  </tr>
</table>
</div>

**データの流れ:**
```
ユーザー行動 → 計測 → 収集 → 保存 → 分析 → ダッシュボード → 意思決定
```

## 主要なコンポーネント

### 1. 計測（Instrumentation）

**何を計測するか:**
- ユーザー行動（ページ閲覧、ボタンクリック、機能利用）
- ビジネスイベント（サインアップ、購入、解約）
- システムメトリクス（レスポンス時間、エラー率）

**計測方法:**
- トラッキングコード（JavaScript SDK）
- アプリ内イベント（モバイルSDK）
- サーバーサイドログ

### 2. 収集（Collection）

**データ収集の仕組み:**
- イベントストリーミング
- バッチ処理
- リアルタイムパイプライン

**データの整形:**
- クリーニング（重複削除、ノイズ除去）
- 標準化（フォーマット統一）
- 検証（データ品質チェック）

### 3. 保存（Storage）

**データの保管場所:**
- データウェアハウス（分析用）
- データレイク（生データ保管）
- OLTP データベース（リアルタイム処理）

**考慮事項:**
- スケーラビリティ（データ量の増加に対応）
- コスト（ストレージとクエリ費用）
- セキュリティ（アクセス制御、暗号化）

### 4. 分析（Analysis）

**メトリクスの計算:**
- NSM/KPIの集計
- コホート分析（リテンション率）
- ファネル分析（コンバージョン率）
- セグメント別分析

**分析手法:**
- SQL クエリ
- BI ツール
- データサイエンス（機械学習、予測モデル）

### 5. 可視化（Visualization）

**ダッシュボードの種類:**
- リアルタイムダッシュボード（現在の状況）
- 経営ダッシュボード（KPIサマリー）
- 詳細分析ダッシュボード（深掘り）

**ダッシュボードの要素:**
- NSMの大きな表示
- Input Metrics（KPI）のトレンド
- アラート（異常値の通知）
- ドリルダウン（詳細への掘り下げ）

## ツールとサービス

### プロダクト分析ツール

- **Google Analytics**: Web解析の定番
- **Mixpanel**: イベントベース分析
- **Amplitude**: プロダクト分析特化
- **Segment**: データ収集の統合プラットフォーム

### データ基盤サービス

- **Google BigQuery**: クラウドデータウェアハウス
- **Amazon Redshift**: AWSのデータウェアハウス
- **Snowflake**: マルチクラウド対応
- **Databricks**: データレイクとAI/ML統合

### 可視化ツール

- **Grafana**: リアルタイムダッシュボード
- **Looker**: BIツール（データ探索）
- **Tableau**: 高度なビジュアライゼーション
- **Metabase**: オープンソースBI
- **Google Data Studio**: 無料ダッシュボード

### 選択基準

**初期段階（MVP）:**
- Google Analytics（無料、すぐ使える）
- Mixpanel / Amplitude（フリープランで開始）
- Google Data Studio（無料、GA連携）

**成長段階:**
- Segment（データ収集の統合）
- BigQuery / Snowflake（スケーラブルなストレージ）
- Looker / Tableau（高度な分析）

**選択時の考慮点:**
- **コスト**: 初期費用、月額、データ量課金
- **スケーラビリティ**: データ量増加への対応
- **統合性**: 既存ツールとの連携
- **学習コスト**: チームの習熟度

## 構築のステップ

### Step 1: 追跡すべき指標の定義

- NSM/KPIを明確にする
- 各指標の計算式を定義
- どのイベントが必要かをリストアップ

### Step 2: イベント設計

- イベント名の命名規則
- イベントプロパティ（属性）の定義
- トラッキング計画書の作成

### Step 3: 計測の実装

- トラッキングコードの埋め込み
- イベント送信のテスト
- データ品質の確認

### Step 4: データパイプラインの構築

- データ収集の自動化
- ETL（抽出、変換、ロード）の設定
- データウェアハウスへの連携

### Step 5: ダッシュボードの作成

- NSMダッシュボードの作成
- KPIダッシュボードの作成
- チーム全体でのアクセス設定

### Step 6: 検証と改善

- データの正確性を検証
- ダッシュボードの改善
- 定期的なレビューと最適化

## データ基盤の成熟度

### レベル1: Basic（基本）

- Google Analyticsでページビューを追跡
- 手動でレポート作成
- 週次/月次でのレビュー

### レベル2: Intermediate（中級）

- イベントベース分析（Mixpanel、Amplitude）
- 自動化されたダッシュボード
- 日次でのKPI追跡
- セグメント別分析

### レベル3: Advanced（上級）

- データウェアハウス統合（BigQuery、Snowflake）
- リアルタイムダッシュボード
- 予測分析、機械学習
- A/Bテストプラットフォーム
- データサイエンスチーム

## チェックリスト

### 計画段階

- [ ] NSM/KPIを定義したか？
- [ ] 追跡すべきイベントをリストアップしたか？
- [ ] ツールを選定したか？
- [ ] 予算は確保されているか？

### 実装段階

- [ ] トラッキングコードを実装したか？
- [ ] データ収集は正常に動作しているか？
- [ ] データは適切に保存されているか？
- [ ] ダッシュボードで可視化できているか？

### 運用段階

- [ ] チーム全体がアクセスできるか？
- [ ] データ品質は担保されているか？
- [ ] 定期的にレビューしているか？
- [ ] プライバシーとセキュリティは考慮されているか？

### コンプライアンス

- [ ] 個人情報保護（GDPR、個人情報保護法）に準拠しているか？
- [ ] ユーザー同意を取得しているか？
- [ ] データの保持期間を設定しているか？
- [ ] アクセス制御が適切か？

## 次のステップ

1. **[North Star Metric](./northstar.md)** - 最重要指標を定義
2. **[KPI](./kpi.md)** - 追跡すべき指標を設定
3. **[ADR](./adr.md)** - データ基盤の技術決定を記録
4. **ダッシュボードの構築** - 実際にダッシュボードを作成し、チームで共有
5. **継続的な改善** - データを活用して、プロダクトを改善
