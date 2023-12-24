---
title: コミットのお作法
tags:
  - Git
  - GitHub
private: false
updated_at: '2023-04-19T21:00:56+09:00'
id: faafd093ac7e9bf0d510
organization_url_name: null
slide: false
ignorePublish: false
---
ある日`git checkout .`を誤って打ってしまい修正ファイルが全て吹き飛びました。

コミットを小さな粒度で行うことで、こういったミスはある程度対処できそうです。

量が多くなるほど適切な名前をつけないと、後々どういった内容のコミットか思い出すことが難しくなります。
またレビュー時のレビュアーが処理を追うためにも重要な役割を果たします。

コードはよく読んでもコミットの方針やメッセージ等を見ることはあまりないと思い「コミットのお作法」をまとめようと思います。

# メッセージの書き方
主に3行で書く場合と１行で書く場合があります。個人的には3行の方が丁寧で良いかと思っています！
3行の例
```
1 <修正を一言で>
2 空白
3 <なぜ修正をしたかの説明>
```

１行の例
#100 fix 〇〇が✖︎✖︎の時エラーになる修正。
* #100はチケットやissueのNo

### 含ませるもの
1. プレフィックス
1. チケットNo
1. 修正の内容

*順番はプロジェクトによって変わる

### 1.プレフィックスをつける
どんな修正か一目でわかる。
ここらあたりで網羅できそう。
```
add：新規（ファイル）機能追加
fix：バグ修正
update：機能修正（バグではない）
remove：削除（ファイル）
style:スタイルの修正
refactor：リファクタリング
upgrade：バージョンアップ
test：テスト関連
```

#### 「覚えるの大変だよ！」
という方はこちらのコマンドを入れていただくとcommit時に参照できます！
1. `$ vim ~/.commit_template`
2. 🔽コピペ
```.commit_template
# ==== Prefix ====
# add:        新規（ファイル）機能追加
# fix:        バグ修正
# update:     機能修正（バグではない）
# remove:     削除（ファイル）
# style:      スタイルの修正
# refactor:   リファクタリング
# upgrade:    バージョンアップ
# test:       テスト関連
```
3.`$ git config --global commit.template ~/.commit_template`
4.`$ git commit`
![スクリーンショット 2022-05-24 22.40.23.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/787586/d2984b6c-3d59-c29d-d97c-906f9a75872e.png)

### 2.チケットNo
チケットを管理しているシステムとの連携。
どのチケットか一目でわかる。

### 3.修正の内容
なぜその変更を行ったかを含ませる。
コードを変更した背景がわかる、後から見返しやすい。
Bad:  〇〇を追加
Good: ✖︎✖︎のため〇〇を追加

現在形で書く。
英語 or 日本語に関してはどちらでも良いがリポジトリで統一する

# 方針
「細かく」行う。レビューがやりやすい、revertやrebase
がやりやすい。
「細かく」とはそれぞれのコミットは意味を持った単位で行う。
SOLID原則で言うところの単一責任の原則。普段のプログラムと同様にコミットもそのコミットが何を示しているか一言で言えるように行う。

### 以前のcommitにtypoやインデントのずれがあったとき
前回のコミットに含ませる。
- 一つ前
```bash
$ git commit --amend
```
- 一つ以上前
```bash
# nはいくつ前のコミットを修正するか
$ git rebase -i HEAD~n
# => pick 059f53cbf <commit message>
# 変更を加えたいコミットのpickを「e」もしくは「edit」に変更してファイル修正
$ git commit --amend
$ git rebase --continue
```

### メッセージを変更したい
- 一つ前
```bash
$ git commit --amend -m "変更したいメッセージ"
```
- 一つ以上前
```bash
$ git rebase -i HEAD~n
# => pick 059f53cbf <commit message>
# 変更したいコミットのpickを「r」か「reword」に変更。
# => reword 059f53cbf <commit message>
# 編集ページが立ち上がるので、変更。
$ git rebase --continue
```
* rebaseの時点でエラーが出たら
```terminal
# エラー無視して続行
$ git commit --amend --no-verify

# やっぱ中止
$ git rebase --abort
```

### コミットを無かったことにしたい
commitの変更分を元に戻すcommitを作り出す。
```bash
$ git revert <無くしたいcommitのhash値>
```
*rebaseのdropでもできますが、履歴を残しておいた方が取消しの取り消しが容易なのでrevertを使う。

### コミットをやり直したい
```bash
# これより前のcommitが取り消される。*ファイルが消える訳ではない
$ git reset <戻したいcommitのhash値>
```

### コミットまとめてと言われた時
rebaseを用い、直前のpickと統合させる。
```bash
$ git rebase -i HEAD~2
# 直前のcommitのpickを「s」もしくは「squash」に変更
# commit messageを編集
```
詳細:[Git squashで複数のコミットを1つでまとめる](https://dev-yakuza.posstree.com/git/git-squash/)

# 追記
最近使ってるテンプレ
```
# ==== Prefix ====
# fix:        不具合の修正
# add:        ちょっとしたファイル・コードの追加 ex)画像ファイル
# feat:       新機能
# refactor:   バグ修正や機能の追加を行わないコードの変更
# test:       テストコードの変更
# style:      コードの処理に影響しない変更（スペースや書式設定など）
# chore:      ドキュメントの生成やビルドプロセス、ライブラリなどの変更
# docs:       ドキュメントのみの変更
# perf:       パフォーマンス改善を行うためのコードの変更
# ci:         CI用の設定やスクリプトに関する変更
```
# 参考
https://qiita.com/itosho/items/9565c6ad2ffc24c09364

https://qiita.com/konatsu_p/items/dfe199ebe3a7d2010b3e

https://qiita.com/gogotanaka/items/8c55f69120965b077737

https://www.praha-inc.com/lab/posts/commit-message

https://suwaru.tokyo/%E3%80%90%E5%BF%85%E9%A0%88%E3%80%91git%E3%82%B3%E3%83%9F%E3%83%83%E3%83%88%E3%81%AE%E6%9B%B8%E3%81%8D%E6%96%B9%E3%83%BB%E4%BD%9C%E6%B3%95%E3%80%90prefix-emoji%E3%80%91/#anc_22

