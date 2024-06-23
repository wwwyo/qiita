---
title: vscode上でtypescriptをサクッと動かす
tags:
  - 'TypeScript'
  - 'VSCode'
  - 'Deno'
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---

# はじめに

TypeScript のコードや型の確認を行うためにサクッと書いて実行したいときありますよね。
以前は TypeScript Playground を使っていたのですが、エディタから離れずにそのまま実行できると便利です。
そこで今回は、VSCode 上で TypeScript をサクッと動かす方法を紹介します。

# 要約

一連のワークフローを登録できる VSCode の [task](https://code.visualstudio.com/docs/editor/tasks) という機能を用います。
開いているファイルを Deno 上で実行するワークフローを task に登録しショートカットから実行できるようにすることで TypeScript をサクッと動かすことができます。

# 詳細

## 1. Deno のインストール

TypeScript をそのまま実行できる [Deno](https://deno.com/) をインストールします。
ちなみに [ts-node](https://www.npmjs.com/package/ts-node) でも良いです。

## 2. tasks.json の作成

⌘ + Shift + P でコマンドパレットを開き、`Tasks: Open User Tasks` を選択します。
tasks に関する設定ファイルが開かれるので、以下のように設定します。

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "Run Deno", // タスクのラベル
      "type": "shell", // シェルコマンドを実行
      "command": "deno", // 実行するコマンド
      "args": ["run", "${file}"], // コマンドの引数
      "group": {
        "kind": "build", // タスクの種類
        "isDefault": true // デフォルトのタスクに設定
      }
    }
  ]
}
```

isDefault を true にすることで、「⌘ + Shift + B」を押すだけで実行できるようになります。
（Windows や Linux の場合は、Ctrl + Shift + B）

# まとめ

VSCode 上で TypeScript をサクッと動かすためには、Deno をインストールし、tasks.json に適切な設定を追加するだけです。これにより、エディタから離れることなく TypeScript のコードをすぐに実行できるようになります。

# 参考文献

- [Integrate with External Tools via Tasks](https://code.visualstudio.com/docs/editor/tasks)
- [Task の設定オプション](https://code.visualstudio.com/docs/editor/tasks-appendix)
- [Deno](https://deno.com/)
- [ts-node](https://www.npmjs.com/package/ts-node)
