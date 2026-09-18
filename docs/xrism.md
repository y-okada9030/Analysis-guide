# XRISM：再処理と広がった線源の解析

この章では、XRISM の Resolve と Xtend について、公開データの取得、イベントのスクリーニング、exposure map の作成、スペクトル抽出、応答の作成までを説明します。特に Resolve の広がった天体を、FoV 全体から解析する場合を中心に扱います。

XRISM の解析は、必ず最新の [ABC Data Reduction Guide](https://heasarc.gsfc.nasa.gov/docs/xrism/analysis/abc_guide/) と、使用するソフトウェアの `fhelp` を確認して進めてください。以下のコマンドは、公開データの構成や CALDB、ソフトウェアのバージョンによって修正が必要になる場合があります。

## 目次

- [1. データを取得する](#1-データを取得する)
- [2. Resolve の cleaned event を作成する](#2-resolve-の-cleaned-event-を作成する)
- [3. Resolve の exposure map を作成する](#3-resolve-の-exposure-map-を作成する)
- [4. Ls イベントを除いたイベントファイルを作成する](#4-ls-イベントを除いたイベントファイルを作成する)
- [5. Resolve のスペクトルを抽出する](#5-resolve-のスペクトルを抽出する)
- [6. Resolve の RMF と ARF を作成する](#6-resolve-の-rmf-と-arf-を作成する)
- [7. Resolve の検出器背景モデル](#7-resolve-の検出器背景モデル)
- [8. Xtend のスペクトルと応答を作成する](#8-xtend-のスペクトルと応答を作成する)
- [9. Resolve と Xtend の結果を比較する](#9-resolve-と-xtend-の結果を比較する)
- [10. 再現性のために記録する情報](#10-再現性のために記録する情報)

## 1. データを取得する

公開データは、対象 ObsID のディレクトリから取得します。例えば、ObsID `000126000` の rev3 データを取得する場合は次のようにします。

```sh
mkdir -p 000126000
cd 000126000
wget -nv -m -np -nH --cut-dirs=6 -R "index.html*" \
  --execute robots=off --wait=1 \
  https://data.darts.isas.jaxa.jp/pub/xrism/data/obs/rev3/0/000126000/
```

実際に解析するときは、ダウンロードしたディレクトリ構造を確認します。Resolve の cleaned event は通常、次の場所にあります。

```text
<obsid>/resolve/event_cl/*_cl.evt.gz
```

観測ログ、processing notes、quick-look products も併せて保存してください。解析に使ったデータの版、CALDB、HEASoft、XRISM ソフトウェアのバージョンを記録すると、後から同じ処理を再現しやすくなります。

## 2. Resolve の cleaned event を作成する

配布されている `*_cl.evt.gz` は、解析の出発点となる cleaned event です。Resolve では、通常の grade 選択に加えて、観測時期や校正方針に応じた追加スクリーニングを行います。

以下は、PI の下限と RISE_TIME、DERIV_MAX、ITYPE、STATUS を用いた追加選別の例です。ファイル名と条件は、対象データに対応する ABC Guide および最新の calibration note に合わせて確認してください。

```sh
ftcopy \
  infile="xa201054010rsl_p0px1000_cl.evt.gz[EVENTS][(PI>=600)&&(((((RISE_TIME+0.00075*DERIV_MAX)>46)&&((RISE_TIME+0.00075*DERIV_MAX)<58))&&ITYPE<4)||(ITYPE==4))&&STATUS[4]==b0]" \
  outfile=xa201054010rsl_p0px1000_cl2.evt \
  copyall=yes clobber=yes history=yes
```

この例で作成される `*_cl2.evt` が、追加スクリーニング後のイベントファイルです。`ftlist` や `fkeyprint` を使い、イベント数、PI 範囲、GTI、検出器情報が意図したものになっている���確認します。

### Gain に問題のある pixel の確認

スペクトル抽出の前に、XRISM の gain report を確認します。

- [XRISM gain reports](https://heasarc.gsfc.nasa.gov/FTP/xrism/postlaunch/gainreports/)
- 特定 pixel の gain に問題がないか
- 使用する観測時期、検出器、processing version に該当する報告か

特に、観測時期によっては PIX27 を解析から除外する必要があります。一方、校正用の PIX12 は通常の解析で自動的に含めない設定になっていることがあります。pixel の扱いはデータ版や校正情報に依存するため、常に最新の gain report と ABC Guide を優先してください。

## 3. Resolve の exposure map を作成する

exposure map は、検出器上の各位置が実効的にどれだけ観測されたかを表す情報です。姿勢、GTI、検出器の有効面積、遮蔽や校正情報を考慮し、画像や ARF の作成に用います。

`xaexpmap` の実行には、event file、EHK、pixel GTI、CALDB が必要です。次は Resolve の exposure map を作成する例です。

```sh
xaexpmap \
  ehkfile=/path/to/auxil/xa201054010.ehk.gz \
  gtifile=xa201054010rsl_p0px1000_cl2.evt \
  instrume=RESOLVE \
  badimgfile=NONE \
  pixgtifile=/path/to/resolve/event_uf/xa201054010rsl_px1000_exp.gti.gz \
  outfile=xa201054010rsl_p0px1000_cl2.expo \
  outmaptype=EXPOSURE \
  delta=20.0 \
  numphi=4 \
  stopsys=SKY \
  instmap=CALDB \
  qefile=CALDB \
  contamifile=CALDB \
  vigfile=CALDB \
  obffile=CALDB \
  fwfile=CALDB \
  gvfile=CALDB \
  maskcalsrc=yes \
  fwtype=FILE \
  specmode=MONO \
  specfile=spec.fits \
  specform=FITS \
  evperchan=DEFAULT \
  abund=1 \
  cols=0 \
  covfac=1 \
  clobber=yes \
  chatter=1 \
  logfile=log/make_expo_xa201054010rsl_p0px1000.log
```

`/path/to/...` は自分のデータの実際のパスに置き換えます。`spec.fits` は exposure correction の想定スペクトルとして使われるため、目的に合ったファイルを指定してください。解析前に `xaexpmap` のヘルプで、使用するバージョンのパラメータ名と既定値を確認します。

## 4. Ls イベントを除いたイベントファイルを作成する

Resolve では、地上試験で想定された量より多い Ls event が含まれることがあり、暗い天体ではバックグラウンドへの影響が無視できない場合があります。以下は、PI 4–20 keV、ITYPE<4 のイベントだけを残し、Ls event を除いたファイルを作成する例です。

```sh
ftcopy \
  infile="xa201054010rsl_p0px1000_cl2.evt[EVENTS][(PI>=4000)&&(PI<=20000)&&(ITYPE<4)]" \
  outfile=xa201054010rsl_p0px1000_cl2noLs.evt \
  copyall=yes clobber=yes
```

このファイルは、後の Resolve RMF 作成に使います。イベントを除外した理由、条件、作成日時を解析ログに残してください。LS event の扱いはデータ版や解析目的によって変わる可能性があるため、公式ガイドと最新の calibration information を優先します。

## 5. Resolve のスペクトルを抽出する

FoV 全体のスペクトルを抽出する例です。ここでは Hp grade と、gain に問題がある pixel を除外する pixel filter を使います。以下は **XSELECT の対話環境内**で入力します。

```sh
xselect
```

```text
xrism
read event xa201054010rsl_p0px1000_cl2.evt .
filter grade 0-0
filter column "PIXEL=0:26,28:35"
extract spectrum
save spectrum xa201054010rsl_p0px1000_fov_Hp.pi
exit
```

- `filter grade 0-0`：Hp grade のイベントを選択する例
- `filter column "PIXEL=0:26,28:35"`：PIX27 を除外する例
- event file、grade、pixel selection は対象観測と gain report に合わせて変更する

スペクトル抽出後は、PHA の `EXPOSURE`、`BACKSCAL`、`RESPFILE`、`ANCRFILE` などを確認します。入力 event file と抽出条件も記録してください。

## 6. Resolve の RMF と ARF を作成する

### 6.1 RMF

Resolve の RMF は、通常、Ls event を除いた event file を使って作成します。FoV 全体の Hp spectrum に対応する RMF の例を示します。

```sh
rslmkrmf \
  infile=xa201054010rsl_p0px1000_cl2noLs.evt \
  outfileroot=xa201054010rsl_p0px1000_fov_HpLnoLs \
  splitrmf=yes \
  elcbinfac=16 \
  splitcomb=no \
  resolist=0 \
  combps=no \
  secondaries=yes \
  regmode=DET \
  regionfile=NONE \
  pixeltest=CENTER \
  pixlist=0-11,13-26,28-35 \
  rmfparamfile=CALDB \
  outrsp=no \
  arfinfile=NONE \
  whichrmf=L \
  nchanin=60000 \
  logfile=log/xa201054010rsl_p0px1000_rmf.log \
  clobber=yes
```

`pixlist` は使用する pixel の範囲に合わせます。ここでは PIX27 を除外しています。RMF の入力イベント、pixel 選択、grade の対応が、スペクトル抽出時の条件と一致していることを確認してください。

### 6.2 ARF

広がった天体では、領域、天体の形、ray-trace 用の入力、exposure map が ARF に影響します。次の例では、flat circle の天体モデルと `rsl_fov.reg` の領域を指定しています。

```sh
xaarfgen \
  xrtevtfile=raytrace_xa201054010rsl_p0px1000_rsl_flatcircle.fits \
  source_ra=220.22 \
  source_dec=-62.672 \
  telescop=XRISM \
  instrume=RESOLVE \
  emapfile=xa201054010rsl_p0px1000_cl2.expo \
  regmode=DET \
  regionfile=rsl_fov.reg \
  sourcetype=FLATCIRCLE \
  flatradius=5.0 \
  rmffile=xa201054010rsl_p0px1000_fov_HpLnoLs.rmf \
  erange="0.3 18.0 0 0" \
  outfile=xa201054010rsl_p0px1000_rsl_HpL_flatcircle.arf \
  numphoton=600000 \
  qefile=CALDB \
  contamifile=CALDB \
  gatevalvefile=CALDB \
  onaxisffile=CALDB \
  onaxiscfile=CALDB \
  mirrorfile=CALDB \
  obstructfile=CALDB \
  frontreffile=CALDB \
  backreffile=CALDB \
  pcolreffile=CALDB \
  scatterfile=CALDB \
  imgfile=NONE \
  clobber=yes \
  mode=h | tee log/xa201054010rsl_p0px1000_raytrace_arf.log
```

`source_ra` と `source_dec` は天体の座標に置き換えます。`regionfile` は、スペクトルを抽出した領域と対応させます。flat circle は例なので、実際の天体の広がりを表す画像やモデルを使う必要がある場合は、ABC Guide の手順に従って入力を変更してください。

## 7. Resolve の検出器背景モデル

FoV 全体に広がった天体では、天体放射を含まない局所背景領域を確保できないことがあります。その場合は、XRISM の NXB model を用いる方法を検討します。

XSPEC で配布または作成したモデルを読み込む例は次のとおりです。

```text
@rsl_nxb_model_v1.mo
```

NXB model は sky background そのものではありません。天体の放射、宇宙 X線背景、銀河系の前景放射などを含むかどうかは、使用するモデルの定義を確認し、必要な成分を別途モデル化します。使用した NXB model の版、パラメータ、規格化の扱いを解析記録に残してください。

## 8. Xtend のスペクトルと応答を作成する

Xtend では、まず cleaned event から画像とスペクトルを作り、抽出領域に対応する RMF と ARF を作成します。Xtend の event file、exposure map、領域ファイルは実際のデータ構成に合わせて置き換えてください。

### 8.1 RMF

FoV 用の PHA から RMF を作る例です。

```sh
xtdrmf xtend_fov0616.pha xtend.rmf
```

### 8.2 ARF

```sh
xaarfgen \
  xrtevtfile=../xtd_raytrace_sw.fits \
  source_ra=220.22 \
  source_dec=-62.672 \
  telescop=XRISM \
  instrume=XTEND \
  emapfile=xtd_201054010.expo \
  regmode=RADEC \
  regionfile=../rsl_allpix_sky.reg \
  sourcetype=FLATCIRCLE \
  flatradius=5.0 \
  rmffile=xtend.rmf \
  erange="0.3 15.0 0 0" \
  outfile=xtend.arf \
  numphoton=600000 \
  minphoton=100 \
  teldeffile=CALDB \
  qefile=CALDB \
  contamifile=CALDB \
  obffile=CALDB \
  fwfile=CALDB \
  gatevalvefile=CALDB \
  onaxisffile=CALDB \
  onaxiscfile=CALDB \
  mirrorfile=CALDB \
  obstructfile=CALDB \
  frontreffile=CALDB \
  backreffile=CALDB \
  pcolreffile=CALDB \
  scatterfile=CALDB \
  imgfile=NONE \
  seed=7 \
  clobber=yes \
  mode=h
```

`regmode=RADEC` の場合、region file は天球座標で保存したものを使います。Resolve の `regmode=DET` と混同しないようにしてください。Xtend の ARF も、天体の位置・広がり・領域・exposure map と整合している必要があります。

## 9. Resolve と Xtend の結果を比較する

同じ天体でも、Resolve と Xtend では視野、抽出領域、PSF、エネルギー分解能、応答が異なります。比較する前に次をそろえて確認します。

- ObsID と観測時刻
- event screening と grade
- source region と sky region
- energy band
- exposure map と CALDB
- RMF、ARF、NXB model

Resolve の高分解能スペクトルと Xtend の広い視野のスペクトルを、同じ抽出領域・同じ応答で得たものとして扱ってはいけません。両者を同時フィットする場合は、それぞれの PHA、RMF、ARF を別データセットとして読み込み、cross-normalization や領域の違いを考慮します。

## 10. 再現性のために記録する情報

最低限、以下を記録します。

```text
Target:
ObsID:
Data release / processing version:
HEASoft version:
XRISM software version:
CALDB version:

Resolve event file:
Additional screening:
Grade selection:
Pixel selection:
Ls-event treatment:
Exposure map:

Resolve spectrum:
Resolve RMF:
Resolve ARF:
Resolve NXB model:

Xtend event file:
Xtend spectrum:
Xtend RMF:
Xtend ARF:

Source region:
Region coordinate system:
Energy range:
Background treatment:
```

解析を完了したら、ログファイル、region file、exposure map、PHA、RMF、ARF、NXB model と、各コマンドの入力条件を同じ解析ディレクトリに保存します。
