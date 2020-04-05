---
title: "Java_dotinstall"
date: 2020-04-05T22:30:05+09:00
---

### 元のhtmlファイルを作成する
```html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="utf-8">
    <title>JavaScript Practice</title>
    <style>
        .box {
            width: 100px;
            height: 100px;
            background-color: skyblue;
            cursor: pointer;
        }
    </style>
</head>
<body>
    <div class="box"></div>
</body>
</html>
```

このコードで，単純に青いboxが1つ表示される。styleでcursorを設定しているおかげで，boxにカーソルを持っていくと一応指差し状態にはなるが，何も起こらない。

### クリックイベントを設定する。
boxをクリックした際に動作を追加する。今回はscriptタグに追加する。scriptタグは，html部分をほとんど読み込んでから実行して欲しいので，`</body>`直前に記入するのが一般的。
javascriptが要素を参考にするときはidをつけておくといい，参照が楽。
今回の場合は，クリックしたらboxの色がpinkになるという機能を追加した。実際にはbodyの中身を以下のように変更しただけで実行できる。
```html
<body>
    <div class="box" id="target"></div>
    <script>
        <!--ブラウザが厳密なエラーチェックを行う-->
        'use strict'; 
        document.getElementById('target').addEventListener('click', () => {
            document.getElementById('target').style.background = 'pink';
        })
    </script>
</body>
```
`対象要素.addEventListener()`クリックされたかどうかなどのイベントを受け取る。
documentとは文章全体，getElementByIdでidが指定されたものだけを取得。
今回の場合クリックされたら，それ以降の内容が実行される。実際はstyleのbackgroundの色を変更する。
{}の中身は関数にあたるので，色々な処理を記載することができ，同時にboxを丸くすることなどもできる。
この関数の記載方法は_アロー関数_といい，JavaScriptに特別で，pythonなどでは見慣れないので詳しく説明を記載する。

#### アロー関数 
[この記事](https://qiita.com/may88seiji/items/4a49c7c78b55d75d693b)を参考にするといい。アロー関数は名前の通り，`=>`を使用して関数リテラルを記述すること。関数リテラルとはソースコードに直接べた書きされた関数のこと。名前を持たない無名関数，匿名関数と言われる。ただし，関数リテラルを変数に入れこんで利用することはできるし，多い？

### 回転してピンクの円に変換 
先ほどは，styleを一つずつ記載していたが，cssのクラスとしてスタイルを指定した方がすっきりする。
ここでは，circleクラスの追加を行う。
さらにここではアニメーションも追加する，cssアニメーションの知識も必要。cssクラスのところにtransitionやtransformプロパティを追加することで実行している。
```html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="utf-8">
    <title>JavaScript Practice</title>
    <style>
        .box {
            width: 100px;
            height: 100px;
            background-color: skyblue;
            cursor: pointer;
            transition: 0.8s;
        }
        .circle {
            background: pink;
            border-radius: 50%;
            transform: rotate(360deg);
        }
    </style>
</head>
<body>
    <div class="box" id="target"></div>
    <script>
        <!--ブラウザが厳密なエラーチェックを行う-->
        'use strict';
        document.getElementById('target').addEventListener('click', () => {
            // document.getElementById('target').style.background = 'pink';
            document.getElementById('target').classList.add('circle');
        })
    </script>
</body>
</html>
```

addではなく，`toggle`を使うことで，もう一度クリックすると元に戻る操作にすることができる。
`document.getElementById('target').classList.add('circle');`を
`document.getElementById('target').classList.toggle('circle');`とする。

### 定数を用いる
上のスクリプトでは，documentから何度もtarget idを取得していて冗長である上に，変更に弱い。`const`を用いて定数を設定することでより良いプログラムにする。

```html
<body>
    <div class="box" id="target"></div>
    <script>
        'use strict';

        const target = document.getElementById('target');

        target.addEventListener('click', () => {
            target.classList.toggle('circle');
        })
    </script>
</body>
```

### JavaScriptでdivを作成する 
そもそもdivタグとは，囲った部分をブロック要素としてまとめて扱うことができるようになるタグのこと。pやh1のようにそれ単体で意味を持つタグと異なる，少し特殊なタグである。まとめて扱うことができるので，レイアウトの指定に便利。 
ここでは，forを回して100個のdivを追加している。

```html
<body>

    <script>
        'use strict';

        for (let i = 0; i < 100; i++) {
            const div = document.createElement('div');
            div.classList.add('box');
            div.textContent = i;

            div.addEventListener('click', () => {
                div.classList.toggle('circle');
            });

            document.body.appendChild(div);
        }
    </script>
</body>
```

### ゲーム化
5つのうちの1つだけがランダムで当たりになっているというゲームの作成 
```html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="utf-8">
    <title>JavaScript Practice</title>
    <style>
        body {
            display: flex;
            flex-wrap: wrap;
        }
        .box {
            width: 100px;
            height: 100px;
            background-color: skyblue;
            cursor: pointer;
            transition: 0.8s;
            margin: 0 8px 8px 0;
            text-align: center;
            line-height: 100px;
        }
        .win {
            background: pink;
            border-radius: 50%;
            transform: rotate(360deg);
        }
        .lose {
            transform: scale(0.9);
        }
    </style>
</head>
<body>

    <script>
        'use strict';

        const num = 5;
        // Math.random()で0以上1未満のランダムな数字を生成
        const winner = Math.floor(Math.random() * num); //0 - 4

        for (let i = 0; i < num; i++) {
            const div = document.createElement('div');
            div.classList.add('box');

            div.addEventListener('click', () => {
                if (i === winner) {
                    div.textContent = 'Win!';
                    div.classList.add('win');
                } else {
                    div.textContent = 'Lose....';
                    div.classList.add('lose');
                }
            });

            document.body.appendChild(div);
        }
    </script>
</body>
</html>
```
