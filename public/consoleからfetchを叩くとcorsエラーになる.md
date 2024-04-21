---
title: consoleからfetchを叩くとcorsエラーになる
tags:
  - 'js'
  - 'browser'
  - 'cors'
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---

# はじめに

cors 難い。今回は 通常のアプリケーション上の挙動では問題なかったが、dev tools 上で fetch を叩いた時、限定で cors エラーが発生したのでその解決方法を記載します。

# 要約

network タグから fetch をコピーすると `cache-control: no-cache` と `pragma: no-cache` header がつきます。これらの header がついた上で [preflight](https://developer.mozilla.org/ja/docs/Glossary/Preflight_request) リクエストが飛ぶような状況では ブラウザが自動で ['Access-Control-Request-Headers'](https://developer.mozilla.org/ja/docs/Web/HTTP/Headers/Access-Control-Request-Headers) に　 cache-control と pragma を追加します。backend の cors 許可ではこれらの header が許可されている必要があるのですが、ここの設定ができていなかったため cors エラーが発生していました。

# 前提

- クロスオリジンへのリクエスト
- preflight が飛ぶ状況

# 詳細

## 1. 背景

ある日、特定の fetch のデバッグのために、dev tools の network タブから fetch をコピーして console 上で実行しました。
通常のアプリケーションの動作時は問題ないのですが、この時のみ cors エラーとなってしまったのです。

## 2. 原因

実は、fetch をコピーするときに、`cache-control: no-cache` と `pragma: no-cache` がついていました。
これらの header がついた上で preflight が飛ぶような状況では、ブラウザが自動で `Access-Control-Request-Headers` に　 cache-control と pragma を追加します。
バックエンドの cors 設定ではこれらの header が許可されている必要があるのですが、ここの設定ができていなかったため cors エラーが発生していました。

## 3. 解決法

下記のいずれかで解決できました。

1. バックエンドの cors 設定で [`Access-Control-Allow-Headers`](https://developer.mozilla.org/ja/docs/Web/HTTP/Headers/Access-Control-Allow-Headers) に `cache-control` と `pragma` を追加する

2. console で実行する際に `cache-control` と `pragma` を削除する

# まとめ

fetch を コピーした際に credential や cache-control など実際の挙動と異なる header がついていることがあるので注意が必要。
