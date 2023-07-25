# 2023年度 統計学C-Ⅰ工（地）水曜４限 2023年7月26日 15:10~16:40
# R実習

## 1. データ解析の再現性
『再現性』とは、同一の結果が同一の実験手法・解析手法によって得られるとき、それら結果の一致の度合いの高さを示す。\
自分の解析結果を研究室内や会社内の他の人が同じ解析をする場合、エクセルなどの表計算ソフトにおけるメニュー操作やコピー＆ペーストを通して行ったデータの集計・加工・分析およびグラフ化（可視化）の作業は記録できず、結果の再現性が低い。\
一方、RやPythonといったプログラミング言語を用いた解析では、スクリプトを作成することでデータの読み込みから結果の出力まで、「手作業」を極力排除してデータ解析を自動化することができ、結果の再現性が高い。
## 2. Rについて
- Rはデータ分析や統計解析のために開発されたソフトウェアで、プログラミング言語としても十分な機能を備えている。
- プログラミング言語といってもCやJavaなどの言語よりも比較的簡単。順番に処理を記述していけば一通りの分析が可能であり、プログラミング言語としうより「分析ツール」という感覚で使用している人も多い。
- 無料で入手できる統計解析に特化したプログラミング言語で、統計解析で最も広く使われている。
- 基本的な統計解析機能が標準パッケージに含まれており、初期状態で一通りの統計分析を行うことが可能。
- 様々な分野に適した拡張パッケージが提供されており、適宜インストールして使用することが可能。
- Rは開発者のRoss Ihaka、Robert Clifford Gentlemanにより開発され、Rという名称は両者のイニシャルでもある。
- 現在は、R Development Core Teamによってメンテナンス・拡張がされている。
## 3. 使用するデータ
[独立行政法人統計センター](https://www.nstac.go.jp/)が公開しているSSDSE（教育用標準データセット：Standardized Statistical Data Set for Education）は、データサイエンス演習、統計教育用にが作成・公開している統計データ。\
今回は、2023年度版の家計調査データ（総務省統計局「家計調査」2020年（令和2年）～2022年（令和4年））を取得し、解析用データとする。\
### 使用するファイルについて
#### 1. 赤枠の『統計を活かす』をクリック
<img width="2556" alt="スクリーンショット 2023-07-25 12 54 07" src="https://github.com/yosui-nojima/statistics-C1_R_2/assets/85273234/c74c37dd-d320-48b5-a5d2-1b0e6c8e4a88">

#### 2. 『SSDSE（教育用標準データセット）』をクッリク
<img width="2559" alt="スクリーンショット 2023-07-25 12 54 16" src="https://github.com/yosui-nojima/statistics-C1_R_2/assets/85273234/181adf1e-2649-4e70-a901-f996464c2d5e">

#### 3. この演習では、『SSDSE-C-2023』のデータをクリックしてダウンロードする
<img width="1281" alt="スクリーンショット 2023-07-25 12 54 31" src="https://github.com/yosui-nojima/statistics-C1_R_2/assets/85273234/d45e187d-b3f1-4428-a5b8-7efbff7787d6">

#### 4. 任意の表計算ソフトで開くと以下の内容を含むデータを確認することができる
数値の単位は、世帯人員は『人』（小数点以下２桁）、他はすべて『円』。
データは平均値を表しており、\
2020年暦年データ ＋ 2021年暦年データ ＋ 2022年暦年データ）÷ ３\
で求められている。\
また、各食料品の項目の数値は、都道府県庁所在市別、二人以上の世帯の１世帯当たりの品目別の年間支出金額を表している。
<img width="2054" alt="スクリーンショット 2023-07-25 13 20 48" src="https://github.com/yosui-nojima/statistics-C1_R_2/assets/85273234/a26e915b-e0af-4636-88ec-4f784023de3f">

#### 5. 使用するデータの詳細
今回使用するデータの詳細（データの出典や単位など）は下記を参照すること。\
[https://www.nstac.go.jp/sys/files/kaisetsu-C-2023.pdf](https://www.nstac.go.jp/sys/files/kaisetsu-C-2023.pdf)

### エクセルファイルをR上で読み込む
エクセルファイルの読み込みはデフォルト状態のRではできないため、```openxlsx```ライブラリーをインストールする必要があります。\
また、今回はサーバーから直接R上に読み込む。（ダウンロードしたファイルは任意のダウンロードファルダに保存されている。）\
下記をR上で実行する。
```
install.packages("openxlsx")
```
R上でエクセルファイルを読み込む。
```
data <- read.xlsx("https://www.nstac.go.jp/sys/files/SSDSE-C-2023.xlsx", colNames = T) #ファイルの読み込み
colnames(data) <- data[1,] #列名の指定
row.names(data) <- data[,2] #行明の指定
data <- data[-1,-c(1:5)] #不要な行・列の削除
```

## 4. 仮説検定のR実装
今回は仮説検定のうち、F検定とt検定について説明します。
### 使用するデータ
読み込んだ家計調査データは、当該期間（2020年（令和2年）～2022年（令和4年））の１世帯あたりの年間支出金額の平均値を示している。\
今回は、『まぐろ』の年間支出金額の全国平均値と『さけ』の年間支出金額の全国平均値を使用する。
本来はこれらのデータについて確率正規プロットなどで、データが正規分布に従っているかどうかを確認するが、今回は正規分布に従っていると仮定して検定を行う。
下記を実行して使用するデータを```data```オブジェクトから抽出し、それぞれ```maguro```と```sake```というオブジェクトに格納する。
```
maguro <- as.numeric(data[,"まぐろ"])
sake <- as.numeric(data[,"さけ"])
```
```as.numeric()```関数は数値データとしてオブジェクトに出力するための関数。（```as.numeric()```関数を使わずに出力すると、文字列として出力されてしまうため。）
### F検定
まずはF検定で等分散かそうでないかを検定する。\
下記を実行する。
```
var.test(maguro, sake, alternative = "two.sided")
```
以下の結果が出力される。\


t検定は```t.test()```関数を使って実行可能。\
### Studentのt検定
