---
title: "NuxtIris"
date: 2020-04-07T23:49:34+09:00
draft: true
---

## Nuxt.jsを始めて使ってみる

Nuxt.jsは、ディレクトリ構成やルーティングやレンダリングなどを行ってくれるVue.jsのフレームワークです。
なんたるかは[ここ](https://qiita.com/Kentaro91011/items/406d8121775f98ddd84d)
中身については[公式ドキュメント](https://ja.nuxtjs.org/guide/installation)が日本語で非常に充実している。
今回は[この記事](https://qiita.com/kurakura0916/items/7a19355e8dc5d63f4631)を参考に，バックエンドをPythonで，フロントをNuxt.jsを用いて簡単なwebアプリを作成する手法を勉強する。
![作るもの](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F270991%2F2a1e433d-73b9-3f48-ddf4-362d0f112a8f.gif?ixlib=rb-1.2.2&auto=format&gif-q=60&q=75&w=1400&fit=max&s=b54923a77ea74266e15e82c62c3fefb0)

この[サイト](https://b1tblog.com/2019/12/24/nuxt-app1/)勉強になりそうなので，次にこれ触るといいかもと思った，というメモ。

## Nuxt.jsでフロントを作成

### プロジェクトの作成

まず，プロジェクトを設置するディレクトリに移動して，下記コマンドを実行。

```bash
npx create-nuxt-app irisNuxt
```

するとたくさんのことを聞かれたが，基本的には参考にしている記事と同様の設定にした。一部変更していたりする。レンダリングモードはSPAを指定している。

```bash
$ npx create-nuxt-app irisNuxt

create-nuxt-app v2.15.0
✨  Generating Nuxt.js project in irisNuxt
? Project name irisNuxt
? Project description My astonishing Nuxt.js project
? Author name TsumaR
? Choose programming language JavaScript
? Choose the package manager Npm
? Choose UI framework Vuetify.js
? Choose custom server framework None (Recommended)
? Choose Nuxt.js modules (Press <space> to select, <a> to toggle all, <i> to invert selection)
? Choose linting tools (Press <space> to select, <a> to toggle all, <i> to invert selection)
? Choose test framework None
? Choose rendering mode Single Page App
? Choose development tools (Press <space> to select, <a> to toggle all, <i> to invert selection)
Warning: name can no longer contain capital letters
⠋ Installing packages with npm
```

すべて返答すると`npm`が必要パッケージをインストールしてくれる。box上で操作しているとエラーが生じてうまくいかなかった。
うまく行くと，指定したアプリ名のディレクトリができる。その中に入って以下のコマンドを入力。

```bash
npm run dev
```

を実施することで，アプリケーションがローカルホスト(`http://localhost:3000`)で起動することを確認できる。スクラッチから始めることもできるらしいが，今回は`create-nuxt-app`で突き進む。

### ディレクトリ構造について

Nuxt.jsの利点の一つはディレクトリ構造を自動で理解してアプリをデプロイ？してくれることである。これはディレクトリ構造を学ぶことの重要性を意味する。
各ディレクトリの役割は公式ドキュメントの「ディレクトリ構造」[ページ](https://ja.nuxtjs.org/guide/directory-structure)参考。
今までのコマンドで作成されるディレクトリやファイルは以下の通り，

```bash
$ ls
README.md               middleware              package.json            store
assets                  node_modules            pages
components              nuxt.config.js          plugins
layouts                 package-lock.json       static
```

今回は，以下のように構成する。

```bash
irisNuxt
├── Dockerfile
├── README.md
├── assets
├── components
├── layouts <- 画面共通部分のディレクトリ
│   ├── README.md
│   └── default.vue
├── middleware
├── nuxt.config.js
├── package-lock.json
├── package.json
├── pages
├── plugins
├── static
├── store
└── yarn.lock
```

