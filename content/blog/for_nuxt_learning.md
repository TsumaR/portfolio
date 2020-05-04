---
title: "Nuxt.jsの学習のために自分がしたこと"
date: 2020-05-04T18:18:06+09:00
draft: false
---


## 記事の概要

とりあえず，1週間くらいで簡単なアプリなら作れるようになるために。
一旦，実装力メイン。

[Nuxt.jsに飛びつく前に~Nuxt.jsを習得するための前提技術と、その勉強方法の紹介~](https://qiita.com/kyohei_ai/items/763b0c228a8451c68865)この記事を見ながら勉強を進めました。
が，自分はあくまで飛びつきながら学習している途中の段階です。。
自分のように，事前準備に時間がかかりすぎて途中でやる気が萎えてしまう人に向けて，ひとまず概要を理解し，飛びつき，知識を吸収しながらよじ登る学習方法について，よじ登りながらまとめている記事。

## Nuxt.jsについて

> Nuxtは、モダンな web アプリケーションを作成する Vue.js に基づいたプログレッシブフレームワークです。Vue.js 公式ライブラリ（vue、vue-router や vuex）および強力な開発ツール（webpack、Babel や PostCSS）に基づいています。 Nuxt の目標は、優れた開発者エクスペリエンスを念頭に置き、Web 開発を強力かつ高性能にすることです。([公式サイト](https://ja.nuxtjs.org/guide/))

Nuxt.jsは何よりも[公式サイト](https://ja.nuxtjs.org/guide/)がわかりやすいので，まず公式サイトを確認すると良いです。
ここにもあるように，Nuxt.jsの中身はVue.jsを基本としているので，Nuxt.jsの学習は，(Vue.jsを知らない場合)主にVue.jsの勉強ということになります。

[『Vue.js&Nuxt.js超入門』](https://www.amazon.co.jp/gp/product/B07X6F1C2P/ref=as_li_tl?ie=UTF8&camp=247&creative=1211&creativeASIN=B07X6F1C2P&linkCode=as2&tag=newt0-22&linkId=182ba8193eba8bdf672a078fe7f871c8)がこの記事の目的には最初に読む本としていいらしい。僕は読んでないです。

## 自分の勉強履歴

* JavaScriptはドットインストールで復習し，[しまぶーさんのyoutube](https://www.youtube.com/channel/UCti6dG0zSAetLGGYcgNML4Q)を見た
* Vue.jsは[この記事](https://www.tsumar.com/blog/vuejsdotinstall/)にまとめた内容で進めた
* Nuxt.jsは[公式サイト](https://ja.nuxtjs.org/guide/)のディレクトリ構造までとVuex ストアを読んだ
* firestoreに関してはqiitaで記事を真似ただけ

Vue.jsの概念自体を理解することが重要。
Nuxt.jsは,Vuexを理解しておくことが非常に重要。それ以外は，公式ドキュメントを読んだり，公式の動画を見た後はqiitaやboothにある記事を基にアプリのコピーを作成した。
非常に影響を受けたのはこの記事[『新型コロナで休校になって暇になった高校生がミニサービス「Yobikake」を3日間で立ち上げた話』](https://qiita.com/nztm/items/2de55be97ac1de9a435b)
高校生が3日でアプリを作っているのに自分は何をやっているんだとやる気がでた。
このアプリは単純なものだったので，Nuxt.jsの構造を学ぶ参考にもした([GitHub](https://github.com/nztm/Yobikake))
自分は[Vuetify](https://vuetifyjs.com/ja/)を使ったので，上記のプログラムをそのままコピーはしなかったが，基本の理解には非常に良いスクリプトだと思う。
Vuetifyの勉強は[公式ページ](https://vuetifyjs.com/ja/components/cards/)を読みまくった。