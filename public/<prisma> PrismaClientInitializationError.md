---
title: <prisma> PrismaClientInitializationErrorでDBに接続できな苦なった
tags:
  - prisma
  - Supabase
private: false
updated_at: '2024-02-06T02:29:40+09:00'
id: d2cd1f980706fbebe31b
organization_url_name: null
slide: false
ignorePublish: false
---

# はじめに

長らく放置していたプロジェクトで急に Prisma のエラーが発生しました。エラー内容は以下の通りです。

```bash
PrismaError {
  message: PrismaClientInitializationError:
  Invalid `prisma.xxxx.findMany()` invocation:


  Can't reach database server at `db.<project-ref>.supabase.co`:`5432`
}
```

上述のエラーは、Prisma の初期化に失敗したことを示しています。
4 半期ほどコードベースでの修正は行っていない、かつ正常に動いていたため、何が原因でこのエラーが発生したのかを調査しました。
原因と解決法をまとめます。

# 要約

Supabase の db 接続アドレスが変更された(厳密にはその後ろで動いている AWS の アドレスが変更された)ことにより、Prisma の初期化に失敗した。
新しいアドレスに自動で書き換わっているので、[Supabase の設定画面](https://supabase.com/dashboard/project/_/settings/database)から、更新された接続情報を取得しプロダクションコードに反映する。

# 前提

- supabase
- prisma
- Next.js

# 詳細

## 1. 原因解明

AWS など大手クラウドベンダーが IPv4 のサポートに料金を請求し始めたことを受け、Supabase も IPv6 をサポートするようになったようです。
そこで、直接 IPv4 のアドレスに直接接続していた場合は以下のいずれかの方法で対応が必要です。

- IPv6 に対応した新しいアドレスに変更する
- API 経由で接続する
- Supervisor と呼ばれる Supabase が用意したコネクションプールを利用する

## 2. 対策

[公式](https://github.com/orgs/supabase/discussions/17817)に従うと Supervisor を使うことをおすすめされているようだったようで今回は Supervisor への接続へ変更することによる解消を行いました。
URL は[Supabase の設定画面](https://supabase.com/dashboard/project/_/settings/database)からプロジェクトを選択すると表示されます。
この URL を使って接続情報の環境変数を書き換えることで解消しました。

# まとめ

9 月ごろから通知はされており、メールも来てたらしい、全然気づかなかった。
メール大事。

# 参考文献

- [公式 issue](https://github.com/orgs/supabase/discussions/17817)
