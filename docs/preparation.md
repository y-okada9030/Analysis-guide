# 解析を始める前に

## 目的と観測内容を記録する

まず、科学目的を短く記します。例えば「超新星残骸の殻の温度を位置ごとに比較する」なら、殻を分割する領域、各領域に適した背景、用いるエネルギー帯を事前に明確にしておきます。

解析前に以下の情報を記録します：
- ObsID、使用検出器、観測モード（FAINT/VFAINT）
- 露出時間、配布データの処理版

Chandra では [Data Caveats](https://cxc.cfa.harvard.edu/ciao/caveats/) と [ACIS Data Preparation](https://cxc.cfa.harvard.edu/ciao/guides/acis_data.html) を確認し、観測時期や検出器に関わる既知の問題がないかを確認します。

## ソフトウェアと校正

Chandra データの処理には CIAO と Chandra CALDB、XRISM データには HEASoft と XRISM CALDB を用います。画像と領域の確認には DS9、スペクトル解析には XSPEC を使います。

CIAO と HEASoft のコマンドが同じ名前でも、設定ファイルや校正の参照先は異なることがあります。異なる端末セッションで個別に有効にして、実行するコマンドが正しい環境から来ていることを確認しましょう。

```sh
command -v chandra_repro
command -v xselect
command -v xspec
```

3つのコマンドすべてが 1つの環境に存在する必要はありません。**その作業で使うソフトが、ツールの要求する場所から実行されること**を確認します。使用した CIAO、HEASoft、CALDB のバージョンと日付を���析記録に残します。

## 元データと処理結果を分ける

配布されたファイルは原本として保存し、再処理や抽出は別の作業ディレクトリで行います。以下は `Work/N132D` のような観測対象ごとの整理例です。自分のプロジェクトに合わせて調整してください。

```text
project/
  README.md                 # 科学目的、ObsID、使用ソフト版
  archive/                  # ダウンロードした原本
  analysis/
    chandra/<OBSID>/        # 観測ごとの再処理と解析出力
    xrism/<OBSID>/          # Resolve と Xtend の出力
  regions/                  # 線源領域、背景領域、除外領域
  logs/                     # コマンド実行履歴と判断記録
  figures/                  # 図の元データと完成図
```

ツールが自動生成する ObsID や `repro/` などのディレクトリ構造は、ツール側の要求に従って維持します。領域ファイルとスクリプトには、どの観測・エネルギー帯・座標系に用いたのかが判明するよう記名します。
