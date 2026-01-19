---
title: ZedでLSPが動かないと思ったらsecure by defaultが原因だった
tags:
  - エディタ
  - ZED
  - LSP
private: false
updated_at: '2026-01-19T12:16:43+09:00'
id: 135a7360db0ab99951b0
organization_url_name: null
slide: false
ignorePublish: false
---

# はじめに

Zed エディタを使っていて「あれ、LSP が動かない...」と困った経験はありませんか？
設定ファイルも正しいはずなのに、補完も型チェックも効かない。そんな時、原因は意外なところにありました。

# 要約

Zed v0.218.2-pre 以降、セキュリティ強化のため **Restricted Mode** がデフォルトで有効になっています。
新しく開いたワークツリーでは LSP サーバーが自動で起動しないため、手動で trust する必要があります。

**解決方法**: タイトルバーの `! Restricted Mode` をクリックして、ワークツリーを trust する

# 前提

- Zed v0.218.2-pre 以降
- 新しいプロジェクト/ワークツリーを開いた場合

# 詳細

## 1. 何が起きていたか

Zed でプロジェクトを開いたところ、他の IDE(ex. vscode, cursor)では動くはずの LSP（Language Server Protocol）が全く動作しませんでした。
Go の LSP、補完も診断も何も出てこない状態。

「設定おかしくしたかな...」と `.zed/settings.json` を確認しても問題なし。

## 2. 原因：Secure by Default

調べてみると、2025 年 12 月に Zed がセキュリティ強化のアップデートを行っていました。

https://zed.dev/blog/secure-by-default

> Zed は初回に開く作業ツリーを **Restricted Mode** で表示し、設定の適用・サーバー起動・インストールを行いません。

つまり、新しく開いたプロジェクトでは以下が無効化されています：

- `.zed/settings.json` のプロジェクト設定の読み込み
- LSP サーバーのダウンロード・起動
- MCP サーバーのインストール・起動

これはセキュリティ脆弱性（CVE-2025-68432、CVE-2025-68433）への対応で、悪意のある設定ファイルを含むリポジトリを開いた際の任意コード実行を防ぐためのものです。

## 3. 解決方法

タイトルバーに `! Restricted Mode` と表示されているので、これをクリックします。

モーダルが表示されるので、以下のいずれかを選択：

- **Trust and Continue**: このワークツリーを信頼して LSP 等を有効化
- **Trust all projects in the <path> folder**: このフォルダ配下すべてを信頼（チェックボックス）
- **Stay in Restricted Mode**: 信頼せずに閉じる

trust すると、LSP が正常に起動し、補完や診断が効くようになりました！

:::note info
trust の設定は永続化されるので、同じプロジェクトを再度開いた時は自動で trust されます。
:::

# まとめ

- Zed v0.218.2-pre 以降、新しいワークツリーは Restricted Mode で開かれる
- Restricted Mode では LSP/MCP サーバーが起動しない
- タイトルバーの `! Restricted Mode` をクリックして trust すれば解決
- セキュリティ上の理由なので、信頼できるプロジェクトのみ trust しましょう

「なんか LSP 動かないな...」と思ったら、まずタイトルバーを確認してみてください！

# 参考文献

- [Zed Moves Toward Secure-by-Default: Introducing Worktree Trust](https://zed.dev/blog/secure-by-default)
