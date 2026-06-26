# Metsuke（目付）

最終更新日：2026-06-24

**目付** — フォルダ自動同期 Premiere Pro UXPプラグイン / Watch folder auto-sync plugin for Premiere Pro

---

## 概要

指定したフォルダを監視し、追加されたメディアファイルを自動的に Premiere Pro のプロジェクトビンにインポートします。
サブフォルダ内のファイルにも対応しています。

有料プラグイン「Watchtower」（$40）の代替として、より手頃な価格での提供を目指して開発しています。

---

## 機能

- 監視フォルダを複数登録
- 手動同期（今すぐ同期ボタン）
- 自動同期（5秒〜5分間隔で設定可能）
- サブフォルダ内のファイルも同期
- 同期ログ表示（ファイル名・時刻）
- 同期完了時のデスクトップ通知（ON/OFF可）
- ファイル種類フィルタ（動画・音声・画像をカテゴリ単位でON/OFF）
- 設定の自動保存・次回起動時復元

---

## 対応環境

- Premiere Pro 2025（v25.6以降）/ 2026（v26系）
- Windows（Mac未検証）

---

## インストール

### .ccx ファイルからインストール（推奨）

1. [Releases](https://github.com/kamneko88/metsuke/releases) から最新の `.ccx` ファイルをダウンロード
2. ダウンロードした `.ccx` ファイルをダブルクリック
3. Creative Cloud デスクトップアプリが開くので「Install」をクリック
4. Premiere Pro を起動 → **Window → UXP Plugins → Metsuke** で開く

### 開発版（UDT使用）

1. [UXP Developer Tool（UDT）](https://developer.adobe.com/photoshop/uxp/devtool/) をインストール
2. Premiere Pro を起動
3. UDT を**管理者として**起動
4. 「Add Plugin」→ このフォルダの `manifest.json` を選択
5. 「Load」をクリック
6. Premiere Pro の **Window → UXP Plugins → Metsuke** で開く

---

## ファイル構成

```
metsuke/
├── manifest.json        # プラグイン定義
├── package.json         # npm設定
├── index.html           # UI + ロジック（単一ファイル）
├── metsuke_dev_notes.md # 開発まとめ・引き継ぎノート
├── README.md
└── .gitignore
```

---

## バージョン履歴

| バージョン | 内容 |
|---|---|
| v1.0.0 | 初回リリース。フォルダ追加・手動同期・自動同期・設定保存 |
| v1.1.0 | 設定画面追加、ログ表示、ステータスバー |
| v1.2.0 | 通知機能、ファイル種類フィルタ、サブフォルダ再帰同期、UIブラッシュアップ |

---

## ライセンス

MIT
