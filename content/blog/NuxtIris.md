---
title: "Nuxt.js１日目。バックエンドPythonで"
date: 2020-04-07T23:49:34+09:00
---

## Nuxt.jsを始めて使ってみる

Nuxt.jsは、ディレクトリ構成やルーティングやレンダリングなどを行ってくれるVue.jsのフレームワークです。
<!--more-->
なんたるかは[ここ](https://qiita.com/Kentaro91011/items/406d8121775f98ddd84d)
中身については[公式ドキュメント](https://ja.nuxtjs.org/guide/installation)が日本語で非常に充実している。
今回は[この記事](https://qiita.com/kurakura0916/items/7a19355e8dc5d63f4631)を参考に，バックエンドをPythonで，フロントをNuxt.jsを用いて簡単なwebアプリを作成する手法を勉強する。
![作るもの](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F270991%2F2a1e433d-73b9-3f48-ddf4-362d0f112a8f.gif?ixlib=rb-1.2.2&auto=format&gif-q=60&q=75&w=1400&fit=max&s=b54923a77ea74266e15e82c62c3fefb0)
スクリプトに関してはほとんど上の記事ままなので，参考になったら上の記事にLGTMしにいってください。

この[サイト](https://b1tblog.com/2019/12/24/nuxt-app1/)勉強になりそうなので，次にこれ触るといいかもと思った，というメモ。

## Nuxt.jsでフロントを作成

### 1.プロジェクトの作成

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

### 2.ディレクトリ構造について

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

* assets：StylesやSASS，Image，Fontのようなコンパイルされていないファイル
* components：Vue.jsのコンポーネントファイル。名前付きの再利用可能なVueインスタンスのこと。
* layouts：サイドバーなど，ページの外観を変更するために使用される。
* middleware：ページやページグループをレンダリングするよりも前に実行されるカスタム関数を定義できる。
* pages：アプリケーションのビューおよび，ルーティングファイルをいれる。このディレクトリないすべての`.vue`ファイルがアプリとして含まれる。

今回は，以下のように構成する([元記事](https://qiita.com/kurakura0916/items/7a19355e8dc5d63f4631)まま。

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

### 3.ヘッダーとフッターの追加

`layouts`ディレクトリ内にある，`default.vue`を変更してヘッダーとフッターを追加する。ここから先は自分の備忘録でもあるので，細かい内容に関しては元記事を見てください。

```vue
<template>
  <v-app>
    <v-toolbar
      app
      color="primary"
      class="white--text"
    >
      <v-toolbar-side-icon class="white--text"/>
      <v-toolbar-title>Title</v-toolbar-title>
    </v-toolbar>

    <v-content>
      <v-container>
      <nuxt/>
      </v-container>
    </v-content>

    <v-footer
      app
    >
      <span>Footer</span>
    </v-footer>
  </v-app>
</template>
```

### 3.入力受付フォームの作成

`components`ディレクトリに画面出力のコンテンツを作成する。1つのvueインスタンスを作成することで，名前に応じて複数回利用されるよう設計できる。逆に言えばコンテンツをパーツ毎にファイル分けして作成できる。
ここで，ここまで来て自分の知識が欠如しており，コンポーネントを理解していなかったので，ここに捕捉する。(完全に自分のためである。)

#### 3.1.(補足)コンポーネント

##### インスタンス

前日までの勉強において，Vue.jsではJavaScriptの中でインスタンスの生成を宣言して利用していた。細かくいうと，`new Vue`でインスタンスを生成，その中のel要素で，htmlの，どの部分を対象としてVueを使用するかを定義していた。html1箇所に対する操作としてvueインスタンスを定義していた。

```html:html
<div id='app'>
</div>
```

```JavaScript:js
new Vue({
  el: '#app'
})
```

上記の例である。しかし，今回Nuxt.jsでは，JavaScriptを直接記載することなく，`.vue`ファイルを生成して，操作ごとを記載していた。操作ごとの定義，`.vue`ファイルとは，について理解がしっくりこなかったため調べた。

##### コンポーネント

上でも記載したように，コンポーネントとは再利用可能なVueインスタンス要素である。複数のコンポーネントを作成して，それを部品のように組み立てることで1つのWebページを作ることができる。先ほどのインスタンスの生成ほうでは，html要素を指定してインスタンスを指定していたため，このような操作ができなかった。

##### vueファイル

コンポーネント自体は通常のインスタンス生成の時と同じようにJavaScriptで定義できる。一方で，`.vue`ファイル一つ一つで一つのコンポーネントを構成することができる。
詳しくは公式ドキュメントかこの[記事](https://qiita.com/kiyokiyo_kzsby/items/980c1dc45e00d2d3cbb4)を参考にするとよい。
vueファイルの中身は主に，

* templateタグ：コンポーネントのhtml要素を埋め込む
* scriptタグ：JavaScriptを記載する
* styleタグ：cssを記載する

からなる。Nuxt.jsでは基本的にVueファイルで全てを？管理する。

#### 3.2.実際のスクリプト

```vue
<template>
  <form>
    <div>
      <v-text-field
        v-model="$store.state.sepalLength"
        label="Sepal Length"
        ></v-text-field>
      <v-text-field
        v-model="$store.state.sepalWidth"
        label="Sepal Width"
        ></v-text-field>
      <v-text-field
        v-model="$store.state.petalLength"
        label="Petal Length"
        ></v-text-field>
      <v-text-field
        v-model="$store.state.petalLength"
        label="Petal Length"
        ></v-text-field>
    </div>
  </form>
</template>

<script>
export default {
  name: "FormText"
}
</script>

<style scoped>
</style>
```

componentのなかのFormText.vueにvueのコンポーネントを記載。
`template`，`form`内に，2つの<div>`で

```vue
<v-text-field
  v-model="$store.state.sepalLength"
  label="Sepal Length"
  ></v-text-field>
```

が４つからなるtext入力フォームと，

```vue
<div class="predict">
  //クリックした時にsbmit/clearメソッドの実行
      <v-btn @click="submit">submit</v-btn>
      <v-btn @click="clear">clear</v-btn>
      <h1 v-if="$store.state.predictLabel">Predicted label is {{ $store.state.predictedLabel }}</h1>
    </div>
```

submitとclearボタンの機能はscriptの中に`sbumit`と`clear`という名前のメソッドで追加する。

うーん。なぜかうまくいかない。
上記のQiitaのまま動かすと，headerが画面の半分以上を占める気持ちわるいUIになってしまう。
ということで書籍の勉強に移る。

### 1.デフォルト画面の作成

ヘッダーとフッターの作成をリベンジ
`layouts/default.vue`を書き換える。
一旦全てを消去し，

```vue
<template>
  <v-app>
    <!-- この中に HTML を記載する -->
  </v-app>
</template>
  
<script lang="ts">
  // この中に TypeScript を記載する
  import Vue from ’Vue’
  import Component from ’nuxt-class-component’

  @Component
  export default class Layout extends Vue {}
</script>

<style scoped>
/* この中に CSS を記載して style を定義 */
</style>
```

に書き換える。と思ったが，うまくいかないし勉強にならないので調べながら一つ一つ書いた。
[【vue.js】Vuefityをマスターする(1)](https://reffect.co.jp/vue/vuetify-for-beginner)という記事が参考になる。1からvuetifyを使って作成する手順が書いてある。
タグの命名規則は下の画像の通り。Vuetifyのタグにはすべて”v-“が先頭につく。
![命名](https://reffect.co.jp/wp-content/uploads/2019/09/vue_dashbord-layout.png)

```vue

<template>
  <v-app>
    <v-toolbar
      app
      color="primary"npm
      class="white--text"
    >
      <v-toolbar-side-icon class="white--text"/>
      <v-toolbar-title v-text="title"/>
    </v-toolbar>

    <v-content>
      <v-container>
        <nuxt/>
      </v-container>
    </v-content>

    <v-footer
      app
    >
      <span>&copy; TsumaR 2020</span>
    </v-footer>
  </v-app>

</template>
  
<script>
  // この中に TypeScript を記載する
  
</script>

<style scoped>
/* この中に CSS を記載して style を定義 */
</style>
```

mountedはコンポーネントの DOM が生成されたあとに走る処理

Vuex アプリケーションの中心にあるものはストアです。"ストア" は、基本的にアプリケーションの 状態（state） を保持するコンテナです。単純なグローバルオブジェクトとの違いが 2つあります。

Vuex ストアはリアクティブです。Vue コンポーネントがストアから状態を取り出すとき、もしストアの状態が変化したら、ストアはリアクティブかつ効率的に更新を行います。

ストアの状態を直接変更することはできません。明示的にミューテーションをコミットすることによってのみ、ストアの状態を変更します。これによって、全ての状態の変更について追跡可能な記録を残すことが保証され、ツールでのアプリケーションの動作の理解を助けます。

#### Vuex の状態を Vue コンポーネントに入れる

[vueの算出プロパティ](https://jp.vuejs.org/v2/guide/computed.html)を使うのが簡単

```JavaScript
// Counter コンポーネントをつくってみましょう
const Counter = {
  template: `<div>{{ count }}</div>`,
  computed: {
    count () {
      // storeのstateのcountを返している
      return store.state.count
    }
  }
}
```

ちなみに，この際のstoreは以下の通り

```JavaScript
const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    }
  }
})
```

 Vuex のストアの状態を変更できる唯一の方法は、ミューテーションをコミットすることです。

 ではNuxt.jsのsotreディレクトリでstore管理する方法。
 [公式サイト](https://ja.nuxtjs.org/guide/vuex-store)を参考にした。

 まずはじめに、store/index.js 内でステートを関数として、ミューテーション、アクションをオブジェクトとしてエクスポートします

 

[getter](https://vuex.vuejs.org/ja/guide/getters.html)の話は戸惑った。

#### actions

アクションはミューテーションと似ていますが、下記の点で異なります:

アクションは、状態を変更するのではなく、ミューテーションをコミットします。
アクションは任意の非同期処理を含むことができます。
API にアクセスして、なにか値を取得したい! という場合は Actions を使います。 Actions は非同期前提で、Actions から Mutations を呼んで、state を更新します。
今回、「アルコール濃度」などのワインの属性値は API 側に持っているためこれを Actions で呼び出して取得しています。

