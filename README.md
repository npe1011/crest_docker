# Windowsユーザーのためのcrestによるコンフォメーション探索

## はじめに

Grimmeらのcrest [https://github.com/grimme-lab/crest] は、tight-binding近似による半経験的計算手法であるxtbを利用してコンフォメーション探索をすることができます。コンフォメーション探索に関して有機化学屋がよく使っているソフトで有名なところだとCONFLEXなどがありますが、まあまあ高かったりMMベースなので錯体とかはうまく行かないなどの問題もあります。xtb は多くの金属錯体の構造についてもかなり妥当な結果を与えるため、非常に便利です。ただLinuxでしか使えない（諸々の依存性のせいでWindowsでビルドするのが難しそうです）のとGUIもついてないので、慣れていない有機化学屋的には少しハードルが高くなります。ここでは、Docker Desktop + WSL2 を利用します。

## Docker の準備

Docker + WSL2 を使うと Windows上にLinuxの仮想環境的なものを手軽に用意できます。これを説明するとめんどくさいので、適当にググってインストールしてください。「WSL2 Docker」 などで検索すると記事が山程出てきます。

## crest用のコンテナ（環境）の準備

このリポジトリのdockerfileディレクトリ内のファイル2つを適当な場所に保存してください。ここでは `D:\dockerfiles\crest` 内に置いたとします。
コマンドラインを立ち上げて `D:\dockerfiles` に移動します。エクスプローラーで `D:\dockerfiles` フォルダを開いて、アドレスバーに `cmd` と入れてエンターすると楽。

```
docker build -t crest crest
```

と入れてイメージ（仮想環境の雛形みたいなもの）を作成します。ちょっと時間がかかります。無事終了したら、

```
docker images
```

とすると 

```
REPOSITORY       TAG       IMAGE ID       CREATED          SIZE
crest            latest    d9ffafa024a6   1 minutes ago   226MB
```

みたいなに表示されます。そうしたらこのイメージから実際の環境を立ち上げますが、そのまま立ち上げると、Windowsからファイルのやり取りがしにくいので、先に Windows上の作業フォルダを決めて作成しておき、そこと仮想環境（コンテナ）のディレクトリを共有させます。ここではWindows上の作業ディレクトリは `D:\work` とします。

```
docker run -it -v D:\work:/work --name crest crest
```
と入力します。 `-v -v D:\work:/work` の部分で、 Windows上の `D:\work` と仮想環境（コンテナ）であるLinux内の `/work` ディレクトリをつなげています。
コマンドラインに

```
root@xxxxxxxx: /work#
```
のように表示されれば、Linux環境が立ち上がり、今はその中で作業していることになります。このコンテナには既に必要なプログラムや環境が設定されています。


## 初期構造ファイルの準備とcrestの実行

初期構造ファイルは xyzファイルで準備します。xyzファイルはテキスト形式で下記のフォーマットです。先頭に原子数を整数値で入れて、コメント行は何でも（空行でも）よく、そのうしろに原子数分の行数の座標情報（単位：オングストローム）が続きます。ここではこれを `init.xyz` とします。注意として `struc.xyz` というファイル名は使わないでください（crestの出力ファイル名とぶつかるので）。

```
原子数
コメント行
C 0.000 0.000 1.000
H 1.000 0.000 0.000
以下原子座標情報
```

これを `D:\work\init.xyz` とします。

そうしたらコンテナに入った状態のコマンドラインから crestを実行しますが、そのまま Windowsとつながっているディレクトリで実行してそこに中間ファイル等を出力していってしまうと動作が遅くなります（これはWindows+WSL2のファイルシステム特有の問題です）。なので一旦インプットファイルをコンテナ内の一時ディレクトリにコピーして、そこでプログラムを実行し、結果をコピーして戻すというのが必要になります。これを毎回やるのは面倒なの、実行するスクリプトが入っています（ `cresturun` で実行できます）。実際にやるべきことは、下記のように実行するだけです。

```
crestrun init.xyz OPTIONS
```

OPTIONSはcrest実行時の諸々の設定です。詳しくは [https://xtb-docs.readthedocs.io/en/latest/crestcmd.html] を読みましょう。とりあえず設定すべきは、並列計算数、電荷、スピン、エネルギーの閾値くらいかと思います（他はデフォルトで順当なことをしてくれます）。
***注意：必ずオプションは初期構造ファイルの後ろにつけてください。*** 

例えば下の例では、並列数4、電荷1、不対電子の数（aスピン-bスピンの数）1、エネルギー閾値 10 kcal/mol になります。xtb や crest ではスピン多重度周りの指定が Gaussianなどと異なるので注意。（閉殻なら `--uhf 0`、二重項の普通のラジカルなら `--uhf 1`、三重項なら `--uhf 2` となります）
`--T` で指定できる並列計算数（スレッド数）は、使うCPUコアの数を指定しましょう。

```
crestrun init.xyz --T 4 --chrg 1 --uhf 1 --ewin 10
```

実行すると画面にログが表示されるとともに、init_crest.log に保存されます（この時点ではそれ以外の出力ファイルはwindows上からは見えないコンテナ内の `/tmp` 以下に保存されていきます）。MMと比べて時間はかかりますので気長に待ちましょう。

## 結果の見方

計算が終わると、元のフォルダ (D:\work == /work) にファイルが色々出力されます。重要なのは `crest_conformers.xyz` です。これにコンフォメーションの構造とエネルギーが全てまとまっています。xyz形式で、エネルギーの低い安定な順番にならんでいて、各コメント行にエネルギーの値（単位：hartrees）が書いてあります。 `crest.energies` を開くとエネルギーの相対値が kcal/mol 単位で書いてあります。 xyz 形式は Jmol などで開いて眺めるとよいです。

## 拘束条件付きのコンフォメーション探索

特定の原子や構造を固定したコンフォメーション探索もできます。これは例えば遷移状態構造のコンフォメーション探索に利用できます。
[https://xtb-docs.readthedocs.io/en/latest/crestxmpl.html#conformers-of-transition-states] が参考になります。
例えば 30原子の分子で原子3,5,9,10,11 を拘束するには下記のようなファイルを作ります。ここではこれを `const.txt` として、 `init.xyz` と同じところに置きます。
`$constrain` の下の部分では拘束する原子の番号、その力、および初期構造ファイルを入れます。 `$metadyn` の下には動かす原子（拘束するやつ以外全部）の番号を書きます。

```
$constrain
  atoms: 3,5,9-11
  force constant=1.0
  reference=init.xyz
$metadyn
  atoms: 1,2,4,6-8,12-30
$end
```

実行するときには次のようにしてこのファイルを読み込ませます（内部ディレクトリへのコピーも勝手にやります）。

```
crestrun init.xyz --cinp const.txt --T 4 --chrg 1 --uhf 0 --ewin 5
```

遷移状態構造のコンフォメーション探索をするときは

1. 置換基とかをざっくり除いた最小限のモデル構造の遷移状態構造を最適化 (Gaussianなどで）。
2. そこに Gaussview などで置換基を置いていく（よほどじゃなければある程度適当でも多分大丈夫）。
3. ファイルをxyz形式にする（GaussviewやChemd3Dは対応してないのでテキストエディタで手作業）。
4. 拘束する原子を決めて条件ファイルを作り、拘束条件ありのコンフォメーション探索をおこなう（上記）。
5. 結果を睨んで安定なの、よさそうなのを選ぶ（xtbのエネルギーと最終的にDFT計算とかまでやったエネルギーが逆転するのはよくあるので、単純にエネルギーで切ると危険）
6. 選んだ構造を初期構造にしてDFTなどで遷移状態構造最適化 (GaussianならOPT=TS)。
7. 直接やってうまく鞍点にいかないときは、DFTレベルで、一度拘束条件をつけて（Gaussianならmodredundantを使う）OPTしたあとにOPT=TSをやるとよいです。4で指定した原子同士の距離などをfreezeするとよいでしょう。

のようにすると、それっぽい遷移状態構造のコンフォメーションが得られます。
 
