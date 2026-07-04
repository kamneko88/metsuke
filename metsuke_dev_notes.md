# Metsuke 開発まとめ

最終更新日：2026-07-04

---

## プロジェクト概要

**Metsuke（目付）** は、Premiere Pro UXP プラグインです。
指定した監視フォルダ内のメディアファイルを Premiere Pro のプロジェクトビンに自動で読み込みます。

- **リポジトリ**: https://github.com/kamneko88/metsuke（非公開）
- **Booth**: https://kamneko.booth.pm/items/8547606
- **対応バージョン**: Premiere Pro 2025（v25.6以降）/ 2026（v26系）、Windows
- **現在のバージョン**: v1.3.2
- **プラグインID**: com.kamneko.metsuke

---

## 命名経緯

- 当初「PatrolTower」として開発開始
- Watchtowerへの類似を避け、江戸幕府の監察役「目付（Metsuke）」に改名
- プラグインIDも `com.yourname.metsuke` → `com.kamneko.metsuke` に変更済み

---

## 販売・配布方針

- **製品版**：Booth にて無料配布（応援版 500円 を併売）
- **おためし版（LE版）**：一時作成したが現在非推奨。製品版を無料公開する方針に変更したため
- Adobe Marketplace への掲載は将来検討

---

## ファイル構成

```
metsuke/
├── manifest.json            # プラグイン定義（v1.3.2）
├── package.json
├── index.html               # 製品版 UI + ロジック
├── metsuke_dev_notes.md     # 本ファイル
├── README.md
├── .gitignore
├── dist/                    # パッケージ済み .ccx（Git管理外）
└── metsuke-le/              # おためし版（参考用・現在非推奨）
    ├── manifest.json
    └── index.html
```

---

## manifest.json の重要ポイント

```json
{
  "manifestVersion": 5,
  "id": "com.kamneko.metsuke",
  "name": "Metsuke",
  "version": "1.3.2",
  "main": "index.html",
  "host": {
    "app": "premierepro",
    "minVersion": "25.6.0"
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

- **OS**: Windows 11 Pro
- **Premiere Pro**: v26.2.2（2026）
- **UXP Developer Tool（UDT）**: v2.2以降
- **エディタ**: VS Code
- **Node.js**: v24系 / npm v11系

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
const uxp = require('uxp');
const localFileSystem = uxp.storage.localFileSystem;
const ppro = require('premierepro');
```

**注意**: `require()` はすべて `try/catch` で囲む。エラー時にページ全体がクラッシュする。

### Premiere Pro API

```javascript
// アクティブプロジェクト取得
const project = await ppro.Project.getActiveProject();
const rootFolderItem = await project.getRootItem();

// ビン作成（Actionパターン必須）
await project.executeTransaction((compoundAction) => {
  compoundAction.addAction(rootFolderItem.createBinAction(binName, false));
}, 'ビン作成');

// FolderItemにキャスト
const bin = ppro.FolderItem.cast(projectItem);

// ファイルインポート
await project.importFiles([filePath1, filePath2], true, targetBin, false);

// ビンからアイテム削除（確認済み）
await project.executeTransaction((ca) => {
  ca.addAction(parentFolderItem.createRemoveItemAction(targetItem));
}, '削除');

// アイテムを別ビンに移動（確認済み）
await project.executeTransaction((ca) => {
  ca.addAction(destFolderItem.createMoveItemAction(targetItem, destFolderItem));
}, '移動');
```

### UXP スタイル制約（重要）

- `<button>` タグは UXP の Spectrum Widget に自動スタイルが上書きされる → `<div role="button">` で代替
- `border-radius` はボタン要素に対してUXP内部でオーバーライドされる
- `Notification` API は UXP 環境では非対応（`ReferenceError: Notification is not defined`）

### ファイルシステム

```javascript
const entry = await localFileSystem.getFolder();
const token = await localFileSystem.createPersistentToken(entry);
const restoredEntry = await localFileSystem.getEntryForPersistentToken(token);
const dataFolder = await localFileSystem.getEntryWithUrl('plugin-data:/');
```

---

## 機能一覧（v1.3.2）

### メイン画面

| 機能 | 説明 |
|---|---|
| フォルダ追加 | クリックでフォルダ選択ダイアログを開く |
| 今すぐ読み込み | 手動で即時読み込みを実行 |
| 読み込みログ表示 | ビン名・ファイル名・時刻をグループ表示 |
| ステータスバー | 読み込み結果・エラーを色付きで表示 |
| ⚙ボタントグル | メイン⇔設定をワンボタンで切替 |

### 設定画面

| 設定項目 | 内容 |
|---|---|
| 監視フォルダ一覧 | 登録フォルダを表示・×で削除（削除時はログも連動削除） |
| 自動読み込み ON/OFF | トグルスイッチ |
| 読み込み間隔 | 10秒 / 30秒（デフォルト）/ 60秒 / 2分 |
| 最大表示件数 | 非表示 / 5 / 10 / 20 / 50件 |
| 時刻表示 | チェックボックス |
| 読み込むファイル種類 | 動画 / 音声 / 画像 をカテゴリ単位でON/OFF |
| サブフォルダのビン構造を再現 | ON/OFF |

### 対応ファイル形式

| カテゴリ | 拡張子 |
|---|---|
| 動画 | .mp4 .mov .avi .mkv .mxf .r3d .braw .mpg .mpeg .m4v .wmv .flv .webm |
| 音声 | .mp3 .wav .aac .aiff .ogg .m4a |
| 画像 | .jpg .jpeg .png .gif .tif .tiff .bmp .webp .psd .psb .ai .eps |

※ `.flac` は Premiere Pro ネイティブ非対応のため除外

---

## 既知の制限

| 項目 | 状況 |
|---|---|
| フォルダのD&D登録 | UXP の制限により不可。クリックダイアログで代替 |
| バックグラウンド自動読み込み | パネルが開いている間のみ動作 |
| 削除・移動の反映 | 監視フォルダ内のファイル削除はビンに反映されない（UXP API制約） |
| デスクトップ通知 | UXP 環境で Notification API 非対応のため未実装 |
| Mac対応 | 未検証 |

---

## 差別化ポイント（vs Watchtower）

| 項目 | Watchtower | Metsuke |
|---|---|---|
| 価格 | $40（約6,000円） | **無料（応援版500円）** |
| 技術基盤 | CEP（旧世代） | **UXP（最新世代）** |
| 読み込みログ表示 | なし | **あり** |
| ビン名を自由設定 | なし | **あり** |
| ファイル種類フィルタ | あり | **あり** |
| サブフォルダ構造再現 | あり | **あり** |
| 削除の反映 | あり（フォルダ側削除のみ） | **なし（制約により未対応）** |

---

## バージョン履歴

| バージョン | 内容 |
|---|---|
| v1.0.0 | 初回リリース。フォルダ追加・手動同期・自動同期・設定保存 |
| v1.1.0 | 設定画面追加、ログ表示、ステータスバー、フォルダ追加をダイアログ直結方式に変更 |
| v1.2.0 | ファイル種類フィルタ追加、アイコン変更（👁）、タイトルスタイル改善、サブフォルダ再帰同期対応、⚙ボタントグル化 |
| v1.3.0 | サブフォルダのビン構造保持オプション追加 |
| v1.3.1 | buttonをdivに変更（UXPスタイル制約回避）、フォルダ削除時ログ連動削除、設定画面スクロール修正、フォントサイズ拡大、ティールカラー統一 |
| v1.3.1LE | おためし版リリース（監視フォルダ1つ・自動読み込み1分固定）※現在非推奨 |
| v1.3.2 | 「同期」→「読み込み」に全表記統一、flac除外、読み込み間隔を4択に整理、通知機能削除（API非対応）、プラグインIDをcom.kamneko.metsukeに変更、Noto Serif JPフォント適用、👀アイコンに変更 |
| v1.3.3 | 字幕ファイル対応追加（.srt .stl .xml .vtt .scc .mcc）、設定画面に「字幕」カテゴリを新設 |

---

## 今後の課題・バックログ

- [ ] 除外フォルダの登録機能
- [ ] ビン名の手動編集（設定画面から）
- [ ] 削除・移動の反映（UXP API対応状況を継続監視）
- [ ] Adobe Creative Cloud Marketplace への掲載
- [ ] Mac環境での動作確認
