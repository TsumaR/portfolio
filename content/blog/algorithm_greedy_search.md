---
title: "Pythonで貪欲法の学習"
date: 2020-05-01T12:11:50+09:00
draft: false
---

# 2-2 猪突猛進! "貪欲法"

> 貪欲法は局所探索法と並んで近似アルゴリズムの最も基本的な考え方の一つである。
このアルゴリズムは問題の要素を複数の部分問題に分割し、それぞれを独立に評価を行い、評価値の高い順に取り込んでいくことで解を得るという方法である。動的計画法と異なり保持する状態は常に一つであり、一度選択した要素を再考する事は無い。このため得られる解は最適解であるという保証は無いが部分問題の解法と単純なソートのみでプログラムを実装することが可能であり、多く問題に対して多項式時間での近似アルゴリズムとなる。

## Addition and Multiplication(abc70-b)

> 問題
square1001 は、電光掲示板に整数1が表示されているのを見ました。
彼は、電光掲示板に対して、以下の操作 A, 操作 B をすることができます。
操作 A： 電光掲示板に表示する整数を「今の電光掲示板の整数を2倍にしたもの」に変える。
操作 B： 電光掲示板に表示する整数を「今の電光掲示板の整数にKを足したもの」に変える。
square1001は、操作A,操作B合計でN回行わなければなりません。そのとき、N回の操作後の、電光掲示板に書かれている整数として考えられる最小の値を求めなさい。
制約

>1≤N,K≤101≤N,K≤10
入力(N,KN,K)はすべて整数である

考察
N回の操作のうち，それぞれで小さくなるものを選ぶという問題に分割できる。

答え

```python
N = int(input())
K = int(input())

display = 1
for i in range(N):
    if display + K <= 2 * display:
        display += K
    else:
        display *= 2

print(display)
```

## 例題2-2-1 硬貨の問題

【キーワード】

* Greedy
* コイン

【コメント】
硬貨の問題がgreedyで良いことはそんなに自明じゃないらしい。

【問題】

> 太郎君はよくJOI雑貨店で買い物をする． JOI雑貨店には硬貨は 
500
 円， 
100
 円， 
50
 円， 
10
 円， 
5
 円， 
1
 円が十分な数だけあり，いつも最も枚数が少なくなるようなおつ りの支払い方をする． 太郎君がJOI雑貨店で買い物をしてレジで 
1000
 円札を 
1
 枚出した時，もらうおつりに含まれる硬貨の枚数を求めるプログ ラムを作成せよ．
例えば入力例 
1
 の場合は下の図に示すように， 
4
 を出力しなければならない．


```python
Charge = 1000 - int(input())

count_coin = 0

coin_kind = [1, 5, 10, 50, 100, 500]

for i in range(1,7):
    t = Charge // coin_kind[-i]
    Charge -= t * coin_kind[-i]
    count_coin += t

print(count_coin)
```

     380


    4


## 例題2-2-2 区間スケジューリング問題

始点，終点が決まっているものについて，できる限り多く選択していく問題。

【キーワード】

* Greedy
* 区間の終端でソート (DEGwer さんの[数え上げテクニック集](https://drive.google.com/file/d/1WC7Y2Ni-8elttUgorfbix9tO1fvYN3g3/view) にも記載あり)

【コメント】

> 区間の終端 (または始端) でソートするのは極めてよくみるテクニックで、今後難しい問題に挑むときにも常に念頭に置いておきたいです。

### [キーエンスプログラミングコンテスト2020 B Robot Arms](https://atcoder.jp/contests/keyence2020/tasks/keyence2020_b)

> ある工場では、 数直線上に 
N
 個のロボットが設置されています。 ロボット 
i
 は座標 
X
i
 に設置されており、数直線の正負の方向にそれぞれ長さ 
L
i
 の腕を伸ばすことができます。
これらのロボットのうちいくつか (
0
 個以上) を取り除き、 残ったどの 
2
 つのロボットについても、腕が動く範囲が共通部分を持たないようにしたいと思います。 ただし、各 
i
 (
1
≤
i
≤
N
) に対して、 ロボット 
i
 の腕が動く範囲とは 数直線上の座標が 
X
i
−
L
i
 より大きく 
X
i
+
L
i
 未満の部分を指します。
取り除かずに残せるロボットの個数の最大値を求めてください。

考察

できるだけたくさんを取りたいので，終点が近い順に選んでいけば良い。→終点昇順で並べかえ，順番に取っていくだけ


```python
N = int(input())
R = [list(map(int, input().split())) for _ in range(N)]

# [終点，始点]のリスト
coordinate = [[i[0] + i[1], max(i[0] - i[1], 0), ] for i in R]
coordinate.sort()

ans = 0
end_point = 0

for i in range(N):
    if end_point <= coordinate[i][1]:
        ans += 1
        end_point = coordinate[i][0]

print(ans)
```

### [A-東京都(KUPC2015-a)](https://atcoder.jp/contests/kupc2015/tasks/kupc2015_a)

> KUPC2015は東京と京都の二箇所でオンサイトが開催されている． あなたはKUPCの告知を手伝うことにした．
英小文字からなる文字列が印字されたテープがある．あなたはこのテープを文字同士の間でのみ好きなだけ自由に切ってもよい．
あなたはtokyoかkyotoのいずれかの文字列を含むテープをなるべくたくさん作りたい．ただし，一旦切ったテープを後でくっつけることはできないものとする．
作る事ができるtokyoもしくはkyotoを含むテープの数の最大値を出力せよ．

考察

今回は後ろ順でsortするとかしなくて良い。単純に前から5文字ずつ見ていく。


```python
T = int(input())
for t in range(T):
    S = input()
    count = 0
    position = 0
    while position < len(S) - 4:
        if S[position:position + 5] == "tokyo" or S[position:position + 5] == "kyoto":
            count += 1
            position += 5
        else:
            position += 1
    print(count)
```

     3
     higashikyoto


    1


     kupconsitokyotokyoto


    2


     kaljoagighakyoto


    1


## 例題 2-2-3 Best Cow Line

【キーワード】

* Greedy
* 辞書順最小なものを求めるGreedy

【コメント】

> 辞書順最小なものを求めよと言われたときにとにかく先頭から最小になることを優先していく考え方は超頻出ですね。

### [Dubious Document 2 (abc076c)](https://atcoder.jp/contests/abc076/tasks/abc076_c)

> E869120 は、宝物が入ってそうな箱を見つけました。
しかし、これには鍵がかかっており、鍵を開けるためには英小文字からなる文字列 
S
 が必要です。
彼は文字列 
S
′
 を見つけ、これは文字列 
S
 の 
0
 個以上 
|
S
|
 個以内の文字が ? に置き換わった文字列であることも分かりました。
ただし、文字列 
A
 に対して、
|
A
|
 を「文字列 
A
 の長さ」とします。
そこで、E869120 はヒントとなる紙を見つけました。
条件1：文字列 
S
 の中に連続する部分文字列として英小文字から成る文字列 
T
 が含まれている。
条件2：
S
 は、条件1を満たす文字列の中で辞書順最小の文字列である。
そのとき、鍵となる文字列 
S
 を出力しなさい。
ただし、そのような文字列 
S
 が存在しない場合は代わりに UNRESTORABLE と出力しなさい。

考察

二箇所以上に入るなら，後ろ側に入れ，前のところは全てaで埋めてしまった方がいい。
つまり，後ろから探索することを考える。


```python
import sys

S = list(input())
T = list(input())

length = len(T)
position = len(S) - length

while position >= 0:
    text = S[position:position+length]
    for i in range(length):
        if text[i] == "?":
            text[i] = T[i]
    if text != T:
        position -= 1
    else:
        S[position:position+length] = T
        position = -10

if position != -10:
    print("UNRESTORABLE")
    sys.exit()

for i in range(len(S)):
    if S[i] == "?":
        S[i] = "a"

print("".join(S))
```

     ?tc????
     coder


    atcoder


ただし，もっと綺麗にかけるはず。例えば

```python
sd = input()
t = input()

n = len(sd)
m = len(t)
s = []

# sd を後ろから見ていき、 t の入りそうな場所を探す
for i in range(n - m, -1, -1):
    t_kamo = sd[i:i + m]
    for j in range(m + 1):
        # 1文字ずつ順に入りうるか調べ、最後まで入るなら "?" を "a" に置き換えて出力
        if j == m:
            print((sd[:i] + t + sd[i + len(t):]).replace("?", "a"))
            exit()
        if t_kamo[j] == "?":
            continue
        elif t_kamo[j] != t[j]:
            break

print("UNRESTORABLE")
```

## 例題2-2-4 Saruman's Army

【キーワード】
* Greedy
* より厳しいところを取っていくGreedy。(一般に「交換しても悪化しない」)

【コメント】

> Greedy アルゴリズムを考えるときに、より厳しいところをとっていく考え方は頻出のイメージです。その最も典型的な例として、一次元的 (二次元量もOK) な数量の大小関係だけでマッチング条件が決まるような問題における最大二部マッチング問題があります。

### [Multiple Gift(abc083c)](https://atcoder.jp/contests/abc083/tasks/arc088_a)

> 高橋君は、日頃の感謝を込めて、お母さんに数列をプレゼントすることにしました。 お母さんにプレゼントする数列 
A
 は、以下の条件を満たす必要があります。
A
 は 
X
 以上 
Y
 以下の整数からなる
すべての 
1
≤
i
≤
|
A
|
−
1
 に対し、
A
i
+
1
 は 
A
i
 の倍数であり、かつ 
A
i
+
1
 は 
A
i
 より真に大きい
高橋君がお母さんにプレゼントできる数列の長さの最大値を求めてください。

考察

Ai +1 = 2*Aiということ。
スタートは必ずXで良い。
X * 2^(a-1) <= Y となる最大のaを求める問題。


```python
x, y = map(int, input().split())

ans = 0

while x <= y:
    x *= 2
    ans += 1

print(ans)
```

     25 100


    3


### [積み重ね(arc006c)](https://atcoder.jp/contests/arc006/tasks/arc006_3)

> 高橋君はもう大人なので、親元を離れて一人暮らしをすることにしました。トラックから引越し先の部屋へと荷物のダンボールを運びたいのですが、部屋の床がダンボールで埋まってしまうと、今日高橋君が寝るための布団がひけません。
　そこで、
1
 箱ずつ広げて置くのではなく、ある程度ダンボールを積み重ねた山を作ることにしました。しかし、ダンボールには重さが決まっており、下にあるダンボールよりも重いダンボールを上に積み重ねると下のダンボールが潰れてしまいます。
 トラックから運ぶ順にダンボールの重さが与えられるので、ダンボールを潰さないような積み重ね方を考えなさい。そして、その積み重ねた山の個数が最小となる場合の山の個数を求めなさい。
 
【コメント】
典型問題

考察

順番に取り出し，載せられる山があれば載せて，無理なら加える。
山の頂点の重さを記録したリストを作成すれば良い。


```python
from collections import deque

N = int(input())
D = [int(input()) for _ in range(N)]
tower = []

def stacking(box, tower):
    if tower == [] or box > max(tower):
        tower.append(box)
    else:
        tower.sort()
        for i in range(len(tower)):
            if tower[i] >= box:
                tower[i] = box
                break
                
    return tower

for j in range(N):
    box = D[j]
    stacking(box, tower)

print(len(tower))
```

     5
     4
     3
     1
     2
     1


    2


### [おいしいたこ焼きの売り方(abc005c)](https://atcoder.jp/contests/abc005/tasks/abc005_3)

> 高橋君は、たこ焼きをどの順番で売るか悩んでいました。というのも、作り置きされたたこ焼きは美味しくないとわかっているので、高橋君はそのようなたこ焼きを売りたくないのですが、できたてばかり売ってしまうと売れるたこ焼きの数が減ってしまいます。
また、お客さんを待たせてばかりだと、次第にお客さんが離れてしまうだろうと高橋君は考えています。
そこで、彼は 
T
 秒以内に作成されたたこ焼きを売り続けることで、お客さんを捌ききれるかどうかを調べることにしました。
たこ焼きは 
A
1
、
A
2
、…、
A
N
 秒後に焼きあがります。
お客さんは 
B
1
、
B
2
、…、
B
M
 秒後にやってきます。
1
 人のお客さんに対して、たこ焼きを 
1
 つ売るとします。すべてのお客さんにたこ焼きを売れるならyes、売れないならnoを出力して下さい。
 
 何故かRE，あとでやり直す必要がある。


```python
import copy
import sys
from collections import deque

# 何秒以内のたこ焼きまで売るか
T = int(input())
# たこ焼きの数
N = int(input())
A = list(map(int, input().split()))
# 来店者数
M = int(input())
B = list(map(int, input().split()))

# 客と
takoyaki = [0] * 100
customer = [0] * 100

for i in A:
    takoyaki[i] += 1
for i in B:
    customer[i] += 1

# メイン
yakiagari = deque([])
for i in range(100):
    for _ in range(takoyaki[i]):
        yakiagari.append(T)
    if len(yakiagari) < customer[i]:
        print("no")
        sys.exit()
    else:
        for _ in range(customer[i]):
            yakiagari.popleft()
    yakiagari = map(lambda x: x-1, yakiagari)
    yakiagari = deque([l for l in yakiagari if l < 0])
    
print("yes")
```

## Fence Repair(POJ No.3253)

【キーワード】

* Greedy
* priority_queue
* ハフマン符号
* 二分木
* ヒープ

【コメント】
ハフマン符号を求める Greedy、priority_queue(優先度付き待ち行列，優先度付きキュー)も用います。
Codeforces の問題ですが、なんとか近いものがありました！

優先度付きキュー： ある優先度（例えば、値の大きな物ほど優先度が高いとか）に従って、 優先度の高いものから順に取り出すことの出来るコレクション
優先度が1番高い要素1つだけを効率よく取り出すことの出来る、 ヒープと呼ばれるデータ構造があり、 通常、優先度付き待ち行列は、このヒープを用いて実装
Pythonでは[heapq](https://python.ms/binary-heap/#_2-%E3%82%B5%E3%83%B3%E3%83%97%E3%83%AB%E3%82%B3%E3%83%BC%E3%83%89)を用いることができる。


```python

```
