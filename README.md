# PatrolTower

Premiere Pro UXP プラグイン — 監視フォルダの自動同期ツール

## 概要

指定したフォルダを監視し、追加されたメディアファイルを自動的に Premiere Pro のプロジェクトビンにインポートします。有料プラグイン「Watchtower」の代替を目指した無料・オープンソース実装です。

## 機能

- 監視フォルダを複数登録
- 手動同期（今すぐ同期ボタン）
- 自動同期（5秒〜5分間隔）
- 同期ログ表示（ファイル名・時刻）
- 設定画面（除外拡張子・表示件数など）

## 対応環境

- Premiere Pro 2025（v25.6以降）/ 2026（v26系）
- Windows（Mac未検証）

## インストール（開発版）

1. [UXP Developer Tool（UDT）](https://developer.adobe.com/photoshop/uxp/devtool/) をインストール
2. Premiere Pro を起動
3. UDT を**管理者として**起動
4. 「Add Plugin」→ このフォルダの `manifest.json` を選択
5. 「Load」をクリック
6. Premiere Pro の **Window → UXP Plugins → PatrolTower** で開く

## ファイル構成

```
patrol-tower/
├── manifest.json   # プラグイン定義
├── package.json    # npm設定
├── index.html      # UI + ロジック（単一ファイル）
└── .gitignore
```

## ライセンス

MIT
