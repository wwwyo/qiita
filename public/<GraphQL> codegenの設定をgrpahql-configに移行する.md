---
title: <GraphQL> graphql-codegen の設定を grpahql-configに移行する
tags:
  - 'graphql'
  - 'typescript'
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---

# はじめに

[GraphQL 成熟度モデル](https://oisham.hatenablog.com/entry/2023/05/01/141829) の記事を読んで、「あ、やベェ LSP の対応していない」となったので、対応しました。と言っても、VSCode の[拡張機能](https://marketplace.visualstudio.com/items?itemName=GraphQL.vscode-graphql)を入れただけなのですが、設定ファイルが graphql-codegen と重複するため、設定ファイルを[graphql-config](https://the-guild.dev/graphql/config/docs)へ移行しました。移行方法です。

# 前提

- graphql: v16
- graphql-codegen/cli: v5
- graphql-config: v5

もともと codegen の config ファイルは`codegen.ts`という ts ファイルに存在している。

# 詳細

## 1. 拡張機能を入れる

https://marketplace.visualstudio.com/items?itemName=GraphQL.vscode-graphql

拡張機能の説明を見ると、root の `graphql-config file` を見ると書いてあるので graphql-config file を入れていく

## 2. graphql-config

```bash
$ npm i graphql-config
```

ファイル形式は yml でも ts でも色々いけるが同じく ts に移した
参考: https://the-guild.dev/graphql/config/docs/user/usage

```bash
$ touch .graphqlrc.ts
```

## 3. 移す

```codegen.ts
import type { CodegenConfig } from '@graphql-codegen/cli'

const config: CodegenConfig = {
  schema: [...],
  documents: [...],
  generates: {
    '<path>': {
      ...
    }
  },
}
```

`extensions.codegen`の中に移す必要があるので間違えないように

```.graphqlrc.ts
import type { IGraphQLConfig } from 'graphql-config'

const config: IGraphQLConfig = {
  schema: [...],
  documents: [...],
  extensions: {
    codegen: {
      generates: {
        '<path>': {
          ...
        }
      }
    }
  }
}
```

# 備考

schema を外部 url にしている場合は`languageService`に追加すると SDL ファイルを作ってそこを参照するみたいなことが書いてあったがうまくジャンプできなかった(?)
まだ experimental みたいなので一旦諦めた

```.graphqlrc.ts
extensions: {
  codegen: {...},
  languageService: {
    cacheSchemaFileForLookup: true, // SDL ファイルを生成
  },
}
```

https://github.com/graphql/graphiql/blob/main/packages/vscode-graphql/README.md#go-to-definition-is-not-working-for-my-url

# 参考文献

- [GraphQL Config](https://the-guild.dev/graphql/config/docs)
