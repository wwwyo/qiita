---
title: Multipassでマウント時に所有者がnobody＆Permission Deniedになる問題の原因と解決策
tags:
  - multipass
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---

# Multipass でマウント時に所有者が nobody＆Permission Denied になる問題の原因と解決策

## 問題の概要

Multipass で Mac のローカルディレクトリを Ubuntu インスタンスにマウントした際、以下の問題が発生。

- マウントしたディレクトリにアクセスできない（Permission denied）
- インスタンス内で`ls -al`コマンドの結果、ファイル所有者が`nobody`になっている

## 調査で判明した原因

### 1. Multipass の`--mount`は UID/GID マッピングが必要

- Mac 側の UID/GID が正しく伝わっていなかった
- マウント時に明示的な指定が必要

```bash
# 例
# Mac 側の UID/GID
id -u
id -g

# Multipass 側の UID/GID にマッピング
multipass mount --uid-map 501:1000 --gid-map 20:1000 /Users/example_user/shared_dir UBUNTU:/mnt/c/shared_dir
```

### 2. Mac 側で所有者情報が壊れていた

- `ls -al`の結果、マウント元のファイルの所有者が 777 など不正な UID になっていた
- おそらく色々触っている時に `chmod`誤って `chown`で叩いている...

## 解決手順

### Mac 側で所有者とグループを正しく戻す

```bash
sudo chown -R example_user:staff /Users/example_user/shared_dir
```
