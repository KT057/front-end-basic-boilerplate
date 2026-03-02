# SCSS 規約 (apps/web)

`apps/web/src/` 配下の `.scss` ファイルを作成・編集する際は以下のルールに従うこと。

## ファイル構成

```
apps/web/src/
├── style/
│   ├── index.scss                         # トップページ（エントリ）
│   └── _section-{name}.scss               # セクション単位の分割ファイル
├── {page-name}/
│   └── style/
│       ├── index.scss                     # ページエントリ
│       └── _section-{name}.scss           # セクション単位の分割ファイル
└── shared/
    └── common/
        └── styles/
            ├── _variable.scss             # 変数定義
            ├── _mixin.scss                # mixin・関数（rem, em, baseFontSizeForRem）
            ├── _media.scss                # メディアクエリmixin
            ├── _import.scss               # @forward でまとめてエクスポート
            ├── _reset.scss                # リセットCSS
            └── _animation.scss            # アニメーション
```

- セクション毎に `_section-{name}.scss` ファイルを分割し、`index.scss` から `@use` で読み込む
- 共有スタイルは `shared/common/styles/` に配置

## ページエントリの書き方

各ページの `index.scss` では以下を行う：

```scss
@use "../shared/common/styles/reset" as *;
@use "../shared/common/styles/import" as *;
@use "section-hero";
@use "section-about";

html {
  @include baseFontSizeForRem();
}
```

- `@use` で `reset` と `import` を読み込む（`as *`）
- `html { @include baseFontSizeForRem(); }` をページ毎に必ず設定する
- セクション別ファイルを `@use` で読み込む

## サイズ指定: rem() 関数を使う

基本的に `px` を使わず `rem()` 関数を使用する。

```scss
// Good
.hero__title {
  font-size: rem(32);
  margin-bottom: rem(16);
  padding: rem(24) rem(40);
}

// OK: 1px などの極小値は px で問題ない
.card {
  border: 1px solid #ccc;
  border-radius: 2px;
}

// Bad: px で大きなサイズ指定
.hero__title {
  font-size: 32px;
  margin-bottom: 16px;
}
```

## スマホ対応: spSizeLess()

スマホ向けのスタイルは `@include spSizeLess()` を使用する。

```scss
.hero__title {
  font-size: rem(48);

  @include spSizeLess() {
    font-size: rem(24);
  }
}
```

利用可能なメディアクエリmixin（`shared/common/styles/_media.scss`）：
- `pcSizeOver()` - PC以上
- `pcSize()` - PC幅
- `pcSizeLess()` - PC以下
- `tabletSizeOver()` - タブレット以上
- `tabletSize()` - タブレット幅
- `tabletSizeLess()` - タブレット以下
- `spSizeLess()` - スマホ以下
- `spSizeOver()` - スマホ以上

## クラス名の省略禁止

`&__element` や `&--modifier` のようにネスト内で `&` を使ったクラス名の省略は避ける。フルのクラス名で書く。

```scss
// Good: フルのクラス名で書く
.card {
  width: 100%;
}

.card__title {
  font-size: rem(24);
}

.card__button {
  padding: rem(12);
}

.card__button--primary {
  background: blue;
}

// Bad: &でクラス名を省略
.card {
  width: 100%;

  &__title {
    font-size: rem(24);
  }

  &__button {
    padding: rem(12);

    &--primary {
      background: blue;
    }
  }
}
```

## `&` の使用が許可されるケース

擬似要素・擬似クラスでの `&` 使用はOK。

```scss
// OK: 擬似要素
.card__title {
  &::before {
    content: "";
  }

  &::after {
    content: "";
  }

  &:hover {
    opacity: 0.8;
  }

  &:first-child {
    margin-top: 0;
  }
}
```

## ネスト制限: 2階層まで

セレクタのネストは2階層以上深くしない。

```scss
// Good: 2階層まで
.hero {
  @include spSizeLess() {
    padding: rem(16);
  }
}

// Bad: 深すぎるネスト
.hero {
  .hero__inner {
    .hero__title {
      font-size: rem(32);
    }
  }
}
```
