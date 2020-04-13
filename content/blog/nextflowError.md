---
title: "NextflowError"
date: 2020-04-10T23:18:53+09:00
---

## Nextflow操作で生じたエラー 

### 1.進行せずに終了する

```bash
~/bin/nextflow run nextflow/qc_for_R.nf -c run.config -resume -with-report log.04.qc_for_R.h
Picked up JAVA_TOOL_OPTIONS: -XX:+UseSerialGC -XX:ParallelGCThreads=2
N E X T F L O W  ~  version 19.10.0
Launching `nextflow/qc_for_R.nf` [drunk_mirzakhani] - revision: 62c0b2ec90
[-        ] process > qc_alignment    -
[-        ] process > qc_distribution -
[-        ] process > qc_insert_size  -
[-        ] process > qc_coverage     -
```

上記のように，%が何も進行せずに急にスクリプトが終了してしまうエラーが生じた。このエラーは以前にも遭遇したことがある気がするが，当時はまとめていなかったので，0から解決しないといけなかった。まとめておくことはだいじ。

このようなエラーは基本的に，channelの指定がおかしいことによって起きる。

channelの中身を適宜`view`コマンドや，`println`コマンドで出力し，処理を行うべき。

### 2.1コマンドで異なる箇所にある同じidファイルを参照する

出力結果のわかりやすさのため，処理途中で生成されるファイルを，サンプルごとではなく，ファイルの種類ごとでまとめた。
この際，途中で出力された複数のサンプルを後半のスクリプトで参照しようとする際に，channelを`combine`で結合するだけではうまくいかないので注意が必要。
結論としては，
`join`コマンドを使用する，もしくは`sortedList`を使用することで解決する。
`join`コマンドの方がシンプルにかけるのでこちらを採用した。`join`コマンドで結合するchannelに，共通のidを持たせることで，同じサンプルのファイルを同時に処理できるようにした。
