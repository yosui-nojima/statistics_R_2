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

#### 2. 赤枠の『SSDSE（教育用標準データセット）』をクッリク
<img width="2559" alt="スクリーンショット 2023-07-25 12 54 16" src="https://github.com/yosui-nojima/statistics-C1_R_2/assets/85273234/181adf1e-2649-4e70-a901-f996464c2d5e">

#### 3. 赤枠の『SSDSE-C-2023』のデータをクリックしてダウンロードする
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
エクセルファイルの読み込みはデフォルト状態のRではできないため、```openxlsx```ライブラリーをインストールする必要がある。\
また、今回はサーバーから直接R上に読み込む。（ダウンロードしたファイルは任意のダウンロードファルダに保存される。）\
下記をR上で実行する。
```
install.packages("openxlsx")
```
R上でエクセルファイルを読み込む。
```
library(openxlsx)
data <- read.xlsx("https://www.nstac.go.jp/sys/files/SSDSE-C-2023.xlsx", colNames = T) #ファイルの読み込み
colnames(data) <- data[1,] #列名の指定
row.names(data) <- data[,2] #行明の指定
data <- data[-c(1,2),-c(1:5)] #不要な行・列の削除
```

## 4. 仮説検定のR実装
今回は仮説検定のうち、F検定とt検定について説明します。
### 使用するデータ
読み込んだ家計調査データは、当該期間（2020年（令和2年）～2022年（令和4年））の１世帯あたりの年間支出金額の平均値を示している。\
今回は、年間支出金額という変数について、『まぐろ』の群と『さけ』の群の2群を使用する。\
本来はこれらのデータについて正規確立プロットなどで、データが正規分布に従っているかどうかを確認するが、今回は正規分布に従っていると仮定して検定を行う。\
下記を実行して使用するデータを```data```オブジェクトから抽出し、それぞれ```maguro```と```sake```というオブジェクトに格納する。**※各自選択した群がわかるオブジェクト名に変更すること**
```
maguro <- as.numeric(data[,"まぐろ"]) #各自選択した群がわかるオブジェクト名に変更すること
sake <- as.numeric(data[,"さけ"]) #各自選択した群がわかるオブジェクト名に変更すること
```
```as.numeric()```関数は数値データとしてオブジェクトに出力するための関数。（```as.numeric()```関数を使わずに出力すると、文字列として出力されてしまうため。）
### データの可視化
『まぐろ』の全国の年間支出金額の平均と『さけ』全国の年間支出金額の平均を棒グラフで可視化する。\
下記を実行する。
```
both <- cbind(maguro, sake)
xm <- apply(both, 2, mean)
xs <- apply(both, 2, sd)
b <- barplot(xm, ylab = "Yen", ylim = c(0, max(xm + xs))) #棒グラフを出力
arrows(b, xm - xs, b, xm + xs, code = 3, lwd = 1, angle = 90, length = 0.1) #エラーバーを出力
```
下図が出力される。\
![Rplot14](https://github.com/yosui-nojima/statistics-C1_R_2/assets/85273234/4dce1e45-319b-45a6-9e35-a068974bdfa6)

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
<img width="428" alt="スクリーンショット 2023-07-25 16 31 03" src="https://github.com/yosui-nojima/statistics-C1_R_2/assets/85273234/266d9af5-199f-488e-81e3-60af5f6a6709">\
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
<img width="437" alt="スクリーンショット 2023-07-25 16 31 16" src="https://github.com/yosui-nojima/statistics-C1_R_2/assets/85273234/4c58830c-9559-4829-8836-2806e6f20fde">\
```p-value = 0.06738```の部分が*P*値を意味する。\
有意水準αを5%とすると、*P*>αのため帰無仮説(H0:µ1=µ2)が採択される。\
つまり、『まぐろ』の全国の年間支出金額の平均と『さけ』の全国の年間支出金額の平均は異なるとは言えない。

#### Studentの*t*検定
念のためStudentの*t*検定の実行方法も記載する。\
```var.equal = T```に変えて下記を実行する。\
下記を実行する。
```
t.test(maguro, sake, var.equal=T, paired=F, alternative = "two.sided")
```
以下の結果が出力される。\
<img width="437" alt="スクリーンショット 2023-07-25 16 31 34" src="https://github.com/yosui-nojima/statistics-C1_R_2/assets/85273234/89f52ff2-ff14-41a6-af54-cfc03cb40666">

## 5. 相関分析のR実装
### 使用するデータ
相関分析では、『パスタ』の年間支出金額と『チーズ』の年間支出金額を使用する。\
これについても、本来は正規確立プロットなどでデータが正規分布に従っているかどうかを確認するが、今回は正規分布に従っていると仮定して検定を行う。\
下記を実行して使用するデータを```data```オブジェクトから抽出し、それぞれ```pasta```と```cheese```というオブジェクトに格納する。**※各自選択した変数がわかるオブジェクト名に変更すること**
```
pasta <- as.numeric(data[,"パスタ"]) #各自選択した変数がわかるオブジェクト名に変更すること
cheese <- as.numeric(data[,"チーズ"]) #各自選択した変数がわかるオブジェクト名に変更すること
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
![Rplot15](https://github.com/yosui-nojima/statistics-C1_R_2/assets/85273234/97d7a672-649f-46f1-91f6-820b522230fc)

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
<img width="387" alt="スクリーンショット 2023-07-25 16 32 17" src="https://github.com/yosui-nojima/statistics-C1_R_2/assets/85273234/24404f51-ec81-469c-8e2d-b77378268c7c">\
一番下の数値がPearsonの相関係数を示しており、0.7を超えている。つまり、『パスタ』の年間支出金額と『チーズ』の年間支出金額には正に強い相関がある。\
また、```p-value = 2.369e-09```の部分が無相関の検定での*P*値を意味する。\
有意水準αを5%とすると、*P*<αのため帰無仮説(H0:ρ=0)は棄却される。\
つまり、『パスタ』の年間支出金額と『チーズ』の年間支出金額は有意な関連が認められる。

## 6. 線形回帰分析のR実装
線形回帰分析は、```lm()```関数を使って実行する。\
『チーズ』の年間支出金額を説明変数、『パスタ』の年間支出金額を目的変数として最小二乗法で標本回帰直線を得る。\
下記を実行する。
```
lr <- lm(cheese ~ pasta, data=data.frame(both2))
abline(lr, col="red")
```
```lm()```関数では、目的関数を入力し、~(チルダ)、説明変数の順番で引数を入力する。```data = ```引数にはデータフレーム型のオブジェクトを指定する。ここでは、```lr```オブジェクトとして出力している。\
```abline()```関数は、で先に出力した散布図に標本回帰直線を追加する。\
下図が出力される。\
![Rplot16](https://github.com/yosui-nojima/statistics-C1_R_2/assets/85273234/5660629c-11ce-4719-89c3-f2db208eae8c)

線形回帰分析の結果を出力する。
下記を実行する。
```
summary(lr)
```
```summary()```関数は、```lm()```関数で出力したオブジェクトの概要を出力する。
以下の結果が出力される。\
<img width="411" alt="スクリーンショット 2023-07-25 16 32 43" src="https://github.com/yosui-nojima/statistics-C1_R_2/assets/85273234/b67d51af-fede-488d-ac98-d7e13df02e81">

出力結果の見方は、下記の通り。\
﻿﻿<img width="707" alt="スクリーンショット 2023-07-25 16 47 08" src="https://github.com/yosui-nojima/statistics-C1_R_2/assets/85273234/3a2a9496-328d-4083-a054-4b6c92bac6ad">

これらの情報のうち、切片、傾き、決定係数を標本回帰直線付き散布図に記入する（パワーポイントなど任意のソフトを用いて）。\
<img width="651" alt="スクリーンショット 2023-07-25 16 46 24" src="https://github.com/yosui-nojima/statistics-C1_R_2/assets/85273234/7b50a4b4-9145-48ee-9e12-6cce6fe23cbc">

# レポート課題
以下の内容をRで実行し、レポートを作成すること。

## 1. 仮説検定
1.  [独立行政法人統計センター](https://www.nstac.go.jp/)が公開している2023年度版の家計調査データ（総務省統計局「家計調査」2020年（令和2年）～2022年（令和4年））について、年間支出金額を変数として任意の2群を選択する。（※『まぐろ』と『さけ』は対象外）
2.  1.で選択した2群について、平均値、標準偏差を算出し、エラーバー付き棒グラフとして可視化する。
3.  1.で選択した2群について、F検定を行い、続いて任意のt検定を行う。

## 2. 相関分析
1.  [独立行政法人統計センター](https://www.nstac.go.jp/)が公開している2023年度版の家計調査データ（総務省統計局「家計調査」2020年（令和2年）～2022年（令和4年））について、任意の2変数を選択する。（※『パスタ』と『チーズ』は対象外）
2.  1.で選択した2変数の関連性ついて、散布図として可視化し、相関分析により相関係数を算出する。

## 3. 線形回帰分析
1.  [独立行政法人統計センター](https://www.nstac.go.jp/)が公開している2023年度版の家計調査データ（総務省統計局「家計調査」2020年（令和2年）～2022年（令和4年））について、任意の2変数を選択する。（※『パスタ』と『チーズ』は対象外）
2.  1.で選択した2変数の関連性ついて、散布図として可視化し、散布図に標本回帰直線、決定係数を記入する。


**レポートには、以下の内容を含めること。**
- 選択した群、または変数の名前
- 実行したRのコード
- 実行後の出力結果
- 実行後に出力された図
- 仮説検定、相関分析、線形回帰分析のそれぞれの分析結果の結論


**※提出期限：2023年8月7日 23:59までにCLEから提出**\
**※PDFファイルをCLEより提出すること。**\
**※ファイル名は「○○○○○○_20230726.pdf」とすること。○○○○○○には各自の学籍番号を入れる。**\
**※期限を過ぎるとCLEから提出ページが消えます。**\
**※CLE以外からの提出（メール添付など）は認めない。**

