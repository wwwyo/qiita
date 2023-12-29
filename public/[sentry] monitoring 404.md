---
title: '[sentry] monitoring 404'
tags:
  - sentry
  - Next.js
private: false
updated_at: '2023-12-29T17:03:11+09:00'
id: 514487e2fce13f3c5947
organization_url_name: null
slide: false
ignorePublish: false
---

# はじめに

Next.js に Sentry を入れた際、captureException などで /monitoring?xxx へのリクエストが飛びます。
このリクエストが 404 エラーになってしまったので、その原因と対応をまとめます。

# 要約

next.config.js の tunnelRoute の path を見直すことで解決しました。

# 前提

Next.js: @13 App Router
sentry/next.js: @7.77
Next.js の basePath や rewrite などでアクセスする path をいじっている

# 詳細

## 1. sentry の初期設定を wizard で行った場合はデフォルトで tunnel が設定される

```bash
npx @sentry/wizard@latest -i nextjs
```

https://docs.sentry.io/platforms/javascript/guides/nextjs/

next.config.js に sentry の設定が生成される.
ちなみに tunnelRoute を設定すると 直接 Sentry に飛ばすのではなく、自前のサーバーを経由して Sentry にログを飛ばすことができる。これをすることによって AdBlocker などでサードパーティへのリクエストがブロックされることを回避できる。

fyi: https://docs.sentry.io/platforms/javascript/guides/nextjs/troubleshooting/?original_referrer=https%3A%2F%2Fwww.google.com%2F#dealing-with-ad-blockers

```next.config.js
// ...
module.exports = withSentryConfig(
  module.exports,
  {
    // For all available options, see:
    // https://github.com/getsentry/sentry-webpack-plugin#options

    // Suppresses source map uploading logs during build
    silent: false,
    authToken: process.env.SENTRY_AUTH_TOKEN,
    org: '<org>',
    project: '<project>',
  },
  {
    // For all available options, see:
    // https://docs.sentry.io/platforms/javascript/guides/nextjs/manual-setup/

    // Upload a larger set of source maps for prettier stack traces (increases build time)
    widenClientFileUpload: true,

    // Transpiles SDK to be compatible with IE11 (increases bundle size)
    transpileClientSDK: true,

    // Routes browser requests to Sentry through a Next.js rewrite to circumvent ad-blockers (increases server load)
    tunnelRoute: '/monitoring',

    // Hides source maps from generated client bundles
    hideSourceMaps: true,

    // Automatically tree-shake Sentry logger statements to reduce bundle size
    disableLogger: true,
  }
)
```

## 2. /monitoring へのリクエストが 404 になる

今回は tunnelRoute が自動で設定されたので、sentry の captureException などでエラーを捕捉しようとすると、monitoring へのリクエストが飛ぶ。

この時、/monitoring へのリクエストが 404 になってしまう。

## 3. 原因

Next.js へのアクセスを reverse proxy 経由でアクセスしており、実際の path は /reverse/XXX のように `/reverse`がつくようになっていた。

そのため、tunnelRoute の設定を`/monitoring`から`/reverse/monitoring`へ変更することで無事に sentry へロギングできるようになった。

# まとめ

1. sentry の初期設定を wizard で行った場合はデフォルトで tunnel が設定される
2. tunnelRoute は /monitoring に設定され、sentry の captureException などでエラーを捕捉しようとすると、monitoring へのリクエストが飛ぶ。
3. 404 になる場合は、tunnelRoute の path を見直すことで解決する。

# 参考文献

- [公式 tunnel について](https://docs.sentry.io/platforms/javascript/guides/nextjs/manual-setup/)
- [base path の issue](https://github.com/getsentry/sentry-javascript/issues/8293)
