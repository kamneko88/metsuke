# Metsuke 開発まとめ

最終更新日：2026-06-24

---

## プロジェクト概要

**Metsuke（目付）** は、Premiere Pro UXP プラグインです。
指定した監視フォルダ内のメディアファイルを自動的に Premiere Pro のプロジェクトビンにインポートします。
有料プラグイン「Watchtower」($40) の代替として、500円での販売を想定して開発しています。

- **リポジトリ**: https://github.com/kamneko88/metsuke
- **対応バージョン**: Premiere Pro 2025（v25.6以降）/ 2026（v26系）
- **技術**: HTML + CSS + JavaScript（単一ファイル構成）、Adobe UXP API

---

## 命名経緯

- 当初は「PatrolTower」という名前で開発開始
- Watchtower（監視塔）へのパクリ臭を避けるため、江戸幕府の監察役「目付（Metsuke）」に改名
- リポジトリ名・manifest・全ファイルを一括リネーム済み

---

## ファイル構成

```
metsuke/
├── manifest.json   # プラグイン定義
├── package.json    # npm設定
├── index.html      # UI + ロジック（単一ファイル）
├── .gitignore
└── README.md
```

### 単一ファイル構成の理由

UXP Developer Tool（UDT）でのロード時、パス解決の制約により CSS・JS を外部ファイルに分離すると読み込めないケースがあるため、すべて `index.html` にインライン記述しています。

---

## manifest.json の重要ポイント

```json
{
  "manifestVersion": 5,
  "id": "com.yourname.metsuke",
  "name": "Metsuke",
  "version": "1.2.0",
  "main": "index.html",
  "host": {
    "app": "premierepro",
    "minVersion": "25.6.0"
  },
  "entrypoints": [
    {
      "type": "panel",
      "id": "metsuke-panel",
      "label": { "default": "Metsuke" }
    }
  ],
  "requiredPermissions": {
    "localFileSystem": "fullAccess",
    "launchProcess": {
      "schemes": ["file"],
      "extensions": [".mp3", ".wav", "..."]
    }
  }
}
```

### ハマりポイント（manifest）

| 問題 | 原因 | 解決策 |
|---|---|---|
| `Cannot resolve module: main.js` | `package.json` に `"main": "index.html"` を書いていた | `package.json` の `main` を削除し、`manifest.json` のルートに `"main": "index.html"` を追記 |
| プラグインが認識されない | `manifestVersion: 7` を使用していた | `manifestVersion: 5` に変更 |
| UDT接続できない | UDTを管理者権限なしで起動していた | 管理者として実行 |

---

## 開発環境

- **OS**: Windows
- **PowerShell**: 7.6.2
- **Node.js**: v24.14.0
- **npm**: v11.11.0
- **エディタ**: Visual Studio Code
- **Premiere Pro**: 26.2.2
- **UXP Developer Tool（UDT）**: v2.2以降

### 開発ループ

1. Premiere Pro を起動
2. UDT を**管理者として**起動
3. 「Add Plugin」→ `manifest.json` を選択して Load
4. コード編集後は UDT で「Reload」
5. デバッグは UDT の「Debug」ボタン → Chrome DevTools で確認

---

## UXP API 重要メモ

### モジュール読み込み

```javascript
// ファイルシステム
const uxp = require('uxp');
const localFileSystem = uxp.storage.localFileSystem;

// Premiere Pro API
const ppro = require('premierepro');
```

**注意**: `require()` はすべて `try/catch` で囲むこと。エラー時にページ全体がクラッシュする。

### ファイルシステム

```javascript
// フォルダ選択ダイアログ
const entry = await localFileSystem.getFolder();

// 永続トークン（次回起動時の復元に使用）
const token = await localFileSystem.createPersistentToken(entry);
const restoredEntry = await localFileSystem.getEntryForPersistentToken(token);

// 設定の保存先
const dataFolder = await localFileSystem.getEntryWithUrl('plugin-data:/');
```

### Premiere Pro API

```javascript
// アクティブプロジェクト取得
const project = await ppro.Project.getActiveProject();

// ルートビン取得（FolderItem）
const rootFolderItem = await project.getRootItem();

// ビン作成（Actionパターン必須）
await project.executeTransaction((compoundAction) => {
  compoundAction.addAction(rootFolderItem.createBinAction(binName, false));
}, 'ビン作成');

// FolderItem にキャスト
const bin = ppro.FolderItem.cast(projectItem);

// ファイルインポート
await project.importFiles(
  [filePath1, filePath2],  // パスの配列
  true,                     // suppressUI
  targetBin,                // インポート先ビン
  false                     // asNumberedStills
);
```

### ビン作成の注意点

`rootItem.createBin()` は存在しない。必ず `executeTransaction` + `createBinAction` を使う。

### プロパティとメソッドの違い

- **プロパティ**: 同期（`await` 不要）
- **メソッド**: 非同期（`await` 必須）

---

## 機能一覧（v1.2.0時点）

### メイン画面

| 機能 | 説明 |
|---|---|
| フォルダ追加 | クリックでフォルダ選択ダイアログを開く |
| 今すぐ同期 | 手動で即時同期を実行 |
| 同期ログ表示 | ビン名・ファイル名・時刻をグループ表示 |
| ステータスバー | 同期結果・エラーを色付きで表示 |
| ⚙ボタントグル | メイン⇔設定をワンボタンで切替 |

### 設定画面

| 設定項目 | 内容 |
|---|---|
| 監視フォルダ一覧 | 登録フォルダを表示・×で削除 |
| 自動同期 ON/OFF | トグルスイッチ |
| 同期間隔 | 5秒 / 10秒 / 30秒（デフォルト）/ 60秒 / 2分 / 5分 |
| 最大表示件数 | 非表示 / 5 / 10 / 20 / 50件 |
| 時刻表示 | チェックボックス |
| 同期完了通知 | デスクトップ通知のON/OFF |
| 同期ファイル種類 | 動画 / 音声 / 画像 をカテゴリ単位でON/OFF |

### 対応ファイル形式

| カテゴリ | 拡張子 |
|---|---|
| 動画 | .mp4 .mov .avi .mkv .mxf .r3d .braw .mpg .mpeg .m4v .wmv .flv .webm |
| 音声 | .mp3 .wav .aac .aiff .flac .ogg .m4a |
| 画像 | .jpg .jpeg .png .gif .tif .tiff .bmp .webp .psd .psb .ai .eps |

---

## 既知の制限・今後の課題

| 項目 | 状況 |
|---|---|
| フォルダのD&D登録 | UXP の制限により現状不可。将来のAPI拡充待ち |
| バックグラウンド自動同期 | パネルが開いている間のみ動作（UXPパネルの仕様） |
| サブフォルダのビン構造再現 | 現状フラットにインポート。構造保持は今後実装予定 |
| Mac対応 | 未検証 |
| After Effects対応 | 未対応 |

---

## 差別化ポイント（vs Watchtower）

| 項目 | Watchtower | Metsuke |
|---|---|---|
| 価格 | $40（約6,000円） | **500円** |
| 技術基盤 | CEP（旧世代） | **UXP（最新世代）** |
| 同期ログ表示 | なし | **あり** |
| ビン名を自由設定 | なし | **あり** |
| ファイル種類フィルタ | チェックボックス式 | **チェックボックス式** |
| 日本語UI | あり | あり |
| 通知 | あり | あり |

---

## バージョン履歴

| バージョン | 内容 |
|---|---|
| v1.0.0 | 初回リリース。フォルダ追加・手動同期・自動同期・設定保存 |
| v1.1.0 | 設定画面追加、ログ表示、ステータスバー、フォルダ追加をダイアログ直結方式に変更 |
| v1.2.0 | 通知機能追加、ファイル種類フィルタ追加（カテゴリ式）、除外拡張子手入力廃止、アイコン変更（👁）、タイトルスタイル改善、サブフォルダ再帰同期対応、⚙ボタントグル化 |

---

## 販売計画

- **価格**: 500円
- **配布方法**: Adobe Creative Cloud Marketplace（予定）
- **配布形式**: `.ccx` パッケージ（UDTの「Package」機能で生成）
- **現状**: 開発・テスト中

### .ccx パッケージの作成手順

1. UDT でプラグインを選択
2. 右の「…」メニュー →「Package」
3. 出力フォルダを選択
4. `com.yourname.metsuke_premierepro.ccx` が生成される

---

## 今後実装予定の機能

- [ ] サブフォルダのビン構造を保ったままインポート
- [ ] 除外フォルダの登録
- [ ] ビン名の手動編集（設定画面から）
- [ ] Adobe Creative Cloud Marketplace への公開
