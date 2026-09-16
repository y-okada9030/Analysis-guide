# スペクトル解析と結果の確認

スペクトルを作っただけでは解析は終わりません。線源 PHA、背景 PHA、RMF、ARF が**同じ観測・時間・領域・イベント選択**を表しているかを確認し、XSPEC に読み込んで、モデルフィットと残差検定を実施します。

## ファイルの対応を確認する

PHA ヘッダーの `BACKFILE`、`RESPFILE`、`ANCRFILE`、`BACKSCAL`、`EXPOSURE` を読みます。`BACKSCAL` は線源領域と Background 領域の面積や有効面積の比に関わるため、正確に設定されていることが不可欠です。

```sh
fkeyprint <SOURCE_PHA> BACKSCAL
fkeyprint <SOURCE_PHA> RESPFILE
fkeyprint <SOURCE_PHA> ANCRFILE
fkeyprint <SOURCE_PHA> BACKFILE
```

上の入力は HEASoft 環境での確認例です。Chandra なら CIAO の `dmlist` や `dmkeypar` でもヘッダーを確認できます。背景ファイルを指定しない場合は、その理由を記録します。

## XSPEC に読み込む

XSPEC は [公式マニュアル](https://heasarc.gsfc.nasa.gov/docs/software/xspec/manual/) に従って使います。PHA のヘッダーに背景・応答への正しい参照が書かれていれば、自動的に読み込まれます。

```sh
xspec
```

```text
data <SOURCE_PHA>
show data
plot data
```

ヘッダーの参照がない、または別のファイルを使う科学的理由がある場合は、対応する Background・応答を明示します。`<...>` は実在するファイルに置き換えます。

```text
backgrnd <BACKGROUND_PHA>
response <SOURCE_RMF>
arf <SOURCE_ARF>
show data
```

広がった線源のスペクトルは場所によって変わり得ます。熱的プラズマ、非熱的放射、吸収などのどの成分を入れるかは、観測帯域と研究目的から判断します。

## 統計量と Background の扱いを決める

カウントが少ない帯域や高分解能の線を扱うとき、見やすくするための強いグルーピングで情報を失う場合があります。統計量とチャンネルのまとめ方を文書に記します。

複数 ObsID や複数機器のスペクトルを扱うときは、観測ごとの PHA、背景、RMF、ARF を維持して同時にフィットするか、公式手順で結合します。スペクトルフィットの記録には、各ファイルの対応と統計法を含めます。

## 結果を点検する

フィットが終わったら、データとモデル、残差を表示します。特定の線やエネルギー帯に系統的な残差があれば、モデルだけでなく背景、応答、領域の定義を見直します。

```text
plot ldata delchi
```

最終的に、元イベントと露出マップ上で領域を再表示し、観測ログ、スクリーニング、背景の規格化、応答の生成入力、フィットの実行記録をそろえて保存します。
