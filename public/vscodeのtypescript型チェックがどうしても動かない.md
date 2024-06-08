---
title: vscodeのTypeScript型チェックがどうしても動かない
tags:
  - 'TypeScript'
  - 'VSCode'
  - '型'
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---

# はじめに

TypeScript を用いた開発では外せなくなってきている VSCode ですがなぜか型チェックが動かない事象が生じました。
基本的な確認事項を実行したのですが復帰せず困り果てていた末に些細なことで解決しました。過ぎ去った時間への追悼として解決方法を記載します。
↓ この赤の波線が表示されないバグ
![動かない画像](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/787586/dbe644f7-d987-9399-2357-19a2ba7ef412.png)

# 要約

GitHub Copilot の拡張機能を再インストールすることで解決しました

# 前提

- VSCode 1.90.0
- Mac
- tsc ではエラーになるが VSCode 上ではエラーにならない。

# 詳細

## 1. Restart TS Server

VSCode の TypeScript サーバーを再起動するコマンド。
とりあえず ts check が動かなくなったら試すものです。一日 10 回以上は打っている。

1. VSCode のコマンドパレットを開く(⌘ + Shift + P)
2. `TypeScript: Restart TS Server` を選択

:::note info
ちなみに TS Server が出てこない場合は builtin で入っている拡張機能がアンインストールされている可能性がある
(JavaScript and Typescript)[https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-typescript-next]
:::

## 2. Reload Window

その名の通り再起動コマンド

1. VSCode のコマンドパレットを開く(⌘ + Shift + P)
2. `Reload Window` を選択

## 3. VSCode を完全に閉じて再起動

## 4. PC の再起動

## 5. VSCode 自体を再 Install

## 6. setting.json や拡張機能を調整する

使っていない設定を消したり、Typescript が関連していそうな設定をデフォルトに戻していく作業

## 7. GitHub Copilot の拡張機能を再インストールする

ここまできてようやく治る。
なぜ copilot の拡張機能が今回の事象に関係していたかは不明(copilot 自体は問題なく動いていた)。

# まとめ

まとめ
VSCode の TypeScript 型チェックが動かない問題は、GitHub Copilot の拡張機能を再インストールすることで解決しました。他にもいくつかの試すべき手順がありますが、最終的には Copilot の再インストールが必要でした。なぜ Copilot の拡張機能が関係していたのかは不明ですが、困リ果てた時はこの方法を試してみてください。

# 参考文献

- https://stackoverflow.com/questions/57318553/vs-code-not-doing-type-checking-on-typescript-files
