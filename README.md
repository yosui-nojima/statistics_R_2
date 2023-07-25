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
今回は、2023年度版の家計調査データ（総務省統計局「家計調査」2020年（令和2年）～2022年（令和4年））を取得し、解析用データとする。
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
今回は、『まぐろ』の年間支出金額と『さけ』の年間支出金額を使用する。
本来はこれらのデータについて正規確立プロットなどで、データが正規分布に従っているかどうかを確認するが、今回は正規分布に従っていると仮定して検定を行う。\
下記を実行して使用するデータを```data```オブジェクトから抽出し、それぞれ```maguro```と```sake```というオブジェクトに格納する。
```
maguro <- as.numeric(data[,"まぐろ"])
sake <- as.numeric(data[,"さけ"])
```
```as.numeric()```関数は数値データとしてオブジェクトに出力するための関数。（```as.numeric()```関数を使わずに出力すると、文字列として出力されてしまうため。）
### データの可視化
『まぐろ』の全国の年間支出金額の平均と『さけ』全国の年間支出金額の平均を棒グラフで可視化する。\
下記を実行する。
```
both <- cbind(maguro, sake)
xm <- apply(both, 2, mean)
xs <- apply(both, 2, sd)
# bar chart
b <- barplot(xm, xlab = "Fish", ylab = "Yen", ylim = c(0, max(xm + xs)))
# error bar
arrows(b, xm - xs, b, xm + xs, code = 3, lwd = 1, angle = 90, length = 0.1)
```
下図が出力される。\
![Rplot09](https://github.com/yosui-nojima/statistics-C1_R_2/assets/85273234/702415b8-37b0-4498-a165-b408b3052d89)

棒の頂点が平均値を示す。\
標準偏差はエラーバー（棒の上についているひげの部分）で示している。\
次に、このグラフで見られる差が本質的な差かどうかを仮説検定により調査する。

### F検定
まずはF検定で等分散かそうでないかを検定する。\
下記を実行する。
```
var.test(maguro, sake, alternative = "two.sided")
```
以下の結果が出力される。\
<img width="435" alt="スクリーンショット 2023-07-25 13 50 03" src="https://github.com/yosui-nojima/statistics-C1_R_2/assets/85273234/f54cd568-8a58-4cc5-992b-ee559015b040">\
```p-value = 1.325e-12```の部分が*P*値を意味する。\
有意水準αを5%とすると、*P*<αのため帰無仮説(H0:σ1=σ2)は棄却される。つまり、『まぐろ』の全国の年間支出金額の分散と『さけ』の全国の年間支出金額の分散は異なると言える。\
したがって、その後に平均の差の検定でt検定を行う場合は、Welchの*t*検定を行う。

### *t*検定
t検定は```t.test()```関数を使って実行する。
#### Welchの*t*検定
```t.test()```関数には以下の引数を指定することが可能。
- ```x = ```: ２群のうち一方のデータを入力
- ```y = ```: ２群のうちもう一方のデータを入力
- ```var.equal = ```: 等分散かどうかを指定する。```F```で等分散でない（つまりWelchの*t*検定）、```T```で等分散（つまりStudentの*t*検定）
- ```paired = ```: 対応があるかどうか。```F```で対応がない、```T```で対応がある
- ```alternative = ```: 両側検定（```"two.sided"```と指定）か、片側検定（```"greater"```または```less```と指定）

下記を実行する。
```
t.test(x = maguro, y = sake, var.equal=F, paired=F, alternative = "two.sided")
```
以下の結果が出力される。\
<img width="437" alt="スクリーンショット 2023-07-25 14 09 01" src="https://github.com/yosui-nojima/statistics-C1_R_2/assets/85273234/782bd0a4-9013-4181-8cbb-826834fc4d58">\
```p-value = 0.06738```の部分が*P*値を意味する。\
有意水準αを5%とすると、*P*>αのため帰無仮説(H0:µ1=µ2)が採択される。\
つまり、『まぐろ』の全国の年間支出金額の平均と『さけ』の全国の年間支出金額の平均は異なるとは言えない。\

#### Studentの*t*検定
念のためStudentの*t*検定の実行方法も記載する。
```var.equal = T```に変えて下記を実行する。
```
下記を実行する。
t.test(maguro, sake, var.equal=T, paired=F, alternative = "two.sided")
```
以下の結果が出力される。\
<img width="443" alt="スクリーンショット 2023-07-25 14 12 48" src="https://github.com/yosui-nojima/statistics-C1_R_2/assets/85273234/3d5f2f1d-4adf-4bdf-9551-5db71994b9c4">

## 5. 相関分析のR実装
### 使用するデータ
相関分析では、『パスタ』の年間支出金額と『チーズ』の年間支出金額を使用する。\
これについても、本来は正規確立プロットなどでデータが正規分布に従っているかどうかを確認するが、今回は正規分布に従っていると仮定して検定を行う。
下記を実行して使用するデータを```data```オブジェクトから抽出し、それぞれ```pasta```と```cheese```というオブジェクトに格納する。
```
pasta <- as.numeric(data[,"パスタ"])
cheese <- as.numeric(data[,"チーズ"])
```
```as.numeric()```関数は数値データとしてオブジェクトに出力するための関数。（```as.numeric()```関数を使わずに出力すると、文字列として出力されてしまうため。）

### データの可視化
『パスタ』の全国の年間支出金額と『チーズ』全国の年間支出金額の関連性を散布図で可視化する。\
下記を実行する。
```
both2 <- cbind(pasta, cheese)
plot(both2)
```
下図が出力される。\
![Rplot11](https://github.com/yosui-nojima/statistics-C1_R_2/assets/85273234/f98c7492-b313-4d94-86c7-322a5ad5b740)

散布図のみでは関連性の主張は定性的であるため、2変数間の線形性の指標である相関係数（Pearsonの相関係数）を算出する。

### 相関分析の実行
相関分析は```cor.test()```関数を使って実行する。\
```cor.test()```関数には以下の引数を指定することが可能。
- ```x = ```: ２変数のうち一方のデータを入力
- ```y = ```: ２変数のうちもう一方のデータを入力
- ```alternative = ```: 両側検定（```"two.sided"```と指定）か、片側検定（```"greater"```または```less```と指定）
- ```method = ```: 相関分析の手法を指定（```pearson```または```spearman```など）
- ```conf.level = ```: 相関係数の区間推定を行う際の信頼係数を指定

下記を実行する。
```
cor.test(x = pasta, y = cheese, alternative = "two.sided", method = "pearson", conf.level = 0.95)
```
以下の結果が出力される。\
<img width="384" alt="スクリーンショット 2023-07-25 14 43 45" src="https://github.com/yosui-nojima/statistics-C1_R_2/assets/85273234/8fbb32ea-f9bf-4cf7-9ad6-cf4a2f32e8c5">\
一番下の数値がPearsonの相関係数を示しており、0.7を超えている。つまり、『パスタ』の年間支出金額と『チーズ』の年間支出金額には正に強い相関がある。\
また、```p-value = 2.369e-09```の部分が無相関の検定での*P*値を意味する。\
有意水準αを5%とすると、*P*<αのため帰無仮説(H0:ρ=0)は棄却される。\
つまり、『パスタ』の年間支出金額と『チーズ』の年間支出金額は有意な関連が認められる。

## 6. 線形回帰分析のR実装
線形回帰分析は、```lm()```関数を使って実行する。\
『チーズ』の年間支出金額を説明変数、『パスタ』の年間支出金額を目的変数として最小二乗法で標本回帰直線を得る。\
下記を実行する。
```
lr <- lm(pasta ~ cheese, data=data.frame(both2))
abline(lr, col="red")
```
```lm()```関数では、目的関数を入力し、~(チルダ)、説明変数の順番で引数を入力する。```data = ```引数にはデータフレーム型のオブジェクトを指定する。ここでは、```lr```オブジェクトとして出力している。\
```abline()```関数は、で先に出力した散布図に標本回帰直線を追加する。\
下図が出力される。\
![Rplot12](https://github.com/yosui-nojima/statistics-C1_R_2/assets/85273234/66e42033-2386-4005-ad41-3ca969cf8116)

線形回帰分析の結果を出力する。
下記を実行する。
```
summary(lr)
```
```summary()```関数は、```lm()```関数で出力したオブジェクトの概要を出力する。
以下の結果が出力される。\
<img width="414" alt="スクリーンショット 2023-07-25 15 40 30" src="https://github.com/yosui-nojima/statistics-C1_R_2/assets/85273234/06928e6b-88ca-4627-9241-196cf77dd168">

出力結果の見方は、下記の通り。\
﻿﻿<img width="1166" alt="スクリーンショット 2023-07-25 15 54 58" src="https://github.com/yosui-nojima/statistics-C1_R_2/assets/85273234/bf0a0a4b-eae0-4899-a086-5da218d8b199">

これらの情報のうち、切片、傾き、決定係数を標本回帰直線付き散布図に入力する。\
<img width="601" alt="スクリーンショット 2023-07-25 15 36 32" src="https://github.com/yosui-nojima/statistics-C1_R_2/assets/85273234/50d11c00-988a-47ae-be12-c7abbef5030c">



