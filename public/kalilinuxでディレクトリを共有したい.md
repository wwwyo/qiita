---
title: UTM × Kali Linux 2025で共有フォルダ設定に苦戦した話【完全解決版】
tags:
  - KaliLinux
  - UTM
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---

# はじめに

macOS の UTM で Kali Linux 2025 を動かしている方で、共有フォルダ設定でお困りの方はいらっしゃいませんか？

私は最近、UTM で Kali Linux 環境を構築する際に、ホスト側のフォルダを共有する設定で `special device … does not exist` エラーに遭遇し、解決まで時間を要しました。

同様の問題でお困りの方に向けて解決法をまとめました。

# 要約

**結論から申し上げると、UTM の VirtFS 共有ではタグ名が `share` で固定されています。**

この仕様を知らずに、任意の名前（`kali`、`shared` など）を設定すると、ゲスト側で認識されずエラーが発生します。

この記事では以下の内容について説明します：

- UTM × Kali Linux 2025 での共有フォルダ設定方法
- `special device … does not exist` エラーの原因と解決法
- 自動マウント設定の安全な方法
- 代替手段（VirtioFS、WebDAV）の選択肢

# 前提環境

検証環境は以下の通りです：

| 項目             | バージョン/設定          |
| ---------------- | ------------------------ |
| UTM              | 4.x（QEMU バックエンド） |
| ゲスト OS        | Kali Linux 2025.1        |
| 共有方式         | VirtFS (9p)              |
| ホスト側フォルダ | `~/任意`                 |
| UTM タグ名       | `share`（**これ重要**）  |
| マウント先       | `/media/share`           |

:::note warn
**重要なポイント**

- QEMU バックエンドの場合、VirtFS でタグ名は `share` 固定
- Apple Virtualization バックエンドの場合は後述の別解を参照
  :::

# 遭遇したエラーと原因

## 発生した症状

最初に以下のコマンドを実行したところ、エラーが発生しました：

```bash
sudo mount -t 9p -o trans=virtio,version=9p2000.L kali /media/kali
```

```
mount: /media/kali: special device kali does not exist.
```

ログを確認すると、以下のメッセージが出力されていました：

```bash
dmesg | grep 9p
# → 9pnet_virtio: no channels available for device kali
```

この `no channels available` が問題解決の重要な手がかりとなりました。

## 原因の特定

エラーの原因は以下の通りです：

1. **UTM の VirtFS 共有ではタグ名が `share` で固定されている**
2. ゲスト側はタグ名でデバイスを探索するため、一致しないと接続できない
3. 任意の名前（`kali`）を設定していたため、デバイスが見つからない

# 解決手順

## 1. UTM 側の設定

まず、Kali VM を停止します：

次に、UTM の設定を開いて以下の手順で設定します：

1. **設定 → 共有** を選択
2. 既存の共有エントリがあれば削除
3. 新しく追加する際の設定：
   - **ディレクトリ**: 任意
   - **モード**: `VirtFS`
4. 保存して VM 起動

## 2. ゲスト側での設定

VM が起動したら、以下のコマンドを実行します：

```bash
# マウントポイント作成(任意)
sudo mkdir -p /media/share

# マウント実行 (shareは固定)
sudo mount -t 9p -o trans=virtio,version=9p2000.L share /media/share

# 動作確認
ls /media/share
```

成功すると、ホスト側のファイルが表示されます。

# まとめ

UTM × Kali Linux 2025 での共有フォルダ設定において重要な点：

1. **タグ名は `share` 固定**が最重要
2. エラーが発生した場合は `dmesg | grep 9p` でログ確認
3. `/etc/fstab` で自動化する際は `_netdev,nofail` を付ける
4. 高速性を重視するなら Apple Silicon + VirtioFS も検討価値あり

最初はエラーの原因が分からずに苦労しましたが、タグ名の仕様を理解してからは問題なく設定できました。

同様の問題でお困りの方の参考になれば幸いです。
