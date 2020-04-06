---
title: "Vue.jsの基礎をお勉強した話"
date: 2020-04-06T10:30:45+09:00
draft: false
---

## Vue.jsとは
JavaScriptでUI部品を作るためのフレームワーク。noteや一休.comに使用されているNuxt.jsはこのVue.jsをベースとし，サーバーサイドレンダリングをフレームワーク化したものである。
<!--more-->
自分でモダンなwebアプリを開発するのにNuxt.jsを勉強するのが良さそうだと思ったので，その基礎としてVue.jsを軽く勉強した。
(参考にした記事)
* [Nuxt.jsとpythonで機械学習アプリ](https://qiita.com/kurakura0916/items/7a19355e8dc5d63f4631)
* その他，個人的にいつも参考にしているp1assさんが作成しているアプリのフロントはNuxt.jsで書かれている。[memoito](https://qiita.com/p1ass/items/1dfb77d5d978568686a1)，[えもじっく](https://qiita.com/p1ass/items/fa0215cee44251bf2e50)

そもそもの一人開発についてはせっかくなのでQiitaの[アドベントカレンダー](https://qiita.com/advent-calendar/2019/solo-developer)を見るのがいいだろう。 


## 勉強法 
[ドットインストール](https://dotinstall.com/lessons/basic_vuejs_v2/43901)を取り敢えず見てみた。
ドットインストール知らなかったんだけど，いい。

## Vue.jsの基本を抑える 
双方向データバインディングを行えるのが大きな特徴。データバインディングとは_データとUIを結びつける_という意味で，双方向はどちらか一方を更新する事でもう片方も更新されるという事。取り敢えず簡単に，ブラウザ上で文字入力すると表示が変わるアプリを作成してみる。 


### Vue.jsを導入 
以下のスクリプトを`index.html`のbodyタグの中に記載することで簡単に導入することができる。
```html
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
<script src="js/main.js"></script>
```
二行目は必須ではなく，連携させるjavaファイルを記載しているだけ。 


### データの作成
データ側である，UIに結びつくモデルの作成を行う。ここでは，html上で`div`タグによって定義された`id="app"`のUIに対するデータを作成している。上記でscript指定したmain.jsを以下のようにする。
```JavaScript:main.js
(function() {
    'use strict';
    var vm = new Vue({
        el: '#app',
        data: {
            name: 'TsumaR'
        }
    });
})();
```
これがVue.jsの書き方である。
* `var`とは：JavaScriptにおける変数定義法。再宣言ができるのでローカル変数の定義に便利。`let`は再代入はできるが，再宣言ができないのが大きな違い。
* `new Vue`：宣言的にデータをDOMに描画している。
* `el`とは：Vue.jsの文法。Vue()の中にあるのだから理にかなっている。mountする要素を指定している。elementの略で，Vue.jsが作用を及ぼす範囲を指定するということ。
* `data`の中身：key:値で指定している。


### データをUIへ反映
二重波括弧の中に，`data`で指定したkeyを記載するとUIに対応づけることができる。
v-modelでkeyを対応づけることで，UIの変更をデータに反映することもできる。ちなみに二重括弧の中はJavaScriptのしきをそのまま書くことができる。
```html
        <div id="app">
            <p>Hello {{name}}!</p>
            <p><input type="text" v-model="name"></p>
        </div>
```


