# 2024年度 統計学B-II 木曜２限 2024年1月23日 10:30~12:00
# R実習

## 目次
### 1. [使用するデータ](https://github.com/yosui-nojima/statistics-C1_R_2/blob/main/README.md#1-%E4%BD%BF%E7%94%A8%E3%81%99%E3%82%8B%E3%83%87%E3%83%BC%E3%82%BF-1)
### 2. [仮説検定のR実装](https://github.com/yosui-nojima/statistics-C1_R_2/blob/main/README.md#2-%E4%BB%AE%E8%AA%AC%E6%A4%9C%E5%AE%9A%E3%81%AEr%E5%AE%9F%E8%A3%85-1)
### 3. [相関分析のR実装](https://github.com/yosui-nojima/statistics-C1_R_2/blob/main/README.md#3-%E7%9B%B8%E9%96%A2%E5%88%86%E6%9E%90%E3%81%AEr%E5%AE%9F%E8%A3%85-1)
### 4. [線形回帰分析のR実装](https://github.com/yosui-nojima/statistics-C1_R_2/blob/main/README.md#4-%E7%B7%9A%E5%BD%A2%E5%9B%9E%E5%B8%B0%E5%88%86%E6%9E%90%E3%81%AEr%E5%AE%9F%E8%A3%85-1)
### 5. [レポート課題](https://github.com/yosui-nojima/statistics-C1_R_2/blob/main/README.md#5-%E3%83%AC%E3%83%9D%E3%83%BC%E3%83%88%E8%AA%B2%E9%A1%8C-1)

## 1. 使用するデータ
[独立行政法人統計センター](https://www.nstac.go.jp/)が公開しているSSDSE（教育用標準データセット：Standardized Statistical Data Set for Education）は、データサイエンス演習、統計教育用に作成・公開している統計データ。\
今回は、2024年版の家計調査データ（総務省統計局「家計調査」2021年（令和3年）～2023年（令和5年））を取得し、解析用データとする。
### 使用するファイルについて
#### 1. 赤枠の『統計を活かす』をクリック
<img width="1728" alt="Image" src="https://github.com/user-attachments/assets/6e762925-20f1-4793-8741-27e0c62dbe0d" />

#### 2. 赤枠の『SSDSE（教育用標準データセット）』をクッリク
<img width="1728" alt="Image" src="https://github.com/user-attachments/assets/e7063c24-1ec3-4982-9a08-8e47baf2d0d1" />

#### 3. 赤枠の『SSDSE-C-2023』のデータをクリックしてダウンロードする
<img width="1728" alt="Image" src="https://github.com/user-attachments/assets/c1f78054-5f4e-4582-85eb-55d62cb072cc" />

#### 4. 任意の表計算ソフトで開くと以下の内容を含むデータを確認することができる
数値の単位は、世帯人員は『人』（小数点以下２桁）、他はすべて『円』。
データは平均値を表しており、\
（2021年歴年データ ＋ 2022年歴年データ ＋ 2023年歴年データ）÷ ３\
で求められている。\
また、各食料品の項目の数値は、都道府県庁所在市別、二人以上の世帯の１世帯当たりの品目別の年間支出金額を表している。
<img width="1728" alt="Image" src="https://github.com/user-attachments/assets/e978ed49-3ca3-4769-9fd1-665423c26f5e" />

#### 5. 使用するデータの詳細
今回使用するデータの詳細（データの出典や単位など）は下記を参照すること。\
[https://www.nstac.go.jp/sys/files/kaisetsu-C-2024.pdf](https://www.nstac.go.jp/sys/files/kaisetsu-C-2024.pdf)

### エクセルファイルをR上で読み込む
RとRStudioは前回（2024年10月31日）の実習講義でインストール済み。\
RStudioによるRの起動方法は前回実習講義の[RStudioによるRの起動](https://github.com/yosui-nojima/statistics-C1_R_1#6-rstudio%E3%81%AB%E3%82%88%E3%82%8Br%E3%81%AE%E8%B5%B7%E5%8B%95)を参照。\
下記の画面が表示される場合は（新しいバージョンに更新するかを聞いている）、『Remind Later』（後で再通知）を選択。

<img width="443" alt="スクリーンショット 2023-07-26 13 36 29" src="https://github.com/yosui-nojima/statistics-C1_R_2/assets/85273234/a0143119-8f7c-4cae-b266-7c3561a02ed5">

Rスクリプトを入力する画面の立ち上げは、前回実習講義の[R実行の準備](https://github.com/yosui-nojima/statistics-C1_R_1#7-r%E5%AE%9F%E8%A1%8C%E3%81%AE%E6%BA%96%E5%82%99)を参照

エクセルファイルの読み込みはデフォルト状態のRではできないため、```openxlsx```ライブラリーをインストールする必要がある。\
また、今回はサーバーから直接R上に読み込む。（ダウンロードしたファイルは任意のダウンロードファルダに保存される。）\
下記をR上で実行する。
```
install.packages("openxlsx")
```
R上でエクセルファイルを読み込む。
```
library(openxlsx)
data <- read.xlsx("https://www.nstac.go.jp/sys/files/SSDSE-C-2024.xlsx") #ファイルの読み込み
colnames(data) <- data[1,] #列名の指定
row.names(data) <- data[,3] #行名の指定
data <- data[-c(1,2),-c(1:5,216:229)] #不要な行・列の削除
data <- data[,-grep("　", colnames(data))] #不要な列の削除
```
どのような項目があるかは、データの列名を確認すればよい。\
下記を実行する。
```
colnames(data)
```
以下の結果が出力される。\
<img width="1447" alt="Image" src="https://github.com/user-attachments/assets/f0abe8b5-8b52-4ff6-a54f-a0d71a739c44" />

以降の**2. 仮説検定のR実装**、**3. 相関分析のR実装**、**4. 線形回帰分析のR実装**ではこの中から任意の列を選択する。

## 2. 仮説検定のR実装
今回は仮説検定のうち、F検定とt検定について説明します。
### 使用するデータ
読み込んだ家計調査データは、当該期間（2020年（令和2年）～2022年（令和4年））の１世帯あたりの年間支出金額の平均値を示している。\
今回は、年間支出金額という変数について、『まぐろ』の群と『さけ』の群の2群を使用する。\
本来はこれらのデータについて正規確率プロットなどで、データが正規分布に従っているかどうかを確認するが、今回は正規分布に従っていると仮定して検定を行う。\
下記を実行して使用するデータを```data```オブジェクトから抽出し、それぞれ```maguro```と```sake```というオブジェクトに格納する。**※各自選択した群がわかるオブジェクト名（英語アルファベット）に変更すること**
```
maguro <- as.numeric(data[,"まぐろ"]) #各自選択した群がわかるオブジェクト名（英語アルファベット）に変更すること
sake <- as.numeric(data[,"さけ"]) #各自選択した群がわかるオブジェクト名（英語アルファベット）に変更すること
```
```as.numeric()```関数は数値データとしてオブジェクトに出力するための関数。（```as.numeric()```関数を使わずに出力すると、文字列として出力されてしまうため。）
### データの可視化
『まぐろ』の全国の年間支出金額の平均と『さけ』全国の年間支出金額の平均を棒グラフで可視化する。\
下記を実行する。
```
both <- cbind(maguro, sake) #maguroとsakeの2つのオブジェクトを結合して、bothという1つのオブジェクトに出力する
xm <- apply(both, 2, mean) #marugoとsakeそれぞれの平均値を算出
xs <- apply(both, 2, sd) #marugoとsakeそれぞれの標準偏差を算出
b <- barplot(xm, ylab = "Yen", ylim = c(0, max(xm + xs))) #棒グラフを出力
arrows(b, xm - xs, b, xm + xs, code = 3, lwd = 1, angle = 90, length = 0.1) #エラーバーを出力
```
下図が出力される。\
<img width="700" alt="Image" src="https://github.com/user-attachments/assets/2b0a2802-5ae3-484b-899e-50fa3dec9a32" />

棒の頂点が平均値を示す。\
標準偏差はエラーバーで示している。\
次に、このグラフで見られる差が本質的な差かどうかを仮説検定により調査する。

### F検定
まずはF検定で等分散かそうでないかを検定する。\
```var.test()```関数には以下の引数を指定することが可能。
- ```x = ```: ２群のうち一方のデータを入力
- ```y = ```: ２群のうちもう一方のデータを入力
- ```alternative = ```: 両側検定（```"two.sided"```と指定）か、片側検定（```"greater"```または```"less"```と指定）
下記を実行する。
```
var.test(x = maguro, y = sake, alternative = "two.sided")
```

以下の結果が出力される。\
<img width="524" alt="Image" src="https://github.com/user-attachments/assets/67d62aea-dca5-4054-8361-a5e7b2603620" />\
```p-value = 8.94e-13```の部分が*P*値を意味する。\
有意水準αを5%とすると、*P*<αのため帰無仮説(H<sub>0</sub>: σ<sub>1</sub>=σ<sub>2</sub>)は棄却される。つまり、『まぐろ』の全国の年間支出金額の分散と『さけ』の全国の年間支出金額の分散は異なると言える。\
したがって、この後に2群の平均の差の検定でt検定を行う場合は、Welchの*t*検定を選択する。

### *t*検定
t検定は```t.test()```関数を使って実行する。
#### Welchの*t*検定
```t.test()```関数には以下の引数を指定することが可能。
- ```x = ```: ２群のうち一方のデータを入力
- ```y = ```: ２群のうちもう一方のデータを入力
- ```var.equal = ```: 等分散かどうかを指定する。```F```で等分散でない（つまりWelchの*t*検定）、```T```で等分散（つまりStudentの*t*検定）
- ```paired = ```: 対応があるかどうか。```F```で対応がない、```T```で対応がある
- ```alternative = ```: 両側検定（```"two.sided"```と指定）か、片側検定（```"greater"```または```"less"```と指定）

下記を実行する。
```
t.test(x = maguro, y = sake, var.equal=F, paired=F, alternative = "two.sided")
```
以下の結果が出力される。\
<img width="538" alt="Image" src="https://github.com/user-attachments/assets/2a41835c-3461-4785-b374-71ea2abe3dc3" />\
```p-value = 0.03505```の部分が*P*値を意味する。\
有意水準αを5%とすると、*P*<αのため帰無仮説(H<sub>0</sub>: µ<sub>1</sub>=µ<sub>2</sub>)は棄却される。\
つまり、『まぐろ』の全国の年間支出金額の平均と『さけ』の全国の年間支出金額の平均は異なるといえる。

#### Studentの*t*検定
F検定で帰無仮説(H<sub>0</sub>: σ<sub>1</sub>=σ<sub>2</sub>)が採択され、2群の平均の差の検定で*t*検定を行う場合は、Studentの*t*検定を選択する。\
```var.equal = T```に変えて下記を実行する。\
下記を実行する。
```
t.test(maguro, sake, var.equal=T, paired=F, alternative = "two.sided")
```
以下の結果が出力される。\
<img width="533" alt="Image" src="https://github.com/user-attachments/assets/103992d1-0b52-44d6-8f3f-4ce7f9c88260" />

## 3. 相関分析のR実装
### 使用するデータ
相関分析では、『パスタ』の年間支出金額と『チーズ』の年間支出金額を使用する。\
これについても、本来は正規確率プロットなどでデータが正規分布に従っているかどうかを確認するが、今回は正規分布に従っていると仮定して検定を行う。\
下記を実行して使用するデータを```data```オブジェクトから抽出し、それぞれ```pasta```と```cheese```というオブジェクトに格納する。**※各自選択した変数がわかるオブジェクト名（英語アルファベット）に変更すること**
```
pasta <- as.numeric(data[,"パスタ"]) #各自選択した変数がわかるオブジェクト名（英語アルファベット）に変更すること
cheese <- as.numeric(data[,"チーズ"]) #各自選択した変数がわかるオブジェクト名（英語アルファベット）に変更すること
```
```as.numeric()```関数は数値データとしてオブジェクトに出力するための関数。（```as.numeric()```関数を使わずに出力すると、文字列として出力されてしまうため。）

### データの可視化
『パスタ』の全国の年間支出金額と『チーズ』全国の年間支出金額の関連性を散布図で可視化する。\
下記を実行する。
```
both2 <- cbind(cheese, pasta) #cheeseとpastaの2つのオブジェクトを結合して、both2という1つのオブジェクトに出力する
plot(both2) #散布図を出力
```
下図が出力される。\
<img width="697" alt="Image" src="https://github.com/user-attachments/assets/61b69d15-5c72-45b3-bcab-3f070ac2282a" />

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
<img width="470" alt="Image" src="https://github.com/user-attachments/assets/c65b3330-895a-4fce-bba3-d9b48f4a68a9" />\
一番下の数値がPearsonの積率相関係数を示しており、0.7を超えている。つまり、『パスタ』の年間支出金額と『チーズ』の年間支出金額には正に強い相関がある。\
また、```p-value = 2.615e-09```の部分が無相関の検定での*P*値を意味する。\
有意水準αを5%とすると、*P*<αのため帰無仮説(H<sub>0</sub>: ρ=0)は棄却される。\
つまり、『パスタ』の年間支出金額と『チーズ』の年間支出金額は有意な関連が認められる。

## 4. 線形回帰分析のR実装
線形回帰分析は、```lm()```関数を使って実行する。\
『チーズ』の年間支出金額を説明変数、『パスタ』の年間支出金額を目的変数として最小二乗法で標本回帰直線を得る。\
下記を実行する。
```
lr <- lm(pasta ~ cheese, data=data.frame(both2))
```
```lm()```関数では、目的関数、~(チルダ)、説明変数の順番で入力する。```data = ```引数にはデータフレーム型のオブジェクトを指定する。ここでは、```lr```オブジェクトとして出力している。

線形回帰分析の結果を出力する。
下記を実行する。
```
summary(lr)
```
```summary()```関数は、```lm()```関数で出力したオブジェクトの概要を出力する。
以下の結果が出力される。\
<img width="507" alt="Image" src="https://github.com/user-attachments/assets/0f94bfd1-aed9-4291-94df-f7ebccc312fc" />

出力結果の見方は、下記の通り。\
<img width="655" alt="Image" src="https://github.com/user-attachments/assets/fe3b8aba-9493-4482-862c-3d006d6701b8" />

有意水準αを5%とすると、傾きの*P*値は*P*<αのため帰無仮説(H<sub>0</sub>: β<sub>1</sub>=0)は棄却される。\
したがって、チーズの年間支出金額が増えると、パスタの年間支出金額も増えると言える。\
チーズの年間支出金額は、パスタの年間支出金額に関連する因子の１つであると考えられる。

続いて、これらの情報のうち、切片、傾き、決定係数を散布図に記入する。\
下記を実行する。
```
plot(both2, main = paste("y = ", round(lr[["coefficients"]][["(Intercept)"]], digits = 3), " + ", round(lr[["coefficients"]][["cheese"]], digits = 3), "x",  ", ", "R^2 = ", round(summary(lr)[["r.squared"]], digits = 3), sep = ""))
```
散布図の上に１次関数式、決定係数（R^2）が記載された下図が出力される。\
<img width="711" alt="Image" src="https://github.com/user-attachments/assets/2cdadc91-9734-4886-b52a-ea9354e54e06" />

最後に標本回帰直線を散布図に追加する。
```
abline(lr, col="red")
```
```abline()```関数は、先に出力した散布図に直線を追加する関数。\
下図が出力される。

<img width="710" alt="Image" src="https://github.com/user-attachments/assets/cdf16740-8051-46df-86b9-52bfc832ec68" />

## 5. レポート課題
以下の内容をRで実行し、レポートを作成すること。

### 1. 仮説検定
1.  [独立行政法人統計センター](https://www.nstac.go.jp/)が公開している2024年版の家計調査データ（総務省統計局「家計調査」2021年（令和3年）～2023年（令和5年））について、年間支出金額を変数として任意の2群を選択する。（※『まぐろ』と『さけ』は対象外）
2.  1.で選択した2群について、平均値、標準偏差を算出し、エラーバー付き棒グラフとして可視化する。
3.  1.で選択した2群について、F検定を行い、続いて任意のt検定を行う。

### 2. 相関分析
1.  [独立行政法人統計センター](https://www.nstac.go.jp/)が公開している2024年版の家計調査データ（総務省統計局「家計調査」2021年（令和3年）～2023年（令和5年））について、任意の2変数を選択する。（※『パスタ』と『チーズ』は対象外）
2.  1.で選択した2変数の関連性ついて、散布図として可視化し、相関分析により相関係数を算出する。

### 3. 線形回帰分析
1.  [独立行政法人統計センター](https://www.nstac.go.jp/)が公開している2024年版の家計調査データ（総務省統計局「家計調査」2021年（令和3年）～2023年（令和5年））について、任意の2変数を選択する。（※『パスタ』と『チーズ』は対象外）
2.  1.で選択した2変数の関連性ついて、散布図として可視化し、散布図に標本回帰直線、決定係数を記入する。


**レポートには、以下の内容を含めること。**
- 選択した群（1. 仮説検定の場合）および 変数の名前（2. 相関分析、3. 線形回帰分析の場合）
- 実行したRのコード
- 実行後の出力結果
- 実行後に出力された図（図の出力方法は、前回実習講義の[図の保存方法](https://github.com/yosui-nojima/statistics-C1_R_1#%E5%9B%B3%E3%81%AE%E4%BF%9D%E5%AD%98%E6%96%B9%E6%B3%95)を参照。）
- 仮説検定、相関分析、線形回帰分析のそれぞれの分析結果の結論


**※提出期限：2025年2月3日 23:59までにCLEから提出**\
**※PDFファイルをCLEより提出すること。**\
**※ファイル名は「○○○○○○_20250123.pdf」とすること。○○○○○○には各自の学籍番号を入れる。**\
**※期限を過ぎるとCLEから提出ページが消えます。**\
**※CLE以外からの提出（メール添付など）は認めない。**\
**※他人のレポートのコピー＆ペーストはしないように。**\
**※家計調査データは199項目あるため、そこから2つを選ぶ場合<sub>199</sub>C<sub>2</sub>=19,701通りある。**
