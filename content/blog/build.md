---
title: "tkinterとpyinstallerでアプリ作成"
date: 2020-04-04T09:23:07+09:00
---

# tkinterでアプリ作成
打ち抜き箇所抽出アプリを作成した際のログ。tkinterというパッケージを利用してアプリ自体を作成。

## 1. 環境設定
自分のローカルPCで作成した環境で操作する。
```
conda activate app_make
```
この際，環境にanacondaを含めると出力されるファイルのサイズが300MBを超えてしまうので気をつける。自分の場合は，pythonだけが入った空っぽの仮想環境を作り，必要パッケージを都度`pip`でインストールした。
巷では`numpy`加える場合はなんたらという話が多いが，基本的に最新版の`pyinstaller`ではそれほどエラーが発生しない。`pip`で入れる際には，依存パッケージが存在するように気をつけて行う。
```
#依存パッケージを含むように
# pip install opencv-contrib-python #エラーがでた
pip install opencv-contrib-python==3.4.5.20
pip install numpy
pip install pandas
pip install
pip install matplotlib
```
このようにanacondaを避けて頑張っても最終的なアプリの容量は93.1MBになってしまった(まだまし)。
ここで，`opencv`のデフォルトバージョンである4>では，
```
ImportError: dlopen(/var/folders/lj/x6k9qjtn5rn77ztp7ttlj46c0000gn/T/_MEIxZRfvC/cv2/cv2.cpython-37m-darwin.so, 2): Library not loaded: @loader_path/libpng16.16.dylib
  Referenced from: /private/var/folders/lj/x6k9qjtn5rn77ztp7ttlj46c0000gn/T/_MEIxZRfvC/libfreetype.6.dylib
  Reason: Incompatible library version: libfreetype.6.dylib requires version 54.0.0 or later, but libpng16.16.dylib provides version 38.0.0
```
というエラーが出てしまうので，`opencv`はバージョン3を指定してインストールする必要があった。

## 2. tkinter
```
python run.py
```
でアプリのウィンドウが起動されるかを確認する。
この状態でデバッグを行い，動作確認がすんでから`pyinstaller`を行う。

## 3. pyinstaller
pyinstallerでビルドして配布可能な形にする。

### 3.1. 試してみる
```
pyinstaller extractpoint.py --onefile --noconsole
```
これだけだとファイルは出力されるが実行することができなかった。specファイルの書き換えなどが必要。
ここで出力されたファイルを起動するとパソコンが使いものにならなくなったので注意。
エラー内容の確認のためには，アプリの中にあるUNIX実行ファイルのpathをコンソールに直接入力することで行うのが良い。

## 2.2. .specファイルの書き換え
`pkg_resources.py2_warn`がないと怒られたので，`.spec`ファイルの`hiddenimports=[]`を`hiddenimports=['pkg_resources.py2_warn']`に書き換える。
```
a = Analysis(['ExtPointer.py'],
             pathex=['/Users/wagatsuma/Documents/Extract_sampling_point'],
             binaries=[],
             datas=[],
             hiddenimports=['pkg_resources.py2_warn'],
             hookspath=[],
             runtime_hooks=[],
             excludes=[],
             win_no_prefer_redirects=False,
             win_private_assemblies=False,
             cipher=block_cipher,
             noarchive=False)
```　

## 2.3.　動かす　
ツールのバージョンを正確にすることで，コンソールから動かすと起動できるようになったが，何故かアプリから起動すると，開いてすぐに落ちてしまうようになってしまった。実行ファイル化の際に，`tornado`というパッケージがないと怒られていたのでインストール
```
pip install tornado
```
同様にコンソールから動かすと起動できるが，アプリアイコンから起動するのはすぐ落ちてしまう。何故なのか。。

#### 2.3.2. 環境
```
331 INFO: PyInstaller: 3.6
331 INFO: Python: 3.7.6 (conda)
```

## 5. エラー
おそらく，macのアプリを起動する際に，公証を受けているなどの認証が必要であることが原因のようである。以下の記事を参考にすること。
[https://techracho.bpsinc.jp/yosuke2/2019_07_31/78252]
[https://github.com/pyinstaller/pyinstaller/issues/4629]
[https://forest.watch.impress.co.jp/docs/news/1226601.html]


コンソールアプリからエラーを確認してみた。主に現れていたのは以下のようなエラー

```
CGBitmapContextCreateImage: invalid context 0x0. If you want to see the backtrace, please set CG_CONTEXT_SHOW_BACKTRACE environmental variable.
```

```
FAIL: PID[4856]: SecTaskCopySigningIdentifier(): [22: Invalid argument]
```

```
Death (BackgroundOnly) of untracked active application: <private>
```

```
Failed to copy signing info for 4794, responsible for file:///Users/wagatsuma/Documents/test_extract/dist/ExtPointer.app/Contents/MacOS/ExtPointer: #-67062: Error Domain=NSOSStatusErrorDomain Code=-67062 "(null)"
```

## 6. バイナリファイルに署名する　
macの`名前なんだっけ`を突破するために，署名を行う。[この記事](https://qiita.com/zeriyoshi/items/78b4c7227217e2eac536)を参考にコード署名用の証明書を作成しする。

```
codesign --force --verify --verbose --sign "extract_position_apps" "ExtPointer.app" --deep --options runtime --timestamp
```
結果の出力が
```
ExtPointer.app: signed app bundle with Mach-O thin (x86_64) [ExtPointer]
```

以下のコマンドで署名を確認すると，
```
codesign -vvv --deep --strict ./dist/ExtPointer.app
```
上記署名前の出力が以下のものから，
```
./dist/ExtPointer.app: code object is not signed at all
```
このように変化した。
```
./dist/ExtPointer.app: valid on disk
./dist/ExtPointer.app: satisfies its Designated Requirement
```

ただ，結果的にダウンしてしまった。コンソールから起動した場合も動かなくなってしまい下記のエラー
```
[11300] Error loading Python lib '/var/folders/lj/x6k9qjtn5rn77ztp7ttlj46c0000gn/T/_MEIzXzJl6/libpython3.7.dylib': dlopen: dlopen(/var/folders/lj/x6k9qjtn5rn77ztp7ttlj46c0000gn/T/_MEIzXzJl6/libpython3.7.dylib, 10): no suitable image found.  Did find:
	/var/folders/lj/x6k9qjtn5rn77ztp7ttlj46c0000gn/T/_MEIzXzJl6/libpython3.7.dylib: code signature in (/var/folders/lj/x6k9qjtn5rn77ztp7ttlj46c0000gn/T/_MEIzXzJl6/libpython3.7.dylib) not valid for use in process using Library Validation: mapped file has no cdhash, completely unsigned? Code has to be at least ad-hoc signed.
	/var/folders/lj/x6k9qjtn5rn77ztp7ttlj46c0000gn/T/_MEIzXzJl6/libpython3.7.dylib: stat() failed with errno=3
  ```

## USBで共有　
USBでアプリファイルを共有し，`app/contents/macOS/`自体を起動することによって他のmacbookでも動作することは確認ずみ。


