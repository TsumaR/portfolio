---
title: "Vue.jsの基礎を"1日無料で”お勉強した話"
date: 2020-04-06T10:30:45+09:00
---

### Vue.jsとは
JavaScriptでUI部品を作るためのフレームワーク。noteや一休.comに使用されているNuxt.jsはこのVue.jsをベースとし，サーバーサイドレンダリングをフレームワーク化したものである。
<!--more-->
自分でモダンなwebアプリを開発するのにNuxt.jsを勉強するのが良さそうだと思ったので，その基礎としてVue.jsを軽く勉強した。
(参考にした記事)
* [Nuxt.jsとpythonで機械学習アプリ](https://qiita.com/kurakura0916/items/7a19355e8dc5d63f4631)
* その他，個人的にいつも参考にしているp1assさんが作成しているアプリのフロントはNuxt.jsで書かれている。[memoito](https://qiita.com/p1ass/items/1dfb77d5d978568686a1)，[えもじっく](https://qiita.com/p1ass/items/fa0215cee44251bf2e50)

そもそもの一人開発についてはせっかくなのでQiitaの[アドベントカレンダー](https://qiita.com/advent-calendar/2019/solo-developer)を見るのがいいだろう。 

ちなみに，フロントの知識皆無の人間が書いている。JavaScriptも昔，少しだけ勉強しようとしたが実際にコーディングした経験はない。htmlの基礎の基礎タグについての軽い知識はある。

***

### Vue.jsの基本を理解する
[ドットインストール](https://dotinstall.com/lessons/basic_vuejs_v2/43901)を取り敢えず見てみた。
ドットインストール知らなかったんだけど，いい。 
Vue.jsは双方向データバインディングを行えるのが大きな特徴。データバインディングとは**データとUIを結びつける**という意味で，双方向はどちらか一方を更新する事でもう片方も更新されるという事。取り敢えず簡単に，ブラウザ上で文字入力すると表示が変わるアプリを作成してみる。 

***
#### 0.ブラウザ入力により表示テキスト変更アプリ
#### 1.Vue.jsを導入 
以下のスクリプトを`index.html`のbodyタグの中に記載することで簡単に導入することができる。
```html
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
<script src="js/main.js"></script>
```
二行目は必須ではなく，連携させるjavaファイルを記載しているだけ。 


#### 2.データの作成
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


#### 3.データをUIへ反映
二重波括弧の中に，`data`で指定したkeyを記載するとUIに対応づけることができる。
v-modelでkeyを対応づけることで，UIの変更をデータに反映することもできる。ちなみに二重括弧の中はJavaScriptの式をそのまま書くことができる。
```html
        <div id="app">
            <p>Hello {{name}}!</p>
            <p><input type="text" v-model="name"></p>
        </div>
```
以上でブラウザ上で文字入力すると表示が変わるアプリを作成することができた。htmlを変更することなく，Vueが完全に制御している。


*** 

### 基礎から学ぶVue.js
ここから先，todoリストを作成しているが有料。課金してもいいのだが，本当の基礎の部分がわかったらあとは記事を探して勉強するので十分な気がするので記事を参考に勉強した。
実際は基礎から学ぶVue.jsの[サポートサイト](https://cr-vue.mio3io.com/)を利用した。このサイトには，To Doリストを作りながら学習しよう！というページや，Netlifyで自動デプロイするまでの記事まであり大変勉強になる。ただし，本章の内容に関しては当然細かい説明はない(本を買って読むべき)。今回はチュートリアルの[ToDoリストを作りながら学習しよう"](https://cr-vue.mio3io.com/tutorials/todo.html#step1-%E3%82%A4%E3%83%B3%E3%82%B9%E3%82%BF%E3%83%B3%E3%82%B9%E3%81%AE%E4%BD%9C%E6%88%90)を動かす。
作成するToDoリストは下のもの
![ToDoimage](https://cr-vue.mio3io.com/images/todo/todo-image.png)

#### ログ 
細かい操作は直接元ページを見た方がいいので記載しない。基本的にドットインストールの無料箇所を見ただけでだいぶ理解が捗った。

##### Vueのmethod
ドットインストールではVueにdataしか宣言しなかったが，`methods`を宣言することができる。
```
const app = new Vue({
  el: '#app',
  data: {
    // 使用するデータ
  },
  methods: {
    // 使用するメソッド
  }
})
``` 

##### データの構想
必要になるデータは
* ToDo のリストデータ
  + 要素の固有ID
  + コメント
  + 今の状態
* 作業中・完了・すべて などオプションラベルで使用する名称リスト
* 現在絞り込みしている作業状態
リストデータをテーブル形式(`table`タグを使用)で記載する。

##### フォーム入力 
ドットインストールでは，フォーム入力は以下のようにし，フォーム入力(UI)とデータを参照した
```html
<p><input type="text" v-model="name"></p>
```
一方で，入力データを画面に表示させず，常にデータとして持っている必要がない場合は`ref`属性を使う。`ref`属性を使って参照するための名前をタグ付けしておくと，その`DOM`に直接アクセスできる。今回の例の場合，
```html
<input type="text" ref="comment">
```
メソッド内でこれを参照するためには
```
this.$refs.comment.value
```
のように，`this`をつける必要がある。ちなみに`DOM`とは「Document Object Model」の略で，ファイルの特定の部分に目印を付けて「この部分」に「こういう事をしたい」という処理を可能にするための取り決めのこと。

少し冗長になるが，
```html
<h2>新しい作業の追加</h2>
    <form class="add-form" v-on:submit.prevent="doAdd">
        コメント <input type="text" ref="comment">
        <button type="submit">追加</button>
    </form>
```
で作業追加の項目を作成できる。説明したように`ref`で`comment`を参照するようになっている。一方で，ここまで出てきたことのなかった`v-on`というものが現れた。ここでは，下で指定されたフォームのサブミットが行われると，それをハンドリングして`doAdd`メソッドが呼び出されるようになっている。

##### Vueのmethods
`v-on`によって呼び出されたdoAddメソッドはVueを用いて，自分で定義するものである。ToDoアプリでは，フォームの入力値を取得して新しいToDoの追加処理を行うようにする。`method`オプションに記載する。
```JavaScript
new Vue({
  // ...
  methods: {
    // ToDo 追加の処理
    doAdd: function(event, value) {
      // ref で名前を付けておいた要素を参照
      var comment = this.$refs.comment
      // 入力がなければ何もしないで return
      if (!comment.value.length) {
        return
      }
      // { 新しいID, コメント, 作業状態 }
      // というオブジェクトを現在の todos リストへ push
      // 作業状態「state」はデフォルト「作業中=0」で作成
      this.todos.push({
        id: todoStorage.uid++,
        comment: comment.value,
        state: 0
      })
      // フォーム要素を空にする
      comment.value = ''
    }
  }
})
```
結果としてはこうなる。$refs以外は普通のJavaScriptの構文。通常の`push`メソッドを利用するだけで，Vueのデータ`todos`にリストの要素として追加される。
`todos`に追加されると，htmlではbodyを下のように記載しているため，
```html
                <tbody>
                    <tr v-for="item in todos" v-bind:key="item.id">
                        <!-- 要素の情報 -->
                        <th>{{ item.id }}</th>
                        <th>{{ item.comment }}</th>
                        <td class="state">
                            <button>{{ item.state }}</button>
                        </td>
                        <td class="button">
                            <button>消去</button>
                        </td>
                    </tr>
                </tbody>
```
htmlに要素が追加された形で表示することができるのである。Vue.jsすごい。
ToDoのリストも，フォームも`<div id="app">`内にあるからできることであることをちゃんと理解しておくこと。


##### ストレージへの保存(watch)
上記で表示上，新たなToDoを追加できるようになったが，このままだとJavaScriptがtodosというリストの要素として記憶しているだけになってしまう。そのため，書き換えが起こった場合にストレージに保存する機能を追加する必要がある。ここに関しても当然のようにVue.jsが行ってくれる。
ここで，操作ごとにストレージ保存のスクリプトを書かずとも，`Vue`のデータが書き換わるごとにストレージに自動で保存してくれる機能を追加することができる。`watch`機能である。
```JavaScript
watch: {
  監視するデータ: function(newVal, oldVal) {
    // 変化した時に行いたい処理
  }
}
```
