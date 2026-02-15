---
title: テストケース（Gherkin形式）
---

> このドキュメントは、ユーザーストーリーを実行可能なGherkin形式のテストケースに変換し、品質を保証するためのテンプレートです。

## 📋 プロジェクト情報

| 項目 | 内容 |
|------|------|
| **関連ドキュメント** | [ジャーニーマップ](./journey_map.md) / [ユーザーストーリー](./user_stories.md) |
| **テスト自動化ツール** | [例: Cucumber, SpecFlow, Behave など] |

---

## 🧪 テストケース一覧

### Feature 1: [機能名を記入]

**ユーザーストーリー:**
```
ユーザーとして、
[何かをしたい]
なぜなら、[理由]だから
```

**優先度:** 高 / 中 / 低
**ステータス:** 未実装 / 実装済み / テスト中 / 完了
**関連ジャーニーフェーズ:** [例: 認知、検討、登録など]

```gherkin
Feature: [機能名]
  As a [ユーザーの種類]
  I want to [やりたいこと]
  So that [目的・理由]

  Scenario: [シナリオ名]
    Given [前提条件]
    When [アクション]
    Then [期待結果]

  Scenario: [別のシナリオ名]
    Given [前提条件]
    When [アクション]
    And [追加のアクション]
    Then [期待結果]
    And [追加の期待結果]
```

---

### Feature 2: [機能名を記入]

**ユーザーストーリー:**
```
ユーザーとして、
[何かをしたい]
なぜなら、[理由]だから
```

**優先度:** 高 / 中 / 低
**ステータス:** 未実装 / 実装済み / テスト中 / 完了
**関連ジャーニーフェーズ:** [例: 初回利用]

```gherkin
Feature: [機能名]

  Scenario: [シナリオ名]
    Given [前提条件]
    When [アクション]
    Then [期待結果]
```

---

## 📊 テストケース管理表

| Feature名 | シナリオ数 | 優先度 | ステータス | 担当者 | 自動化 | 備考 |
|-----------|-----------|--------|-----------|--------|--------|------|
| [Feature名] | X | 高 | 完了 | [名前] | ✅ | [備考] |
| [Feature名] | X | 中 | テスト中 | [名前] | 🔄 | [備考] |
| [Feature名] | X | 低 | 未実装 | [名前] | ❌ | [備考] |

**凡例:**
- ✅: 自動化済み
- 🔄: 自動化中
- ❌: 手動テストのみ

---

# 📖 使い方ガイド

## Gherkin形式とは

Gherkin（ガーキン）は、**人間が読めて、かつ自動テストツールで実行できる形式**で要件やテストケースを記述するための言語です。ビジネス関係者、開発者、QAが同じ言語で仕様を共有できるため、BDD（振る舞い駆動開発）の中核となります。

### Gherkinの特徴

- 📖 **読みやすい**: 技術知識がなくても理解できる
- 🤖 **実行可能**: Cucumber、SpecFlowなどのツールで自動テスト化
- 🌍 **多言語対応**: 英語だけでなく、日本語でも記述可能
- 🔗 **Living Documentation**: コードと仕様が同期し、常に最新

## Gherkinの基本構文

### Feature（機能）

```gherkin
Feature: ユーザー登録
  As a 新規ユーザー
  I want to アカウントを作成したい
  So that サービスを利用できる
```

- **Feature**: テストする機能の名前
- **As a / I want to / So that**: ユーザーストーリー形式（省略可）

### Scenario（シナリオ）

```gherkin
Scenario: メールアドレスで登録する
  Given ユーザーが登録ページにアクセスしている
  When メールアドレスとパスワードを入力する
  And 「登録」ボタンをクリックする
  Then アカウントが作成される
  And ウェルカムメールが送信される
```

- **Given**: 前提条件（テスト開始時の状態）
- **When**: アクション（ユーザーが行う操作）
- **Then**: 期待結果（何が起こるべきか）
- **And / But**: 複数の条件やアクションを繋ぐ

### Scenario Outline（シナリオアウトライン）

複数のデータパターンで同じテストを実行したい場合に使用します。

```gherkin
Scenario Outline: 異なるメールアドレス形式で登録する
  Given ユーザーが登録ページにアクセスしている
  When "<email>" を入力する
  Then <結果>

  Examples:
    | email               | 結果                 |
    | user@example.com    | アカウントが作成される |
    | invalid-email       | エラーメッセージが表示される |
    | user@              | エラーメッセージが表示される |
```

### Background（背景）

すべてのシナリオで共通の前提条件を記述します。

```gherkin
Feature: ダッシュボード

  Background:
    Given ユーザーがログインしている
    And ダッシュボードページを表示している

  Scenario: プロジェクト一覧を表示
    When ページをリロードする
    Then 自分のプロジェクト一覧が表示される

  Scenario: 新規プロジェクトを作成
    When 「新規プロジェクト」ボタンをクリック
    Then プロジェクト作成フォームが表示される
```

## ジャーニーマップからテストケースへの変換

[ジャーニーマップ](./journey_map.md)で特定したペインポイントと改善機会を、テストケースに変換します。

### ステップ1: ペインポイントからユーザーストーリーへ

ペインポイントを特定し、ユーザーストーリー形式で記述します。

```
ユーザーとして、
[何かをしたい]
なぜなら、[理由]だから
```

### ステップ2: ユーザーストーリーからGherkinへ

ユーザーストーリーをGherkin形式のテストケースに変換します。

## テスト自動化ツールとの連携

### Cucumber (JavaScript/TypeScript)

```javascript
// features/step_definitions/registration.steps.js
const { Given, When, Then } = require('@cucumber/cucumber');

Given('ユーザーがサインアップページにアクセスしている', async function () {
  await this.page.goto('https://example.com/signup');
});

When('メールアドレス {string} を入力する', async function (email) {
  await this.page.fill('#email', email);
});

Then('アカウントが作成される', async function () {
  const successMessage = await this.page.textContent('.success');
  assert.equal(successMessage, 'アカウントが作成されました');
});
```

### SpecFlow (C#)

```csharp
// Steps/RegistrationSteps.cs
[Binding]
public class RegistrationSteps
{
    [Given(@"ユーザーがサインアップページにアクセスしている")]
    public void ユーザーがサインアップページにアクセスしている()
    {
        Driver.Navigate().GoToUrl("https://example.com/signup");
    }

    [When(@"メールアドレス ""(.*)"" を入力する")]
    public void メールアドレスを入力する(string email)
    {
        Driver.FindElement(By.Id("email")).SendKeys(email);
    }
}
```

## 次のステップ

テストケースを作成したら、以下のステップで品質保証を自動化します。

1. **テスト自動化ツールの導入**
   - Cucumber, SpecFlow, Behave などを選択
   - プロジェクトにセットアップ

2. **ステップ定義の実装**
   - Gherkin のステップをコードに変換
   - 上記「テスト自動化ツールとの連携」を参照

3. **CI/CDパイプラインへの統合**
   - GitHub Actions, GitLab CI, Jenkins などに組み込み
   - プルリクエストごとに自動テスト実行

4. **テスト駆動開発（BDD）**
   - テストケースを先に書く
   - テストを通すように実装
   - リファクタリング

5. **継続的な改善**
   - テスト結果を分析
   - 失敗したテストから学ぶ
   - カバレッジを向上
　
