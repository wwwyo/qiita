---
title: npmのパッケージバージョンを固定する運用に切り替えた話
tags:
  - npm
private: false
updated_at: '2025-05-23T09:52:35+09:00'
id: ae417acc3d99a98953b9
organization_url_name: null
slide: false
ignorePublish: false
---

# npmのパッケージバージョンを固定する運用に切り替えた話

## はじめに

npmの依存パッケージ管理について、これまで「package-lock.jsonを管理していれば十分」と考えていました。しかし、チーム開発や自動アップデートツール（dependabot等）の導入が進む中で、バージョンの不一致や意図しないアップデートによるトラブルが増えてきました。

そこで、**パッケージのバージョンを完全に固定する運用**へ切り替えたので、その背景と手順をまとめます。

---

## 背景・課題

- `package-lock.json`のコンフリクトが頻発し、解消のたびにバージョンがずれることがあった
- `^`や`~`でバージョン指定していると、lockファイルを消して再インストールした際に、意図しない新しいバージョンが入ることがあった
- 結果として、「意図せず挙動が変わった」「最新バージョンのバグを踏んでしまう」といった問題が起きやすくなった


## バージョン固定の方針

- すべての依存パッケージを「完全一致」で管理する
- `package.json`のバージョン表記も信頼できる状態にする
- lockファイルを消しても、同じバージョンで再現できるようにする


## 実際の手順

### 1. 依存パッケージを最新化

```bash
npm update --save
```
これで、現状のバージョン範囲内で最新のものに揃えます。
`--save` をつけることで、`package.json`のバージョンが更新されます。
変わらないものは手動で`package.json`を修正しました。

### 2. lockファイルと`node_modules`を削除

```bash
rm -rf package-lock.json node_modules
npm cache clear --force
```
一度クリーンな状態にします。

### 3. `.npmrc`で「バージョン固定」設定

```bash
touch .npmrc
```
`.npmrc`に以下を記述します。

```bash
save-exact=true
```
これにより、`npm install`時にバージョンが完全一致で記録されるようになります。

### 4. パッケージを再インストール

```bash
npm i
```
これで`package.json`のバージョン指定がすべて「完全一致」になり、以降のインストールもブレなくなります。

---

## 注意点・補足

- 既存の`^`や`~`が残っている場合は、手動で修正が必要です
- lockファイルを消す場合は、必ずバージョン指定を見直してください

---

## 参考リンク

- [Renovate公式：Dependency Pinning](https://docs.renovatebot.com/dependency-pinning/)

---

## まとめ

パッケージのバージョンを固定することで、
- チーム全体で同じ環境を再現しやすくなる
- 意図しないアップデートによるトラブルを防げる
といったメリットがありました。
