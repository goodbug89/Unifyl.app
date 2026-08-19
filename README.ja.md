<p align="center">
  <img src="docs/screenshots/icon.png" alt="Unifyl" width="128" height="128">
</p>

<h1 align="center">Unifyl</h1>

<p align="center">
  <strong>macOS向けのAIデュアルペインファイルマネージャ。</strong><br>
  Total Commanderの深さ、ForkLiftの仕上がり、組み込みのAI知性。
</p>

<p align="center">
  <img src="https://img.shields.io/badge/macOS-14%2B-000?logo=apple&logoColor=white" alt="macOS 14+">
  <img src="https://img.shields.io/badge/Swift-6.0-F05138?logo=swift&logoColor=white" alt="Swift 6">
  <img src="https://img.shields.io/badge/Apple%20Silicon-Optimized-8E8E93" alt="Apple Silicon">
</p>

<p align="center">
  <a href="https://unifyl.app/ja/">ウェブサイト</a> &middot;
  <a href="https://github.com/goodbug89/Unifyl.app/wiki">ドキュメント</a> &middot;
  <a href="https://github.com/goodbug89/Unifyl.app/releases/latest">ダウンロード</a>
</p>

<p align="center">
  <strong>言語:</strong>
  <a href="README.md">English</a> &middot;
  <strong>日本語</strong> &middot;
  <a href="README.zh-Hans.md">简体中文</a> &middot;
  <a href="README.de.md">Deutsch</a> &middot;
  <a href="README.es.md">Español</a> &middot;
  <a href="README.fr.md">Français</a>
</p>

---

![Unifyl スクリーンショット](docs/screenshots/01-dualpane.png)

## 概要

Unifylは、プロ向けパワーツールとローカルAI知性を組み合わせたmacOSデュアルペインファイルマネージャです。あなたのファイルがMacを離れることはありません。

- **デュアルペイン** — シングル、デュアル、トリプル、自由分割のレイアウト、タブ、ワークスペース、各ペインにインラインターミナル
- **AI駆動** — セマンティック検索、AIリネーム、スマートタグ付け、重複検出 — すべてApple Neural Engineでローカル実行
- **リモート & クラウド** — FTP、SFTP、WebDAV、SMB、S3、Google Drive、Dropbox、OneDrive、Docker、Kubernetes
- **50以上のプロ向けツール** — マルチリネーム、テキスト/バイナリ/画像比較、ディレクトリ同期、3-wayマージ、EXIF一括編集
- **アーカイブを仮想フォルダに** — ZIP、7z、TAR、GZ、BZ2、XZ、ZSTD、RAR(CJKレガシーZIPの文字コード自動検出を含む)
- **キーボードファースト** — コマンドパレット、Vimスタイルのナビゲーション、120以上の再割り当て可能なショートカット
- **15言語対応** — 英語、韓国語、日本語、中国語(簡体・繁体)、ドイツ語、フランス語、スペイン語、ポルトガル語、イタリア語、ロシア語、トルコ語、アラビア語、タイ語、ベトナム語
- **CJK 強化** (1.3.0): `.alz` アーカイブ展開 (バンドル `unalz`)、`.hwpx` インラインプレビュー、ピンイン / ローマ字 / ハングル-漢字の多言語ファイル名検索
- **CJK 追加強化** (1.3.1): メッセンジャー/クラウドフォルダの自動ブックマーク(KakaoTalk · LINE · WeChat · iCloud Drive など 12 種)、全角 ↔ 半角 検索フォールディング、EUC-KR/Shift-JIS/GBK → UTF-8 エンコーディング変換(取り消し対応)、韓国政府様式のヒューリスティック検出

---

## インストール

[unifyl.app/download](https://github.com/goodbug89/Unifyl.app/releases/latest) から最新リリースを取得。

[Mac App Store](https://apps.apple.com/app/unifyl/id6773977314?mt=12) でも入手できます（サンドボックス版 — 機能の違いは FAQ を参照）。

1. `.dmg` ファイルを開く
2. Unifylをアプリケーションフォルダにドラッグ
3. 起動し、求められたらファイルアクセスを許可

---

## 価格

| | **Free** | **Pro — $24.99** |
|---|---|---|
| ペインレイアウト | デュアル/トリプル/自由分割（すべて） | シングル、デュアル、トリプル、自由分割 |
| 検索 | 基本 + Spotlight | 完全な正規表現 + コンテンツ検索 |
| AIツール | — | すべてのAI機能 |
| リモート接続 | FTP、SFTP | すべてのプロトコル + クラウドストレージ |
| アーカイブ | ZIP | ZIP、7z、TAR、GZ、BZ2、XZ、ZSTD、RAR、**ALZ** |

> **エディションについて。** Mac App Store 版はサンドボックスのため、Git パネル、Docker/Kubernetes、SSH トンネル、インラインターミナル、ポート/プロセスビューア、ffmpeg によるメディア変換を含みません — macOS がそこでは許可しないためです。それ以外は同一で、Pro の一回払い価格も両方で同じです。これらが必要な場合は 直接ダウンロード版をお選びください。

買い切り、サブスクリプションなし。Mac App Storeでは無料でダウンロードでき、ProはApp内課金（Apple IDに紐付き、ファミリー共有対応）。直接ダウンロード版も無料で、ProはLemonSqueezy経由（最大3台のデバイスで利用可能）。初回起動時に14日間の無料Proトライアル付き。

---

## 主なキーボードショートカット

| ショートカット | アクション |
|---|---|
| `⌘ + ⇧ + P` | コマンドパレット |
| `Tab` | アクティブペイン切り替え |
| `Return` | ファイル開く / フォルダ入る |
| `⇧ + Return` | 名前変更 |
| `Space` | クイックルック |
| `⌘ + Delete` | ゴミ箱に移動 |
| `⌃ + G` | サムネイルグリッド表示 |
| `⌘ + T` | 新規タブ |
| `⌘ + 2` | デュアルペイン |
| `⌘ + K` | ターミナル |

全120以上のショートカットは「環境設定 > キーボードショートカット」でカスタマイズ可能。

---

## ソースからのビルド

```bash
git clone https://github.com/goodbug89/Unifyl.app.git
cd unifyl
make doctor    # ツールチェイン確認
make gen       # Xcodeプロジェクト生成
make build     # 全パッケージ順次ビルド
```

詳細な機能リスト、貢献ガイドライン、プラグイン開発については [English README](README.md) を参照してください。

---

## ライセンス

プロプライエタリ。Copyright 2024–2026 Unifyl. All rights reserved.

<p align="center">
  <a href="https://unifyl.app/ja/">ウェブサイト</a> &middot;
  <a href="https://github.com/goodbug89/Unifyl.app/wiki">ドキュメント</a>
</p>
