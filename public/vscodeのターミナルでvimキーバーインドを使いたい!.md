---
title: vscodeのターミナルでvimキーバーインドを使いたい!
tags:
  - 'vscode'
  - 'vim'
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---

# はじめに
vscodeのターミナルで文字を消す際にdelキーを使ったり、カーソル移動に矢印キーを使っていませんか？
そんなことをしていてはカーソル操作だけで一日が終わってしまうため、vimキーバインドを使って効率的に作業しましょう！


# 要約
vscodeのターミナルでvimキーバインドを使う方法を紹介します。
.zshrcに一行加えるとvscのターミナルでキーバインドが効きます。
```.bashrc
# .zshrc に追加
bindkey -v
```

# 前提
vscのターミナルのshellがzshを使っている想定です。
zshではない場合は 「⌘ + shift + p」から `select default profile` を選択すると変更できます。

# 詳細

## 1. zshの bindkey -v でvimキーバインドを使う
zshにはbindkeyというキーバインドを割り当てる関数が存在します。
デフォルトではEmacsのキーバインドが割り当てられているため、vimのキーバインドを使いたい場合は以下のコマンドを.zshrcに追加すればok。

```bash
# .zshrc に追加
bindkey -v
```

これで 「esc」からvimのキーバインドが使えるようになります。もう矢印キーを使う必要はありません。

## 2. Emacsキーバインドとは
Emacsはテキストエディタの一つで、かつてはviとのエディタ戦争があったほど有名でした。
[Emacs用のキーバインド](https://www.aise.ics.saitama-u.ac.jp/~gotoh/EmacsKeybind.html)もあるので好みで選んでください
![alt text](https://i.gzn.jp/img/2021/07/23/the-era-of-visual-studio-code/2020-09-20-text-editor-popularity.png)
> 画像引用元: [GIGAZINE](https://gigazine.net/news/20210723-the-era-of-visual-studio-code/)
# まとめ
vscodeのターミナルでvimキーバインドを使う方法を紹介しました。
エディタはvscodeを使っていても、vimのキーバインドを併用することで効率的に作業ができます。
是非、試してみてください！

# 参考文献

- [Use vim keybindings in visual studio code terminal]([URL](https://stackoverflow.com/questions/68807956/use-vim-keybindings-in-visual-studio-code-terminal))