---
title: コーディング規約
description: Star Novel の開発におけるコーディング規約
---

## 概要

本ドキュメントでは、Star Novel プロジェクトで統一されたコード品質を維持するためのコーディング規約を定義します。

### 技術スタック

| レイヤー | 技術 |
|---------|------|
| フロントエンド | Next.js 14, TypeScript, Tailwind CSS |
| バックエンド | Cloudflare Workers, Hono, TypeScript |
| ORM | Drizzle ORM |
| 状態管理 | Zustand, TanStack Query |

---

## TypeScript 規約

### 基本設定

```json
// tsconfig.json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "exactOptionalPropertyTypes": true
  }
}
```

### 型定義

#### 明示的な型定義を優先

```typescript
// ✅ Good
const user: User = {
  id: "usr_abc123",
  name: "太郎",
};

// ❌ Bad
const user = {
  id: "usr_abc123",
  name: "太郎",
};
```

#### 型推論が明確な場合は省略可

```typescript
// ✅ Good - 型推論が明確
const count = 0;
const items = ["a", "b", "c"];

// ✅ Good - 関数の戻り値は明示
function getUser(id: string): User | null {
  // ...
}
```

#### any の使用禁止

```typescript
// ❌ Bad
function process(data: any) {
  // ...
}

// ✅ Good
function process(data: unknown) {
  if (isValidData(data)) {
    // 型ガードで絞り込み
  }
}

// ✅ Good - ジェネリクスを使用
function process<T>(data: T): ProcessedData<T> {
  // ...
}
```

### インターフェースと型エイリアス

```typescript
// オブジェクト型は interface を使用
interface User {
  id: string;
  name: string;
  email: string;
}

// ユニオン型や複雑な型は type を使用
type Role = "reader" | "writer" | "admin";
type UserWithRole = User & { role: Role };
```

### Enum は避け、const object を使用

```typescript
// ❌ Bad
enum Status {
  Draft = "draft",
  Published = "published",
}

// ✅ Good
const Status = {
  Draft: "draft",
  Published: "published",
} as const;

type Status = (typeof Status)[keyof typeof Status];
```

---

## 命名規則

### ファイル・ディレクトリ

| 種類 | 形式 | 例 |
|------|------|-----|
| コンポーネント | PascalCase | `UserProfile.tsx` |
| フック | camelCase | `useAuth.ts` |
| ユーティリティ | camelCase | `formatDate.ts` |
| 型定義 | camelCase | `user.types.ts` |
| 定数 | camelCase | `constants.ts` |
| ディレクトリ | kebab-case | `user-profile/` |

### 変数・関数

| 種類 | 形式 | 例 |
|------|------|-----|
| 変数 | camelCase | `userName`, `isLoading` |
| 関数 | camelCase | `getUser`, `handleClick` |
| 定数 | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` |
| 型/インターフェース | PascalCase | `User`, `ApiResponse` |
| コンポーネント | PascalCase | `UserCard`, `NavBar` |

### Boolean 変数のプレフィックス

```typescript
// ✅ Good
const isLoading = true;
const hasError = false;
const canEdit = true;
const shouldUpdate = false;

// ❌ Bad
const loading = true;
const error = false;
```

### イベントハンドラの命名

```typescript
// ✅ Good
const handleClick = () => {};
const handleSubmit = () => {};
const onUserSelect = () => {};

// ❌ Bad
const click = () => {};
const submit = () => {};
```

---

## React/Next.js 規約

### コンポーネント構造

```typescript
// components/UserCard/UserCard.tsx

import { type FC, memo } from "react";
import { cn } from "@/lib/utils";
import type { User } from "@/types/user";

// Props の型定義
interface UserCardProps {
  user: User;
  className?: string;
  onSelect?: (user: User) => void;
}

// コンポーネント定義
export const UserCard: FC<UserCardProps> = memo(function UserCard({
  user,
  className,
  onSelect,
}) {
  const handleClick = () => {
    onSelect?.(user);
  };

  return (
    <div
      className={cn("rounded-lg border p-4", className)}
      onClick={handleClick}
      role="button"
      tabIndex={0}
    >
      <h3 className="font-bold">{user.displayName}</h3>
      <p className="text-muted-foreground">{user.bio}</p>
    </div>
  );
});
```

### フックの構造

```typescript
// hooks/useUser.ts

import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";
import { api } from "@/lib/api";
import type { User } from "@/types/user";

export function useUser(userId: string) {
  return useQuery({
    queryKey: ["user", userId],
    queryFn: () => api.users.get(userId),
    staleTime: 5 * 60 * 1000, // 5分
  });
}

export function useUpdateUser() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (data: Partial<User>) => api.users.update(data),
    onSuccess: (updatedUser) => {
      queryClient.setQueryData(["user", updatedUser.id], updatedUser);
    },
  });
}
```

### Server Components vs Client Components

```typescript
// Server Component（デフォルト）
// app/novels/[id]/page.tsx
import { getNovel } from "@/lib/api";
import { NovelDetail } from "@/components/NovelDetail";

export default async function NovelPage({
  params,
}: {
  params: { id: string };
}) {
  const novel = await getNovel(params.id);

  return <NovelDetail novel={novel} />;
}

// Client Component（インタラクティブな機能が必要な場合）
// components/BookmarkButton.tsx
"use client";

import { useState } from "react";
import { useBookmark } from "@/hooks/useBookmark";

export function BookmarkButton({ novelId }: { novelId: string }) {
  const { isBookmarked, toggle } = useBookmark(novelId);

  return (
    <button onClick={toggle}>
      {isBookmarked ? "ブックマーク解除" : "ブックマーク"}
    </button>
  );
}
```

### コンポーネントのエクスポート

```typescript
// components/UserCard/index.ts
export { UserCard } from "./UserCard";
export type { UserCardProps } from "./UserCard";

// 使用側
import { UserCard } from "@/components/UserCard";
```

---

## Hono (Backend) 規約

### ルート定義

```typescript
// src/routes/novels.ts

import { Hono } from "hono";
import { zValidator } from "@hono/zod-validator";
import { z } from "zod";
import { authMiddleware } from "@/middleware/auth";
import { novelService } from "@/services/novel";

const app = new Hono();

// スキーマ定義
const createNovelSchema = z.object({
  title: z.string().min(1).max(100),
  synopsis: z.string().max(2000).optional(),
  genre: z.enum(["fantasy", "romance", "scifi", "mystery", "other"]),
  isR18: z.boolean().default(false),
});

// ルート定義
app.get("/", async (c) => {
  const novels = await novelService.list();
  return c.json({ data: novels });
});

app.post(
  "/",
  authMiddleware,
  zValidator("json", createNovelSchema),
  async (c) => {
    const data = c.req.valid("json");
    const userId = c.get("userId");
    const novel = await novelService.create(userId, data);
    return c.json(novel, 201);
  }
);

app.get("/:id", async (c) => {
  const id = c.req.param("id");
  const novel = await novelService.get(id);

  if (!novel) {
    return c.json({ error: "Not found" }, 404);
  }

  return c.json(novel);
});

export default app;
```

### サービス層

```typescript
// src/services/novel.ts

import { db } from "@/db";
import { novels, chapters } from "@/db/schema";
import { eq } from "drizzle-orm";
import type { CreateNovelInput, Novel } from "@/types/novel";

export const novelService = {
  async list(): Promise<Novel[]> {
    return db.select().from(novels).orderBy(novels.updatedAt);
  },

  async get(id: string): Promise<Novel | null> {
    const result = await db
      .select()
      .from(novels)
      .where(eq(novels.id, id))
      .limit(1);

    return result[0] ?? null;
  },

  async create(authorId: string, input: CreateNovelInput): Promise<Novel> {
    const id = generateId("nvl");
    const now = new Date().toISOString();

    const [novel] = await db
      .insert(novels)
      .values({
        id,
        authorId,
        ...input,
        createdAt: now,
        updatedAt: now,
      })
      .returning();

    return novel;
  },
};
```

### エラーハンドリング

```typescript
// src/lib/errors.ts

export class AppError extends Error {
  constructor(
    public code: string,
    message: string,
    public status: number = 500,
    public details?: Record<string, unknown>
  ) {
    super(message);
    this.name = "AppError";
  }
}

export class NotFoundError extends AppError {
  constructor(resource: string, id: string) {
    super("NOT_FOUND", `${resource} not found: ${id}`, 404);
  }
}

export class UnauthorizedError extends AppError {
  constructor(message = "Unauthorized") {
    super("UNAUTHORIZED", message, 401);
  }
}

export class ValidationError extends AppError {
  constructor(errors: Array<{ field: string; message: string }>) {
    super("VALIDATION_ERROR", "Validation failed", 400, { errors });
  }
}
```

```typescript
// src/middleware/errorHandler.ts

import { Hono } from "hono";
import { AppError } from "@/lib/errors";

export const errorHandler = (err: Error, c: Context) => {
  console.error(err);

  if (err instanceof AppError) {
    return c.json(
      {
        type: `https://api.star-novel.com/errors/${err.code.toLowerCase()}`,
        title: err.code,
        status: err.status,
        detail: err.message,
        ...err.details,
      },
      err.status
    );
  }

  return c.json(
    {
      type: "https://api.star-novel.com/errors/internal-error",
      title: "Internal Server Error",
      status: 500,
      detail: "An unexpected error occurred",
    },
    500
  );
};
```

---

## Drizzle ORM 規約

### スキーマ定義

```typescript
// src/db/schema/users.ts

import { sqliteTable, text, integer } from "drizzle-orm/sqlite-core";
import { relations } from "drizzle-orm";

export const users = sqliteTable("users", {
  id: text("id").primaryKey(),
  email: text("email").notNull().unique(),
  username: text("username").notNull().unique(),
  passwordHash: text("password_hash"),
  displayName: text("display_name"),
  avatarUrl: text("avatar_url"),
  bio: text("bio"),
  role: text("role", { enum: ["reader", "writer", "admin"] })
    .notNull()
    .default("reader"),
  locale: text("locale").notNull().default("ja"),
  emailVerified: integer("email_verified", { mode: "boolean" })
    .notNull()
    .default(false),
  createdAt: text("created_at").notNull().default(sql`CURRENT_TIMESTAMP`),
  updatedAt: text("updated_at").notNull().default(sql`CURRENT_TIMESTAMP`),
});

export const usersRelations = relations(users, ({ many }) => ({
  novels: many(novels),
  bookmarks: many(bookmarks),
  follows: many(follows),
}));
```

### クエリパターン

```typescript
// 基本的なクエリ
const user = await db.select().from(users).where(eq(users.id, id)).limit(1);

// リレーション込みのクエリ
const novelWithAuthor = await db.query.novels.findFirst({
  where: eq(novels.id, novelId),
  with: {
    author: true,
    tags: true,
  },
});

// 複雑なクエリ
const popularNovels = await db
  .select({
    id: novels.id,
    title: novels.title,
    author: {
      id: users.id,
      displayName: users.displayName,
    },
  })
  .from(novels)
  .leftJoin(users, eq(novels.authorId, users.id))
  .where(and(eq(novels.status, "ongoing"), gt(novels.viewsCount, 1000)))
  .orderBy(desc(novels.viewsCount))
  .limit(10);
```

---

## CSS/Tailwind 規約

### クラス名の順序

```tsx
// 順序: レイアウト → サイズ → 余白 → 背景 → ボーダー → テキスト → その他
<div className="flex items-center justify-between w-full h-12 px-4 py-2 bg-white border rounded-lg text-gray-900 hover:bg-gray-50 transition-colors">
```

### cn ユーティリティの使用

```typescript
// lib/utils.ts
import { type ClassValue, clsx } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}

// 使用例
<div className={cn(
  "rounded-lg border p-4",
  isActive && "border-primary bg-primary/10",
  className
)} />
```

### カスタムクラスの定義

```css
/* globals.css */
@layer components {
  .btn-primary {
    @apply px-4 py-2 bg-primary text-primary-foreground rounded-md hover:bg-primary/90 transition-colors;
  }

  .card {
    @apply rounded-lg border bg-card p-6 shadow-sm;
  }
}
```

---

## Git コミット規約

### コミットメッセージ形式

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Type 一覧

| Type | 説明 |
|------|------|
| feat | 新機能 |
| fix | バグ修正 |
| docs | ドキュメント |
| style | フォーマット（動作に影響なし） |
| refactor | リファクタリング |
| perf | パフォーマンス改善 |
| test | テスト追加・修正 |
| chore | ビルド・ツール関連 |

### 例

```
feat(auth): Google OAuth 認証を実装

- OAuth フローの実装
- セッション管理の追加
- プロフィール同期機能

Closes #12
```

```
fix(translation): 翻訳キューのタイムアウト問題を修正

翻訳処理が5分以上かかる場合にタイムアウトする問題を修正。
タイムアウト値を10分に延長し、リトライロジックを改善。

Fixes #45
```

---

## コードレビュー基準

### 必須チェック項目

- [ ] TypeScript の型エラーがないこと
- [ ] ESLint エラーがないこと
- [ ] テストが追加/更新されていること
- [ ] 命名規則に従っていること
- [ ] 不要な `console.log` が残っていないこと
- [ ] セキュリティ上の問題がないこと

### 推奨チェック項目

- [ ] コードの可読性
- [ ] パフォーマンスへの影響
- [ ] エラーハンドリングの適切さ
- [ ] ドキュメントの更新

---

## ツール設定

### ESLint

```javascript
// .eslintrc.js
module.exports = {
  extends: [
    "next/core-web-vitals",
    "plugin:@typescript-eslint/recommended",
    "plugin:@typescript-eslint/recommended-requiring-type-checking",
    "prettier",
  ],
  rules: {
    "@typescript-eslint/no-explicit-any": "error",
    "@typescript-eslint/explicit-function-return-type": "warn",
    "@typescript-eslint/no-unused-vars": [
      "error",
      { argsIgnorePattern: "^_" },
    ],
    "no-console": ["warn", { allow: ["warn", "error"] }],
  },
};
```

### Prettier

```json
// .prettierrc
{
  "semi": true,
  "singleQuote": false,
  "tabWidth": 2,
  "trailingComma": "es5",
  "printWidth": 80
}
```

---

## 関連ドキュメント

| ドキュメント | 内容 |
|-------------|------|
| [API設計書](/specs/api-design/) | RESTful API 仕様 |
| [DB設計書](/specs/db-design/) | データベース設計 |
| [テスト戦略](/specs/testing-strategy/) | テスト方針 |
