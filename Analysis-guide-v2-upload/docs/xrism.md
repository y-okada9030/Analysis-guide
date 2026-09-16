# XRISM：広がった線源

XRISM の解析は [ABC Data Reduction Guide v2.0](https://heasarc.gsfc.nasa.gov/docs/xrism/analysis/abc_guide/) を基準にします。これは Resolve と Xtend を一冊で扱いますが、両者のイベント選択と応答の作り方は別です。この章は超新星残骸や銀河団などを想定し、**線源の広がりと視野内の明るさの分布を応答にどう反映するか**を中心に説明します。ABC Guide の N132D 例の領域、座標、ピクセル設定を他の天体へコピーしてはいけません。

## 1. 観測の状態と cleaned event を確認する

データを取得したら、観測ログ、Resolve の processing notes、quick-look products を読みます。[ABC Guide 第5章](https://heasarc.gsfc.nasa.gov/docs/xrism/analysis/abc_guide/XRISM_Data_Analysis.html) は HEASoft と CALDB の更新、標準スクリーニング、姿勢の安定性、追加スクリーニングを確認する順序を示しています。配布済み cleaned event から始めてよいか、再処理や特定時間帯の除外が必要かをここで判断します。

Resolve では明るさによる grade の分布、PSP overflow、ノイズの多いピクセル、粒子由来の Background を確認します。Xtend では窓モード、flickering pixels、宇宙線の echo、flare、pile-up を確認します。これは「問題がありそうなら後で直す」作業ではありません。**除外した時間・ピクセル・grade は、PHA と応答の入力を揃えるために最初に決めます。**

```sh
fhelp xselect
fhelp xaexpmap
fhelp xaarfgen
```

ツールのパラメータは版で変わるため、上のヘルプと ABC Guide の現行版を照合します。

## 2. Resolve：高分解能スペクトル

Resolve では各ピクセルが空の異なる範囲から光子を受け取ります。広がった対象は望遠鏡の PSF により隣接領域の放射が混ざるので、**どのピクセルのスペクトルを抽出し、線源をどの画像・輝度分布で表すか**を決めます。まず [ABC Guide 6.2–6.6](https://heasarc.gsfc.nasa.gov/docs/xrism/analysis/abc_guide/Resolve_Data_Analysis.html) に従い、対象の明るさ、追加スクリーニング、grade とピクセル選択を確認します。

次は Hp grade の抽出を理解するための XSELECT 入力例です。**XSELECT 内**で入力します。`<RESOLVE_CLEANED_EVT>` は実在する配布ファイルに置き換えます。実際の広がった線源では、公式 6.6 の pixel region と適切な screening を加え、単に全ピクセルを採用しないでください。

```sh
xselect
```

```text
read events <RESOLVE_CLEANED_EVT> .
filter GRADE 0:0
extr spectrum
save spectrum resolve_hp.pi
```

PHA を作った後、[ABC Guide 6.7](https://heasarc.gsfc.nasa.gov/docs/xrism/analysis/abc_guide/Resolve_Data_Analysis.html) の RMF+ARF または `rslmkrsp` の方法を選びます。RMF には対象の grade、ピクセル、時間帯を反映させます。ARF は点源の `POINT` 入力ではなく、**extended-source の IMAGE mode** など、空の輝度分布を表す入力に従って作ります。その画像は「見栄えのために平滑化した図」とは異なり、ray tracing に渡す線源モデルです。線源の形と抽出ピクセルに応じた適用範囲を確認します。

`xaarfgen` には姿勢、ray-tracing、画像、エネルギー格子など観測固有の入力が多いため、この章では埋めかけの汎用コマンドを掲載しません。[公式の Extended Source: IMAGE Mode と Alternative Methods](https://heasarc.gsfc.nasa.gov/docs/xrism/analysis/abc_guide/Resolve_Data_Analysis.html) から、対象の形態に合う完全な例を選んでください。複数の空領域からの放射を分光的に分けたいときは同章の spatial-spectral mixing を使います。NXB は [公式 6.8](https://heasarc.gsfc.nasa.gov/docs/xrism/analysis/abc_guide/Resolve_Data_Analysis.html) に従い、単純に隣のピクセルを「空の背景」とみなしません。

## 3. Xtend：広い視野の画像と CCD スペクトル

Xtend は画像で放射の広がりと周辺天体を見られますが、チップ境界や窓モード、露出の違いを意識して領域を作ります。[ABC Guide 7.3–7.6](https://heasarc.gsfc.nasa.gov/docs/xrism/analysis/abc_guide/Xtend_Data_Analysis.html) に従い、cleaned event と追加 screening を確認した上で、XSELECT で画像を作ります。以下は **XSELECT 内**の入力例です。

```text
read events <XTEND_CLEANED_EVT> .
set image sky
extract image
save image xtend_sky.img
```

DS9 で元画像を見て、線源領域と Background 領域を保存します。対象が視野全体に広がる場合、見かけ上暗い部分でも天体の背景放射が残っている可能性があります。Background の選び方を先に決め、[公式 7.6.4–7.6.6](https://heasarc.gsfc.nasa.gov/docs/xrism/analysis/abc_guide/Xtend_Data_Analysis.html) の領域形式と座標系に合わせて線源・Background PHA を抽出します。領域ファイルは XSELECT と応答生成ツールで使う座標系が一致するか再表示して確認します。

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

線源と Background の `BACKSCAL`、GTI、screening が合うか確認します。RMF は [ABC Guide 7.7.1](https://heasarc.gsfc.nasa.gov/docs/xrism/analysis/abc_guide/Xtend_Data_Analysis.html) の `xtdrmf`、ARF は 7.7.2 の `xaarfgen` を使います。広がった線源の ARF では線源の画像・輝度分布、抽出領域、ray-tracing、露出マップが関係します。点源用の指定を流用すると、明るい場所から暗い場所への PSF 混入や位置依存の有効面積を正しく扱えません。NXB と pile-up は [公式 7.8–7.9](https://heasarc.gsfc.nasa.gov/docs/xrism/analysis/abc_guide/Xtend_Data_Analysis.html) を確認します。

## 4. Resolve と Xtend を比較するとき

同じ対象でも、両機器の視野、抽出領域、エネルギー分解能は違います。Resolve の高分解能線と Xtend の広い領域のスペクトルをそのまま同一の放射成分とみなさず、各機器の抽出範囲に入る線源成分を確認します。モデルを同時に当てる場合は、機器間の規格化や sky background（空の背景放射）を検討し、**各スペクトルに固有の Background・RMF・ARF**を保持します。異なる grade や領域の応答を一つにコピーしないでください。
