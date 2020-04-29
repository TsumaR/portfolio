---
title: "Pythonでアルゴリズムの勉強~全探索"
date: 2020-04-30T01:52:09+09:00
draft: true
---

[AtCoder 版！蟻本 (初級編)](https://qiita.com/drken/items/e77685614f3c6bf86f44)でアルゴリズムの勉強。自力で解けなかった問題もあるので復讐が必要。

# 全てのきほん "全探索"

## 例題 2-1-1 部分和問題

再起関数を用いて全探索を行う。
参考にした例では，

```
def dfs():
    if 最後まで来ていたら:
         return
    まだ途中なら:
    　　　処理内容
```

といった形を取っていた。

### abc045c

1 以上 9 以下の数字のみからなる文字列 
S
 が与えられます。 この文字列の中で、あなたはこれら文字と文字の間のうち、いくつかの場所に + を入れることができます。 一つも入れなくてもかまいません。 ただし、+ が連続してはいけません。

このようにして出来る全ての文字列を数式とみなし、和を計算することができます。

ありうる全ての数式の値を計算し、その合計を出力してください。
 
 #### 部分話の例
 
 与えられた整数値の全てを合計する


```python
def total(n):
    if n < 1:
        return 0

    return n + total(n - 1)

print(total(100))
```

    5050


全探索で前から+をいれる箇所と入れない箇所をリストとして保持しておけば良い。


```python
s = input()
l = len(s)

def dfs(i, sentence):
    # 再起が最後まで終了した時
    if i == l - 1:
        return sum(list(map(int, sentence.split("+"))))
    return dfs(i+1, sentence + "+" + s[i+1]) + dfs(i+1, sentence+s[i+1])

print(dfs(0, s[0]))
```

     9999999999


    12656242944


### abc079c train ticket

駅の待合室に座っているjoisinoお姉ちゃんは，切符を眺めています。
切符には4つの0以上9以下の整数A B C Dが整理番号としてこの順に書かれています。
Aop1 Bop2 Cop3 D = 7となるように，op1op2op3に+か-を入れて式を作ってください。
なお，答えが存在しない入力は与えられず，また答えが複数存在する場合はどれを出力しても良いものとします。

0
≦
A
,
B
,
C
,
D
≦
9


```python
s = input()

def dfs(i, f, sum):
    if i == 3:
        if sum == 7:
            print(f + "=7")
            exit()
    else:
        dfs(i+1, f+"+"+s[i+1], sum + int(s[i+1]))
        dfs(i+1, f+"-"+s[i+1], sum - int(s[i+1]))

dfs(0, s[0], int(s[0]))
```

     3242


    3+2+4-2=7
    3-2+4+2=7


### abc104c-allGreen

プログラミングコンペティションサイト AtCode は、アルゴリズムの問題集を提供しています。 それぞれの問題には、難易度に応じて点数が付けられています。 現在、
1
 以上 
D
 以下のそれぞれの整数 
i
 に対して、
100
i
 点を付けられた問題が 
p
i
 問存在します。 これらの 
p
1
+
…
+
p
D
 問が AtCode に収録された問題のすべてです。

AtCode のユーザーは 総合スコア と呼ばれる値を持ちます。 ユーザーの総合スコアは、以下の 
2
 つの要素の和です。

基本スコア: ユーザーが解いた問題すべての配点の合計です。
コンプリートボーナス: 
100
i
 点を付けられた 
p
i
 問の問題すべてを解いたユーザーは、基本スコアと別にコンプリートボーナス 
c
i
 点を獲得します 
(
1
≤
i
≤
D
)
。
AtCode の新たなユーザーとなった高橋くんは、まだ問題を 
1
 問も解いていません。 彼の目標は、総合スコアを 
G
 点以上にすることです。 このためには、少なくとも何問の問題を解く必要があるでしょうか？
 
 制約
1
≤
D
≤
10
1
≤
p
i
≤
100
100
≤
c
i
≤
10
6
100
≤
G
入力中のすべての値は整数である。
c
i
,
G
 はすべて 
100
 の倍数である。
総合スコアを 
G
 点以上にすることは可能である。

中途半端なのを複数解くくらいなら得点の高い方を，ボーナスがもらえるまで解いてしまった方が早い。よって，中途半端に解く問題は0以上1種類以下。
完全に解くかを決める→残りの配点のうちもっとも高いものが中途半端に解く問題となる。
再起関数で定義する。
総合点-完全に解く数

関数内でglobal変数を宣言することで記載をコンパクトに


```python
d, g = map(int, input().split())
pc = [list(map(int, input().split())) for i in range(d)]
ans = float("inf")
# 解いていない問題の配点をrestとして保持しておく。
# iは解いた問題の種類(配点/100)
# countは解いた問題の数
def dfs(i,sum,count,rest):
    # global変数としてansを引継ぎ
    global ans
    if i == d:
        # 合計がgoalを超えていなければrestに残っている問題の中で最大の得点の問題を解いていく。
        if sum < g:
            use = max(rest)
            n = min(pc[use-1][0], -(-(g-sum)//(use*100)))
            count += n
            sum += n*use * 100
            
        # 合計値が与えられた得点を超えていたら，最小の問題数を答える。
        if sum >= g:
            ans = min(ans, count)
            
    else:
        # 総合スコア，解いた問題数，まだ解いていない問題を更新する。
        # i得点の問題を解かないで次に進める場合
        dfs(i + 1, sum, count, rest)
        # i得点の問題を解いて先に進める場合，得点と問題数を加え，restからその得点要素を消去
        dfs(i + 1, sum + pc[i][0] * (i+1) * 100 + pc[i][1], count + pc[i][0], rest - {i+1})


dfs(0,0,0,set(range(1,d + 1)))

# 関数内で定義したグローバル変数を出力する
print(ans)
```

     再起



    ---------------------------------------------------------------------------

    KeyboardInterrupt                         Traceback (most recent call last)

    ~/anaconda/lib/python3.6/site-packages/ipykernel/kernelbase.py in _input_request(self, prompt, ident, parent, password)
        884             try:
    --> 885                 ident, reply = self.session.recv(self.stdin_socket, 0)
        886             except Exception:


    ~/anaconda/lib/python3.6/site-packages/jupyter_client/session.py in recv(self, socket, mode, content, copy)
        802         try:
    --> 803             msg_list = socket.recv_multipart(mode, copy=copy)
        804         except zmq.ZMQError as e:


    ~/anaconda/lib/python3.6/site-packages/zmq/sugar/socket.py in recv_multipart(self, flags, copy, track)
        474         """
    --> 475         parts = [self.recv(flags, copy=copy, track=track)]
        476         # have first part already, only loop while more to receive


    zmq/backend/cython/socket.pyx in zmq.backend.cython.socket.Socket.recv()


    zmq/backend/cython/socket.pyx in zmq.backend.cython.socket.Socket.recv()


    zmq/backend/cython/socket.pyx in zmq.backend.cython.socket._recv_copy()


    ~/anaconda/lib/python3.6/site-packages/zmq/backend/cython/checkrc.pxd in zmq.backend.cython.checkrc._check_rc()


    KeyboardInterrupt: 

    
    During handling of the above exception, another exception occurred:


    KeyboardInterrupt                         Traceback (most recent call last)

    <ipython-input-22-ccbd79fa75cb> in <module>
    ----> 1 d, g = map(int, input().split())
          2 pc = [list(map(int, input().split())) for i in range(d)]
          3 ans = float("inf")
          4 # 解いていない問題の配点をrestとして保持しておく。
          5 # iは解いた問題の種類(配点/100)


    ~/anaconda/lib/python3.6/site-packages/ipykernel/kernelbase.py in raw_input(self, prompt)
        858             self._parent_ident,
        859             self._parent_header,
    --> 860             password=False,
        861         )
        862 


    ~/anaconda/lib/python3.6/site-packages/ipykernel/kernelbase.py in _input_request(self, prompt, ident, parent, password)
        888             except KeyboardInterrupt:
        889                 # re-raise KeyboardInterrupt, to truncate traceback
    --> 890                 raise KeyboardInterrupt
        891             else:
        892                 break


    KeyboardInterrupt: 


## 例題2-1-2 Lake Counting

キーワード

* DFS
* (弱)連結成分分解
* グリッドグラフ上の探索

[DFS](https://qiita.com/drken/items/4a7869c5e304883f539b)は深さ優先探索(縦型探索)のこと。
条件に合う葉を何でもいいから一つ探し出したいときに向いている。(逆に横型探索が向いているのは将棋の局面など全てを算出したいとき。)
深さ優先(DFS)では新しく検出したノードをスタックとして追加して，読みこむ。注意として，根つき木に限らず，グラフに対する探索法。
また，DFSは再帰関数と相性が良い。「まだ探索していない頂点があるうちは、探索していない頂点 vv を一つ選んで vv を始点とした DFS を実施することを繰り返す」という考え方をする。ただし，再帰で解くと，深くなりすぎたときにスタックオーバーフローになることがあるらしい。とりあえず再帰の練習の一環ので再帰で解く。
c++によるテンプレートは以下の通り。

```c++
import sys
sys.setrecursionlimit(10**7) #再帰関数の呼び出し制限
h,w = map(int,input().split())
c = [list(input()) for i in range(h)]

def dfs(x,y):
    if not(0 <= x < h) or not(0 <= y < w) or c[x][y] == "#": # 壁に当たったり、探索範囲外になった場合はreturn
        return
    if c[x][y] == "g": # ゴールを見つけたら”Yes”を出力して終了
        print("Yes")
        sys.exit()
        
    c[x][y] = "#" #探索済みを示すためのマーキング
    dfs(x+1,y)
    dfs(x-1,y)
    dfs(x,y+1)
    dfs(x,y-1)

for i in range(h):
    for j in range(w):
        if c[i][j] == "s":
            sx, sy = i, j # スタート位置
            
dfs(sx, sy)
print("No")
```

行きがけ順。
![行きかけ](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F182963%2F462e78fe-8907-7e50-fb32-581326c3637d.png?ixlib=rb-1.2.2&auto=format&gif-q=60&q=75&w=1400&fit=max&s=bdb3f88c1e9222e91cc86a7a7c6920a2)

帰りかけ順。
![帰りかけ](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F182963%2F7fd74230-5be2-26ec-96ff-d5f350b15bab.png?ixlib=rb-1.2.2&auto=format&gif-q=60&q=75&w=1400&fit=max&s=8154b47a8101974d4e6c9ef13b69d62c)

### atc001a 深さ優先探索

> 高橋君の住む街は長方形の形をしており、格子状の区画に区切られています。 長方形の各辺は東西及び南北に並行です。 各区画は道または塀のどちらかであり、高橋君は道を東西南北に移動できますが斜めには移動できません。 また、塀の区画は通ることができません。
高橋君が、塀を壊したりすることなく道を通って魚屋にたどり着けるかどうか判定してください。

```
s: 家
g: 魚屋
.: 道
#: 堀
```

迷路で道があるか問題。深さ優先探索による塗り潰し。全箇所から4方向への移動を試す。試していないところを覚えておく。とりあえず行き止まりまで猪突猛進。

*　考え方 *
位置を引数にした再帰関数を使う。自分から4方向への呼び出しを行う。
再帰関数なので呼び出した後に戻ってくる。

#### 疑似コード

```
関数 search (x,y) {
  if (場所 (x,y)が壁か迷路の外) return
  if (既に (x,y)に一度到達している　return
    (x,y)に到達したということを記録。
  
  // 4方向を試す
  search(x+1, y)
  search(x-1, y)
  search(x, y-1)
  search(x, y+1)
}
```


```python
import sys
sys.setrecursionlimit(10**7) 

h,w = map(int ,input().split())
c = [list(input()) for i in range(h)]
d = [[0] * w for i in range(h)]

def deep_search(x,y):
    if not(0 <= x < h) or not(0 <= y < w) or c[x][y] == "#" or d[x][y] ==1:
        return
    if c[x][y] == "g": # ゴールを見つけたら”Yes”を出力して終了
        print("Yes")
        sys.exit()
    d[x][y] == 1
    deep_search(x+1, y)
    deep_search(x-1, y)
    deep_search(x, y+1)
    deep_search(x, y-1)

# スタート地点から開始
for i in range(h):
    for j in range(w):
        if c[i][j] == "s":
            sx, sy = i, j

deep_search(sx, sy)
print("No")
```

### arc031b 埋め立て

> とある所に島国がありました。島国にはいくつかの島があります。このたび、この島国で埋め立て計画が立案されたのですが、どこを埋め立てるか決まっていません。できることなら埋め立てによって島を繋いで、
1
 つの島にしてしまいたいのですが、たくさん埋め立てるわけにもいきません。
10
 マス × 
10
 マスのこの島国の地図が与えられるので、
1
 マスを埋め立てた時に 
1
 つの島にできるか判定してください。ただし、地図で陸地を表すマスが上下左右につながっている領域のことを島と呼びます。

#### 考察

埋める候補は最悪で10*10 = 100なので，全ての場合について考えて良さそう
あるマスから全てのマスに到達することができるかをグラフ探索で全探索すれば良い。
i,jを0~100まで順にみる。
要素がxの時，oに置き換えてその点からDFSで探索し，一度通った島をxに変換。
全て探索を終えたときにマス全てがxになっているかを調べる。

多次元配列のリストコピーには`copy`ライブラリの
`copy.deepcopy`をする必要がある。


```python
import copy
import itertools
import sys

A = [list(input()) for i in range(10)]

def umetate(x,y):
    if not(0 <= x < 10) or not(0 <= y < 10) or A_cop[x][y] == "x":
        return
    if A_cop[x][y] == "o":
        A_cop[x][y] = "x"
        umetate(x+1, y)
        umetate(x-1, y)
        umetate(x, y-1)
        umetate(x, y+1)


for i in range(10):
    for j in range(10):
        if A[i][j] == "x":
            A_cop = copy.deepcopy(A)
            A_cop[i][j] = "o"
            umetate(i,j)
            flat_A_copy = list(itertools.chain.from_iterable(A_cop))
            if "o" not in flat_A_copy:
                print("YES")
                sys.exit()

print("NO")
```

     xxxxoxxxxx
     xxxxoxxxxx
     xxxxoxxxxx
     xxxxoxxxxx
     ooooxooooo
     xxxxoxxxxx
     xxxxoxxxxx
     xxxxoxxxxx
     xxxxoxxxxx
     xxxxoxxxxx


    YES



    An exception has occurred, use %tb to see the full traceback.


    SystemExit



    /Users/wagatsuma/anaconda/lib/python3.6/site-packages/IPython/core/interactiveshell.py:3334: UserWarning: To exit: use 'exit', 'quit', or Ctrl-D.
      warn("To exit: use 'exit', 'quit', or Ctrl-D.", stacklevel=1)


## 例題2-1-3 迷路の最短経路

[キーワード]

* [BFS](https://qiita.com/drken/items/996d80bcae64649a6580)
* BFSを用いる最短経路問題
* グリッド上の探索

DFSがスタックに基づいていたのに対して，幅優先探索(BFS)はキューに基づく。候補を一つでも算出したい際に用いたDFSと比べて，全部の候補の中から最善を選ぶ場合などに用いるのがBFS。ただし，BFSでできることはほとんどがDFSで実装できる。そりゃそうか。
例えば，迷路が辿り着けるかや，島は一つになっているか？はDFSで解いた。
それに比べノード間の最短距離を求める問題などはBFS,幅優先探索を用いる。(ただし，各エッジに重みがないことが条件)

深さ優先探索の場合は，再帰関数とスタックによる実装の両方を行うことができたが，幅優先探索ではキューを使うことがほとんど一般的。

迷路の最短経路のdistの値は下図のようになり，この例では16回でゴールにつく。


![最短経路](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F182963%2F3aef75d2-3bf2-5e43-aab9-6ee293b4a38a.png?ixlib=rb-1.2.2&auto=format&gif-q=60&q=75&w=1400&fit=max&s=54ce4355b43777ecd23c02b04b250654)


### [幅優先探索(abc007c)](https://atcoder.jp/contests/abc007/tasks/abc007_3)

たかはし君は迷路が好きです。今、上下左右に移動できる二次元盤面上の迷路を解こうとしています。盤面は以下のような形式で与えられます。

まず、盤面のサイズと、迷路のスタート地点とゴール地点の座標が与えられる。
次に、それぞれのマスが通行可能な空きマス(.)か通行不可能な壁マス(#)かという情報を持った盤面が与えられる。盤面は壁マスで囲まれている。スタート地点とゴール地点は必ず空きマスであり、スタート地点からゴール地点へは、空きマスを辿って必ずたどり着ける。具体的には、入出力例を参考にすると良い。
今、彼は上記の迷路を解くのに必要な最小移動手数を求めたいと思っています。どうやって求めるかを調べていたところ、「幅優先探索」という手法が効率的であることを知りました。幅優先探索というのは以下の手法です。

スタート地点から近い(たどり着くための最短手数が少ない)マスから順番に、たどり着く手数を以下のように確定していく。説明の例として図1の迷路を利用する

接したことのある点をqueに保持する。


```python
from collections import deque

r, c = map(int, input().split())
sy, sx = map(int, input().split())
gy, gx = map(int, input().split())
maze = [list(input()) for i in range(r)]

sy -= 1
sx -= 1
gy -= 1
gx -= 1

# キューを使って解く
# キューはdeque([])で定義し，popleft()で取り出す。

def find_shortest():
    # たどり着くまでにかかった回数をいれる多重リスト。
    d = [[float("inf")] * c for i in range(r)]
    # 1ノード降りるときにforで回せるようリスト化しておく
    dx = [1, 0, -1, 0]
    dy = [0, 1, 0, -1]

    # 接したポイントを取り込む用のqueを定義
    que = deque([])
    # スタート地点をqueに加える。探索の際はqueから取り出すループを回すので最初だけ先に入れておく
    que.append((sy,sx))
    # スタートからスタートにかかる距離は0なのでinfを0にかえる。
    d[sy][sx] = 0

    # queがあるかぎり続ける
    while que:
        # queから座標を取り出す。
        p = que.popleft()
        # 取り出した点がゴールなら終了
        if p[0] == gy and p[1] == gx:
            break
        #　それ以外の時は，接している点を4方向順に取得。一つずつ処理する。
        for i in range(4):
            nx = p[0] + dx[i]
            ny = p[1] + dy[i]

            # 新しく取得した座標に対しての処理
            # 迷路内に存在する座標であり，壁ではなく，まだ通ったことがなくdがinfの点なら処理する = 新しく取得した道候補の1つ
            if 0 <= nx < r and 0 <= ny < c and maze[nx][ny] != "#" and d[nx][ny] == float("inf"):
                # 道候補をqueに加える。
                que.append((nx, ny))
                # その道候補への経路までにかかった経路をdに加える。
                d[nx][ny] = d[p[0]][p[1]] + 1
    return d[gy][gx]

ans = find_shortest()
print(ans)
```

### [チーズ(Cheeze) JOI2010予選Eチーズ](https://atcoder.jp/contests/joi2011yo/tasks/joi2011yo_e)

問題
> 今年も JOI 町のチーズ工場がチーズの生産を始め，ねずみが巣から顔を出した．JOI 町は東西南北に区画整理されていて，各区画は巣，チーズ工場，障害物，空き地のいずれかである．ねずみは巣から出発して全てのチーズ工場を訪れチーズを 
1
 個ずつ食べる．
この町には， 
N
 個のチーズ工場があり，どの工場も 
１
 種類のチーズだけを生産している．チーズの硬さは工場によって異なっており，硬さ 
1
 から 
N
 までのチーズを生産するチーズ工場がちょうど 
1
 つずつある．
ねずみの最初の体力は 
1
 であり，チーズを 
1
 個食べるごとに体力が 
1
 増える．ただし，ねずみは自分の体力よりも硬いチーズを 食べることはできない．
ねずみは，東西南北に隣り合う区画に 
1
 分で移動することができるが，障害物の区画には入ることができない．チーズ工場をチーズを食べずに通り過ぎる こともできる．すべてのチーズを食べ終えるまでにかかる最短時間を求めるプログラ ムを書け．ただし，ねずみがチーズを食べるのにかかる時間は無視できる．
 
出力
> すべてのチーズを食べ終えるまでにかかる最短時間（分）を表す整数を 
1
 行で出力せよ．

考察

最短経路なので幅優先探索で解くことを考える。
1を食べると2になり，その時点で食べれるチーズは2だけなので，順番に食べていくしかない。
つまり，チーズを食べるごとにその時点の工場をスタート，1つ硬いチーズの工場をゴールとして最短経路の探索をループすれば良い。

4版だけTLEしてしまったのであとでやり直す必要がある。


```python
from collections import deque

h,w,n = map(int, input().split())
maze = [list(input()) for i in range(h)]

def find_shortest():
    dx = [1, 0, -1, 0]
    dy = [0, 1, 0, -1]
    result_distance = 0
    # スタートをリセットしながら回す。
    # 都度queを設定して最初から回す。
    for height in range(h):
        for width in range(w):
            if maze[height][width] == "S":
                sx, sy = height, width
    for hardness in range(n+1):
        # リセットされた中での距離
        d = [[float("inf")] * w for i in range(h)]
        que = deque([])
        que.append((sx, sy))
        d[sx][sy] = 0
        # 現在のスタート値から次のレベルに到達するまでにかかる距離をbfsで検索
        while que:
            p = que.popleft()
            # リセットされた中でゴールについたら
            if maze[p[0]][p[1]] == str(hardness+1):
                result_distance += d[p[0]][p[1]]
                sx = p[0]
                sy = p[1]
                break
            for i in range(4):
                nx = p[0] + dx[i]
                ny = p[1] + dy[i]
                if 0 <= nx < h and 0 <= ny < w and maze[nx][ny] != "X" and d[nx][ny] == float("inf"):
                    que.append((nx, ny))
                    d[nx][ny] = d[p[0]][p[1]] + 1
    return result_distance
        
ans = find_shortest()
print(ans)
```

## 2-1-4 特殊な状態の列挙

【キーワード】

* n!通りの全探索
* next_permutaion

【コメント】
> 小さな n でしか適用できないので難しい問題になると逆に見かけないですが、n! 通りの全探索ができることも重要なステップになります。n が少し大きくなると bit DP が想定解法であるケースが多いです。

### [One-stroke Path(abc054-c)](https://atcoder.jp/contests/abc054/tasks/abc054_c)

自己ループと二重辺を含まない 
N
 頂点 
M
 辺の重み無し無向グラフが与えられます。
i
(
1
≦
i
≦
M
)
 番目の辺は頂点 
a
i
 と頂点 
b
i
 を結びます。
ここで、自己ループは 
a
i
=
b
i
(
1
≦
i
≦
M
)
 となる辺のことを表します。
また、二重辺は 
a
i
=
a
j
 かつ 
b
i
=
b
j
(
1
≦
i
<
j
≦
M
)
 となる辺のことを表します。
頂点 
1
 を始点として、全ての頂点を1度だけ訪れるパスは何通りありますか。
ただし、パスの始点と終点の頂点も訪れたものとみなします。

#### 制約

2
≦
N
≦
8
0
≦
M
≦
N
(
N
−
1
)
/
2
1
≦
a
i
<
b
i
≦
N
与えられるグラフは自己ループと二重辺を含まない。

考察

深さ優先探索を行う。(一度通った点は二度と通れないとして)
更新できなくなったときに，全ての点を通過していればok。
N=8なので，とりあえず再帰を回せば解けそう。

__グラフを考えるときに，二重リストで，繋がっている点を1，繋がっていない場合に0として記録しておくと良い__


```python
N,M = map(int, input().split())

# 繋がっている点を記載する二重リストを準備する。
adj_matrix = [[0]* N for _ in range(N)]
for i in range(M):
    a, b = map(int, input().split())
    adj_matrix[a-1][b-1] = 1
    adj_matrix[b-1][a-1] = 1

# 点を通ったかを記録するリスト
used = [False] * N
used[0] = True


# 深さ探索を行うため再帰関数を定義する。
# 現在いる点をvとしての再帰
def dfs(v, used):
    # 再帰を繰り返してusedの中身が全てTrueになったときのみ1を返すようにする。
    if not False in used:
        return 1

    ans = 0
    for i in range(N):
        if adj_matrix[v][i] != 1:
            continue
        if used[i]:
            continue
        used[i] = True
        ans += dfs(i, used)
        used[i] = False
    # usedの中にFalseがあるとき用のreturn(再帰で用いられるようのreturn)
    return ans

print(dfs(0, used))
```

     3 3
     1 2
     1 3
     2 3


    2


### カード並べ(JOI-D2010)

> 花子さんは 
n
 枚 
(
4
≤
n
≤
10
)
 のカードを並べて 遊んでいる． それぞれのカードには 
1
 以上 
99
 以下の整数 が 
1
 つずつ書かれている． 花子さんは，これらのカードの中から 
k
 枚 
(
2
≤
k
≤
4
)
 を選び， 横一列に並べて整数を作ることにした． 花子さんは全部で何種類の整数を作ることができるだろうか．
例えば， 
1
,
2
,
3
,
13
,
21
 の 
5
 枚のカードが与えられ，そこから 
3
 枚を選び整数を作ることを考える． 
2
,
1
,
13
 をこの順に並べると，整数 
2113
 を作ることができる． また， 
21
,
1
,
3
 をこの順に並べても，同じ整数 
2113
 を作ることができる． このように，異なるカードの組み合わせから同じ整数が作られることもある．
n
 枚のカードに書かれた整数が与えられたとき， その中から 
k
 枚を選び，横一列に並べることで作ることができる整数の個数を求めるプログ ラムを作成せよ．

考察

最大でnPkなので，5040通りの数ができる。permutationをitertoolsで取り出せば良い。


```python
import itertools

N = int(input())
k = int(input())
card = [input() for i in range(N)]

number = set()
for i in itertools.permutations(card, k):
    number.add("".join(i))

print(len(number))
```

