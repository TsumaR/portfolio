---
title: "HugoとNetlifyでホームページ作成"
date: 2020-03-30T01:20:49+09:00
---

このホームページを作成した時のログ

# portfolioを作る
HUGOとnetifyで作成したportfolioにすることを目指した，ログブログ。
git submoduleの使い方に詰まったので，[この記事](https://qiita.com/sotarok/items/0d525e568a6088f6f6bb)などを参考にした。

### HUGOのインストール　
```sh
brew update
brew install HUGO
```

### HUGOの準備
まず，自分のportfolioを設置するディレクトリを作成し，`.git`を作成する。
```sh
hugo new site portfolio
cd portfolio
git init
git add .
git commit -m 'Initial commit'
```

### HUGOテーマのintroductionをsubmoduleとして導入

今回のportfolioにはHUGOの[introduction](https://themes.gohugo.io/hugo-theme-introduction/)テーマを用いた。
先ほど作成したportfolioディレクトリに上記のテーマのサブモジュールを追加する。
*(注釈)*
git submodule は、外部の git リポジトリを、自分の git リポジトリのサブディレクトリとして登録し、特定の commit を参照する仕組みです。
通常のレポジトリがブランチ単位で管理するのに対して，サブモジュールはCommitID単位で管理するというのが大きな違い。
`push`権限のため，自分のgithubにフォークしたレポジトリのサブモジュールを加えた。この方法が正しいのかは不確か。

```sh
git submodule add https://github.com/TsumaR/hugo-theme-introduction.git themes/introduction
git submodule init
git submodule update
git submodule update —remote themes/introduction
```

```sh
git merge --allow-unrelated-histories origin/master
```
で強行突破

### configファイルの編集
```sh
cd ../../ #元のportfolioディレクトリに移動
cp ./themes/introduction/exampleSite/config.toml ./config.toml
```
言語設定を日本語に，contentDirの設定を`content`に変更する。
```sh
vim config.toml
git add config.toml
git commit -m "setting config file"
```

### 基本ファイルの作成　

とりあえず，骨格となる書類を追加していく。
```sh
hugo new home/index.md
hugo new blog/_index.md
hugo new work/_index.md
```
形式はsubmoduleした`themes/introduction/exampleSite`の中にあるファイルを参考にしてvimで編集した。

### ローカルでサイトの確認
```sh
hugo server
```
上記のコマンドにより，ポート1313のローカルサーバーが立ち上がる。ブラウザから`http://localhost:1313/`にアクセスして挙動を確認する。

### netlifyでの公開
次に作成したホームページを公開する。
```sh
hugo
```
でpublicディレクトリの作成。さらに，netlifyの公式[サイト](https://gohugo.io/hosting-and-deployment/hosting-on-netlify/)よりコピーした`netlify.toml`を[これらのサイト](https://qiita.com/jrfk/items/4c6df87ca72a76e30224)を参考にして修正。

## ここまででのエラー　
1. 写真が表示されない　
2. latest post，Read more，all postが表示されない。　

### 1. 写真が表示されないについて
SSL認証による可能性があるので，独自ドメインを取得して，HTTPS設定をした。
参考にしたのはこの[サイト](https://jamstack.jp/blog/how_to_set_custom_domain/)
Netlifyではwwwありドメインを強く推奨しているらしい。

結果的にHTTPSにしただけでは表示されなかった。結果的には，おそらく間違いが大量に存在した`config.toml`を修正して表示された。

### 2. blogページなどが正しい挙動を示さない
blogの文章がほとんど全て表示されてしまい，'Read more'や'All Post'ボタンも表示されないについて
前半の全て表示されてしまうについては，Hugoが文字数カウントをスペースで行っていることにより，日本語で記事を書いた際に，大量の文章が表示されてしまうことが原因だった。そのため，`config.toml`に
```
has CJKLanguage = ture
```
を書き加えることで解決した。

一方で，'Read more'のページが表示されていなかった。これらの問題はHugoのテーマにあるMultiple language対応のための設定を読み落としていることが原因だった。themesに日本語用の設定ファイルを追加してあげることで解決した。具体的には
```
themes/introduction/layouts/i18n/ja.toml
```
ファイルを`en.toml`をコピーする形で追加することで解決した。

[参考文献1](https://blog.tomoya.dev/2019/01/hugo-with-netlify/)
[参考文献2](https://r17n.page/2019/07/24/create-hexo-blog-process/)


# 注意　
submoduleを理解していないせいで
```sh
git merge --allow-unrelated-histories origin/master
```
をする羽目になった。
そもそもの基本で，GitHub側でREADMEを作ったならまずそれをpullしないと時系列がずれてしまう。
submoduleに関しては，submoduleディレクトリで`commit`，`push`したのちに親ディレクトリでその`commit`があったということを`commit`しないといけない。

