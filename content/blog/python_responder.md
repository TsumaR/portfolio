---
title: "Python_responder"
date: 2020-04-15T23:58:27+09:00
draft: false
---

## メモ

### 勉強しないといけないPythonの知識

`async`や，`@`，


## responderを使う

作者が`pipenv`の作者であることもあり，基本的に`pipenv`を使用することを推奨されているよう。
今回は，condaを消去する勇気がなかったのでcondaで作った環境上にpipenvをインストールして作業している。
仮想環境ないではpipしか使わないよう統一化して気をつける必要がある，と思う。

```sh
conda create -n responder python=3.6
conda activate responder
pip install pyenv
mkdir project_name && cd project_name
pipenv install
```

pipenv自体初めてなので吐き出されるログを記載

```
✔ Successfully created virtual environment!
Virtualenv location: /Users/wagatsuma/.local/share/virtualenvs/responder_test-rl88sVq2
Creating a Pipfile for this project…
Pipfile.lock not found, creating…
Locking [dev-packages] dependencies…
Locking [packages] dependencies…
Updated Pipfile.lock (ca72e7)!
Installing dependencies from Pipfile.lock (ca72e7)…
  🐍   ▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉ 0/0 — 00:00:00
To activate this project's virtualenv, run pipenv shell.
Alternatively, run a command inside the virtualenv with pipenv run.
```

結果的にPipfileとPipfile.lockが生成されている。

```sh
pipenv install responder
pipenv shell
```

ここまでで必要パッケージをインストールして仮想環境に入り込んでいる。

### ルーティング

ルーティングをひと言でいうと「クライアントのリクエスト内容と、サーバーの処理内容を紐付ける作業」

# 2020/04/15　コメント

Nuxt.jsの勉強を主目的として始めたがresponderの勉強が同時に必要で，Nuxt.jsが定着しなさそうなので一旦ストップする。
ネットに落ちているNuxt.jsだけで作ったアプリに似たものを自分で作ることを目的としてNuxt.jsが定逆してからresponderの勉強に戻ってくることとする。
