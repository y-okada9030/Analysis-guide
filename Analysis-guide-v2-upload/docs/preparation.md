# 解析を始める前に

## 目的と観測を確認する

まず、何を測るのかを短く書きます。たとえば「超新星残骸の殻の温度を場所ごとに比べる」なら、殻を分割する領域、各領域に適した Background、位置依存の応答が必要です。「一枚の画像を作る」場合とは必要な処理が異なります。対象が視野より広いか、点源が重なるか、複数の観測を合わせるかも確認します。

解析前に ObsID、機器、観測モード、露出時間、配布データの処理版を記録します。Chandra では [Data Caveats](https://cxc.cfa.harvard.edu/ciao/caveats/) と [ACIS Data Preparation](https://cxc.cfa.harvard.edu/ciao/guides/acis_data.html) を確認します。XRISM では [ABC Guide の updates と ongoing issues](https://heasarc.gsfc.nasa.gov/docs/xrism/analysis/abc_guide/) に加え、観測ログと Resolve の processing notes を確認します。既知の問題がその観測に当てはまるなら、後のスクリーニングや応答生成を変える必要があります。

## ソフトと校正

Chandra のイベント再処理は CIAO と Chandra CALDB、XRISM のデータ処理は HEASoft と XRISM CALDB の組み合わせを使います。DS9 は画像と領域の確認、XSPEC はスペクトルフィットに使えます。導入と更新は [CIAO 公式ダウンロード](https://cxc.cfa.harvard.edu/ciao/download/)、[HEASoft 公式ページ](https://heasarc.gsfc.nasa.gov/docs/software/lheasoft/)、[XRISM ABC Guide](https://heasarc.gsfc.nasa.gov/docs/xrism/analysis/abc_guide/)に従ってください。macOS、特に Apple Silicon では、その時点の公式対応プラットフォームも確認します。

CIAO と HEASoft のコマンドが同名でも、設定ファイルや校正の参照先は同じとは限りません。別々の端末セッションで有効にし、実行するコマンドの場所を確認すると、誤った環境で処理する事故を減らせます。

```sh
command -v chandra_repro
command -v xselect
command -v xspec
```

上の三つが一つの環境ですべて見つかる必要はありません。**その作業で使うソフトが想定した場所から実行されること**を確認します。使った CIAO・HEASoft・CALDB の版、実行日、変更した設定を解析ノートに残します。

## 元データと作業結果を分ける

配布されたファイルは原本として保存し、再処理や抽出は別の場所で行います。以下は `Work/N132D` のような観測対象ごとの整理を参考にしつつ、第三者が見て追いやすいように簡略化した例です。実在のユーザーパスを前提にはしません。

```text
project/
  README.md                 # 目的、ObsID、使用版
  archive/                  # ダウンロードした原本
  analysis/
    chandra/<OBSID>/        # 観測ごとの再処理と出力
    xrism/<OBSID>/          # Resolve と Xtend の出力
  regions/                  # 線源・Background・除外領域
  logs/                     # 実行履歴と判断
  figures/                  # 図の元データと完成図
```

ツールが自動生成する ObsID や `repro/` の構造は維持します。領域ファイルとスクリプトには、どの観測・エネルギー帯・座標系に使ったかが分かる名前を付けます。複数 ObsID を重ねる場合でも、まず観測ごとに校正、Background、応答を作ります。
