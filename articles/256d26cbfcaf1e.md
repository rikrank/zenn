---
title: "node.jsで画像の圧縮とwebp生成を自動化する"
emoji: "✨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [npm,nodejs,javascript,tech,自動化]
published: true
---

こんにちは。
Web制作で画像を扱う際に度々発生するのが、容量の圧縮やwebp化といった最適化の処理。
node.jsを使うことでそれらの作業をコマンドひとつで済ませられます。
今回は簡単に実行できるスクリプトを用意したので、メモがてらに記録します。

※サクッと使えるようにしたい方向けなので、今回はtypescriptなどは導入していません。

---------------------------------------
   
1. パッケージのインストール

下記パッケージをインストールします。（下記動作環境）

* "imagemin-gifsicle": "^7.0.0",
* "imagemin-keep-folder": "^5.3.2",
* "imagemin-mozjpeg": "9.0.0",
* "imagemin-pngquant": "^9.0.2",
* "imagemin-svgo": "9.0.0",
* "imagemin-webp": "6.0.0"

npm：
```
npm install --save-dev imagemin-keep-folder imagemin-pngquant@9.0.0 imagemin-webp@6.0.0 imagemin-svgo@9.0.0 imagemin-mozjpeg imagemin-gifsicle
```


yarn：
``` 
yarn add -D imagemin-keep-folder imagemin-pngquant@9.0.0 imagemin-webp@6.0.0 imagemin-svgo@9.0.0 imagemin-mozjpeg imagemin-gifsicle
```

<br/>

2. ディレクトリの作成

プロジェクトディレクトリの中に新たにjsファイルを作成します。（今回は、`imagemin.js`とします）
また、同階層には`src`ディレクトリの中に`img`ディレクトリを作成します。（ディレクトリはコードに合わせて適宜変えてください）
`img`ディレクトリの中には、適当な画像格納してください。

<br/>

3. スクリプトの作成

`imagemin.js`には下記コードを追記します。

```
const imagemin = require("imagemin-keep-folder");
const imageminPngquant = require("imagemin-pngquant");
const imageminWebp = require("imagemin-webp");
const imageminSvgo = require("imagemin-svgo");
const imageminMozjpeg = require("imagemin-mozjpeg");
const imageminGifsicle = require("imagemin-gifsicle");

const srcDir = "./src/img/**/*.{jpg,jpeg,png,gif,svg}";
const outDir = "./dist/img/**/*";

const convertWebp = (targetFiles) => {
  imagemin([targetFiles], {
    use: [imageminWebp({ quality: 50 })], // qualityを指定しないと稀にwebpが走らない場合があるので注意する。（{ quality: 50 }）で指定すれば大体いけそう
  });
};

imagemin([srcDir], {
  plugins: [
    imageminMozjpeg(),
    imageminPngquant(),
    imageminGifsicle(),
    imageminSvgo(),
  ],
  replaceOutputDir: (output) => {
    return output.replace(/img\//, "../dist/img/");
  },
}).then(() => {
  convertWebp(outDir);
  console.log("Images optimized!");
});
```

【個人的メモ】
* `imagemin-keep-folder`を利用することで、エントリーポイント配下（./src）のディレクトリ構造を維持した状態で、吐き出し先に最適化した画像群を送ることができます。
* 画像圧縮をしたのちwebp化をおこなう関数（`convertWebp`）をプロミスチェーンでつなげることで、画像圧縮とwebp化をひとつの処理の中で実行させています。
* `imagemin-mozjpeg`、`imagemin-svgo`、`imagemin-webp` など、そのまま最新版でパッケージをインストールをしてしまうと上記の処理が走らなくなるので、適宜バージョンを変更してください。

<br/>

4. 実行

`node imagemin.js`

▶︎ 上記コマンドで、distフォルダに圧縮された画像とweb化された画像をいい感じに格納してくれます。
npm scriptsで簡単に実行できるように`imgmin: node imagemin.js`みたいな感じでスクリプト登録しておくとよいでしょう。

ちなみに今回作成したプロジェクトは別のプロジェクトにも組み込みやすいような最小構成になっています。

# まとめ
* 単純作業は自動化してしまえ。
* node.jsは偉大。