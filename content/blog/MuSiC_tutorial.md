---
title: "MuSiCのチュートリアルを動かす(pancreas)"
date: 2020-03-30T17:57:23+09:00
---

# MuSiC 
scRNA-seqのデータからbulkのRNA-seqのdeconvolutionを行い，セルタイプがどれだけいるかを推定するツール。 
Nature Communicationに2019に報告された。

# tutorialを実際に動かしてみる(pancreas) 
基本的にはチュートリアルに書いてある通りに動かせば全てうまく行く。
この記事はpancreasのチュートリアル。免疫細胞に対してのみやっている。

## インストール 
githubから直接インストールすることを推奨している。
```r
install.packages('devtools')
devtools::install_github('xuranw/MuSiC')
library(MuSiC)
```

## データの準備　
RDS形式でデータが配布されているので読み込むだけ，
膵臓のバルクデータ(89 subjects)とsingle cellデータを読み込む。どちらもカウントデータとして処理されている。BMIなどそれ以外の項目も含んでいる。
確認できていないが，下流の解析的にsingle cellのデータはセルタイプまで推測されているデータを与えないといけない模様。そもそも自分で解析するときはこのデータ型を作るとが重要そう。

```r
GSE50244.bulk.eset = readRDS('./GSE50244bulkeset.rds')
EMTAB.eset = readRDS('EMTABesethealthy.rds')
```
`EMTAB.eset`の中身を確認
```
ExpressionSet (storageMode: lockedEnvironment)
assayData: 25453 features, 1097 samples 
  element names: exprs 
protocolData: none
phenoData
  sampleNames: AZ_A10 AZ_A11 ... HP1509101_P9 (1097 total)
  varLabels: sampleID SubjectName cellTypeID cellType
  varMetadata: labelDescription
featureData: none
experimentData: use 'experimentData(object)'
Annotation: 
```
`EMTAB.eset@phenoData`の中身は
```
An object of class 'AnnotatedDataFrame'
  sampleNames: AZ_A10 AZ_A11 ... HP1509101_P9 (1097 total)
  varLabels: sampleID SubjectName cellTypeID cellType
  varMetadata: labelDescription
```


## 推測 
今回はメインであり，isletの90％を構成する6種類の細胞(alpha,beta,delta,gamma,acinar,ductal)が，89 subjectsのbulkデータそれぞれにどれだけいるかを`music_porp`関数で推測する。

```r
library(xbioc)

# Estimate cell type proportions
Est.prop.GSE50244 = music_prop(bulk.eset = GSE50244.bulk.eset, sc.eset = EMTAB.eset, clusters = 'cellType',
                               samples = 'sampleID', select.ct = c('alpha', 'beta', 'delta', 'gamma',
                                                                   'acinar', 'ductal'), verbose = F)
```
`Est.prop.GSE50244`には以下のデータが含まれる。89 subjectsそれぞれに推測を行っているのでmatrixが5つ出てくる。
```
Est.prop.weighted: バルクの各サンプルに対して，MuSiCにより推定された細胞タイプの割合を記したデータフレーム 
Est.prop.allgene: バルクの各サンプルに対して，NNLS(非負最小二乗法)により推定された細胞タイプの割合を記したデータフレーム 
Weight.gene: matrix, MuSiC estimated weight for each gene, genes by subjects; MuSiCが推定した各遺伝子の重みmatrix，各サンプルごとに
r.squared.full:  vector of R squared from MuSiC estimated proportions for each subject(よく分からなかった。ベクトル)
Var.prop: サンプル*細胞タイプのmatrix。MuSiC estimateの分散を示すらしい。
```

## 結果の作図　
2箇所修正が必要だった。
1. reshape2とcowplotをロードする必要がある。
2. `m.prop.GSE50244`を定義するところの名前が間違っていた。`GSE50244.EMTAB.prop`から`Est.prop.GSE50244`に変更　

```r
if (!require("reshape2")) install.packages("reshape2")
if (!require("cowplot")) install.packages("cowplot")

library(reshape2)
library(cowplot) #plot_gridに必要

jitter.fig = Jitter_Est(list(data.matrix(Est.prop.GSE50244$Est.prop.weighted),
                             data.matrix(Est.prop.GSE50244$Est.prop.allgene)),
                        method.name = c('MuSiC', 'NNLS'), title = 'Jitter plot of Est Proportions')

# A more sophisticated jitter plot is provided as below. We seperated the T2D subjects and normal
#subjects by their HbA1c levels.

#GSE50244.EMTAB.prop→Est.prop.GSE50244
m.prop.GSE50244 = rbind(melt(Est.prop.GSE50244$Est.prop.weighted),
                        melt(Est.prop.GSE50244$Est.prop.allgene))

colnames(m.prop.GSE50244) = c('Sub', 'CellType', 'Prop')
m.prop.GSE50244$CellType = factor(m.prop.GSE50244$CellType, levels = c('alpha', 'beta', 'delta', 'gamma', 'acinar', 'ductal'))
m.prop.GSE50244$Method = factor(rep(c('MuSiC', 'NNLS'), each = 89*6), levels = c('MuSiC', 'NNLS'))
m.prop.GSE50244$HbA1c = rep(GSE50244.bulk.eset$hba1c, 2*6)
m.prop.GSE50244 = m.prop.GSE50244[!is.na(m.prop.GSE50244$HbA1c), ]
m.prop.GSE50244$Disease = factor(c('Normal', 'T2D')[(m.prop.GSE50244$HbA1c > 6.5)+1], levels = c('Normal', 'T2D'))
m.prop.GSE50244$D = (m.prop.GSE50244$Disease == 'T2D')/5
m.prop.GSE50244 = rbind(subset(m.prop.GSE50244, Disease == 'Normal'), subset(m.prop.GSE50244, Disease != 'Normal'))

jitter.new = ggplot(m.prop.GSE50244, aes(Method, Prop)) +
  geom_point(aes(fill = Method, color = Disease, stroke = D, shape = Disease),
             size = 2, alpha = 0.7, position = position_jitter(width=0.25, height=0)) +
  facet_wrap(~ CellType, scales = 'free') + scale_colour_manual( values = c('white', "gray20")) +
  scale_shape_manual(values = c(21, 24))+ theme_minimal()

plot_grid(jitter.fig, jitter.new, labels = 'auto')
```

![jitter](https://xuranw.github.io/MuSiC/articles/images/GSE50244jitter.jpg)


#### プロットの解釈 
T2Dが進むとbeta細胞が減少するらしい。このfigからだとそうとも言えないように見えてしまうが。。。 

## T2D疾患を判別 
T2Dのもう一つの特徴として，HbA1c(hemoglobin A1c)が増加することがあり，主に，検査のターゲットとなっているらしい。
今回MuSiCとNNLSにより判別したbeta cellの割合と，HbA1cの実際の発現量を同時に確認して確からしさを調べる。 

```r
# Create dataframe for beta cell proportions and HbA1c levels
m.prop.ana = data.frame(pData(GSE50244.bulk.eset)[rep(1:89, 2), c('age', 'bmi', 'hba1c', 'gender')],
                        ct.prop = c(Est.prop.GSE50244$Est.prop.weighted[, 2], 
                                    Est.prop.GSE50244$Est.prop.allgene[, 2]), 
                        Method = factor(rep(c('MuSiC', 'NNLS'), each = 89), 
                        levels = c('MuSiC', 'NNLS')))
colnames(m.prop.ana)[1:4] = c('Age', 'BMI', 'HbA1c', 'Gender')
m.prop.ana = subset(m.prop.ana, !is.na(HbA1c))
m.prop.ana$Disease = factor( c('Normal', 'T2D')[(m.prop.ana$HbA1c > 6.5) + 1], c('Normal', 'T2D') )
m.prop.ana$D = (m.prop.ana$Disease == 'T2D')/5

ggplot(m.prop.ana, aes(HbA1c, ct.prop)) + 
  geom_smooth(method = 'lm',  se = FALSE, col = 'black', lwd = 0.25) +
  geom_point(aes(fill = Method, color = Disease, stroke = D, shape = Disease), size = 2, alpha = 0.7) +  facet_wrap(~ Method) + 
  ggtitle('HbA1c vs Beta Cell Type Proportion') + theme_minimal() + 
  scale_colour_manual( values = c('white', "gray20")) + 
  scale_shape_manual(values = c(21, 24))
```

![fig](https://xuranw.github.io/MuSiC/articles/images/GSE50244beta.jpg) 

確かにT2Dにおけるbeta細胞の割合の減少はNNLSと比較してMuSiCでわかりやすく見られるは見られる。　
この後，年齢やBMIを考慮した上で，実際にbeta細胞とHbA1cに非の相関関係があることを統計的に評価している。 
```r
lm.beta.MuSiC = lm(ct.prop ~ HbA1c + Age + BMI + Gender, data = subset(m.prop.ana, Method == 'MuSiC'))
lm.beta.NNLS = lm(ct.prop ~ HbA1c + Age + BMI + Gender, data = subset(m.prop.ana, Method == 'NNLS'))
summary(lm.beta.MuSiC)
#
#Call:
#lm(formula = ct.prop ~ HbA1c + Age + BMI + Gender, data = subset(m.prop.ana,
#    Method == "MuSiC"))
#
#Residuals:
#     Min       1Q   Median       3Q      Max
#-0.27768 -0.13186 -0.01096  0.10661  0.35790
#
#Coefficients:
#              Estimate Std. Error t value Pr(>|t|)
#(Intercept)   0.877022   0.190276   4.609 1.71e-05 ***
#HbA1c        -0.061396   0.025403  -2.417   0.0182 *
#Age           0.002639   0.001772   1.489   0.1409
#BMI          -0.013620   0.007276  -1.872   0.0653 .
#GenderFemale -0.079874   0.039274  -2.034   0.0457 *
#---
#Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
#
#Residual standard error: 0.167 on 72 degrees of freedom
#Multiple R-squared:  0.2439,   Adjusted R-squared:  0.2019
#F-statistic: 5.806 on 4 and 72 DF,  p-value: 0.0004166

summary(lm.beta.NNLS)
#
#Call:
#lm(formula = ct.prop ~ HbA1c + Age + BMI + Gender, data = subset(m.prop.ana,
#    Method == "NNLS"))
#
#Residuals:
#     Min       1Q   Median       3Q      Max
#-0.04671 -0.02918 -0.01795  0.01394  0.19362
#
#Coefficients:
#               Estimate Std. Error t value Pr(>|t|)
#(Intercept)   0.0950960  0.0546717   1.739   0.0862 .
#HbA1c        -0.0093214  0.0072991  -1.277   0.2057
#Age           0.0005268  0.0005093   1.035   0.3044
#BMI          -0.0015116  0.0020906  -0.723   0.4720
#GenderFemale -0.0037650  0.0112844  -0.334   0.7396
#---
#Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
#
#Residual standard error: 0.04799 on 72 degrees of freedom
#Multiple R-squared:  0.0574,   Adjusted R-squared:  0.005028
#F-statistic: 1.096 on 4 and 72 DF,  p-value: 0.3651
```

# benchmark
single cellデータから人工的にbulkデータを作成しbenchmarkする。`bulk_construct`データで自分でもできるのはいいところ。
ここでは，実際のbulkのcontentsを推測するのに使ったsingle cellと別の実験で取得したsingle cell RNA-seqデータから仮想のbulkデータを作成し，仮想データと実際のデータでbenchmarkする。

#### 1. 4種類の細胞だけからなる人工bulkデータの作成
ここではpancreasのsingle cellデータからbulkデータを作成している。

```r
XinT2D.construct.full = bulk_construct(XinT2D.eset,
                                       clusters = 'cellType',
                                        samples = 'SubjectName')
```
`XinT2D.construct.full`は人工bulkデータの`Bulk.count`と実際の細胞種カウントmatrixの`num.real`を含む。

#### 2. 人工bulkデータの実際の細胞種をカウント
人工bulkデータを生成したsingle cell RNA-seqデータと別のsingle cellデータを，推測用のデータとして使用。

```r
XinT2D.construct.full$prop.real = relative.ab(XinT2D.construct.full$num.real, by.col = FALSE)
head(XinT2D.construct.full$prop.real)
```
結果は
```
#              alpha      beta      delta      gamma
#Non T2D 1 0.7162162 0.1756757 0.06756757 0.04054054
#Non T2D 2 0.1666667 0.5416667 0.08333333 0.20833333
#Non T2D 3 0.6428571 0.2380952 0.07142857 0.04761905
#Non T2D 4 0.5185185 0.3703704 0.00000000 0.11111111
#Non T2D 5 0.4423077 0.4230769 0.09615385 0.03846154
#Non T2D 6 0.7500000 0.1458333 0.08333333 0.02083333
```

#### 3. 人工データの推測
```r
Est.prop.Xin = music_prop(bulk.eset = XinT2D.construct.full$Bulk.counts, sc.eset = EMTAB.eset,
                          clusters = 'cellType', samples = 'sampleID',
                          select.ct = c('alpha', 'beta', 'delta', 'gamma'))
```
4種類だけだから，`music_prop`でいい。

#### 4. 定量的に比較
`Eval_multi`関数により，人工データとリアルbulkデータの細胞割合を以下の項目で評価し比較する。
```
root-mean-squared deviation (RMSD);
mean absolute difference (mAD);
Pearson’s correlation (R).
```
`Prop_comp_multi`, `Abs_diff_multi`,  `Scatter_multi`で可視化することができる。
```r
# Estimation evaluation

Eval_multi(prop.real = data.matrix(XinT2D.construct.full$prop.real),
           prop.est = list(data.matrix(Est.prop.Xin$Est.prop.weighted),
                           data.matrix(Est.prop.Xin$Est.prop.allgene)),
           method.name = c('MuSiC', 'NNLS'))

#           RMSD     mAD      R
#MuSiC   0.09881 0.06357 0.9378
#NNLS    0.17161 0.11749 0.8159

library(cowplot)
prop.comp.fig = Prop_comp_multi(prop.real = data.matrix(XinT2D.construct.full$prop.real),
                                prop.est = list(data.matrix(Est.prop.Xin$Est.prop.weighted),
                                                data.matrix(Est.prop.Xin$Est.prop.allgene)),
                                method.name = c('MuSiC', 'NNLS'),
                                title = 'Heatmap of Real and Est. Prop' )

abs.diff.fig = Abs_diff_multi(prop.real = data.matrix(XinT2D.construct.full$prop.real),
                              prop.est = list(data.matrix(Est.prop.Xin$Est.prop.weighted),
                                              data.matrix(Est.prop.Xin$Est.prop.allgene)),
                              method.name = c('MuSiC', 'NNLS'),
                              title = 'Abs.Diff between Real and Est. Prop' )

plot_grid(prop.comp.fig, abs.diff.fig, labels = "auto", rel_widths = c(4,3))
```
![result](https://xuranw.github.io/MuSiC/articles/images/benchmark.jpg)


