---
title: "Atac-seq pipeline 作成(バージョン1)"
date: 2020-03-30T01:30:13+09:00
draft:true
---

# 方針
このパイプラインを作成したときのデータのクオリティがあまりにも低かったため，大枠を作成するまでのログを残した。
今後データを見て，最適化していく。

### リファレンスの準備
リファレンス配列は通常，ensebmleかUCSCのゲノムブラウザーからダウンロードする。今回はFASTAファイルをUCSC genome browserから，gtfファイルをensembleから持ってきた。gtfファイルはensembleの方が情報量が多いので詳細解析をする時はgtfの方が良いらしい。　
注意しないといけないのが，ensembleのgene idは単なる数字なのに対してUCSCではchr1のように異なる。それを揃えるためensembleから持ってきたgtfファイルの先頭にchrを加えていく。
```sh
grep -v \# Homo_sapiens.GRCh38.98.gtf | awk '{print "chr" $0 }' > Homo_sapiens.GRCh38.98.addchr.gtf
```

### 仮想環境の準備
`conda create -n atac_seq python3.7.1 numpy`
でatac_seq用の仮想環境を作成　
`conda install -c bioconda macs2`
で必要なmacs2をインストールしておく。　

# 手順
### クリーニング　
`trimmomatic`を利用してアダプター配列や末端の配列を除去する。　
```sh
java -jar $trimmomatic \
    PE -phred33 -threads $cpu \
    -trimlog $qc_dir/${id}_log.txt \
    ${id}_1.fastq \
    ${id}_2.fastq \
    $cln1 $uncln1 \
    $cln2 $uncln2 \
    ILLUMINACLIP:$adapter:2:30:10 \
    LEADING:20 \
    TRAILING:20 \
    SLIDINGWINDOW:4:15 \
    MINLEN:36
```


### qc
`fastq`を利用してqcファイルの作成。この際，`trimmomatic`を利用してクリーニングした後の配列と，していない配列両方に対してqcreportを作成。その結果を`../result/qcreport`ディレクトリに保存している。　
パイプラインと別に後から，`../result/qcreport`ディレクトリで，
```sh
multiqc .
```
を用いて１つのqcレポートにまとめる。 ~~その予定だが，`networkx`パッケージを見つけることができないというエラーを吐かれて実行できない。環境が壊れてしまっているのか，改めて入れ直して実行してもうまくいかない。~~　
仕方なく，`pip install networkx`したらうまくいった。

### mapping
`STAR`を用いてmappingする。 この際`STAR`は最低32GBのメモリを確保しておくことが推奨されているそうなのでそこに気をつける。
```sh
$STAR --runThredN $cpu \
    --genomeDir $star_index \
    --readFileIn $cln1 $cln2 \
    --outSAMtype BAM SortedByCoordinate \
    --outFileNamePrefix ${id}.
```

### 重複している配列の除去，ミトコンドリア配列の除去
ピークコールする前に`picard`を用いて重複配列の除去を行う。MultiQCによるQCレポートを見るとわかるが，今回はduplicateが異常に多かった(約95%)，そのため，この操作の際にファイル容量が大幅に減少した。 　

```sh
java -jar $picard MarkDuplicates \
    I=${id}.bamAligned.sortedByCoord.out.bam \
    M=${id}_dupl.bam \
    O=$cln_bam
```

さらに，ミトコンドリアとY染色体にあったっているリードを除去する。ただし，今回はミトコンドリアに当たっているリードは0だった。　
`samtools`を用いてそれらのクリーニングを行なった後に，のちの解析のため，sortとindex作成を行なっておく。

```sh
$samtools view -h -F4 $ddup_bam | grep -v chrM | $samtools view -b > $MT_bam
$samtools view -h -F4 $MT_bam | grep -v chrY | $samtools view -b > $cln_bam
$samtools sort -o $cln_bam $cln_bam
$samtools index $cln_bam
```

### ピークコール　
`macs2`を用いてピークコールする。
```sh
macs2 callpeak \
    -t $cln_bam \
    -n $id \
    -f BAMPE \
    -g $species \
    --nomodel \
    --nolambda \
    --keep-dup all \
    --call-summits
```

### ピークのアノテーション付け
HOMERの`annotatePeaks.pl`を用いて，MACS2によって得られたピークにアノテーション付けを行う。
```sh
$annotate ${id}_peaks.narrowPeak hg38 > $an_peak
```
間違っている気もするので，中身の確認をしっかりと行う必要がある。

### ピークマージと比較
得られたピークをマージする。
```sh
cat a.bed b.bed c.bed | bedtools merge -i - > merged.bed
```
マージしたピークとリファレンスゲノムの比較を行うため，重複を調べる。この際，-fパラメータを指定しないので1塩基でも重複していたら重複とした。
`bedtools insert`で重複を確認する。どのように行なっているかは下図参照。
![bedtools insert](https://github.com/TsumaR/memo/blob/master/figure/intersect-glyph.png)

```sh
bedtools intersect -wa -a ../omni_atac/SRR5427886/SRR5427886_peaks.narrowPeak -b merged_atac5.bed > atac5_intersect.bed
```
```sh
wc -l SRR5427886_peaks.narrowPeak
```
でOmni-ATACのピーク数を確認すると79297だった。
一方で，
```sh
wc -l merged.bed
```
では，927しかピークが検出されなかった。

この結果を`uniq -u intersect_omni.bed | wc -l `で確認したところ，95のみ，


bedtools intersect -wa -a omni_atac/SRR5427886/SRR5427886_peaks.narrowPeak -b 191101_atac5/merged_atac5.bed > atac5_intersect.bed

### R，pythonでの解析
以上の操作によって得られたファイル，マージのピークファイル，アノテーションファイル(今回はマージしていないもの)，


uniq -u atac5_intersect.bed | wc -l
4214

