# youtube-summary-viewer

Consolidated Manager HTML（YouTube/RSS要約ビューア）の閲覧用公開ページ。

## 公開の仕組み

- `index.html` を更新して `main` ブランチへ push すると、GitHub Actions が自動でGitHub Pagesへ反映します
- ページには `noindex` 設定がされており、検索エンジンにはインデックスされません（URLを知っている人のみ閲覧可能）
- 公開URL: リポジトリの Settings > Pages で確認できます（`https://ochi1216.github.io/youtube-summary-viewer/`）

## 更新手順（自宅PC側）

1. `consolidated_html_summary_manager.py` を実行し、`_Consolidated_Manager.html` を生成する
2. このリポジトリのフォルダに `index.html` としてコピーする
3. `git add index.html && git commit -m "update" && git push`
