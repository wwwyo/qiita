---
title: '[Docker]docker-compose up でコンテナが立ち上がらない'
tags:
  - Docker
private: false
updated_at: '2022-03-13T16:32:32+09:00'
id: a7fd6efc69dc46e58faa
organization_url_name: null
slide: false
ignorePublish: false
---
```docker-compose up -d```を打ったときのエラー。
一部のコンテナが立ち上がらなかった。

# エラー内容
今回はhogeコンテナが立ち上がらない。他のコンテナは立ち上がっているのに。。。
```bash
ERROR: for hoge  Cannot start service hoge: network 4819b288c36316e18ef29e3f0ed22fb84f654ae4f659a2ae87707791d62b2242 not found
ERROR: Encountered errors while bringing up the project.
```

# 解決
どうやら前回docker-compose downした際にプロジェクトを止めることができていなかったよう。
```--remove-orphans```オプションで一旦全て削除してから```docker-compose up```するとうまくいった
```bash
$ docker-compose down --remove-orphans
$ docker-compose up
```
*--remove-orphansは未定義コンテナという意味。

具体的な原因はprojectにdocker-compose.ymlが二つ(A,B)あり、AでUpした一方でBでDownさせた際にこのようなことが起こった。


