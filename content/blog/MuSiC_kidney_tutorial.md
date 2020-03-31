---
title: "solid tissueに対するMuSiCチュートリアル"
date: 2020-03-31T21:10:16+09:00
---

#### MuSiCとは
bulkのRNA-seqデータにおける細胞種の割合を，scRNA-seqデータを用いて推測するツール。2019年Nature Communication
簡単な方の例はこの[ページ](https://www.tsumar.com/blog/music_tutorial/)にまとめた。
内容が重複しすぎてもなので，先に上の記事を読んでからこの記事を読むことを前提としてこの記事は作成している。


# MuSiCチュートリアル(kidney)
固形組織には密接に関連した細胞タイプが含まれることが多く、これらの細胞タイプ間の遺伝子発現の相関はコリネアリティにつながり、バルクデータにおけるそれらの相対的な割合を解決することが困難になります。コリニアリティに対処するために、MuSiCは、密接に関連した細胞タイプを再帰的にズームインするツリーガイド手順を採用しています。簡単に言えば、最初に類似した細胞タイプを同じクラスタにグループ化し、クラスタの割合を推定してから、各クラスタ内でこの手順を再帰的に繰り返す。各再帰段階では、クラスタ内の分散が低い遺伝子、すなわち細胞間で一致する遺伝子のみを使用します。これは、高い分散を持つ遺伝子の平均発現推定値が、scRNA-seq実験の細胞キャプチャーにおける広範なバイアスの影響を受けるため、信頼性の高いリファレンスとしての役割を果たすことができないため、非常に重要である。
(本文ママ)

簡単にいうと，腎臓などの組織はそのまま推定するのが難しい。近いグループごとで段階的に推測していく。論文自体ではこのやり方に触れている。

## 細胞タイプの推測 

### シングルセルデータのクラスタリング 
`library(xbioc)`を加えないと`music_basesi`にエラーが生じる。
`music_basis`関数にシングルセルのデータだけを渡し，design matrix(計画行列)などを計算する。
desgin matrixとは説明変数をまとめた行列のこと。RPMLによると
```
「（入力変数に関して非線形な関数である）基底関数をモデルのパラメータだけ右の列に並べ、更に入力変数の数だけ下の行に並べた行列」
```
一般化線形モデルを考えるとわかりやすい。RPML読めばわかる。例えば今回は遺伝子の発現から細胞タイプを推定するわけであるから，同じようなことである。

```r
library(xbioc)
Mousesub.basis = music_basis(Mousesub.eset, clusters = 'cellType', samples = 'sampleID', 
                             select.ct = c('Endo', 'Podo', 'PT', 'LOH', 'DCT', 'CD-PC', 'CD-IC', 'Fib',
                                           'Macro', 'Neutro','B lymph', 'T lymph', 'NK'))

```

ここで`music_basis`によって出力されるのは，
```
Disgn.mtx: 細胞タイプに対する遺伝子の計画行列
S: subjectに対する細胞タイプのライブラリーサイズ(subject=クラスター)
M.S: 各細胞タイプの平均ライブラリーサイズ;
M.theta: 細胞タイプに対する各遺伝子の平均相対存在量matrix
Sigma: 細胞タイプに対する各遺伝子のcross-subject variation.
```

`music_basis`によって得られた結果に対して，`hclust`で最長距離法(完全連結法)による階層的クラスタリングを行った。
ここでは`Desgn.mtx`の距離行列と，`M.theta`のcross-subject mean matrix(平均相対存在量のことのはず)に対して行った。
階層的クラスタリングの基礎はここらへんの[記事](https://qiita.com/Haruka-Ogawa/items/fcda36cc9060ba851225)が参考になる。

```r
# Plot the dendrogram of design matrix and cross-subject mean of realtive abundance
par(mfrow = c(1, 2))
d <- dist(t(log(Mousesub.basis$Disgn.mtx + 1e-6)), method = "euclidean")
# Hierarchical clustering using Complete Linkage
hc1 <- hclust(d, method = "complete" )
# Plot the obtained dendrogram
plot(hc1, cex = 0.6, hang = -1, main = 'Cluster log(Design Matrix)')
d <- dist(t(log(Mousesub.basis$M.theta + 1e-8)), method = "euclidean")
# Hierarchical clustering using Complete Linkage
# hc2 <- hclust(d, method = "complete" )
hc2 <- hclust(d, method = "complete")
# Plot the obtained dendrogram
plot(hc2, cex = 0.6, hang = -1, main = 'Cluster log(Mean of RA)')
```
![hclust](https://xuranw.github.io/MuSiC/articles/images/Cluster.jpg)

距離行列による方の結果をもとに細胞タイプを4つのグループに分類する。
```
Group 1: Neutro
Group 2: Podo
Group 3: Endo, CD-PC, CD-IC, LOH, DCT, PT
Group 4: Fib, Macro, NK, B lymph, T lymph
```
まぁまぁそれっぽい。

### bulkの細胞タイプを細胞タイプグループ化により推定
最初に述べたように，大まかな細胞グループごとでbulkのサンプルの中に存在する細胞の存在比を調べ，さらにそのグループ内で調べていく。Rの処理的には一つの関数で完結するらしい。

#### 1. クラスタータイプでシングルセルデータをラベリング 
```r
clusters.type = list(C1 = 'Neutro', C2 = 'Podo', C3 = c('Endo', 'CD-PC', 'LOH', 'CD-IC', 'DCT', 'PT'), C4 = c('Macro', 'Fib', 'B lymph', 'NK', 'T lymph'))

cl.type = as.character(Mousesub.eset$cellType)

for(cl in 1:length(clusters.type)){
  cl.type[cl.type %in% clusters.type[[cl]]] = names(clusters.type)[cl]
}
pData(Mousesub.eset)$clusterType = factor(cl.type, levels = c(names(clusters.type), 'CD-Trans', 'Novel1', 'Novel2'))
```
#### 2. 実際に細胞種推定
tutorialのままだとうまくいかない。この[issue](https://github.com/xuranw/MuSiC/issues/27)を参考に書き換え実行した。ただし，C1やC2など1種類の細胞からなるグループの割合は分散が大きいらしい。今後の更新をチェックした方がいい。この記事作成時点(2020/03/31)では作者からの返信はなかった。 
これは前の記事で書いた`music_prop`に`group`と`group.markers`パラメータを加えた`music_prop.cluster`関数によって行われる。

```r
load('./IEmarkers.RData')
Endo <- Epith.marker
Macro <- Immune.marker
Neutro <- NULL
Podo <- NULL
IEmarkers <-list（ "C1" = Neutro、 "C2" = Podo、 "C3" = Endo、 "C4" = Macro）
Est.mouse.bulk = music_prop.cluster(bulk.eset = Mouse.bulk.eset,
                                    sc.eset = Mousesub.eset,
                                    group.markers = IEmarkers,
                                    clusters = 'cellType',
                                    group = 'clusterType',
                                    samples = 'sampleID',
                                    clusters.type = clusters.type)
```
ここで，C3やC4の複数の細胞種からなるグループに関しては，そのグループ内のそれぞれの細胞種に特徴的な遺伝子リストを作成し，与える必要がある。今回の場合はそれがIEmarkersである。選び方については作者がこの[issue](https://github.com/xuranw/MuSiC/issues/15)で触れている。

## additional
カスタムシングルセルデータセットの作り方は[ここ](https://github.com/xuranw/MuSiC/issues/2)みること。
