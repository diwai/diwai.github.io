# CLAUDE.md — diwai-website（岩井大輔 個人サイト）

Hugo 製の個人研究者サイト。**このファイルは Claude Code に前提知識を渡すためのもの**（人向け README は意図的に置いていない）。

## 概要

- 公開 URL: **https://diwai.github.io/**（GitHub Pages ユーザーサイト、リポジトリ `diwai/diwai.github.io`）
- 独自ドメイン `daisukeiwai.org` は**使わない**（旧 WordPress がさくらサーバで稼働中のまま。CNAME は削除済み）
- 2言語：**和文がデフォルトで `/`、英文は `/en/`**（`defaultContentLanguage = "ja"`）
- ユーザーとのやり取りは日本語。コミット／プッシュは**ユーザー自身が行う**ので、変更後は「コミットしますか？」と確認する

## ビルドとデプロイ

- ローカル: `hugo --gc --minify`（ローカル Hugo は 0.145、CI は 0.163.3）。プレビューは `.claude/launch.json` の `hugo`（`hugo server`）
- デプロイ: `main` に push → `.github/workflows/hugo.yml` → GitHub Pages
- **Pages の Source は必ず「GitHub Actions」**。ブランチ配信に変わると純正 Jekyll ビルドが走って 404 になる（過去に発生）
- ワークフローが git から算出して Hugo に渡す環境変数（ローカルではフォールバック）:
  - `HUGO_PARAMS_COPYRIGHTYEAR`（content/data 最終コミットの年 → フッターの © 年。ローカルは `now.Year`）
  - `HUGO_PARAMS_CONTENTLASTMOD`（同コミットの日時 → sitemap の `<lastmod>`。ローカルは出力なし）

## データが唯一の情報源（テンプレートを触らずデータで更新する）

- `data/publications.yaml` … 全業績（英文＋和文を統合）。セクションに `ja_only: true` があれば和文ページのみ表示
- `data/cv.yaml` … 受賞・経歴・講演・特許・学会活動・メディア掲載
- `data/projects.yaml` … トップのプロジェクトギャラリー
- `content/_index.{ja,en}.md` … トップの本文、`content/publications/_index.{ja,en}.md` … 業績ページ

### en / ja の表示ルール（重要）
- 英文ページ: `en` を持つエントリのみ表示
- 和文ページ: `ja` があれば `ja`、無ければ `en` を表示
- → **日本語のみの項目は `ja` だけ書く**（英文ページに出したくないものは `en` を付けない）
- `en == ja` の重複は awards / patents では削除済み（表示は同じ）

### cv.yaml の区分フィールド
- 受賞: `by: "self"` / `by: "coauthor"`（共著者・学生の受賞は h3 サブ見出し＋折りたたみ）
- 講演: `by: "coauthor"` を付けると「共著者の講演」（h3＋折りたたみ、初期は閉）に分離
- `selected: true` の受賞は常時表示、他は「その他 N 件を表示」の折りたたみ

## レイアウトの決まりごと

- リンクの書き方: 各項目テキスト末尾に `[ラベル](URL)` を書くと、`layouts/shortcodes/cv.html` が抽出して **🌐 チップ**（`.prose a.chip`）にする。講演・受賞・特許・学会活動で共通。複数リンク可、`(, )` の空括弧は自動除去
- 業績ページは年別ビュー（`layouts/partials/publist_byyear.html`）のみ。カテゴリ別ビューは廃止済み。カテゴリごとに B/J/C/N/M の通し番号（古い順）
- 背景帯: `layouts/partials/section-bands.html` が `.prose` を `<h2>` 単位で白／グレー交互の全幅帯に分ける（トップ＝セクション、業績＝年）。**h3 は帯を分けない**（この性質でサブセクションを同じ帯に収めている）
- 言語切替: ヘッダーのセグメント型スイッチ（`.lang-switch`）。選択は `localStorage.preferredLang` に保存
- 言語自動転送: `head.html` のトップ限定 JS。`/` に来た非日本語ブラウザを `/en/` へ。**クローラー UA は除外**（Google に重複判定された経緯あり）。検証中の `/en/` 重複判定が不合格なら**自動転送の廃止**が次の手
- 「トップへ戻る」ボタン、プロジェクトの「さらに表示」は全表示⇄折りたたみのトグル
- `static/share/paper/` には出版社サイトに無い PDF **8 本**のみ残す方針（他は削除済み）

## SEO / Google Search Console（2026-08〜）

- 所有権確認は HTML タグ方式（`params.googleSiteVerification`）。robots.txt・canonical・OGP・hreflang（x-default 含む）・sitemap `lastmod` 出力済み
- 状況: `/` と `/en/publications/` は登録済み。`/publications/` はクロール済み待ち。`/en/` は「重複」判定で検証中
- **サイトマップ送信の「取得できませんでした」は見た目だけ**（Googlebot は取得済み、全 URL 認識済み）。`sitemap.xml` は登録対象外なので「修正を検証」は押さない。これ以上の対処は不要と結論済み
- 効く施策は被リンク（XR グループサイト・researchmap・大阪大学研究者総覧から `https://diwai.github.io/` へ）

## やらないと決めたこと

- 特許番号を登録番号へ更新する作業（ユーザー判断で中止）
- Google Analytics・プライバシーポリシー（導入後に撤去）
- README.md（削除済み。復活させない）
- `<meta name="description">`／`og:description` の文面を作り込むこと（2026-09 撤去。Google 検索結果のスニペットが「AI が要約整理した文章」に見えるとの判断。`content` の `description` フロントマターも `hugo.toml` の `params.description` も置かない。ページに `.Description` が無ければ head.html はタグ自体を出力しない → 検索エンジン・SNS のクローラーに本文から自動で要約させる）
- 装飾目的の多色グラデーションバー（旧 `.spectrum`／`--spectrum`。プロフィール下・見出し下にあった「投影光の分散」を模した意匠。AI 生成サイトっぽく見えるとの理由で完全に削除。ナビのアクティブ下線は単色 `--accent` のまま残存）

## よく使う確認コマンド

```bash
hugo --gc --minify 2>&1 | grep -iE "error|warn|Total"   # ビルド
grep -oc 'pub-tag pub-tag-' public/en/publications/index.html   # 英文業績の件数（2026-09 時点 211、追記で増える）
grep -oc 'pub-tag pub-tag-' public/publications/index.html      # 和文業績の件数（2026-09 時点 547、追記で増える）
curl -sI https://diwai.github.io/ | head -3               # 公開状態
```
