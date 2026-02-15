---
title: コーディング規約
description: 開発におけるコーディング規約
---

Astranote プロジェクトで統一されたコード品質を維持するためのコーディング規約です。

## 技術スタック概要

- **Frontend**: Astro, React, TypeScript, Tailwind CSS
- **Backend**: Cloudflare Workers, Hono, Drizzle ORM
- **State**: Zustand, TanStack Query

## TypeScript 規約

### 型定義
- **明示的な型定義**: `any` の使用は原則禁止。`unknown` を使用し型ガードを行う。
- **Interface vs Type**: オブジェクト定義には `interface`、Union型やUtilityには `type` を使用。
- **Enum禁止**: `const object` + `keyof typeof` パターンを使用する。

```typescript
// Good
const Status = {
  Draft: "draft",
  Published: "published",
} as const;
type Status = (typeof Status)[keyof typeof Status];
```

## 命名規則

| 種類 | 形式 | 例 |
|------|------|-----|
| コンポーネント | PascalCase | `UserCard.tsx` |
| 関数/変数 | camelCase | `getUserProfile` |
| 定数 | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` |
| ディレクトリ | kebab-case | `user-profile/` |
| Boolean | verb prefix | `isLoading`, `hasError` |

## React/Astro 規約

- **Server Components優先**: 可能なら `.astro` や Server Component で実装し、必要な部分のみ `client:load` で React Island化する。
- **Props Interface**: コンポーネントPropsは必ず Interface で定義しエクスポートする。
- **Custom Hooks**: ロジックはCustom Hooksに分離し、ViewとLogicを分ける。

## Drizzle ORM / Hono 規約

- **Schema**: `src/db/schema` に集約。
- **Validation**: Zod を使用してAPI入出力を検証する。
- **Error Handling**: `AppError` クラスを使用し、統一されたエラーレスポンスを返す。

## Git コミット規約

`type(scope): subject` 形式を使用。

| Type | 説明 |
|------|------|
| feat | 新機能 |
| fix | バグ修正 |
| refactor | リファクタリング |
| docs | ドキュメント |
| chore | ビルド設定など |

**例**: `feat(auth): Add Google OAuth login`
