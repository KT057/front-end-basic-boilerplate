# Pug テンプレート規約 (apps/web)

`apps/web/src/` 配下の `.pug` ファイルを作成・編集する際は以下のルールに従うこと。

## ディレクトリ構成

```
apps/web/src/
├── template/
│   └── index.pug                          # トップページ
├── {page-name}/                           # 各ページ（例: about/, contact/ 等）
│   ├── template/
│   │   ├── index.pug                      # ページエントリ（レイアウト継承 + セクション include）
│   │   └── _section-{name}.pug            # セクション単位の分割ファイル
│   ├── script/
│   │   └── index.ts
│   ├── style/
│   │   └── index.scss
│   └── images/
├── script/
│   └── index.ts
├── style/
│   └── index.scss
└── shared/
    ├── templates/
    │   ├── layout/
    │   │   └── _default.pug               # 共通レイアウト（extends で継承）
    │   └── module/
    │       └── _{module-name}.pug          # 共通コンポーネント（header, footer 等）
    ├── common/
    │   ├── scripts/utils/
    │   └── styles/
    ├── scripts/
    └── styles/
```

- トップページは `src/` 直下の `template/`, `script/`, `style/` を使用
- 各ページは `src/{page-name}/` 配下に `template/`, `script/`, `style/`, `images/` を持つ
- 共有ファイルはファイル名の先頭に `_` を付ける

## セクション分割

ページの `index.pug` を1つの大きなファイルにしない。セクション単位でファイルを分割し `include` で読み込む。

```pug
//- index.pug
extends /shared/templates/layout/_default

block vars
  -
    var title = 'ページタイトル'
    var description = ''
    var keywords = ''
    var url = ''

block main
  include _section-hero
  include _section-about
  include _section-feature
```

```pug
//- _section-hero.pug
section.hero
  .hero__inner
    h1.hero__title メインタイトル
    p.hero__text(class='js-hero-text') テキスト
```

- セクションファイルは `_section-{name}.pug` の命名規則
- 各セクションファイルはセクションのルート要素から記述する（`block` は書かない）

## クラス命名: BEM

クラス名は BEM (Block Element Modifier) で記述する。

```pug
//- Block
.card
  //- Element
  .card__header
    h2.card__title タイトル
  .card__body
    p.card__text テキスト
  //- Modifier
  .card__button.card__button--primary ボタン
```

- Block: コンポーネント名（例: `card`, `hero`, `nav`）
- Element: `__` で接続（例: `card__title`）
- Modifier: `--` で接続（例: `card__button--primary`）
- ネストは2階層まで（`block__element` まで。`block__element__sub` は避ける）

## JS 用クラスの分離

スタイル用のクラスと JavaScript で使うクラスは分離する。JS で参照する要素には `.js-*` プレフィックスのクラスを付与する。

```pug
//- Good: スタイルとJS用クラスを分離
button.button.button--primary(class='js-submit-button') 送信

//- Bad: スタイル用クラスをJSのセレクタに使う
button.button.button--primary 送信
//- JS側で .button--primary を querySelector するのはNG
```

- `.js-*` クラスにはスタイルを当てない
- スタイル用の BEM クラスを JS のセレクタとして使わない
- 1つの要素に複数の `.js-*` クラスを付けてもよい

## 共通テンプレート

複数ページで使い回すコンポーネントは `apps/web/src/shared/templates/module/` に配置する。

- ファイル名は `_{module-name}.pug`（先頭 `_` 付き）
- レイアウトから `include` で読み込む（例: `include ../module/_header`）
- ページ固有のコンポーネントは各ページの `template/` 配下に置く

## レイアウト継承

各ページの `index.pug` は共通レイアウトを `extends` で継承する。

```pug
extends /shared/templates/layout/_default
```

- `block vars` でページ固有の変数（title, description, keywords, url）を定義
- `block main` でページのメインコンテンツを記述
- レイアウトファイルのパスは `/shared/templates/layout/_default` （basedir が `src/` に設定済み）

## レスポンシブ画像: `picture` タグで PC/SP を出し分ける

画像は `picture` + `source` タグを使い、PC用とSP用を出し分ける。ブレークポイントは `shared/common/styles/_variable.scss` の `$SP_SIZE`（現在 480px）を基準とする。

```pug
//- Good: picture タグで PC/SP 画像を出し分け
picture.hero__picture
  source(media='(max-width: 480px)', srcset='./images/hero/hero-img-01-sp.png')
  img.hero__image(src='./images/hero/hero-img-01-pc.png', alt='ヒーローイメージ')

//- Bad: img タグのみ（PC/SP で同じ画像）
img.hero__image(src='./images/hero/hero-img-01.png', alt='ヒーローイメージ')
```

- `source` の `media` 属性で SP 判定 — ブレークポイントは `_variable.scss` の `$SP_SIZE` を確認して合わせる
- `img` タグが PC 用（デフォルト）のフォールバック
- `img` には必ず `alt` 属性を付ける（pug-lint ルール準拠）
- `media` 属性の値は SCSS 変数を直接参照できないため、`_variable.scss` の `$SP_SIZE` の値を確認してハードコードする

### 画像ファイル命名規則

- PC用: `{name}-pc.{ext}`（例: `concept-img-01-pc.png`）
- SP用: `{name}-sp.{ext}`（例: `concept-img-01-sp.png`）

### Figma MCP 利用時のガイドライン

Figma から画像を取得する際、PC フレームと SP フレームそれぞれから画像を書き出す。

- PC フレーム: デスクトップサイズのノードからスクリーンショット/アセットを取得
- SP フレーム: モバイルサイズのノードからスクリーンショット/アセットを取得
- 同一画像でもサイズ・トリミングが異なる場合があるため、必ず両方のフレームを確認する

## pug-lint 準拠

`apps/web/.pug-lint.json` の設定に従うこと。主要ルール:

- インデント: スペース2つ
- 属性のクォート: シングルクォート（`'`）
- 属性の区切り: `, `（カンマ + スペース）
- `div` タグは省略（`.class-name` と書く、`div.class-name` としない）
- `img` には `alt` 属性必須
- `input` には `name`, `type` 属性必須
- `html` には `lang` 属性必須
- クラスリテラルは ID リテラルより前に書く
- HTML テキストの直書き禁止（Pug のテキスト記法を使う）
