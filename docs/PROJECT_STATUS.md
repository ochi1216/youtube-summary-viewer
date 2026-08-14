# PROJECT_STATUS.md

最終更新: S01（セッション引継ぎ管理の導入）

このファイルは、リポジトリ内のコード・README・Git履歴・ユーザーから共有された
既存資料から**確認できる事実のみ**を記載する。確認できない項目は「未確認」とする。

---

## プロジェクト概要

プロジェクト名: **(H)Podcast オーガナイザー**

リポジトリ名は `youtube-summary-viewer` であり、README には「YouTube/RSS要約ビューア」と
記載されている。**本リポジトリは公開ビューアの配信専用であり、Podcast オーガナイザー本体
（収集・要約・統合を行うスクリプト群）はこのリポジトリの外（ユーザーの家PC）にある。**

現状の全体像（ユーザーから共有された既存Pythonコード、および `index.html` から確認）:

```
[家PC・リポジトリ外]
  youtube_summary_list.py         YouTube収集・要約 → summary_<CAT>_<日時>.html
  rss_organizer.py                RSS/note収集・要約 → summary_RSS_<日時>.html
  consolidated_html_summary_manager.py
                                   上記を集約し index.html 相当のHTMLを生成
       ↓ publish_to_iphone.bat でコピー
[本リポジトリ]
  index.html                      公開ビューア本体（生成物）
       ↓ push
[GitHub Actions → GitHub Pages]
  https://ochi1216.github.io/youtube-summary-viewer/
```

Podcast を新しい情報ソースとして既存パイプラインに追加する計画がある
（ユーザー共有の引き継ぎ資料「Podcast要約自動化 ― Claude Code 引き継ぎ資料」に基づく）。
現時点ではまだ実装されていない（`index.html` 内に Podcast データは存在しない）。

---

## 実行環境

- **リポジトリ内**: 静的サイト。ビルド不要。GitHub Actions が `index.html` をそのまま
  GitHub Pages へデプロイする（`.github/workflows/pages.yml`）。
- **リポジトリ外（家PC、未確認）**: Python 3系。確認できた依存ライブラリ
  （ユーザー共有コードの import 文より）: `selenium`, `webdriver_manager`,
  `youtube_transcript_api`, `google.generativeai`, `feedparser`, `playwright`,
  `beautifulsoup4 (bs4)`, `tkinter`, `psutil`。実行OS・Pythonバージョンは未確認。

---

## 主要ファイルと役割

### リポジトリ内

| ファイル | 役割 |
|---|---|
| `index.html` | 公開ビューア本体。**生成物であり手編集しない**（`CLAUDE.md` 参照） |
| `README.md` | 公開の仕組みと、家PC側での更新手順の説明 |
| `.github/workflows/pages.yml` | push時に GitHub Pages へ自動デプロイ |
| `.nojekyll` | Jekyll処理を無効化 |
| `apple-touch-icon.png` | iOSホーム画面追加時のアイコン |

### リポジトリ外（家PC・未管理）

ユーザーから共有されたコードを解析した限りでの役割。**このリポジトリでは管理していない。**

| ファイル | 役割 |
|---|---|
| `youtube_summary_list.py` | YouTube プレイリストの収集・文字起こし取得・Gemini要約・`summary_<CAT>_<日時>.html` 出力 |
| `rss_organizer.py` | RSS/note記事の収集・本文取得・Gemini要約・`summary_RSS_<日時>.html` 出力 |
| `consolidated_html_summary_manager.py` | 上記の `summary_*.html` 群を走査・解析し、ダッシュボード集計込みで `_Consolidated_Manager.html` を生成 |

---

## 現在実装済みの機能

`index.html` を解析して確認できた機能:

- カテゴリ別フィルタ（実データで N / S / Short / A / V / M / B の7種）
- 時間絞込（直近24時間 / 直近7日間）
- ダッシュボード（件数・カテゴリ別チップ・注目記事・7日間トレンドグラフ）
- 要約カード表示（サムネイル・尺インジケーター・チャンネル名・登録者数・キーワード・要旨・
  結論・主なポイントのアコーディオン）
- Web Speech API による音声読み上げ（5段階モード、速度可変）
- 既読管理（`localStorage`、ファイル名+項目インデックス単位）
- お気に入り表示（★マーク、ハイライト配色）
- RSSカードのオンデマンド詳細生成（ブラウザ側から Gemini API を直接呼び出し）

---

## 確定済みの仕様

- 生成HTMLのファイル命名規則: `summary_<CATEGORY>_<YYYYMMDD>_<HHMMSS>.html`
  （`CATEGORY` にアンダースコアを含めてはならない。カテゴリ抽出が
  `filename.split('_')[1]` のため）
- 要約結果の item データは13キー（YouTube系）: `type` / `title` / `summary` /
  `conclusion` / `points` / `thumbnail` / `channel` / `subscriber` / `duration` /
  `is_favorite` / `is_error` / `keywords` / `url`
- 読み上げが参照するのは `title` / `summary` / `points` / `conclusion` の4フィールドのみ
- 既読管理のキーは `` `${filename}_${itemIndex}` ``（URLやタイトルではない）
- `localStorage` のキーは `summaryManagerState` と `gemini_api_key` の2つのみ

---

## 現在の開発状態

- 直近の作業: Podcast を新しい情報ソースとして統合するための設計調査
  （既存コード解析・Section21形式のアウトプット作成）。**コード変更は未着手。**
- Git: `main` と作業ブランチ `claude/podcast-summary-integration-eu2w7n` は
  同一コミットで同期済み。

---

## 既知の問題

ユーザー共有コードの解析で判明した、今後の実装時に注意すべき既存の挙動:

- `consolidated_html_summary_manager.py` は、既知のカード構造（`div.video-card` /
  `div.thread-card`）に一致しない `summary_*.html` を **例外なく items 0件として
  破棄**し、しかも元ファイルを `archive/` へ移動する。新しいソース種別を追加する際は
  先にパーサ対応を入れないと、データが復旧しづらい形で失われる。
- HTML生成側のエスケープ処理が弱く（`'` の置換のみ）、タイトル等に `</script>` が
  含まれると公開ページ全体が壊れうる。
- README に記載の更新手順（`consolidated_html_summary_manager.py` を実行 →
  `index.html` としてコピー → push）は YouTube/RSS 前提で書かれている。
  Podcast 対応後に更新が必要になる可能性があるが、本セッションでは変更していない。

---

## テスト方法

未確認（リポジトリ内に自動テストは存在しない。手動でのブラウザ表示確認が前提と推測されるが、
明文化された手順は README にはない）。

---

## 必要な環境変数

ユーザー共有コードから確認できたもの（**値はここに記載しない**）:

- `YT_SUMMARY_OUTPUT_DIR` — 要約HTML群の出力先ディレクトリ（YouTube/RSS双方で共有）
- `YT_CONSOLIDATION_BATCH` — 統合バッチのパス
- `GEMINI_API_KEY` — Gemini API キー（家PC側の実行環境で使用）

いずれもリポジトリ外（家PC側）の実行環境の話であり、本リポジトリの GitHub Actions
ワークフローは環境変数を使用していない（`.github/workflows/pages.yml` に環境変数の記載なし）。

---

## 外部サービスへの依存

- **GitHub Pages**（本リポジトリの公開先）
- **Google Gemini API**（家PC側の要約処理、および `index.html` 内のRSSオンデマンド生成機能）
- Web Speech API（ブラウザ内蔵、外部サービスではないがブラウザ実装に依存）

---

## 変更禁止事項

`CLAUDE.md` の「1. 変更禁止事項」を参照。要点:

- `index.html` を手編集しない
- 既存ファイルの削除・移動・リネームをしない
- Git履歴を書き換えない
- 認証情報を記載しない

---

## 後方互換性に関する注意

- `index.html` の `localStorage` は `summaryManagerState` に既読履歴・UI設定を保持している。
  データ構造（特に既読キー `filename_itemIndex`）を変更すると、ユーザーの既存の既読状態が
  失われる。
- カテゴリコードは表示・読み上げモードの両方に影響する（`mgr` 側の正規表現が
  ファイル名プレフィックスで読み上げモードを自動選択している）。新カテゴリ追加時は
  この一貫性を壊さないよう注意する。
