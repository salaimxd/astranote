---
title: テスト戦略
description: Star Novel のテスト方針と実装ガイドライン
---

## 概要

Star Novel プロジェクトでは、品質を担保しつつ開発速度を維持するために、テストピラミッドに基づいたテスト戦略を採用します。

### テストピラミッド

```
                    ┌─────────┐
                    │  E2E   │  少数・重要フロー
                    │ Tests  │
                   ─┴───────┴─
                  ┌───────────┐
                  │Integration│  API・DB統合
                  │  Tests    │
                 ─┴─────────┴─
                ┌─────────────┐
                │    Unit     │  多数・高速
                │   Tests     │
               ─┴───────────┴─
```

### テストカバレッジ目標

| レイヤー | 目標カバレッジ | 優先度 |
|---------|--------------|--------|
| ビジネスロジック | 90%+ | 最高 |
| API エンドポイント | 80%+ | 高 |
| UI コンポーネント | 70%+ | 中 |
| E2E | 主要フロー100% | 高 |

---

## テストツール

### フロントエンド

| ツール | 用途 |
|--------|------|
| Vitest | ユニットテスト |
| React Testing Library | コンポーネントテスト |
| Playwright | E2E テスト |
| MSW (Mock Service Worker) | API モック |

### バックエンド

| ツール | 用途 |
|--------|------|
| Vitest | ユニット・統合テスト |
| Miniflare | Cloudflare Workers ローカルテスト |
| Testcontainers | DB テスト（オプション） |

---

## ユニットテスト

### テスト対象

- ユーティリティ関数
- ビジネスロジック
- カスタムフック
- バリデーション
- 型ガード

### ユーティリティ関数のテスト

```typescript
// lib/utils/formatDate.ts
export function formatDate(date: Date, locale: string = "ja"): string {
  return new Intl.DateTimeFormat(locale, {
    year: "numeric",
    month: "long",
    day: "numeric",
  }).format(date);
}

export function formatRelativeTime(date: Date): string {
  const now = new Date();
  const diff = now.getTime() - date.getTime();
  const minutes = Math.floor(diff / 60000);
  const hours = Math.floor(diff / 3600000);
  const days = Math.floor(diff / 86400000);

  if (minutes < 1) return "たった今";
  if (minutes < 60) return `${minutes}分前`;
  if (hours < 24) return `${hours}時間前`;
  if (days < 7) return `${days}日前`;
  return formatDate(date);
}
```

```typescript
// lib/utils/formatDate.test.ts
import { describe, it, expect, vi, beforeEach, afterEach } from "vitest";
import { formatDate, formatRelativeTime } from "./formatDate";

describe("formatDate", () => {
  it("日本語フォーマットで日付を返す", () => {
    const date = new Date("2025-01-15");
    expect(formatDate(date, "ja")).toBe("2025年1月15日");
  });

  it("英語フォーマットで日付を返す", () => {
    const date = new Date("2025-01-15");
    expect(formatDate(date, "en")).toBe("January 15, 2025");
  });
});

describe("formatRelativeTime", () => {
  beforeEach(() => {
    vi.useFakeTimers();
    vi.setSystemTime(new Date("2025-01-15T12:00:00Z"));
  });

  afterEach(() => {
    vi.useRealTimers();
  });

  it("1分未満は「たった今」を返す", () => {
    const date = new Date("2025-01-15T11:59:30Z");
    expect(formatRelativeTime(date)).toBe("たった今");
  });

  it("1時間未満は「X分前」を返す", () => {
    const date = new Date("2025-01-15T11:30:00Z");
    expect(formatRelativeTime(date)).toBe("30分前");
  });

  it("24時間未満は「X時間前」を返す", () => {
    const date = new Date("2025-01-15T06:00:00Z");
    expect(formatRelativeTime(date)).toBe("6時間前");
  });

  it("7日未満は「X日前」を返す", () => {
    const date = new Date("2025-01-12T12:00:00Z");
    expect(formatRelativeTime(date)).toBe("3日前");
  });

  it("7日以上は日付フォーマットを返す", () => {
    const date = new Date("2025-01-01T12:00:00Z");
    expect(formatRelativeTime(date)).toBe("2025年1月1日");
  });
});
```

### ビジネスロジックのテスト

```typescript
// services/translation/translator.ts
export interface TranslationResult {
  translatedText: string;
  cost: number;
  model: string;
}

export async function translateText(
  text: string,
  sourceLang: string,
  targetLang: string,
  glossary: Map<string, string>
): Promise<TranslationResult> {
  // 前処理: 用語集の適用
  let processedText = applyGlossary(text, glossary);

  // 翻訳実行
  const result = await callTranslationApi(processedText, sourceLang, targetLang);

  // 後処理: ルビの復元など
  const finalText = postProcess(result.text, sourceLang, targetLang);

  return {
    translatedText: finalText,
    cost: result.cost,
    model: result.model,
  };
}

export function applyGlossary(text: string, glossary: Map<string, string>): string {
  let result = text;
  for (const [term, replacement] of glossary) {
    result = result.replaceAll(term, `[[GLOSS:${replacement}]]`);
  }
  return result;
}
```

```typescript
// services/translation/translator.test.ts
import { describe, it, expect, vi } from "vitest";
import { applyGlossary, translateText } from "./translator";

// API呼び出しをモック
vi.mock("./api", () => ({
  callTranslationApi: vi.fn().mockResolvedValue({
    text: "Translated text with [[GLOSS:Demon Lord Castle]]",
    cost: 0.001,
    model: "claude-3.5-sonnet",
  }),
}));

describe("applyGlossary", () => {
  it("用語をプレースホルダーに置換する", () => {
    const text = "魔王城に向かう勇者";
    const glossary = new Map([["魔王城", "Demon Lord Castle"]]);

    const result = applyGlossary(text, glossary);

    expect(result).toBe("[[GLOSS:Demon Lord Castle]]に向かう勇者");
  });

  it("複数の用語を置換する", () => {
    const text = "勇者アリスは魔王城へ向かった";
    const glossary = new Map([
      ["勇者アリス", "Hero Alice"],
      ["魔王城", "Demon Lord Castle"],
    ]);

    const result = applyGlossary(text, glossary);

    expect(result).toContain("[[GLOSS:Hero Alice]]");
    expect(result).toContain("[[GLOSS:Demon Lord Castle]]");
  });

  it("空の用語集では元のテキストを返す", () => {
    const text = "魔王城に向かう勇者";
    const glossary = new Map<string, string>();

    const result = applyGlossary(text, glossary);

    expect(result).toBe(text);
  });
});

describe("translateText", () => {
  it("翻訳結果を返す", async () => {
    const text = "テスト文章";
    const glossary = new Map<string, string>();

    const result = await translateText(text, "ja", "en", glossary);

    expect(result).toHaveProperty("translatedText");
    expect(result).toHaveProperty("cost");
    expect(result).toHaveProperty("model");
  });
});
```

### カスタムフックのテスト

```typescript
// hooks/useBookmark.ts
import { useState, useCallback } from "react";
import { useMutation, useQuery, useQueryClient } from "@tanstack/react-query";
import { api } from "@/lib/api";

export function useBookmark(novelId: string) {
  const queryClient = useQueryClient();

  const { data: isBookmarked = false, isLoading } = useQuery({
    queryKey: ["bookmark", novelId],
    queryFn: () => api.bookmarks.check(novelId),
  });

  const addMutation = useMutation({
    mutationFn: () => api.bookmarks.add(novelId),
    onSuccess: () => {
      queryClient.setQueryData(["bookmark", novelId], true);
    },
  });

  const removeMutation = useMutation({
    mutationFn: () => api.bookmarks.remove(novelId),
    onSuccess: () => {
      queryClient.setQueryData(["bookmark", novelId], false);
    },
  });

  const toggle = useCallback(() => {
    if (isBookmarked) {
      removeMutation.mutate();
    } else {
      addMutation.mutate();
    }
  }, [isBookmarked, addMutation, removeMutation]);

  return {
    isBookmarked,
    isLoading,
    toggle,
    isPending: addMutation.isPending || removeMutation.isPending,
  };
}
```

```typescript
// hooks/useBookmark.test.ts
import { describe, it, expect, vi } from "vitest";
import { renderHook, act, waitFor } from "@testing-library/react";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { useBookmark } from "./useBookmark";

// API をモック
vi.mock("@/lib/api", () => ({
  api: {
    bookmarks: {
      check: vi.fn(),
      add: vi.fn(),
      remove: vi.fn(),
    },
  },
}));

import { api } from "@/lib/api";

function createWrapper() {
  const queryClient = new QueryClient({
    defaultOptions: {
      queries: { retry: false },
    },
  });
  return ({ children }: { children: React.ReactNode }) => (
    <QueryClientProvider client={queryClient}>{children}</QueryClientProvider>
  );
}

describe("useBookmark", () => {
  it("ブックマーク状態を取得する", async () => {
    vi.mocked(api.bookmarks.check).mockResolvedValue(true);

    const { result } = renderHook(() => useBookmark("nvl_123"), {
      wrapper: createWrapper(),
    });

    await waitFor(() => {
      expect(result.current.isBookmarked).toBe(true);
    });
  });

  it("toggle でブックマークを追加する", async () => {
    vi.mocked(api.bookmarks.check).mockResolvedValue(false);
    vi.mocked(api.bookmarks.add).mockResolvedValue(undefined);

    const { result } = renderHook(() => useBookmark("nvl_123"), {
      wrapper: createWrapper(),
    });

    await waitFor(() => {
      expect(result.current.isLoading).toBe(false);
    });

    act(() => {
      result.current.toggle();
    });

    await waitFor(() => {
      expect(api.bookmarks.add).toHaveBeenCalledWith("nvl_123");
    });
  });

  it("toggle でブックマークを削除する", async () => {
    vi.mocked(api.bookmarks.check).mockResolvedValue(true);
    vi.mocked(api.bookmarks.remove).mockResolvedValue(undefined);

    const { result } = renderHook(() => useBookmark("nvl_123"), {
      wrapper: createWrapper(),
    });

    await waitFor(() => {
      expect(result.current.isBookmarked).toBe(true);
    });

    act(() => {
      result.current.toggle();
    });

    await waitFor(() => {
      expect(api.bookmarks.remove).toHaveBeenCalledWith("nvl_123");
    });
  });
});
```

---

## コンポーネントテスト

### テスト対象

- UI コンポーネントのレンダリング
- ユーザーインタラクション
- 条件付きレンダリング
- アクセシビリティ

### コンポーネントテスト例

```typescript
// components/NovelCard/NovelCard.tsx
interface NovelCardProps {
  novel: Novel;
  onBookmark?: () => void;
  isBookmarked?: boolean;
}

export function NovelCard({ novel, onBookmark, isBookmarked }: NovelCardProps) {
  return (
    <article className="card">
      <img src={novel.cover} alt={novel.title} />
      <h3>{novel.title}</h3>
      <p>{novel.author.displayName}</p>
      <div className="stats">
        <span>{novel.stats.viewsCount.toLocaleString()} views</span>
        <span>★ {novel.stats.rating.toFixed(1)}</span>
      </div>
      {onBookmark && (
        <button
          onClick={onBookmark}
          aria-pressed={isBookmarked}
          aria-label={isBookmarked ? "ブックマーク解除" : "ブックマーク追加"}
        >
          {isBookmarked ? "★" : "☆"}
        </button>
      )}
    </article>
  );
}
```

```typescript
// components/NovelCard/NovelCard.test.tsx
import { describe, it, expect, vi } from "vitest";
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { NovelCard } from "./NovelCard";

const mockNovel: Novel = {
  id: "nvl_123",
  title: "異世界転生物語",
  cover: "https://example.com/cover.jpg",
  author: {
    id: "usr_456",
    displayName: "作家太郎",
  },
  stats: {
    viewsCount: 12345,
    rating: 4.5,
  },
};

describe("NovelCard", () => {
  it("小説情報を表示する", () => {
    render(<NovelCard novel={mockNovel} />);

    expect(screen.getByText("異世界転生物語")).toBeInTheDocument();
    expect(screen.getByText("作家太郎")).toBeInTheDocument();
    expect(screen.getByText("12,345 views")).toBeInTheDocument();
    expect(screen.getByText("★ 4.5")).toBeInTheDocument();
  });

  it("カバー画像を表示する", () => {
    render(<NovelCard novel={mockNovel} />);

    const img = screen.getByRole("img", { name: "異世界転生物語" });
    expect(img).toHaveAttribute("src", "https://example.com/cover.jpg");
  });

  it("ブックマークボタンをクリックするとコールバックが呼ばれる", async () => {
    const user = userEvent.setup();
    const onBookmark = vi.fn();

    render(
      <NovelCard novel={mockNovel} onBookmark={onBookmark} isBookmarked={false} />
    );

    const button = screen.getByRole("button", { name: "ブックマーク追加" });
    await user.click(button);

    expect(onBookmark).toHaveBeenCalledTimes(1);
  });

  it("ブックマーク済みの状態を表示する", () => {
    render(
      <NovelCard
        novel={mockNovel}
        onBookmark={() => {}}
        isBookmarked={true}
      />
    );

    const button = screen.getByRole("button", { name: "ブックマーク解除" });
    expect(button).toHaveAttribute("aria-pressed", "true");
  });

  it("onBookmark が渡されない場合、ボタンを表示しない", () => {
    render(<NovelCard novel={mockNovel} />);

    expect(
      screen.queryByRole("button", { name: /ブックマーク/ })
    ).not.toBeInTheDocument();
  });
});
```

---

## 統合テスト

### API エンドポイントのテスト

```typescript
// routes/novels.test.ts
import { describe, it, expect, beforeAll, afterAll } from "vitest";
import { Hono } from "hono";
import { testClient } from "hono/testing";
import app from "./novels";
import { db } from "@/db";
import { novels, users } from "@/db/schema";

describe("GET /v1/novels", () => {
  beforeAll(async () => {
    // テストデータのセットアップ
    await db.insert(users).values({
      id: "usr_test",
      email: "test@example.com",
      username: "testuser",
    });

    await db.insert(novels).values([
      {
        id: "nvl_001",
        authorId: "usr_test",
        title: "テスト小説1",
        genre: "fantasy",
        status: "ongoing",
      },
      {
        id: "nvl_002",
        authorId: "usr_test",
        title: "テスト小説2",
        genre: "romance",
        status: "completed",
      },
    ]);
  });

  afterAll(async () => {
    // テストデータのクリーンアップ
    await db.delete(novels);
    await db.delete(users);
  });

  it("小説一覧を取得できる", async () => {
    const client = testClient(app);
    const res = await client.index.$get();

    expect(res.status).toBe(200);

    const data = await res.json();
    expect(data.data).toHaveLength(2);
    expect(data.pagination).toBeDefined();
  });

  it("ジャンルでフィルタできる", async () => {
    const client = testClient(app);
    const res = await client.index.$get({
      query: { genre: "fantasy" },
    });

    const data = await res.json();
    expect(data.data).toHaveLength(1);
    expect(data.data[0].genre).toBe("fantasy");
  });

  it("ページネーションが機能する", async () => {
    const client = testClient(app);
    const res = await client.index.$get({
      query: { limit: "1", page: "1" },
    });

    const data = await res.json();
    expect(data.data).toHaveLength(1);
    expect(data.pagination.totalPages).toBe(2);
  });
});

describe("POST /v1/novels", () => {
  it("認証なしでは作成できない", async () => {
    const client = testClient(app);
    const res = await client.index.$post({
      json: {
        title: "新しい小説",
        genre: "fantasy",
      },
    });

    expect(res.status).toBe(401);
  });

  it("認証済みユーザーは小説を作成できる", async () => {
    const client = testClient(app);
    const res = await client.index.$post(
      {
        json: {
          title: "新しい小説",
          genre: "fantasy",
          synopsis: "テスト概要",
        },
      },
      {
        headers: {
          Authorization: "Bearer valid_token",
        },
      }
    );

    expect(res.status).toBe(201);

    const data = await res.json();
    expect(data.title).toBe("新しい小説");
    expect(data.id).toMatch(/^nvl_/);
  });

  it("バリデーションエラーを返す", async () => {
    const client = testClient(app);
    const res = await client.index.$post(
      {
        json: {
          title: "", // 空のタイトル
          genre: "invalid_genre", // 無効なジャンル
        },
      },
      {
        headers: {
          Authorization: "Bearer valid_token",
        },
      }
    );

    expect(res.status).toBe(400);

    const data = await res.json();
    expect(data.errors).toBeDefined();
  });
});
```

---

## E2E テスト

### テスト対象フロー

| フロー | 優先度 | 説明 |
|--------|--------|------|
| ユーザー登録・ログイン | 最高 | 認証フロー全体 |
| 小説閲覧 | 最高 | 検索→詳細→章読み |
| 小説投稿 | 高 | 作成→章追加→公開 |
| ブックマーク | 中 | 追加・削除・一覧 |
| コメント | 中 | 投稿・返信 |

### E2E テスト例

```typescript
// e2e/auth.spec.ts
import { test, expect } from "@playwright/test";

test.describe("認証フロー", () => {
  test("新規ユーザー登録", async ({ page }) => {
    await page.goto("/register");

    // フォーム入力
    await page.fill('input[name="email"]', "newuser@example.com");
    await page.fill('input[name="username"]', "newuser");
    await page.fill('input[name="password"]', "SecurePassword123");
    await page.fill('input[name="passwordConfirm"]', "SecurePassword123");

    // 送信
    await page.click('button[type="submit"]');

    // 登録成功を確認
    await expect(page).toHaveURL("/dashboard");
    await expect(page.locator("text=ようこそ")).toBeVisible();
  });

  test("ログイン", async ({ page }) => {
    await page.goto("/login");

    await page.fill('input[name="email"]', "existing@example.com");
    await page.fill('input[name="password"]', "ExistingPassword123");

    await page.click('button[type="submit"]');

    await expect(page).toHaveURL("/dashboard");
  });

  test("無効な認証情報でエラー表示", async ({ page }) => {
    await page.goto("/login");

    await page.fill('input[name="email"]', "wrong@example.com");
    await page.fill('input[name="password"]', "WrongPassword");

    await page.click('button[type="submit"]');

    await expect(page.locator("text=メールアドレスまたはパスワードが正しくありません")).toBeVisible();
  });
});
```

```typescript
// e2e/novel-reading.spec.ts
import { test, expect } from "@playwright/test";

test.describe("小説閲覧フロー", () => {
  test("小説を検索して読む", async ({ page }) => {
    // ホームページにアクセス
    await page.goto("/");

    // 検索
    await page.fill('input[type="search"]', "異世界");
    await page.press('input[type="search"]', "Enter");

    // 検索結果を確認
    await expect(page).toHaveURL(/\/search\?q=異世界/);
    await expect(page.locator("article")).toHaveCount.greaterThan(0);

    // 最初の小説をクリック
    await page.click("article:first-child");

    // 小説詳細ページを確認
    await expect(page.locator("h1")).toBeVisible();
    await expect(page.locator("text=第1章")).toBeVisible();

    // 第1章を読む
    await page.click("text=第1章");

    // 章本文が表示されることを確認
    await expect(page.locator("article.chapter-content")).toBeVisible();

    // 次の章に進む
    await page.click('button[aria-label="次の章"]');
    await expect(page.locator("text=第2章")).toBeVisible();
  });

  test("言語切り替え", async ({ page }) => {
    await page.goto("/novels/nvl_123");

    // デフォルトは日本語
    await expect(page.locator("h1")).toContainText("異世界");

    // 英語に切り替え
    await page.click('button[aria-label="言語切り替え"]');
    await page.click("text=English");

    // 英語版が表示される
    await expect(page.locator("h1")).toContainText("Isekai");
  });
});
```

```typescript
// e2e/novel-creation.spec.ts
import { test, expect } from "@playwright/test";

test.describe("小説投稿フロー", () => {
  test.use({
    storageState: "e2e/.auth/writer.json", // 作家としてログイン済み
  });

  test("新しい小説を作成して公開する", async ({ page }) => {
    // ダッシュボードにアクセス
    await page.goto("/dashboard");

    // 新規作成ボタンをクリック
    await page.click("text=新しい作品を作成");

    // 小説情報を入力
    await page.fill('input[name="title"]', "テスト小説");
    await page.fill('textarea[name="synopsis"]', "これはテスト用の小説です。");
    await page.selectOption('select[name="genre"]', "fantasy");

    // 作成
    await page.click('button[type="submit"]');

    // 作品編集ページに遷移
    await expect(page).toHaveURL(/\/dashboard\/novels\/nvl_/);

    // 章を追加
    await page.click("text=新しい章を追加");
    await page.fill('input[name="title"]', "第1章：始まり");
    await page.fill('textarea[name="content"]', "物語の始まりです。");

    // 章を保存
    await page.click("text=下書き保存");
    await expect(page.locator("text=保存しました")).toBeVisible();

    // 公開設定
    await page.click("text=公開設定");
    await page.click("text=公開する");
    await page.click("text=確認");

    // 公開完了
    await expect(page.locator("text=公開しました")).toBeVisible();
  });
});
```

---

## テスト実行

### コマンド

```bash
# ユニットテスト
pnpm test

# ユニットテスト（ウォッチモード）
pnpm test:watch

# カバレッジレポート
pnpm test:coverage

# E2E テスト
pnpm test:e2e

# E2E テスト（UI モード）
pnpm test:e2e:ui

# 特定のファイルのみ
pnpm test src/services/translation
```

### CI 設定

```yaml
# .gitlab-ci.yml
test:
  stage: test
  script:
    - pnpm install
    - pnpm test:coverage
  coverage: '/All files[^|]*\|[^|]*\s+([\d\.]+)/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml

e2e:
  stage: test
  script:
    - pnpm install
    - pnpm build
    - pnpm test:e2e
  artifacts:
    when: always
    paths:
      - playwright-report/
    expire_in: 30 days
```

---

## モック戦略

### MSW によるAPI モック

```typescript
// mocks/handlers.ts
import { http, HttpResponse } from "msw";

export const handlers = [
  // 認証
  http.post("/v1/auth/login", async ({ request }) => {
    const body = await request.json();

    if (body.email === "test@example.com") {
      return HttpResponse.json({
        user: { id: "usr_test", email: "test@example.com" },
        tokens: { accessToken: "mock_token" },
      });
    }

    return HttpResponse.json(
      { error: "Invalid credentials" },
      { status: 401 }
    );
  }),

  // 小説一覧
  http.get("/v1/novels", () => {
    return HttpResponse.json({
      data: [
        {
          id: "nvl_001",
          title: "モック小説1",
          author: { displayName: "テスト作家" },
        },
      ],
      pagination: { page: 1, total: 1 },
    });
  }),

  // 小説詳細
  http.get("/v1/novels/:id", ({ params }) => {
    return HttpResponse.json({
      id: params.id,
      title: "モック小説",
      synopsis: "これはモックです",
    });
  }),
];
```

```typescript
// mocks/server.ts
import { setupServer } from "msw/node";
import { handlers } from "./handlers";

export const server = setupServer(...handlers);
```

```typescript
// vitest.setup.ts
import { beforeAll, afterAll, afterEach } from "vitest";
import { server } from "./mocks/server";

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

---

## 関連ドキュメント

| ドキュメント | 内容 |
|-------------|------|
| [コーディング規約](/specs/coding-standards/) | コード品質基準 |
| [API設計書](/specs/api-design/) | API 仕様 |
| [アーキテクチャ](/specs/architecture/) | システム設計 |
