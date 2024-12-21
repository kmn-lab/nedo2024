# nedo2024
Motion Decoding Using Biosignals Skateboarder Center of Gravity Prediction Challengeの5位解法コードです。

# データの配置
.ipnybファイルと同じ階層に"dataset","output"フォルダを作成する。
"dataset"には事務局配布のファイル一式（reference.mat、test.mat、train.mat、sample_submit.json）を格納する。
提出モデルは、時系列データを画像化したデータで学習・推論する。.ipnybファイルを実行すると、"output"フォルダ下に実験名フォルダが生成され、その下に画像データ(.pkl)、モデルファイル(.pth)、学習曲線などを出力する。

# モデルの構成

## 1_1段目モデル

#### 8227_ImageFeature-Effnetb0TorchV

efficientnetb0で速度ベクトルを学習・推論するモデル

#### 8248_ImageFeature-Effnetb0TorchV_EvalGMM

8227で推論した速度ベクトルを3クラスに分類するモデル

## 2_2段目モデル

3クラス分類のラベルを、画像中にメタ情報として埋め込み、学習・推論するモデル
efficientnetb0,b1,b2,v2s,MaxVit,MobileNetV4の6種類をバックボーンに使用
（b0のモデルを基本としいるが、多様性を持たせるために、他の5種類を追加している）

モデル毎に画像サイズが異なる。b2やv2sでは画像サイズを合わせるために、256x256とは異なるデータ配置・埋込みをしている。
- b0, b1, maxvit, mobilentv4は256x256の画像
- b2は288x288の画像を作成（簡易なssc, aacも埋込み)
- v2sは384x384の画像を作成（簡易なssc, aacも埋込み)
    - Slope Sign Change (SSC): SSCは波形が符号を変更する回数、筋電位信号の変動を捉える。筋活動の変化が多いほどSSCの値は大きい。
    - Average Amplitude Change (AAC): AACは連続するサンプル間の振幅差の平均を計算。信号の変動性を評価。

#### 2-1_通常モデル\\***

0001から0005のtrainデータで学習するモデル

#### 2-2_Test5追加モデル\\***

0001から0005のtrainデータと、0005のtestデータで学習するモデル

#### 2-3_User別モデル\\***

ユーザーID別に学習・推論するモデル　(例：0001の推論は、0001を学習した重みを用いる)。pretrainedとして、当該ユーザー以外のデータで学習。Augmentationで信号領域にノイズ付与。

## 3_Stackingモデル

2_2段目モデルのOOF予測値を使い、Lasso回帰でStacking

