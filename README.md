# daisukeiwai.org — Hugo static site

大阪大学・岩井大輔サイトの Hugo 版です。装飾を抑えた最小構成で、**英語・日本語の切替**、GitHub Pages への**自動デプロイ**、独自ドメイン設定に対応しています。コンテンツはすべて Markdown で編集します。

---

## 1. ディレクトリ構成

サイトは **2 ページ構成**です。「業績（Publications）」だけ独立ページで、
それ以外（略歴・研究プロジェクト・受賞・経歴・講演・研究費・特許・学会活動・
メディア掲載）はすべて **1 枚のトップページ**にまとまっています。ナビの
Projects / Awards / CV はトップページ内の各セクション（`#projects` `#awards`
`#cv`）へのジャンプです。

```
hugo.toml                     # サイト設定（言語・メニュー・独自ドメイン）
content/
  _index.en.md / _index.ja.md # トップページ（略歴・プロジェクト・受賞・経歴・CV・メディア）
  publications/ _index.{en,ja}.md   # 業績ページ（見出し文だけ。中身は data/ から生成）
data/
  publications.yaml           # 全業績（英文＋和文）を統合した唯一のデータ源
layouts/                      # テンプレート（自前・外部テーマ非依存）
  partials/publist_byyear.html # data/ の YAML から業績 HTML を生成（年別）
assets/css/main.css           # デザイン（1ファイル）
static/images/                # 画像はここに置く
static/CNAME                  # 独自ドメイン（daisukeiwai.org）
.github/workflows/hugo.yml    # GitHub Pages 自動デプロイ
```

言語はファイル名で切り替わります（`_index.en.md` ⇄ `_index.ja.md`）。英語は `/`、日本語は `/ja/` に出力され、現行サイトと同じ URL 構造です。トップページ内のセクションは `## 見出し {#id}` の形で id を付けており、ナビからその id にジャンプします。

---

## 2. ローカルで確認する

Hugo（extended 版）をインストールします。

- macOS: `brew install hugo`
- Windows: `winget install Hugo.Hugo.Extended`
- 参照: <https://gohugo.io/installation/>

プロジェクトのフォルダで:

```bash
hugo server
```

ブラウザで <http://localhost:1313/> を開くと、保存するたびに自動でリロードされます。

---

## 3. 内容を編集する

すべて Markdown です。`content/` の各ファイルを編集してください。

- **経歴・研究費・受賞**などの表は、Markdown のテーブル記法（`| ... | ... |`）で書きます。
- **業績**は Markdown ではなく、**1 つのデータファイル**（`data/publications.yaml`）で管理します（下記参照）。
- 日本語版の「キーワード / 学生の方へ / 企業の方へ」の囲みは、Markdown の引用（`>`）で表現しています。
- ナビ（Home / Projects / Publications / Awards / CV）は `hugo.toml` の `menu` で設定します。

### 業績（`data/` の 1 データベースから自動生成）

業績は **`content/` ではなく `data/` の YAML** を唯一のデータ源として管理します。
英語ページ・日本語ページの HTML は、この YAML から `layouts/partials/publist_byyear.html`
が自動生成します。業績データは **`data/publications.yaml` の 1 ファイルに統合**されており、
各セクションの表示範囲は `ja_only` フラグで切り替わります（二重管理は不要）。

- **`ja_only` なしのセクション**（books / journal / conference / non-archival / misc）
  … 英文業績。**英・日 両ページに表示**（ここに 1 件足すだけで両方に出ます）
- **`ja_only: true` のセクション**（books-ja / journal-ja / domestic-conference / others-ja）
  … 和文業績。**日本語ページのみ**に表示

1 件のデータはこの形です（`- cite:` と `links:` を足すだけ）:

  ```yaml
  sections:
    - id: journal                       # 見出しごとに番号が 1 から振り直されます
      title_en: "Journal Articles and Letters"
      title_ja: "学術論文・レター"        # 言語ごとの見出し
      entries:
        - cite: "著者A, 著者B, **Daisuke Iwai**, “論文タイトル,” Venue, Vol. 32, pp. 1-10, 2026."
          links:
            - { icon: "📄", label: "IEEE",     url: "https://doi.org/..." }
            - { icon: "📺", label: "Teaser",   url: "https://youtu.be/..." }
            - { icon: "🏆", label: "Best Paper Award", url: "https://..." }
  ```

  - `cite`＝書誌情報。`**Daisuke Iwai**` のように `**…**` で太字、`\*` で「＊」を出せます。
  - `links`＝各項目のリンク。`icon`（絵文字）でリンクの種類を表します:
    - `📄` 論文 / PDF / プレプリント　`📺` 動画（Teaser・Talk・YouTube）　`🌐` プロジェクトページ
    - `📢` プレスリリース　`🏆` 受賞　`🎖️` 佳作・ノミネート　`✨` メディア掲載
  - リンクが無い項目は `links: []` にします。
  - **論文ファイル・YouTube・受賞へのリンクは、その項目の `links:` に 1 行足すだけ**で反映されます。

  見た目（チップや番号）の調整は `assets/css/main.css` の「publications」セクションでできます。
  これらの YAML は Notion の業績ページから自動生成したものです。

### プロジェクト（`data/projects.yaml` の 1 データベース）

プロジェクトのカードギャラリーは **`data/projects.yaml` の 1 ファイル**を唯一の
データ源とし、**英語・日本語ページの両方**をここから生成します。タイトルだけ言語別
（`title_en` / `title_ja`）、会議名・画像・リンクは共通なので、**1 件を 1 か所に
書けば両ページに反映**され、二重管理は不要です。トップページには `{{</* projects */>}}`
と 1 行書くだけで、`assets/css/main.css` のギャラリー表示になります（最初は `show` 件、
「さらに表示」ボタンで 6 件ずつ展開・JavaScript 無効時は全件表示）。

  ```yaml
  show: 6            # 最初に見せる枚数
  items:
    - title_en: "Casper DPM: ..."
      title_ja: "Casper DPM: 手への…"
      venue: "ACM SIGGRAPH ASIA 2024"
      image: "/images/proj/casperdpm.jpg"   # 任意（static/images/ 配下）
      links:
        - { icon: "🌐", label: "Project Page", url: "https://..." }
        - { icon: "📄", label: "ACM",          url: "https://..." }
        - { icon: "📺", label: "Teaser",       url: "https://youtu.be/..." }
  ```

  - `icon` は業績と同じ絵文字（`📄`論文 `📺`動画 `🌐`プロジェクトページ `📢`報道 `📎`補足）。
  - レンダリングは `layouts/shortcodes/projects.html`、見た目は `assets/css/main.css` の
    「project gallery」セクション、ボタンの文言は `i18n/{en,ja}.toml` の
    `showMore` / `showLess` で調整できます。

### 経歴・受賞・講演・研究費・特許・学会活動・メディア（`data/cv.yaml`）

CV に相当する 7 セクション（略歴・受賞・講演・研究費・特許・学会活動・取材協力）は、
**`data/cv.yaml` の 1 ファイル**を唯一のデータ源とし、英語・日本語ページの両方を
ここから生成します（トップページには `{{</* cv */>}}` の 1 行だけ）。各エントリーは
`en` / `ja` を持てます:

- **英語ページ**＝`en` があるエントリーを表示（`en` の文で描画）
- **日本語ページ**＝すべてのエントリーを表示（`ja` があれば `ja`、無ければ `en`）

つまり **英語だけの項目（国際講演・国際的な学会活動など）は `en` を 1 回書けば
英日両ページに出ます**（日本語ページには英語のまま表示）。日本語だけの項目
（国内の受賞・講演・査読・メディア掲載など）は `ja` に書けば日本語ページのみに出ます。
略歴・研究費・特許のように英日で文面が異なるものは `en` と `ja` の両方を書きます。

  ```yaml
  sections:
    - id: talks
      type: talks                 # table / awards / talks / funding / numbered / service / list
      title_en: "Keynotes, Invited Talks & Tutorials"
      title_ja: "基調講演・招待講演・チュートリアル"
      entries:
        - year: "2026"
          en: '**Keynote**, "…," …, 21 Apr. 2026. [Link](https://…)'   # ← 国際講演は en だけでOK
        - year: "2024"
          ja: "招待講演, "…," …, 2024年10月3日."                      # ← 国内講演は ja
  ```

  - 受賞は `selected: true` が初期表示、`false` は「その他」ボタン（`details`）で展開。
  - レンダリングは `layouts/shortcodes/cv.html`、見た目は `assets/css/main.css`。
  - `data/cv.yaml` は現在の英日ページから自動生成したものです。以後はここが正になります。

---

## 4. 画像の移行（重要）

現在の画像は WordPress サーバー上にあり、**有料サーバーを解約すると消えます**。解約前に原本を保存してください。以下を実行すると、必要な画像を `static/images/` に取得できます（ファイル名は本サイトの参照と一致済み）。

```bash
cd static/images
base="https://daisukeiwai.org/wp/wp-content/uploads"
curl -L -o avatar.jpeg            "$base/2022/12/avatar.jpeg"
curl -L -o casperdpm.jpg          "$base/2024/12/casperdpm.jpg"
curl -L -o Kusuyama_ISMAR24.jpg   "$base/2024/10/Kusuyama_ISMAR24.jpg"
curl -L -o distfreedeblur.jpg     "$base/2024/06/distfreedeblur.jpg"
curl -L -o pminbright.jpg         "$base/2024/06/pminbright.jpg"
curl -L -o logo_B.png             "$base/2022/12/logo_B-300x78.png"
```

### 論文ファイル（プレプリント PDF 等）

旧サーバーの `daisukeiwai.org/share/` に置いていた論文 PDF を、本リポジトリの **`static/share/paper/`** に
移行しています。データ内のリンクも `https://daisukeiwai.org/share/…` から相対パス **`/share/…`** に張り替えて
あるため、GitHub Pages に公開後は同じ `daisukeiwai.org/share/…` の URL でこのサイト自身から配信されます。

出版社サイトで入手可能な論文の PDF（および補足資料・報道画像）は削除し、リンクも各ページから外しました。
現在 `static/share/paper/` に残しているのは、**出版社サイトに無い 15 本の PDF**（国内誌・研究会・ワークショップ等の
著者版）のみです（合計約 52MB・最大ファイル 10MB で、GitHub の 100MB 制限内）。

---

## 5. GitHub Pages に公開する

1. GitHub で新しいリポジトリを作成し、このフォルダを push します。
   ```bash
   git init
   git add .
   git commit -m "Initial site"
   git branch -M main
   git remote add origin https://github.com/<ユーザー名>/<リポジトリ名>.git
   git push -u origin main
   ```
2. リポジトリの **Settings → Pages → Build and deployment → Source** を
   **「GitHub Actions」** に設定します。
3. 以降、`main` に push するたびに `.github/workflows/hugo.yml` が自動でビルド・公開します。

---

## 6. 独自ドメイン（daisukeiwai.org）

`static/CNAME` に `daisukeiwai.org` が入っています。GitHub Pages 側と DNS を設定します。

1. **Settings → Pages → Custom domain** に `daisukeiwai.org` を入力。
2. ドメインの DNS（レジストラの管理画面）で以下を設定:
   - apex（`daisukeiwai.org`）に GitHub Pages の **A レコード**:
     `185.199.108.153`, `185.199.109.153`, `185.199.110.153`, `185.199.111.153`
     （IPv6 の AAAA も設定可）
   - `www` を使うなら **CNAME** を `<ユーザー名>.github.io` に向ける。
3. DNS 反映後、Pages 設定で **Enforce HTTPS** にチェック（証明書は自動発行）。

最新の正確な値は GitHub の公式ドキュメントで確認してください:
<https://docs.github.com/pages/configuring-a-custom-domain-for-your-github-pages-site>

> 旧サーバーの DNS からの切替タイミングに注意。新サイトの表示を確認してから、旧サーバーを解約するのが安全です。

---

## 7. デザインについて

- 配色・字体・余白はすべて `assets/css/main.css` の上部（CSS 変数）で調整できます。
- アクセント色は 1 色（コバルト）に絞り、名前の下の細い虹色ライン（投影光の分光をイメージ）だけを“印”にしています。全体は静かなグレースケールです。
- 本文・見出しとも **Open Sans**（daisukeiwai.org と同じ）、日付・番号などの細かな部分だけ IBM Plex Mono を使用しています。フォントは `layouts/partials/head.html` で読み込み、`assets/css/main.css` の `--sans` 変数で切り替えられます。
