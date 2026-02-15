---
title: テスト戦略
description: 単体・統合・E2E・負荷テスト
sidebar:
  badge: Template
---

:::danger[このドキュメントについて]
このドキュメントはテスト戦略を定義するためのテンプレートです。
テストの種類、カバレッジ目標、テストツールを明確にしてください。
:::

テストピラミッド(ユニット・統合・E2E)、各テストの目的・フレームワーク・実行方法・書き方、パフォーマンステスト(負荷テスト)、ビジュアルリグレッションテスト、ベストプラクティス、CI/CDでの自動実行、カバレッジ目標を定義します。

## テストピラミッド

```
        /\
       /  \  E2E Tests (少数)
      /____\
     /      \
    / Integration Tests (中程度)
   /________\
  /          \
 / Unit Tests (多数)
/______________\
```

---

## ユニットテスト

### 概要

- **目的**: 関数・コンポーネント単位のテスト
- **フレームワーク**: Jest / Vitest
- **カバレッジ目標**: 80%以上

### 実行方法

```bash
# テスト実行
npm run test

# ウォッチモード
npm run test:watch

# カバレッジ
npm run test:coverage
```

### テストの書き方

#### 関数のテスト

```typescript
// utils/math.ts
export function add(a: number, b: number): number {
  return a + b;
}

// utils/math.test.ts
import { add } from './math';

describe('add', () => {
  it('should add two numbers', () => {
    expect(add(1, 2)).toBe(3);
  });

  it('should handle negative numbers', () => {
    expect(add(-1, -2)).toBe(-3);
  });

  it('should handle zero', () => {
    expect(add(0, 5)).toBe(5);
  });
});
```

#### コンポーネントのテスト（React）

```typescript
// components/Button.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { Button } from './Button';

describe('Button', () => {
  it('should render with text', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByText('Click me')).toBeInTheDocument();
  });

  it('should call onClick when clicked', () => {
    const handleClick = jest.fn();
    render(<Button onClick={handleClick}>Click me</Button>);

    fireEvent.click(screen.getByText('Click me'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it('should be disabled when disabled prop is true', () => {
    render(<Button disabled>Click me</Button>);
    expect(screen.getByText('Click me')).toBeDisabled();
  });
});
```

### ベストプラクティス

- **AAA パターン**: Arrange（準備）, Act（実行）, Assert（検証）
- **1テスト1検証**: 各テストケースは1つのことをテスト
- **テストの独立性**: テスト間で状態を共有しない
- **わかりやすい名前**: テストケース名は何をテストするか明確に

---

## 統合テスト

### 概要

- **目的**: 複数のモジュールの連携テスト
- **対象**: APIエンドポイント、データベース連携

### API統合テスト

```typescript
// tests/api/users.test.ts
import request from 'supertest';
import app from '../../src/app';

describe('User API', () => {
  describe('POST /api/users', () => {
    it('should create a new user', async () => {
      const response = await request(app)
        .post('/api/users')
        .send({
          email: 'test@example.com',
          password: 'password123',
          name: 'Test User',
        })
        .expect(201);

      expect(response.body).toHaveProperty('id');
      expect(response.body.email).toBe('test@example.com');
    });

    it('should return 400 for invalid email', async () => {
      await request(app)
        .post('/api/users')
        .send({
          email: 'invalid-email',
          password: 'password123',
        })
        .expect(400);
    });
  });

  describe('GET /api/users/:id', () => {
    it('should return user by id', async () => {
      const response = await request(app)
        .get('/api/users/123')
        .expect(200);

      expect(response.body).toHaveProperty('id', '123');
    });

    it('should return 404 for non-existent user', async () => {
      await request(app)
        .get('/api/users/999')
        .expect(404);
    });
  });
});
```

### データベーステスト

```typescript
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

describe('User Repository', () => {
  beforeEach(async () => {
    // テストデータのクリーンアップ
    await prisma.user.deleteMany();
  });

  afterAll(async () => {
    await prisma.$disconnect();
  });

  it('should create and retrieve user', async () => {
    const user = await prisma.user.create({
      data: {
        email: 'test@example.com',
        name: 'Test User',
      },
    });

    const retrieved = await prisma.user.findUnique({
      where: { id: user.id },
    });

    expect(retrieved).toMatchObject({
      email: 'test@example.com',
      name: 'Test User',
    });
  });
});
```

---

## E2Eテスト

### 概要

- **目的**: ユーザーフロー全体のテスト
- **フレームワーク**: Playwright / Cypress
- **実行環境**: ステージング環境相当

### 実行方法

```bash
# Playwright
npm run test:e2e

# ヘッドレスモード
npm run test:e2e:headless

# 特定のブラウザ
npm run test:e2e -- --project=chromium
```

### テストの書き方（Playwright）

```typescript
// tests/e2e/login.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Login Flow', () => {
  test('should login successfully with valid credentials', async ({ page }) => {
    // ログインページに移動
    await page.goto('/login');

    // フォームに入力
    await page.fill('input[name="email"]', 'user@example.com');
    await page.fill('input[name="password"]', 'password123');

    // ログインボタンをクリック
    await page.click('button[type="submit"]');

    // ダッシュボードに遷移することを確認
    await expect(page).toHaveURL('/dashboard');
    await expect(page.locator('h1')).toContainText('Welcome');
  });

  test('should show error with invalid credentials', async ({ page }) => {
    await page.goto('/login');

    await page.fill('input[name="email"]', 'user@example.com');
    await page.fill('input[name="password"]', 'wrongpassword');
    await page.click('button[type="submit"]');

    // エラーメッセージが表示されることを確認
    await expect(page.locator('.error')).toContainText('Invalid credentials');
  });
});
```

### テストの書き方（Cypress）

```typescript
// cypress/e2e/login.cy.ts
describe('Login Flow', () => {
  beforeEach(() => {
    cy.visit('/login');
  });

  it('should login successfully', () => {
    cy.get('input[name="email"]').type('user@example.com');
    cy.get('input[name="password"]').type('password123');
    cy.get('button[type="submit"]').click();

    cy.url().should('include', '/dashboard');
    cy.contains('h1', 'Welcome').should('be.visible');
  });

  it('should show error with invalid credentials', () => {
    cy.get('input[name="email"]').type('user@example.com');
    cy.get('input[name="password"]').type('wrongpassword');
    cy.get('button[type="submit"]').click();

    cy.contains('.error', 'Invalid credentials').should('be.visible');
  });
});
```

---

## パフォーマンステスト

### 負荷テスト

#### k6を使用した例

```javascript
// tests/load/api-load.js
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  vus: 100, // 仮想ユーザー数
  duration: '30s', // テスト期間
  thresholds: {
    http_req_duration: ['p(95)<200'], // 95%のリクエストが200ms以内
    http_req_failed: ['rate<0.01'], // エラー率1%未満
  },
};

export default function () {
  const response = http.get('https://api.example.com/users');

  check(response, {
    'status is 200': (r) => r.status === 200,
    'response time < 200ms': (r) => r.timings.duration < 200,
  });

  sleep(1);
}
```

実行:
```bash
k6 run tests/load/api-load.js
```

---

## ビジュアルリグレッションテスト

### Chromatic / Percy

```typescript
// tests/visual/button.stories.tsx
import { Button } from '../../src/components/Button';

export default {
  title: 'Components/Button',
  component: Button,
};

export const Primary = () => <Button variant="primary">Primary</Button>;
export const Secondary = () => <Button variant="secondary">Secondary</Button>;
export const Disabled = () => <Button disabled>Disabled</Button>;
```

---

## テストのベストプラクティス

### Do

- ✅ テストはシンプルに保つ
- ✅ 意味のあるテストケース名
- ✅ テストデータは独立性を保つ
- ✅ モックは最小限に
- ✅ CI/CDで自動実行

### Don't

- ❌ 実装の詳細をテストしない
- ❌ 複数のことを1つのテストでチェックしない
- ❌ テスト間で状態を共有しない
- ❌ 外部サービスに依存しない（モックを使用）
- ❌ テストをスキップしない

---

## CI/CDでのテスト実行

### GitHub Actions 例

```yaml
name: Test

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm ci

      - name: Run unit tests
        run: npm run test:coverage

      - name: Run E2E tests
        run: npm run test:e2e

      - name: Upload coverage
        uses: codecov/codecov-action@v3
```

---

## テストカバレッジ目標

| テスト種類 | カバレッジ目標 |
|-----------|---------------|
| ユニットテスト | 80%以上 |
| 統合テスト | 主要APIエンドポイント |
| E2Eテスト | クリティカルなユーザーフロー |

---

## トラブルシューティング

### テストが不安定（Flaky）

- タイムアウトを増やす
- 非同期処理を適切に待つ
- テストデータの独立性を確保

### テストが遅い

- 並列実行を有効化
- 不要なモックを削除
- テストデータベースを最適化
