# DS9 を用いた Chandra SNR 画像解析

このページでは、Chandra ACIS の再処理済みイベントファイルから、超新星残骸（SNR）の画像を DS9 で確認し、energy-band image、3-color image、contour、他の波長の画像との重ね合わせ、region の保存を行う方法を説明します。

入力は、`chandra_repro` で作成した Level 2 event file です。

```text
repro/acisfXXXXX_repro_evt2.fits
```

FITS ファイルのヘッダーや WCS、`fv` の使い方は、[FITS ファイルとヘッダーの読み方](fits.md)を参照してください。

## 目次

- [1. 作業の流れ](#1-作業の流れ)
- [2. DS9 の起動と基本表示](#2-ds9-の起動と基本表示)
- [3. Energy band 画像を作る](#3-energy-band-画像を作る)
- [4. 3-color image を作る](#4-3-color-image-を作る)
- [5. Contour を作る](#5-contour-を作る)
- [6. 他の画像を重ねる](#6-他の画像を重ねる)
- [7. Region を作成して保存する](#7-region-を作成して保存する)
- [8. 画像と作業状態を保存する](#8-画像と作業状態を保存する)
- [9. 再現性のために記録する情報](#9-再現性のために記録する情報)
- [よくある間違い](ds9-common-mistakes.md)

## 1. 作業の流れ

1. `chandra_repro` 後の event file を DS9 で開く。
2. soft、medium、hard の energy band image を作る。
3. 3 枚の画像が同じ sky grid と WCS を持つことを確認する。
4. soft → red、medium → green、hard → blue の順に RGB frame へ読み込む。
5. scale、表示範囲、smoothing を調整する。
6. 必要に応じて contour を作り、WCS 座標で別の画像に重ねる。
7. region、contour、DS9 の作業状態、完成画像をそれぞれ保存する。

完成画像だけでは解析条件を再現できません。入力 FITS、energy band、bin size、scale limits、smoothing、contour levels を記録してください。

## 2. DS9 の起動と基本表示

### 2.1 起動

CIAO を有効にした端末で、次を実行します。

```sh
ds9 repro/acisfXXXXX_repro_evt2.fits &
```

event file を開くと、DS9 は event table を sky 座標上で画像化して表示します。この表示画像が自動的に FITS として保存されるわけではありません。保存したい場合は、別途画像を作成します。

### 2.2 最初に確認する項目

- 対象天体が想定した位置にあるか
- chip gap、視野端、bad pixel に由来する構造がないか
- 座標表示が WCS、FK5 または ICRS になっているか
- event file の `energy` の単位が eV であること
- 使用中のファイル名と frame 番号が一致しているか

画像のヘッダーや WCS を確認する方法は、[FITS ファイルとヘッダーの読み方](fits.md#2-まず確認するヘッダー)にまとめています。

### 2.3 Scale と Colormap

X 線画像は明るさの範囲が広いため、目的に応じて scale を選びます。

- 暗い構造を見やすくする：`Scale → sqrt` または `asinh`
- 明るさの範囲を広く表示する：`Scale → log`
- pixel 値を直接比較する：`Scale → linear`
- 自動範囲を使う：`Scale → zscale`
- 範囲を手動で指定する：`Scale → Scale Parameters`

マウスで color bar を左右に動かすと contrast、上下に動かすと bias を調整できます。見栄えだけで決めず、必要に応じて pixel 値と背景レベルも確認してください。

### 2.4 Bin、Block、Zoom の違い

- **Zoom**：画面の拡大率を変える。元データは変わらない。
- **Block**：表示時に複数 pixel をまとめる。元ファイルは変わらない。
- **event file の bin**：event を画像化するときの pixel size を決める。
- **`dmcopy` の bin**：新しい binned image FITS を作る。

解析を再現するには、画面上の見た目だけでなく、画像作成時の bin size を記録します。

### 2.5 Smoothing

DS9 の Smooth は表示だけを平滑化する操作です。Gaussian smoothing の radius や sigma を変更しても、元の FITS pixel 値は変わりません。

低カウント領域では、平滑化によって実際には存在しない構造が見えることがあります。3-color image では、原則として 3 band に同じ smoothing 条件を適用してください。`csmooth` などで解析用の平滑化画像を作る場合は、新しい FITS と入力条件を保存します。

### 2.6 複数 frame の比較

- `Frame → Match → WCS`：表示範囲を一度合わせる
- `Frame → Lock → WCS`：pan、zoom、回転などを継続して同期する
- `Frame → Tile`：画像を並べる
- `Frame → Blink`：画像を切り替えて位置ずれや形態差を確認する

WCS の match や lock は表示を同期するだけで、画像データを再投影する操作ではありません。

## 3. Energy band 画像を作る

### 3.1 `dmcopy` で counts image を作る

`dmcopy` では、event file の `energy` を通常 eV 単位で指定します。次の例では、3 band で同じ sky range と bin size を使います。

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

mkdir -p image

dmcopy "${EVT}[energy=${SOFT_LO}:${SOFT_HI}][bin x=${XMIN}:${XMAX}:${BIN},y=${YMIN}:${YMAX}:${BIN}]" \
  image/soft_counts.fits clobber=yes

dmcopy "${EVT}[energy=${MEDIUM_LO}:${MEDIUM_HI}][bin x=${XMIN}:${XMAX}:${BIN},y=${YMIN}:${YMAX}:${BIN}]" \
  image/medium_counts.fits clobber=yes

dmcopy "${EVT}[energy=${HARD_LO}:${HARD_HI}][bin x=${XMIN}:${XMAX}:${BIN},y=${YMIN}:${YMAX}:${BIN}]" \
  image/hard_counts.fits clobber=yes
```

この例の band は soft=0.5–1.2 keV、medium=1.2–2.0 keV、hard=2.0–7.0 keV です。SNR の目的に応じて、O、Ne、Mg、Si、S、Fe-L、Fe-K、continuum などを意識した band に変更できます。ただし、狭い band の画像を単純に特定元素の分布とみなしてはいけません。continuum、隣接輝線、吸収、検出器応答の energy 依存性が混ざ���ためです。

画像を確認します。

```sh
ds9 image/soft_counts.fits image/medium_counts.fits image/hard_counts.fits &
```

### 3.2 `fluximage` で exposure-corrected image を作る

chip 間や視野内の exposure、effective area の違いを補正して形態を比較する場合は、`fluximage` を使います。

```sh
punlearn fluximage
fluximage "repro/acisfXXXXX_repro_evt2.fits" flux/ \
  bands=csc bin=2 clobber=yes
```

主な出力は次のとおりです。

```text
flux/soft_flux.img
flux/medium_flux.img
flux/hard_flux.img
```

`fluximage` の `bands` と、`dmcopy` の `energy` filter では単位が異なる場合があるため、使用する CIAO のマニュアルを確認してください。

単一の effective energy による補正は、band 内の実際の spectrum と一致しない場合があります。特に soft band では ACIS contamination の影響に注意してください。定量解析では、目的に合った effective energy や spectral weights を検討します。

### 3.3 画像の使い分け

- photon の実カウント分布を確認する：counts image
- chip gap や exposure variation を補正して形態を比較する：exposure-corrected image
- `csmooth` の入力にする：原則として counts image。必要に応じて background や scale map を与える
- 表示用 RGB に使う：どちらでもよいが、3 channel で方針を統一する
- count 単位で contour level を定義する：counts image

## 4. 3-color image を作る

### 4.1 色の割り当て

一般的には、soft → red、medium → green、hard → blue とします。これは表示上の符号化であり、X 線光子が実際にその可視色を持つわけではありません。

### 4.2 GUI で作る

1. `Frame` メニューから RGB Frame を作る。
2. RGB dialog を開く。
3. Red channel に soft image を読み込む。
4. Green channel に medium image を読み込む。
5. Blue channel に hard image を読み込む。
6. RGB の座標 system を WCS にする。
7. point source、rim、chip edge などで 3 channel の位置が一致することを確認する。
8. channel ごとに scale と limits を調整する。
9. 全 channel を表示して、色の balance を確認する。

各 band は count range が異なるため、最初から全 channel の scale limits を共通にする必要はありません。ただし、採用した limits を記録し、特定の channel だけを恣意的に強調しないようにします。

### 4.3 XPA で再現する

DS9 を起動した状態で、次を実行します。

```sh
RED_IMAGE="image/soft_counts.fits"
GREEN_IMAGE="image/medium_counts.fits"
BLUE_IMAGE="image/hard_counts.fits"

xpaset -p ds9 rgb
xpaset -p ds9 rgb system wcs

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

scale limits を固定する場合は、実際の値に置き換えます。

```sh
xpaset -p ds9 rgb channel red
xpaset -p ds9 scale limits RED_MIN RED_MAX
xpaset -p ds9 rgb channel green
xpaset -p ds9 scale limits GREEN_MIN GREEN_MAX
xpaset -p ds9 rgb channel blue
xpaset -p ds9 scale limits BLUE_MIN BLUE_MAX
```

## 5. Contour を作る

Contour は、背景画像とは別の量の空間分布を線で示すために使います。例えば、X 線画像上に hard X-ray、radio continuum、CO integrated intensity の contour を重ねます。

低カウントの画像から直接 contour を作ると、pixel noise を追った線になることがあります。必要に応じて binning や smoothing を行い、処理条件を保存してください。

### 5.1 同じ frame 上に表示する

1. contour source image を開く。
2. `Analysis → Contours` を有効にする。
3. `Analysis → Contour Parameters` で level、scale、smoothness、method を設定する。
4. 必要なら自動 level ではなく、物理的に意味のある値を指定する。
5. 色、線幅、実線／破線を設定する。

level は見栄えだけで決めず、background rms、significance、peak の一定割合、または surface brightness を基準にします。

### 5.2 別画像へ貼り付ける

1. contour source image の frame で contour を作る。
2. `Analysis → Contours → Copy Contours` を選ぶ。
3. 背景にする画像の frame に移る。
4. `Analysis → Contours → Paste Contours` を選ぶ。
5. 座標系を WCS にする。
6. point source や既知の天体位置で alignment を確認する。

WCS が正しければ、画像サイズや pixel scale が異なっていても、contour を sky 座標で貼り付けられます。WCS のない PNG、JPEG、不正な FITS header の画像には正しく重なりません。

```sh
xpaset -p ds9 contour method smooth
xpaset -p ds9 contour smooth 3
xpaset -p ds9 contour levels 3 5 10 20
xpaset -p ds9 contour generate
xpaset -p ds9 contour copy
xpaset -p ds9 contour paste wcs cyan 2 no
```

### 5.3 Contour を保存する

```sh
xpaset -p ds9 contour save hard_xray.ctr wcs fk5
xpaset -p ds9 contour load hard_xray.ctr
```

Contour file には、座標系、level、線の属性、contour point が保存されます。

## 6. 他の画像を重ねる

目的に応じて方法を選びます。

- **Contour overlay**：背景画像を保ちながら、別画像の形態を線で示す。Chandra RGB と radio / CO contour などに向く。
- **RGB channel として合成**：複数の画像を色で比較する。ただし scale によって印象が変わる。
- **Mask overlay**：検出領域や exposure の低い領域を半透明色で示す。
- **WCS match / lock と blink**：画像を合成せず、同じ sky area を交互表示する。

```sh
xpaset -p ds9 match frame wcs
xpaset -p ds9 lock frame wcs
xpaset -p ds9 blink interval 0.5
xpaset -p ds9 blink yes
```

RGB 合成、ratio map、pixel-by-pixel comparison では、表示の WCS を合わせるだけでなく、同じ配列サイズ、pixel scale、WCS grid にそろえる必要があります。外部画像を Chandra image に合わせる例は次のとおりです。

```sh
reproject_image \
  infile=radio_input.fits \
  matchfile=image/broad_counts.fits \
  outfile=radio_on_chandra_grid.fits \
  method=average \
  clobber=yes
```

counts の総和を保存したい画像には `method=sum`、exposure や surface brightness 型の画像には `method=average` を検討します。入力画像の単位と pixel 値の定義を確認してください。再投影は astrometric correction ではないため、元画像の WCS のずれは別途確認します。

## 7. Region を作成して保存する

Region は解析領域、位置の印、ラベル、scale bar などに使います。

1. mouse mode を Region にする。
2. circle、ellipse、box、polygon などを選ぶ。
3. 画像上で region を作る。
4. region を double click し、座標、サイズ、色、線幅、label を編集する。
5. source region と background region を色や tag で区別する。

異なる画像でも再利用する region は、`Region → Save Regions` で DS9 format、fk5 または icrs を選びます。

```text
# Region file format: DS9 version 4.1
global color=green width=2
fk5
circle(14:41:10.0,-62:38:00,20")
```

image 座標で保存すると、pixel grid の異なる画像では同じ sky area に重ならないことがあります。CIAO の filter に使う region と、画像の装飾用の region は区別してください。text、ruler、compass などは CIAO filter として使えません。

## 8. 画像と作業状態を保存する

### 8.1 完成図を保存する

`File → Save Image` は、現在の DS9 window の snapshot を保存します。colormap、scale、smoothing、contour、region、grid などの表示内容が含まれます。

```sh
xpaset -p ds9 saveimage snr_rgb.png
```

JPEG は非可逆圧縮なので、論文や発表用には PNG または TIFF を推奨します。

### 8.2 FITS として保存する

```sh
xpaset -p ds9 save fits displayed_image.fits image
xpaset -p ds9 save rgbimage snr_rgb.fits
```

これは画像データの保存であり、colormap、contour、region などの画面装飾を FITS pixel に焼き込む操作ではありません。

### 8.3 Region、Contour、作業状態を保存する

```sh
xpaset -p ds9 contour save radio_contours.ctr wcs fk5
xpaset -p ds9 backup snr_work.bck
xpaset -p ds9 restore snr_work.bck
```

Backup file と補助 directory が作成された場合は、両方を同じ場所に保ちます。energy band、scale limits、smoothing、contour levels もテキストファイルに残してください。

## 9. 再現性のために記録する情報

最低限、次を記録します。

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
Coordinate system:
Astrometric correction applied: yes / no
Output files:
```

バージョン確認例：

```sh
ciaover -v
ds9 -version
```

## 付録：一連のコマンド例

同じ Chandra event file から counts RGB を作り、DS9 に読み込みます。sky range は対象天体に合わせて変更してください。

```sh
EVT="repro/acisfXXXXX_repro_evt2.fits"
OUTDIR="image"
XMIN=3000; XMAX=5000
YMIN=3000; YMAX=5000
BIN=2
SOFT="500:1200"
MEDIUM="1200:2000"
HARD="2000:7000"

mkdir -p "$OUTDIR"

dmcopy "${EVT}[energy=${SOFT}][bin x=${XMIN}:${XMAX}:${BIN},y=${YMIN}:${YMAX}:${BIN}]" "${OUTDIR}/soft_counts.fits" clobber=yes
dmcopy "${EVT}[energy=${MEDIUM}][bin x=${XMIN}:${XMAX}:${BIN},y=${YMIN}:${YMAX}:${BIN}]" "${OUTDIR}/medium_counts.fits" clobber=yes
dmcopy "${EVT}[energy=${HARD}][bin x=${XMIN}:${XMAX}:${BIN},y=${YMIN}:${YMAX}:${BIN}]" "${OUTDIR}/hard_counts.fits" clobber=yes

ds9 &
```

DS9 起動後に実行します。

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
xpaset -p ds9 saveimage "${OUTDIR}/snr_rgb.png"
xpaset -p ds9 backup "${OUTDIR}/snr_rgb_work.bck"
```

!!! warning
    `dmcopy` の event filter では energy を eV で指定します。一方、`fluximage` の `bands` は通常 keV で指定します。単位を取り違えないでください。
