---
title: テスト戦略
description: テスト方針と実装ガイドライン
---

品質を担保しつつ開発速度を維持するために、テストピラミッドに基づいた戦略を採用します。

## テストピラミッドと目標

| レイヤー | ツール | カバレッジ目標 | 優先度 |
| -------- | ------ | -------------- | ------ |
| **E2E** | Playwright | 主要フロー 100% | 高 |
| **Integration** | Vitest | API 80%+ | 高 |
| **Unit** | Vitest | ビジネスロジック 90%+ | 最高 |

## 技術スタック

- **Unit/Integration**: Vitest
- **E2E**: Playwright
- **Component**: React Testing Library
- **Mock**: MSW (Mock Service Worker)

## テスト実装ガイドライン

### 1. ユニットテスト (Unit Tests)

ユーティリティ関数、カスタムフック、純粋なビジネスロジックを対象にします。

```typescript
describe("formatDate", () => {
  it("日本語フォーマットで日付を返す", () => {
    expect(formatDate(new Date("2025-01-01"))).toBe("2025年1月1日");
  });
});
```

### 2. 統合テスト (Integration Tests)

Hono APIエンドポイントや、DBを含む一連のフローをテストします。
`hono/testing` の `testClient` や、インメモリDBを活用します。

```typescript
it("小説一覧を取得できる", async () => {
  const res = await client.index.$get();
  expect(res.status).toBe(200);
  const data = await res.json();
  expect(data.data).toHaveLength(2);
});
```

### 3. E2Eテスト (End-to-End)

ユーザーの重要体験（登録、投稿、閲覧）をブラウザ環境でテストします。

**必須シナリオ**:
- ユーザー登録・ログインフロー
- 作品の投稿・公開フロー
- 読者の作品検索・閲覧・ブックマークフロー

## CI/CD 連携

- PR作成時に Unit/Integration テストを自動実行。
- develop/main ブランチマージ時に E2E テストを実行。
- カバレッジ低下を防ぐためのチェックを導入。
