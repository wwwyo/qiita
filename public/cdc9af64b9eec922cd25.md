---
title: '[Prisma] db migrate時に毎回データを削除されそうになる'
tags:
  - prisma
  - Supabase
private: false
updated_at: '2023-02-09T18:28:51+09:00'
id: cdc9af64b9eec922cd25
organization_url_name: null
slide: false
ignorePublish: false
---
```:環境
prisma: ^4.7.1
db: postgresql(supabase)
```

# 問題
以前prisma-postgresqlの拡張機能を試しました。

https://qiita.com/www_y118/items/349435ad1e093ff53435

これ以降 `prisma migrate dev`するたびにデータが失われることが求められます。これは非常に困る。

実際のエラー
```bash
The following is a summary of the differences between the expected database schema given your migrations files, and the actual schema of the database.

It should be understood as the set of changes to get from the expected schema to the actual schema.

[+] Added extensions
  - uuid-ossp

- The following migration(s) are applied to the database but missing from the local migrations directory: 20230xxxxxx_migration

✔ We need to reset the "public" schema at "db.xxxxx.supabase.co:5432"
Do you want to continue? All data will be lost. … no
```

# 解決法
正攻法でない気がしますが、とりあえず拡張の部分を消すとデータの削除は防げます。
```js:schema.prisma
generator client {
  provider        = "prisma-client-js"
  // 削除 previewFeatures = ["postgresqlExtensions"]
}

datasource db {
  provider   = "postgresql"
  url        = env("DATABASE_URL")
  // 削除 extensions = [citext(schema: "public"), uuid_ossp(map: "uuid-ossp")]
}
```

# やったこと
dbからschemapullしたあとにmigrateしてもダメでした。
1. `prisma db pull`
1. `prisma migrate dev`

おそらくsupabase側のextensionsと衝突してる気がします。
ちなみにsupabase側の`uuid-ossp`は停止できなかった。
![スクリーンショット 2023-02-09 18.27.56.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/787586/5068a46c-596b-aa81-1173-9f840e2ddb60.png)
