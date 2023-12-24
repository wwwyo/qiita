---
title: '[sentry] monitoring 404'
tags:
  - sentry
  - Next.js
private: false
updated_at: '2023-12-24T16:18:04+09:00'
id: 514487e2fce13f3c5947
organization_url_name: null
slide: false
ignorePublish: false
---

# はじめに

Next.js に Sentry を入れた際、captureException などで /monitoring?xxx へのリクエストが飛びます。
このリクエストが 404 エラーになってしまったので、その原因と対応をまとめます。

# 要約

/base

# 前提

Next.js: @13 app router
sentry/next.js: @7.77

# 詳細

## 1. 詳細情報 1

かくかくしかじかの説明
コードとともになら尚良しだが長くなりすぎるのに注意

## 2. 詳細情報 2

かくかくしかじかの説明
コードとともになら尚良しだが長くなりすぎるのに注意

# まとめ

# 参考文献

- [公式 tunnel について](https://docs.sentry.io/platforms/javascript/guides/nextjs/manual-setup/)
- [base path の issue](https://github.com/getsentry/sentry-javascript/issues/8293)
