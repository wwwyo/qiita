---
title: '[firebase]firebase deploy --only functionsでデプロイできない'
tags:
  - Firebase
  - FirebaseCloudFunctions
private: false
updated_at: '2022-05-01T21:08:46+09:00'
id: dfce0d4af4e6785ecde5
organization_url_name: null
slide: false
ignorePublish: false
---
# 概要
firebase cloudFunctionsをローカルで変更。
デプロイしようと`$ firebase deploy --only functions:hoge-func`
以下のエラーが出て一向にデプロイできない。

```bash
- Error Failed to update function hoge-func in region 
```

firebase落ちてるのではと思ったがそんなわけもなく。
https://status.cloud.google.com/summary

# 解決法
```bash
npm install [依存ライブラリ]
```

依存してるnode_modulesが入っていなかった。
フロントとfunctionsで共通のmoduleを使っていたため、functionから参照できていないとのこと。

しかもちゃんとエラー出ててた(上の方)。
`Did you list all required modules in the package.json dependencies?`


ログはざっと目を通した方が良い！
