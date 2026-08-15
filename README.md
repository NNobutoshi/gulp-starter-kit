# gulp_starter-kit

中規模の静的Webサイトを高速に開発・構築するためのGulpベースのテンプレートです。

Pug / Sass / webpack / 画像最適化 / sprite / icon font / Browsersync を組み合わせ、静的サイト制作に必要な定型作業を自動化します。

## 概要

このプロジェクトは、次のようなケースを想定しています。

- 中規模〜大規模の静的サイトを制作したい
- HTML / CSS / JS を明確に分離した構成で開発したい
- 変更した範囲だけを高速にビルドしたい
- 画像・SVG・アイコン・バンドルなどの反復作業を省略したい

## 特徴

- Pug によるHTML生成
- Sass によるCSS処理と autoprefixer 対応
- webpack によるJavaScriptバンドリング
- 画像圧縮、SVGスプライト、アイコンフォント生成
- Browsersync によるローカルプレビュー
- Git の差分を用いた差分ビルド
- 開発モード / 単発実行モード / 本番モードの3つの運用パターン

## ディレクトリ構成

```text
.
├── gulpfile.js                    # Gulp の定義とエントリーポイント
├── gulpkit/                       # Gulp タスク・設定の本体
│   ├── config/                    # 各種環境設定
│   │   ├── common.js
│   │   ├── config_preview_orig.js
│   │   └── ...
│   ├── lib/                       # 共通ユーティリティ
│   ├── tasks/                     # Gulp タスク群
│   ├── utilities/
│   ├── pug_tmp_engine/            # Excel から Pug を生成する補助ツール
│   ├── index.js                   # Gulp の全体定義
│   ├── run_task_for_diff_refs.js
│   └── ...
├── src/                           # ソースファイル群
│   ├── _bases/
│   ├── _data/
│   ├── _mixins/
│   ├── _parts/
│   ├── _templates/
│   ├── css/
│   ├── examples/
│   ├── fonts/
│   ├── img/
│   ├── js/
│   ├── index.pug
│   └── ...
├── dist/                          # ビルド成果物
├── package.json                   # npm scripts と依存関係
├── Vagrantfile                    # Vagrant 用設定
├── vagrant_config_orig.yml        # Vagrant サンプル設定
├── jsdoc.conf.json                # JSDoc 設定
├── .editorconfig
├── .stylelintrc
├── .gitignore
├── README.md
├── README_master_version.md       # master ベースの参照版
├── npm-shrinkwrap.json
├── gulpkit.js                     # 旧構成の互換ファイル（必要に応じて参照）
└── ...
```

> 現在の develop ベースの構成では `gulpkit/` を利用していますが、基本的な運用思想は master 側の README をベースに整理しています。

## 前提条件

- Node.js 22 系相当
- npm 10 系前後
- Git
- 必要に応じて Vagrant / VirtualBox

## セットアップ

依存パッケージをインストールします。

```bash
npm install
```

BrowserSync を使う場合は、ローカル用設定ファイルを作成します。

```bash
cp gulpkit/config/config_preview_orig.js gulpkit/config/config_preview.js
```

`config_preview.js` はローカル環境専用の設定ファイルとして扱い、Git 管理の対象外にするのが安全です。

## 実行コマンド

### 開発モード

```bash
npm run dev
```

以下の環境変数を含む実行です。

```text
NODE_ENV=development ENABLE_WATCH=1 ENABLE_PREVIEW=1 ENABLE_DIFF=1
```

- source map を有効化
- `Gulp.watch()` による監視を有効化
- BrowserSync によるローカルサーバーを起動
- git diff ベースの差分ビルドを有効化

### 単発ビルドモード

```bash
npm run one
```

```text
NODE_ENV=development ENABLE_WATCH=0 ENABLE_PREVIEW=0 ENABLE_DIFF=0
```

- watch しない
- server を起動しない
- 差分ビルドを無効化
- 1回だけのビルドを確認したいときに使う

### 本番モード

```bash
npm run prd
```

```text
NODE_ENV=production ENABLE_WATCH=0 ENABLE_PREVIEW=1 ENABLE_DIFF=0
```

- CSS / JS の圧縮を有効にする
- 出力先や設定を本番向けに切り替える
- サーバーを起動して確認できる

### 通常ビルド

```bash
npm run bld
```

- `ENABLE_WATCH=0` / `ENABLE_PREVIEW=0` / `ENABLE_DIFF=0`
- 事前調整が終わった通常の静的生成を実行する際に利用する

### 単体タスクの実行

上記のコマンドには、末尾にタスク名を付けて個別実行できます。

```bash
npm run dev html_pug
npm run dev css_sass
npm run prd js_webpack
```

例えば `html_pug` だけを実行すれば、Pug のコンパイルのみを確認できます。

### Pug テンプレート生成補助

```bash
npm run pug_tmp_engine
```

Excel の表形式データをもとに、Pug ファイルを自動生成します。
既に生成済みのファイルがある場合は、データ側だけを更新する構成になっています。

### JSDoc 生成

```bash
npm run jsdoc
```

API ドキュメントや内部ドキュメントを生成する場合に利用します。

## 主要なタスク

- `clean` : `dist` を整理して再生成
- `copy_to` : 画像やメディアファイルをコピー
- `html_pug` : Pug を HTML に変換
- `css_sass` : Sass をコンパイル
- `css_lint_scss` : Stylelint で CSS を検証
- `img_min` : 画像を圧縮
- `icon_font` : SVG からアイコンフォントを生成
- `img_sprite` : PNG スプライトを生成
- `img_sprite_svg` : SVG スプライトを生成
- `js_eslint` : ESLint で JS を検証
- `js_webpack` : webpack で JS をバンドル
- `preview_init` / `preview_reload` : BrowserSync の初期化と更新
- `watcher` : 変更ファイルを監視して再ビルド

## 差分ビルド

このキットでは、ほぼすべてのタスクで差分ビルドを採用しています。

Git の `diff` コマンドを使って、現在の作業差分に含まれるファイルだけをビルドの対象にできます。

- Git の差分を使って変更対象を自動判定できる
- 2つのブランチやコミットハッシュを指定して、任意の差分をビルド対象にできる
- Pug の `extends` や `include`、SCSS の partial など依存関係を解析し、必要ファイルをまとめてビルドできる
- `icon_font` や `img_sprite_svg` のように複数ファイルから1つの成果物が作られるタスクでも、関連ファイルを解析して対象として扱える
- 1ファイルから1ファイルへ変換するような処理では `gulp.lastRun()` を併用して高速化できる
- JavaScript 関連は webpack のキャッシュを利用して再ビルド時間を抑える

この仕組みにより、静的サイトの開発時に大きな差分があっても、必要な範囲だけを正しく再生成でき、開発の高速化につながります。

## グルーピング

このキットは、生成物のグルーピングにも対応しています。

- `icon_font` や `sprite` は任意のグループ単位で生成できる
- JS ファイルは、任意のファイル名をエントリーポイントとして扱い、複数のエントリーを自動検知できる
- JSON ファイルで設定することで、`splitChunks` をディレクトリ単位で出力可能
- 出力物ごとに振る舞いをまとめて管理できるため、中規模サイトの構成に向いている

例として、アイコンの元ファイル群、JS のエントリー群、共通チャンク群をそれぞれ独立したグループとして扱えます。

## 開発時の流れ

1. `src/` 配下にページやスタイル、JS、画像を配置する
2. `npm run dev` を起動する
3. ファイル変更を検知して自動的に対象タスクが再実行される
4. ブラウザでライブプレビューを確認する
5. 変更が確定したら `npm run prd` で本番向けビルドを行う

## カスタマイズ

### BrowserSync 設定

`gulpkit/config/config_preview_orig.js` をもとに、ローカル環境向けに `config_preview.js` を作成します。

```bash
cp gulpkit/config/config_preview_orig.js gulpkit/config/config_preview.js
```

このファイルには、次のような設定を置きます。

- server の host / port
- Proxy の URL
- ローカル開発用のルート設定

### Gulp 設定の調整

`gulpkit/config/common.js` や各 `config_*.js` で、出力先ディレクトリや Sass 設定、webpack 設定、差分ビルドの有効化条件などを調整できます。

必要に応じて次の項目を変更します。

- `dist` の出力先
- `NODE_ENV` ごとの設定
- sourcemap の有効化 / 無効化
- CSS の圧縮や browserslist の設定

## Vagrant

Vagrant でも開発環境を構築できます。

```bash
vagrant up
vagrant ssh
cd myproject
```

その後は通常どおり `npm run dev` などを実行します。

含まれている環境は次のとおりです。

- Ubuntu 24.04
- Node.js 22.17.0
- Nginx

ローカル環境ごとの IP や設定は `vagrant_config.yml` を使って調整します。

## サンプル

サンプルページのURL例です。

[http://sample1.nobbtossi.jp/](http://sample1.nobbtossi.jp/)

## 想定運用

- プロジェクトは Git でバージョン管理する
- 依存パッケージは開発開始時に最新化し、必要に応じて `npm shrinkwrap` 等で共有する
- `dist` も成果物として管理対象に置く前提で運用する

---

※ 本READMEは、master ベースの構成を踏まえつつ、develop ブランチの実際の構成に合わせて整理しています。
