# XRISM：広がった線源

XRISM の解析は [ABC Data Reduction Guide v2.0](https://heasarc.gsfc.nasa.gov/docs/xrism/analysis/abc_guide/) を基準にします。これは Resolve と Xtend を一冊で扱いますが、観測モード、検出器構成、screening は異なるため、それぞれの章を読み分ける必要があります。

## 1. 観測の状態と cleaned event を確認する

データを取得したら、観測ログ、Resolve の processing notes、quick-look products を読みます。[ABC Guide 第5章](https://heasarc.gsfc.nasa.gov/docs/xrism/analysis/abc_guide/XRI_00_ABC_Guide.html) の screening が観測に適用されているか確認します。

Resolve では明るさによる grade の分布、PSP overflow、ノイズの多いピクセル、粒子由来の Background を確認します。Xtend では窓モード、flickering pixels、宇宙線によるノイズを見ます。

```sh
fhelp xselect
fhelp xaexpmap
fhelp xaarfgen
```

ツールのパラメータは版で変わるため、上のヘルプと ABC Guide の現行版を照合します。

## 2. Resolve：高分解能スペクトル

Resolve では各ピクセルが空の異なる範囲から光子を受け取ります。広がった対象は望遠鏡の PSF により隣接領域の放射が混ざるので、**どのピクセル範囲を抽出するか**が結果に大きく影響します。

次は Hp grade の抽出を理解するための XSELECT 入力例です。**XSELECT 内**で入力します。`<RESOLVE_CLEANED_EVT>` は実在する配布ファイルに置き換えます。実際の解析では ABC Guide の指示に従ってください。

```sh
xselect
```

```text
read events <RESOLVE_CLEANED_EVT> .
filter GRADE 0:0
extr spectrum
save spectrum resolve_hp.pi
```

PHA を作った後、[ABC Guide 6.7](https://heasarc.gsfc.nasa.gov/docs/xrism/analysis/abc_guide/Resolve_Data_Analysis.html) の RMF+ARF または `rslmkrsp` の方法を選びます。RMF には grade や position dependence が反映されます。

`xaarfgen` には姿勢、ray-tracing、画像、エネルギー格子など観測固有の入力が多いため、この章では埋めかけの汎用コマンドを掲載しません。[公式 ABC Guide](https://heasarc.gsfc.nasa.gov/docs/xrism/analysis/abc_guide/Resolve_Data_Analysis.html) に従って実行します。

## 3. Xtend：広い視野の画像と CCD スペクトル

Xtend は画像で放射の広がりと周辺天体を見られますが、チップ境界や窓モード、露出の違いを意識して領域を作ります。[ABC Guide 7.3–7.6](https://heasarc.gsfc.nasa.gov/docs/xrism/analysis/abc_guide/Xtend_Data_Analysis.html) を参照し、screening と grade を確認します。

XSELECT で画像を作ります。

```text
read events <XTEND_CLEANED_EVT> .
set image sky
extract image
save image xtend_sky.img
```

DS9 で元画像を見て、線源領域と Background 領域を保存します。対象が視野全体に広がる場合、見かけ上暗い部分でも天体の背景放射が残っている可能性があります。Background を選ぶ際は source-free region を確認します。

線源領域と Background 領域それぞれからスペクトルを抽出します。

```text
filter region source.reg
extr spectrum
save spec xtend_source.pi
clear region
filter region background.reg
extr spectrum
save spec xtend_background.pi
clear region
```

線源と Background の `BACKSCAL`、GTI、screening が合うか確認します。RMF は [ABC Guide 7.7.1](https://heasarc.gsfc.nasa.gov/docs/xrism/analysis/abc_guide/Xtend_Data_Analysis.html) の手順で作ります。ARF には effective area と、detector の検量が含まれます。

## 4. Resolve と Xtend を比較するとき

同じ対象でも、両機器の視野、抽出領域、エネルギー分解能は違います。Resolve の高分解能線と Xtend の広い領域のスペクトルをそのまま同一の放射モデルでフィットすべきではありません。比較する際は、ツール、視野、screen、抽出領域を明記します。
