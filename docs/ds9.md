# DS9 を用いた Chandra SNR 画像解析

このページでは、Chandra ACIS の再処理済みイベントファイルを用いて、超新星残骸（SNR）の画像を DS9 で確認し、energy-band 画像、3-color image、contour、他画像との重ね描き、region の保存までを説明します。

入力は `chandra_repro` で作成した Level 2 event file です。

```text
repro/acisfXXXXX_repro_evt2.fits
```

本ページでは、次の用語を区別します。

- event file：各 X 線光子の位置、energy、時刻などを含む表形式の FITS ファイル
- counts image：指定した energy band のイベント数を sky pixel ごとに数えた画像
- exposure-corrected image：有効面積や露出時間の空間変化を補正した画像
- display smoothing：DS9 上の表示だけを滑らかにする操作。元の FITS データは変わらない
- processed smoothed image：`csmooth` などで作成した新しい FITS 画像

以下の例は CIAO 4.18 を基準とします。DS9 のバージョンによってメニュー名がわずかに異なる場合があります。

## 目次

- [1. 最短の作業手順](#1-最短の作業手順)
- [2. DS9 の起動と基本表示](#2-ds9-の起動と基本表示)
- [3. Energy band 画像を作る](#3-energy-band-画像を作る)
- [4. 3-color image を作る](#4-3-color-image-を作る)
- [5. Contour を作る](#5-contour-を作る)
- [6. 別の画像と重ねる](#6-別の画像と重ねる)
- [7. Region を作成して保存する](#7-region-を作成して保存する)
- [8. 画像と作業状態を保存する](#8-画像と作業状態を保存する)
- [9. 再現性のために残す記録](#9-再現性のために残す記録)
- [付録A：一連の command 例](#付録a一連の-command-例)
- [よくある間違い](ds9-common-mistakes.md)

## 1. 最短の作業手順

初めて作業するときは、次の順番で進めます。

1. `chandra_repro` 後の event file を DS9 で開く。
2. soft、medium、hard band の counts image または exposure-corrected image を作る。
3. 3 枚が同じ sky grid と WCS を持つことを確認する。
4. DS9 の RGB frame に soft → red、medium → green、hard → blue の順に読み込む。
5. 各 channel の scale、表示範囲、smoothing を調整する。
6. contour に用いる画像を別 frame で開き、contour を作成して WCS 座標で対象 frame に貼り付ける。
7. region、contour、DS9 の作業状態、完成画像をそれぞれ別に保存する。

完成画像だけでは解析条件を再現できません。少なくとも、入力 FITS、energy band、bin size、scale、scale limits、smoothing、contour levels を記録してください。

## 2. DS9 の起動と基本表示

### 2.1 CIAO 環境から起動する

CIAO を有効にした端末で次を実行します。

```sh
ds9 repro/acisfXXXXX_repro_evt2.fits &
```

event file を開くと、DS9 は event table を sky 座標で binning して表示します。これは表示用にその場で作られた画像であり、画像 FITS が自動的に保存されたわけではありません。

### 2.2 最初に確認する項目

- 対象天体が視野内の想定位置にあるか
- chip gap、視野端、bad pixel に由来する構造がないか
- 座標表示が WCS、FK5 または ICRS になっているか
- event file の energy 単位が eV であることを理解しているか
- 使用中のファイル名と frame 番号が一致しているか

座標は画面上部の座標表示から変更できます。複数波長との比較では WCS と FK5 または ICRS を使います。`image` や `physical` は pixel 座標であり、異なる画像間の位置比較には原則として使わないことが基本です。

### 2.3 Scale と Colormap

X 線画像では dynamic range が広いため、最初は次を試します。

- faint structure を見たい：`Scale → sqrt` または `asinh`
- 明るい部分と暗い部分を広く見る：`Scale → log`
- pixel 値を直感的に比較したい：`Scale → linear`
- 自動範囲：`Scale → zscale`
- 手動範囲：`Scale → Scale Parameters` で minimum と maximum を指定

マウスで color bar を左右に動かすと contrast、上下に動かすと bias を調整できます。見栄えだけで決めず、必要に応じて pixel 値と background level を確認します。

### 2.4 Bin、Block、Zoom の違い

- Zoom：画面表示の拡大率だけを変える。
- Block：表示時に複数 pixel をまとめる。表示の見え方は変わるが、元ファイルは変わらない。
- event file の Bin：event を画像化するときの pixel size を変える。
- `dmcopy` の `bin x=..., y=...`：新しい binned image FITS を作る。

解析結果を再現する場合は、DS9 上の見た目だけではなく、画像作成時の bin size を記録する必要があります。

### 2.5 Smoothing

DS9 の Smooth は display smoothing です。たとえば Gaussian smoothing を選択し、radius と sigma を調整します。表示を保存した PNG には smoothing が反映されますが、元の FITS pixel 値は変わりません。

注意点は次のとおりです。

- smoothing は低カウント領域に存在しない構造を作って見せることがある
- 3-color image では、原則として 3 band に同じ smoothing 設定を適用する
- contour の科学的な再現性が必要な場合は、表示 smoothing だけに依存せず、平滑化済み FITS と条件を保存する
- counts image に `csmooth` を使う場合、入力条件と significance threshold を記録する

### 2.6 複数 frame の比較

複数の画像を開いた後、基準にする frame を選び、次を使います。

- `Frame → Match → WCS`：現在の frame に他の frame の表示範囲を一度だけ合わせる
- `Frame → Lock → WCS`：pan、zoom、回転などを継続して同期する
- `Frame → Tile`：画像を並べる
- `Frame → Blink`：frame を切り替えて位置ずれや形態差を見る

WCS lock は画像データを再投影しません。各画像を、それぞれの WCS を用いて対応する空の位置に表示しているだけです。

## 3. Energy band 画像を作る

### 3.1 方法 A：`dmcopy` で counts image を作る

同じ event file から 3 band を切り出す基本例を示します。energy は eV 単位で指定します。

可変部

```sh
EVT="repro/acisfXXXXX_repro_evt2.fits"

XMIN=3000
XMAX=5000
YMIN=3000
YMAX=5000
BIN=2

SOFT_LO=500
SOFT_HI=1200
MEDIUM_LO=1200
MEDIUM_HI=2000
HARD_LO=2000
HARD_HI=7000
```

`XMIN` などは例です。対象を含む範囲は DS9 または `dmlist` で確認して変更してください。3 画像で同一の sky range と bin size を使います。

基盤

```sh
mkdir -p image
```

処理部

```sh
dmcopy "${EVT}[energy=${SOFT_LO}:${SOFT_HI}][bin x=${XMIN}:${XMAX}:${BIN},y=${YMIN}:${YMAX}:${BIN}]" \
    image/soft_counts.fits clobber=yes

dmcopy "${EVT}[energy=${MEDIUM_LO}:${MEDIUM_HI}][bin x=${XMIN}:${XMAX}:${BIN},y=${YMIN}:${YMAX}:${BIN}]" \
    image/medium_counts.fits clobber=yes

dmcopy "${EVT}[energy=${HARD_LO}:${HARD_HI}][bin x=${XMIN}:${XMAX}:${BIN},y=${YMIN}:${YMAX}:${BIN}]" \
    image/hard_counts.fits clobber=yes
```

この例では soft=0.5–1.2 keV、medium=1.2–2.0 keV、hard=2.0–7.0 keV としています。これは Chandra Source Catalog の標準 band に対応します。SNR の科学目的に応じて、O、Ne、Mg、Si、S、Fe-L、Fe-K、continuum などを意識した band に変更してよいですが、狭い band を単純に特定元素の分布とみなしてはいけません。continuum、隣接輝線、吸収、応答の energy 依存性が混ざります。

画像を確認する

```sh
ds9 image/soft_counts.fits image/medium_counts.fits image/hard_counts.fits &
```

DS9 で Tile、Match WCS、Lock WCS を使い、視野と位置が一致することを確認します。

### 3.2 方法 B：`fluximage` で exposure-corrected image を作る

チップ間や視野内の exposure、effective area の違いを補正した morphology を比較する場合は `fluximage` を使います。

```sh
punlearn fluximage
fluximage "repro/acisfXXXXX_repro_evt2.fits" flux/ \
    bands=csc bin=2 clobber=yes
```

`bands=csc` は soft、medium、hard の 3 band を作る短縮指定です。主な出力は次のとおりです。

```text
flux/soft_flux.img
flux/medium_flux.img
flux/hard_flux.img
```

標準 band は次のとおりです。

| Band | Energy range | 標準 effective energy |
| --- | --- | --- |
| soft | 0.5–1.2 keV | 0.92 keV |
| medium | 1.2–2.0 keV | 1.56 keV |
| hard | 2.0–7.0 keV | 3.8 keV |
| broad | 0.5–7.0 keV | 2.3 keV |

任意 band は `lo:hi:effective_energy` の形式で指定できます。

```sh
fluximage "repro/acisfXXXXX_repro_evt2.fits" flux_si/ \
    bands=1.7:2.1:1.85 bin=2 clobber=yes
```

単色 effective energy による exposure correction は、band 内の実際の spectrum と一致しない場合があります。特に soft band では ACIS contamination の影響に注意が必要です。定量解析では、目的に合う effective energy または spectral weights を検討します。

### 3.3 Counts image と exposure-corrected image の使い分け

- photon の実カウント分布を確認：counts image
- chip gap や exposure variation を抑えて morphology を比較：exposure-corrected image
- `csmooth` の基本入力：原則として counts image。必要に応じて background、scale map を与える
- 表示用 RGB：目的に応じてどちらでもよいが、3 channel で処理方針を統一する
- contour level を count 単位で定義：counts image
- 異なる観測・機器の surface brightness を比較：calibration と単位を確認した補正画像

## 4. 3-color image を作る

### 4.1 色の割り当て

X 線の energy を可視色に対応させる一般的な割り当ては次のとおりです。

| RGB channel | X 線 band | 意味 |
| --- | --- | --- |
| Red | soft | 低 energy |
| Green | medium | 中間 energy |
| Blue | hard | 高 energy |

これは表示上の符号化であり、X 線 photon が実際にその可視色を持つわけではありません。

### 4.2 GUI で作る

1. `Frame` メニューから新しい RGB Frame を作る。
2. RGB dialog を開く。
3. Red channel を選び、`File → Open` で `soft_counts.fits` または `soft_flux.img` を開く。
4. Green channel を選び、medium image を開く。
5. Blue channel を選び、hard image を開く。
6. RGB の座標 system を WCS にする。
7. 3 channel が同じ位置に重なることを point source、rim、chip edge などで確認する。
8. channel ごとに表示を調整し、scale と limits を決める。
9. 全 channel を表示し、色の balance を確認する。

最初から channel 間の scale limits を lock しない方がよいです。各 band は count range が異なるため、同一の数値範囲では一部の channel が消えることがあります。ただし、恣意的に 1 channel だけ強調しすぎないようにし、採用した limits を記録してください。

### 4.3 XPA で再現する

DS9 を起動した状態で、次のコマンドを実行できます。

可変部

```sh
RED_IMAGE="image/soft_counts.fits"
GREEN_IMAGE="image/medium_counts.fits"
BLUE_IMAGE="image/hard_counts.fits"
```

基盤

```sh
xpaset -p ds9 rgb
xpaset -p ds9 rgb system wcs
```

処理部

```sh
xpaset -p ds9 rgb channel red
xpaset -p ds9 fits "$RED_IMAGE"
xpaset -p ds9 scale asinh

xpaset -p ds9 rgb channel green
xpaset -p ds9 fits "$GREEN_IMAGE"
xpaset -p ds9 scale asinh

xpaset -p ds9 rgb channel blue
xpaset -p ds9 fits "$BLUE_IMAGE"
xpaset -p ds9 scale asinh

xpaset -p ds9 rgb view red yes
xpaset -p ds9 rgb view green yes
xpaset -p ds9 rgb view blue yes
xpaset -p ds9 wcs align yes
```

手動で良い表示範囲を決めた後、各 channel に scale limits を設定すると再現しやすくなります。

```sh
xpaset -p ds9 rgb channel red
xpaset -p ds9 scale limits RED_MIN RED_MAX

xpaset -p ds9 rgb channel green
xpaset -p ds9 scale limits GREEN_MIN GREEN_MAX

xpaset -p ds9 rgb channel blue
xpaset -p ds9 scale limits BLUE_MIN BLUE_MAX
```

`RED_MIN` などは実際の数値に置き換えます。

### 4.4 RGB で不自然な色を作らないための確認

- 3 画像の WCS、画像範囲、pixel size が同じか
- 3 channel で異なる smoothing scale を使っていないか
- exposure correction の有無が channel 間で混在していないか
- background level の差が色として現れていないか
- chip gap や視野端が特定色になっていないか
- point source の pile-up や readout streak が色を支配していないか
- absorption の違いを温度や元素組成の違いと即断していないか

## 5. Contour を作る

### 5.1 どの画像から contour を作るか

Contour は、背景画像とは別の量の空間分布を線で示すときに使います。SNR 解析では次の例が多いです。

- broad-band X 線画像上に hard X-ray contour
- X 線画像上に radio continuum contour
- X 線画像上に CO integrated-intensity contour
- optical image 上に X 線 contour

低カウントの raw image から直接 contour を作ると、pixel noise を追った線になる可能性があります。必要に応じて binning または smoothing を行い、処理条件を保存します。

### 5.2 同じ frame 上に contour を表示する

1. contour source image を開く。
2. `Analysis → Contours` を有効にする。
3. `Analysis → Contour Parameters` を開く。
4. level 数、scale、smoothness、method を設定する。
5. 必要なら自動 level ではなく、物理的に意味のある数値を直接指定する。
6. 色、線幅、実線／破線を設定する。

level は「見栄えの良い本数」だけで決めないでください。background rms、significance、peak の一定割合、または明示した surface-brightness 値を基準にするのがよいです。

例：radio image の background rms が `sigma_rms` のとき

```sh
3 sigma_rms, 5 sigma_rms, 10 sigma_rms, 20 sigma_rms, ...
```

### 5.3 別画像へ contour を貼り付ける

1. contour source image の frame で contour を作る。
2. `Analysis → Contours → Copy Contours` を選ぶ。
3. 背景にしたい image の frame に移動する。
4. `Analysis → Contours → Paste Contours` を選ぶ。
5. paste の座標系を WCS にする。
6. point source や既知の天体位置で alignment を確認する。

WCS が正しければ、画像サイズや pixel scale が異なっていても contour を sky 座標で貼り付けられます。WCS がない PNG、JPEG、または不正な FITS header を持つ画像には正しく重なりません。

XPA では次のように操作できます。

```sh
# contour source frame で実行
xpaset -p ds9 contour method smooth
xpaset -p ds9 contour smooth 3
xpaset -p ds9 contour levels 3 5 10 20
xpaset -p ds9 contour generate
xpaset -p ds9 contour copy

# background frame に移動した後に実行
xpaset -p ds9 contour paste wcs cyan 2 no
```

### 5.4 Contour を保存する

GUI では `Contour → Save Contours` を使います。座標系は WCS/FK5 を推奨します。

```sh
xpaset -p ds9 contour save hard_xray.ctr wcs fk5
```

再読込は次で行います。

```sh
xpaset -p ds9 contour load hard_xray.ctr
```

Contour file は、coordinate system、level、線の属性、contour point を含む ASCII file です。

## 6. 別の画像と重ねる

DS9 で「画像を重ねる」ときは、目的に応じて方法を選びます。

### 6.1 Contour overlay

最も一般的で、位置関係を読みやすい方法です。背景画像の intensity を保ちつつ、もう一方の morphology を線で示せます。

推奨例：Chandra RGB image + radio / CO contour。

### 6.2 RGB channel として合成

2 枚または 3 枚の画像を各 color channel に割り当てます。共通構造と異なる構造を色で見分けやすい一方で、scale の選び方によって印象が大きく変わります。

推奨例：soft / medium / hard X-ray、または radio / optical / X-ray の概観図。

### 6.3 Mask overlay

特定の値を持つ領域、検出領域、exposure の低い領域などを半透明色で示す方法です。連続強度画像の比較より、領域の有無を示す用途に向きます。

```sh
xpaset -p ds9 mask system wcs
xpaset -p ds9 mask color red
xpaset -p ds9 mask transparency 50
xpaset -p ds9 mask load mask.fits
```

### 6.4 WCS match / lock と blink

画像を直接合成せず、同じ sky area を交互表示します。細かな位置ずれ、point source の一致、epoch 間の変化を確認しやすいです。

```sh
xpaset -p ds9 match frame wcs
xpaset -p ds9 lock frame wcs
xpaset -p ds9 blink interval 0.5
xpaset -p ds9 blink yes
```

### 6.5 画像を同一 grid に再投影する

RGB 合成、pixel-by-pixel comparison、ratio map では、単なる WCS 表示同期ではなく、同じ配列サイズ、pixel scale、WCS grid に揃える必要があります。

基準画像を Chandra image とし、外部画像を合わせる例は次のとおりです。

```sh
reproject_image \
    infile=radio_input.fits \
    matchfile=image/broad_counts.fits \
    outfile=radio_on_chandra_grid.fits \
    method=average \
    clobber=yes
```

`reproject_image` の出力は基準画像と同じ dimensions と WCS を持ちます。一般に、counts の総和を保存したい画像には `method=sum`、exposure や surface-brightness 型の画像には `method=average` を検討します。入力画像の単位と pixel 値の定義を確認して選んでください。

再投影は astrometric correction ではありません。元画像の WCS が系統的にずれていれば、再投影後も天体位置はずれたままです。必要に応じて共通 point source を用いて astrometry を検証してください。

## 7. Region を作成して保存する

Region は解析領域、位置の印、ラベル、scale bar などに使えます。

### 7.1 基本操作

1. mouse mode を Region にする。
2. circle、ellipse、box、polygon などを選ぶ。
3. 画像上で region を作る。
4. region を double click し、座標、サイズ、色、線幅、label を編集する。
5. source region と background region を色や tag で区別する。

### 7.2 保存時の座標系

異なる画像でも再利用する region は、`Region → Save Regions` で DS9 format、fk5 または icrs を選びます。

```text
# Region file format: DS9 version 4.1
global color=green width=2
fk5
circle(14:41:10.0,-62:38:00,20")
```

image 座標で保存すると、同じ sky area でも pixel grid の異なる画像ではずれてしまうことがあります。Chandra の同一 event file だけに対する CIAO filtering では physical 座標も有用ですが、他波長比較用の保存は sky 座標を基本とします。

Region を画像の装飾に使う場合と、CIAO の解析 filter に使う場合は区別してください。DS9 region の一部、たとえば text、ruler、compass などは CIAO filter として使えません。

## 8. 画像と作業状態を保存する

### 8.1 完成図を PNG / JPEG / TIFF / EPS として保存

`File → Save Image` を使います。これは現在の DS9 window の snapshot であり、colormap、scale、smoothing、contour、region、grid などの表示内容を含みます。

```sh
xpaset -p ds9 saveimage snr_rgb.png
```

JPEG は非可逆圧縮のため、論文・発表用の中間保存には PNG または TIFF を推奨します。

出力解像度は DS9 window size に依存します。保存前に window を十分大きくし、文字、線幅、color bar が読めるサイズを確認します。

### 8.2 表示中の image data を FITS として保存

`File → Save` または次を使います。

```sh
xpaset -p ds9 save fits displayed_image.fits image
xpaset -p ds9 save rgbimage snr_rgb.fits
```

これは snapshot 保存とは異なり、colormap、contour、region などの画面装飾を FITS pixel に焼き込む操作ではありません。

### 8.3 Region を保存

`Region → Save Regions` を使い、DS9 format と座標系を���びます。解析領域は figure とは別ファイルとして保存します。

推奨名：

```text
src.reg
bkg.reg
rim_regions_fk5.reg
point_sources_excluded.reg
```

### 8.4 Contour を保存

```sh
xpaset -p ds9 contour save radio_contours.ctr wcs fk5
```

### 8.5 DS9 の作業状態を保存

`File → Backup` を使うと、frame、読み込んだ data、colormap、contour、region などを含む save set を作成できます。再開時は `Restore` で戻せます。

```sh
xpaset -p ds9 backup snr_work.bck
xpaset -p ds9 restore snr_work.bck
```

Backup file と補助 directory が作成された場合は、両方を同じ場所に保ちます。Backup は解析手順書の代わりではないため、energy band、scale limits、smoothing、contour levels も text file に残してください。

## 9. 再現性のために残す記録

最低限、次の情報を残します。

```text
Target:
ObsID:
Input event file:
CIAO version:
CALDB version:
DS9 version:

Image type: counts / exposure-corrected
Sky grid: x range, y range, bin size

Red band:
Green band:
Blue band:

Scale function:
Scale limits for each channel:
Smoothing function and parameters:

Contour input file:
Contour levels and units:
Contour smoothing:
Contour colour and width:

Coordinate system:
Astrometric correction applied: yes / no
Output files:
```

バージョン確認例：

```sh
ciaover -v
ds9 -version
```

## 付録A：一連の command 例

次は、同じ Chandra event file から counts RGB を作り、DS9 に読み込む最小構成です。sky range は対象に合わせて変更してください。

可変部

```sh
EVT="repro/acisfXXXXX_repro_evt2.fits"
OUTDIR="image"

XMIN=3000
XMAX=5000
YMIN=3000
YMAX=5000
BIN=2

SOFT="500:1200"
MEDIUM="1200:2000"
HARD="2000:7000"
```

基盤

```sh
mkdir -p "$OUTDIR"
```

処理部

```sh
dmcopy "${EVT}[energy=${SOFT}][bin x=${XMIN}:${XMAX}:${BIN},y=${YMIN}:${YMAX}:${BIN}]" \
    "${OUTDIR}/soft_counts.fits" clobber=yes

dmcopy "${EVT}[energy=${MEDIUM}][bin x=${XMIN}:${XMAX}:${BIN},y=${YMIN}:${YMAX}:${BIN}]" \
    "${OUTDIR}/medium_counts.fits" clobber=yes

dmcopy "${EVT}[energy=${HARD}][bin x=${XMIN}:${XMAX}:${BIN},y=${YMIN}:${YMAX}:${BIN}]" \
    "${OUTDIR}/hard_counts.fits" clobber=yes

ds9 &
```

DS9 が起動した後に実行します。

```sh
xpaset -p ds9 rgb
xpaset -p ds9 rgb system wcs

xpaset -p ds9 rgb channel red
xpaset -p ds9 fits "${OUTDIR}/soft_counts.fits"
xpaset -p ds9 scale asinh

xpaset -p ds9 rgb channel green
xpaset -p ds9 fits "${OUTDIR}/medium_counts.fits"
xpaset -p ds9 scale asinh

xpaset -p ds9 rgb channel blue
xpaset -p ds9 fits "${OUTDIR}/hard_counts.fits"
xpaset -p ds9 scale asinh

xpaset -p ds9 rgb view red yes
xpaset -p ds9 rgb view green yes
xpaset -p ds9 rgb view blue yes
xpaset -p ds9 wcs align yes
```

表示を調整した後、次を保存します。

```sh
xpaset -p ds9 saveimage "${OUTDIR}/snr_rgb.png"
xpaset -p ds9 backup "${OUTDIR}/snr_rgb_work.bck"
```

## よくある間違い

- [Energy の単位を取り違える](ds9-common-mistakes.md#energy-の単位を取り違える)
- [RGB channel の画像範囲が異なる](ds9-common-mistakes.md#rgb-channel-の画像範囲が異なる)
- [PNG や JPEG を天球座標で重ねようとする](ds9-common-mistakes.md#png-や-jpeg-を天球座標で重ねようとする)
- [WCS match を再投影とみなす](ds9-common-mistakes.md#wcs-match-を再投影とみなす)
- [Display smoothing を解析済み画像とみなす](ds9-common-mistakes.md#display-smoothing-を解析済み画像とみなす)
- [Contour の低い level を採用しすぎる](ds9-common-mistakes.md#contour-の低い-level-を採用しすぎる)
- [Exposure map の端を実構造とみなす](ds9-common-mistakes.md#exposure-map-の端を実構造とみなす)
- [色を物理量に直接対応させる](ds9-common-mistakes.md#色を物理量に直接対応させる)

!!! warning
    `dmcopy` の event filter では energy を eV で指定します。一方、`fluximage` の `bands` は keV です。単位を取り違えないでください。
