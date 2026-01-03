<!-- タイトル
Next.js(Turbopack)でresolveAliasを使用してモジュールを差し替える -->

# はじめに

テストやCI環境では別のモジュール（関数）を使用したい場合があるかと思います．
例えば，API通信を行うモジュールをテスト用のモジュールに差し替えたい場合などです．

Next.js v16以降ではTurbopackがデフォルトのバンドラーとなったため，今回はTurbopackでのモジュール差し替え方法を紹介します．

# この記事でわかること

- resolveAlias を使用してモジュールを差し替える方法

# この記事で扱わないこと

- Next.js の基本的なこと

# 詳細

Next.js と言っても，今回はほとんど TS 要素しかないです．
全体感としては，doSomething関数があって，その中でAPI通信を行うfetchSomeData関数を呼び出す形になります．このfetchSomeData関数を環境変数によって差し替えることを行います．

## ファイル構造

```tree
├── app
│   └── page.tsx
├── lib
│   ├── doSomething.ts
│   ├── fetcher.mock.ts
│   └── fetcher.ts
├── next.config.ts
└── package.json
```

## 実装

`lib/fetcher.ts` が本来のAPI通信を行うモジュールで，`lib/fetcher.mock.ts` がテスト用のモジュールになります．doSomething関数の中でfetchSomeData関数を呼び出しています．

### lib/fetcher.ts

```ts
export const fetchSomeData = (): string => {
  return "Fetch from API";
};
```

### lib/fetcher.mock.ts

```ts
export const fetchSomeData = (): string => {
  return "Fetch from Mock";
};
```

### lib/doSomething.ts

```ts
import { fetchSomeData } from "@/lib/fetcher";

export const doSomething = (): string => {
  const response = fetchSomeData();
  return `Result: ${response}`;
};
```

### 設定

次に，`next.config.ts` でresolveAliasを使用して，環境変数によってモジュールを差し替えます．
`USE_MOCK`という環境変数が設定されている場合に，`lib/fetcher.mock.ts` を使用するようにしています．
`@/lib/fetcher`からのインポートが`@/lib/fetcher.mock`に差し替えられます．

```ts
// next.config.ts
const nextConfig: NextConfig = {
  turbopack: process.env.USE_MOCK
    ? {
        resolveAlias: {
          "@/lib/fetcher": "@/lib/fetcher.mock",
        },
      }
    : {},
...
```

### 実行

doSomething関数を `app/page.tsx` で適当に呼び出します．
毎回環境変数を設定するのは大変なので，`package.json` にスクリプトを追加しときます．

```jsonc
...
"scripts": {
  "dev": "next dev",
  "dev:mock": "USE_MOCK=true next dev",
...
```

これで，`npm run dev`で通常のAPI通信モジュールが使用され，`npm run dev:mock`でテスト用のモジュールが使用されるようになります．

### 実行結果

- 動作環境

パッケージマネージャ: bun
Next.js バージョン: 16.1.1

- `bun dev` の場合

```bash
Result: Fetch from API
```

- `bun dev:mock` の場合

```bash
Result: Fetch from Mock
```

# まとめ

今回は Next.js(Turbopack) で resolveAlias を使用してモジュールを差し替える方法を紹介しました．	
`next.config.ts` で環境変数を使用して差し替えを行うことで，簡単にモジュールの切り替えが可能になります．

# 参考文献

- [https://nextjs.org/docs/app/api-reference/config/next-config-js/turbopack#resolving-aliases](https://nextjs.org/docs/app/api-reference/config/next-config-js/turbopack#resolving-aliases)
