---
title: Next.jsでbuild時にout of memoryになる問題を解決した話
tags:
  - 'Next.js'
  - 'Node.js'
  - 'Amplify'
  - 'TypeScript'
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---

# はじめに

Next.js を使ったプロジェクトを AWS Amplify にデプロイしようとした際、`next build` の途中で「JavaScript heap out of memory」というエラーに遭遇し、ビルドが失敗してしまう問題に直面しました。ローカル環境では問題なくビルドできていたため、最初は原因がわからず少しハマってしまいました。

この記事では、同様の問題に直面している方や、Next.js のビルド時のメモリ管理に関心のある方に向けて、原因の特定から解決に至るまでの過程、そして試したことを具体的に共有します。

# この記事でわかること（TL;DR）

- Next.js のビルドがメモリ不足で失敗する主な原因
- ビルド時のメモリ使用量を確認する方法 (`--experimental-debug-memory-usage`)
- Node.js のヒープサイズを調整する方法
- Next.js の Webpack メモリ最適化オプションについて
- TypeScript の型チェックがビルド時のメモリ使用量に与える影響と対策

結論から言うと、私の環境では **Node.js のヒープサイズ上限を引き上げ**、さらに **Webpack のメモリ最適化オプションを有効化**することで、この問題を解決できました。また、型チェックに起因するメモリ不足についても触れています。

# 前提

- Next.js: v15.3
- Node.js: v20.19
- ホスティング環境: AWS Amplify
- ローカルPCでの `next build` は問題なく成功する状態

# 🔥 直面した問題：Amplify でのビルド失敗

AWS Amplify で `next build` を実行すると、プロセスがクラッシュしてしまいました。Amplify のログには、以下のような不穏なメッセージが…

```bash
[WARNING]: <--- Last few GCs --->
[1455:0x6ed5f90]    80488 ms: Scavenge 1983.0 (2076.4) -> 1982.7 (2084.4) MB, 11.30 / 0.00 ms  (average mu = 0.733, current mu = 0.476) allocation failure;
[1455:0x6ed5f90]    80519 ms: Scavenge 1989.5 (2084.4) -> 1989.4 (2084.4) MB, 6.67 / 0.00 ms  (average mu = 0.733, current mu = 0.476) allocation failure;
[1455:0x6ed5f90]    80533 ms: Scavenge 1990.4 (2084.4) -> 1989.7 (2100.4) MB, 9.77 / 0.00 ms  (average mu = 0.733, current mu = 0.476) allocation failure;

<--- JS stacktrace --->
FATAL ERROR: Reached heap limit Allocation failed - JavaScript heap out of memory
```

はい、`heap out of memory` ですね。Node.js のヒープメモリが足りなくなってしまったようです。

# 🛠️ やったこと：原因調査と対策

## 1. まずはメモリ使用量の現状把握

Next.js には、ビルド時のメモリ使用状況をデバッグするための実験的なフラグ `--experimental-debug-memory-usage` が用意されています。これを使ってローカルでビルドを実行し、手がかりを探ってみました。

```bash
next build --experimental-debug-memory-usage
```

すると、このようなレポートが出力されました。

```bash
Memory usage report at "periodic memory snapshot":
 - RSS: 3695.52 MB
 - Heap Used: 2584.34 MB
 - Heap Max: 4144.00 MB
 - Percentage Heap Used: 62.36%
⚠ Long running GC detected: 285.69ms
```

各項目の意味はざっくり以下の通りです。

| 項目        | 意味                                                                 |
| ----------- | -------------------------------------------------------------------- |
| RSS         | Resident Set Size（物理メモリ上で実際に使用中のサイズ）                |
| Heap Used   | 動的に確保されたオブジェクトで現在使用中のヒープメモリサイズ           |
| Heap Max    | `--max-old-space-size` で設定されたヒープ上限（またはデフォルト上限） |
| GC          | ガーベジコレクションの実行時間。長時間のGCは性能低下のサインです。     |

ローカルですら約2.5GB (`Heap Used: 2584.34 MB`) もヒープを使っていることがわかりました。

## 2. Node.js のヒープサイズ上限を確認・調整

Node.js のデフォルトのヒープ上限は、環境にもよりますが、Amplifyでは 2096MB になっていました。以下のコマンドで確認できます。

```bash
node -p "v8.getHeapStatistics().heap_size_limit / 1024 / 1024 + ' MB'"
2096
```

上記より、Amplify上でのbuild時のヒープサイズの上限を引き上げる必要がありそうです。
Amplify のビルド設定 (`amplify.yml` など) で、環境変数 `NODE_OPTIONS` を使って `--max-old-space-size` を指定します。今回は余裕をもって 4GB (4096MB) に設定してみました。

```yaml
# amplify.yml (抜粋)
preBuild:
  commands:
    - export NODE_OPTIONS="--max-old-space-size=4096" # 4GBに設定
    # ... 他のコマンド
```

:::note info
Amplify のメモリ制限については、こちらの GitHub Issue のコメントも参考になりました。
[Amplify build fails with "JavaScript heap out of memory" #654 (comment)](https://github.com/aws-amplify/amplify-hosting/issues/654#issuecomment-2341572651)
:::

## 3. Webpack メモリ最適化の有効化 (Next.js v15.0 以降)

Next.js v15.0 から、Webpack のメモリ使用量を最適化する実験的なオプション `webpackMemoryOptimizations` が追加されました。藁にもすがる思いで、こちらも有効にしてみます。

```ts:next.config.ts
const nextConfig = {
  experimental: {
    webpackMemoryOptimizations: true,
  },
};

export default nextConfig;
```

公式ドキュメント
[How to optimize memory usage - Next.js Docs](https://nextjs.org/docs/app/guides/memory-usage)

私の環境では、このオプションだけで劇的にメモリ使用量が減ることはありませんでしたが、ヒープサイズ拡張と併用することで、ビルドが安定するようになりました。

## 4. 【補足】TypeCheck の無効化（ビルド時のみ）

上記までの対応で `next build` 自体は通るようになったのですが、今度は Lint と型チェックのフェーズでメモリ不足らしき挙動に遭遇しました。

```bash
[INFO]: Linting and checking validity of types ...
[WARNING]: Next.js build worker exited with code: null and signal: SIGKILL
```

`SIGKILL` は、メモリ不足でプロセスが強制終了された可能性を示唆しています。
調査を進めると、TypeScript v5.8 以降で型チェック時のメモリ使用量が増加するケースがあるという情報を見つけました。
参考: [Next.js GitHub Issue Comment](https://github.com/vercel/next.js/issues/76704#issuecomment-2707416535)

CI環境ですでにtscのチェックは行なっているので、ビルドプロセスから型チェックをスキップすることで、メモリ使用量を抑えます。

```ts:next.config.ts
const nextConfig = {
  // ... 他の設定
  typescript: {
    ignoreBuildErrors: true, // ビルド時の型エラーを無視する
  },
  experimental: {
    webpackMemoryOptimizations: true,
  },
};

export default nextConfig;
```

:::note warn
`ignoreBuildErrors: true` を設定すると、型エラーがあってもビルドが成功してしまいます。CI環境では別途 `tsc --noEmit` などで型チェックを行い、品質を担保することが重要です。
:::

# まとめ

今回の Next.js ビルド時の `out of memory` 問題は、以下の手順で解決に至りました。

1.  `--experimental-debug-memory-usage` でビルド時のメモリ使用量を調査。
2.  Node.js のヒープ上限 (`--max-old-space-size`) を引き上げる。
3.  Next.js の `webpackMemoryOptimizations` オプションを有効にする。
4.  (補足) TypeScript の型チェックが原因でメモリを圧迫している場合は、ビルド時の型チェックを無効化する (`ignoreBuildErrors: true`) ことも検討。

特に大規模なアプリケーションや、多くの依存関係を持つプロジェクトでは、メモリ管理が重要になってくるのだと改めて感じました。同様の問題に直面した際の参考になれば幸いです！

# 参考文献

- [Amplify build fails with "JavaScript heap out of memory" #654 (comment)](https://github.com/aws-amplify/amplify-hosting/issues/654#issuecomment-2341572651) - Amplify での `NODE_OPTIONS` 設定について
- [How to optimize memory usage - Next.js Docs](https://nextjs.org/docs/app/guides/memory-usage) - Next.js のメモリ最適化について
- [Next.js GitHub Issue Comment regarding TypeScript memory usage](https://github.com/vercel/next.js/issues/76704#issuecomment-2707416535) - TypeScript のバージョンとメモリ使用量に関する議論