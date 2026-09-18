# FITS ファイルとヘッダーの読み方

Chandra や XRISM の観測データは、主に **FITS（Flexible Image Transport System）** 形式で保存されています。このページでは、FITS ファイルの構造、X線解析で重要なヘッダー、`fv` やコマンドラインツールを用いた確認方法を説明します。

- [FITS Standard（NASA/HEASARC）](https://fits.gsfc.nasa.gov/fits_standard.html)
- [Chandra Data Archive](https://cxc.harvard.edu/cda/)
- [XRISM Data Archive / DARTS](https://data.darts.isas.jaxa.jp/pub/xrism/data/)

## 目次

- [1. FITS の構造](#1-fits-の構造)
- [2. まず確認するヘッダー](#2-まず確認するヘッダー)
- [3. `fv` で確認する](#3-fv-で確認する)
- [4. コマンドラインで確認する](#4-コマンドラインで確認する)
- [5. Chandra と XRISM で重要な情報](#5-chandra-と-xrism-で重要な情報)
- [6. 解析でヘッダーを参照する場面](#6-解析でヘッダーを参照する場面)
- [7. 注意点](#7-注意点)

## 1. FITS の構造

FITS ファイルは、1つ以上の **HDU（Header/Data Unit）** から構成されます。各 HDU は、ヘッダーとデータ部を持ちます。

```text
FITS file
├── Primary HDU
│   ├── Header
│   └── Data（画像など）
├── Extension 1
│   ├── Header
│   └── Data（EVENTS、SPECTRUM など）
└── Extension 2 ...
```

Chandra や XRISM の event file では、通常 `EVENTS` 拡張にイベント表が入っています。そこには `TIME`、`PI`、位置、pixel、grade などの列があります。画像、スペクトル、RMF、ARF、exposure map も、それぞれ異なる HDU や FITS ファイルとして保存されます。

## 2. まず確認するヘッダー

### 2.1 ファイル形式とデータ配列

| キーワード | 意味 |
|---|---|
| `SIMPLE` | 標準 FITS 形式か |
| `BITPIX` | データの格納形式。`-32` は 32 bit 浮動小数点など |
| `NAXIS` | データ配列の軸数 |
| `NAXIS1`、`NAXIS2` | 各軸の要素数 |
| `EXTNAME` | 拡張名。`EVENTS`、`SPECTRUM` など |
| `BUNIT` | データの物理単位 |

`BITPIX` は物理単位ではありません。keV、eV、count などの単位は `BUNIT` や、イベント表の列定義を確認します。

### 2.2 天体、観測、���出器

| キーワード | 意味 |
|---|---|
| `OBJECT` | 天体名 |
| `TELESCOP` | 望遠鏡名。`CHANDRA`、`XRISM` など |
| `INSTRUME` | 検出器名。`ACIS`、`RESOLVE`、`XTEND` など |
| `OBS_ID` / `OBSID` | 観測 ID |
| `DATE-OBS`、`DATE-END` | 観測開始・終了時刻 |
| `TSTART`、`TSTOP` | ミッション時刻での時間範囲 |
| `EXPOSURE` | 有効露出時間 |
| `LIVETIME` | 生存時間。定義はミッションごとに確認 |
| `DATAMODE` | 観測・データモード |

同じキーワードでも、HDU やミッションにより意味や格納場所が異なる場合があります。`OBS_ID` と `OBSID` の両方を確認してください。

### 2.3 位置、座標、WCS

天体の位置を確認するときは、次のキーワードを調べます。

| キーワード | 意味 |
|---|---|
| `RA_OBJ`、`DEC_OBJ` | 観測対象の赤経・赤緯 |
| `RA_NOM`、`DEC_NOM` | 観測の nominal pointing |
| `CTYPE1`、`CTYPE2` | 座標軸の種類 |
| `CRVAL1`、`CRVAL2` | 基準 pixel の座標 |
| `CRPIX1`、`CRPIX2` | 基準 pixel |
| `CDELT1`、`CDELT2` | pixel あたりの座標間隔 |
| `CUNIT1`、`CUNIT2` | 座標の単位 |
| `EQUINOX` | 座標系の元期 |

`CTYPE1=RA---TAN`、`CTYPE2=DEC--TAN` は、赤経・赤緯を TAN 投影で表す WCS の例です。ARF の `source_ra`、`source_dec`、region の座標系を決めるときに参照します。

### 2.4 スペクトル、応答、領域

| キーワード | 意味 |
|---|---|
| `BACKFILE` | 背景 PHA への参照 |
| `RESPFILE` | RMF への参照 |
| `ANCRFILE` | ARF への参照 |
| `BACKSCAL` | 抽出領域の面積スケール情報 |
| `AREASCAL` | チャンネルごとの面積補正 |
| `CHANTYPE` | `PI`、`PHA` などのチャンネル種別 |
| `TLMINn`、`TLMAXn` | 列の値の範囲 |

スペクトルを XSPEC に読み込む前に、応答ファイルの参照先、露出時間、抽出領域、背景の規格化を確認します。複数スペクトルを結合した場合は、`BACKSCAL=1` になっていても面積補正が不要とは限りません。

## 3. `fv` で確認する

`fv` は HEASoft/FTOOLS に含まれる FITS Viewer です。HDU の一覧、ヘッダー、イベント表、画像を GUI で確認できます。

```sh
fv file.fits
# 例
fv xa201054010rsl_p0px1000_cl2.evt
```

起動後、HDU の一覧から `PRIMARY`、`EVENTS`、`SPECTRUM` などを選びます。

- **Header**：キーワードと値を確認
- **Table**：イベント表の列名、単位、データ型を確認
- **Image**：画像配列を確認

`fv` はファイルを編集して保存できるため、原本を直接変更しないでください。確認用コピーを使い、保存操作を行わないことを基本とします。

### `fv` を使うための環境

`fv` は HEASoft の一部です。HEASoft の導入と初期化については、[解析を始める前に：ソフトウェアと校正](preparation.md#ソフトウェアと校正)を参照してください。

環境を初期化した後、次を確認します。

```sh
command -v fv
command -v ftlist
command -v fkeyprint
heainit
```

環境によっては `heainit` が不要な場合もあります。使用している OS、HEASoft のインストール方法、シェルの設定に合わせてください。

## 4. コマンドラインで確認する

### 4.1 HDU とヘッダー

```sh
ftlist file.fits K
ftlist file.fits[EVENTS] K
fkeyprint file.fits[EVENTS] TELESCOP
fkeyprint file.fits[EVENTS] INSTRUME
fkeyprint file.fits[EVENTS] OBS_ID
fkeyprint file.fits[EVENTS] EXPOSURE
```

イベント表の列定義を確認します。

```sh
ftlist file.fits[EVENTS] K include kol
```

列の一部を表示する場合は、行数を制限します。

```sh
ftlist file.fits[EVENTS] T columns=TIME,PI maxrows=10
```

### 4.2 Chandra のコマンド

```sh
dmlist "repro/acisfXXXXX_repro_evt2.fits[EVENTS]" header
dmlist "repro/acisfXXXXX_repro_evt2.fits[EVENTS]" cols
dmkeypar repro/acisfXXXXX_repro_evt2.fits RA_NOM echo+
dmkeypar repro/acisfXXXXX_repro_evt2.fits DEC_NOM echo+
dmkeypar repro/acisfXXXXX_repro_evt2.fits EXPOSURE echo+
```

Chandra の `dmcopy` では、event file の `energy` を通常 eV で指定します。

```sh
dmcopy "repro/acisfXXXXX_repro_evt2.fits[energy=500:7000]" filtered_evt.fits clobber=yes
```

### 4.3 XRISM のコマンド

```sh
ftlist xa201054010rsl_p0px1000_cl2.evt K
ftlist xa201054010rsl_p0px1000_cl2.evt[EVENTS] K include kol
fkeyprint xa201054010rsl_p0px1000_cl2.evt[EVENTS] TELESCOP
fkeyprint xa201054010rsl_p0px1000_cl2.evt[EVENTS] INSTRUME
fkeyprint xa201054010rsl_p0px1000_cl2.evt[EVENTS] OBS_ID
fkeyprint xa201054010rsl_p0px1000_cl2.evt[EVENTS] TSTART
fkeyprint xa201054010rsl_p0px1000_cl2.evt[EVENTS] TSTOP
```

イベント表では、`TIME`、`PI`、`GRADE`、`ITYPE`、`PIXEL`、`STATUS` などを確認します。列名と定義はデータ版によって変わる可能性があるため、`ftlist ... K include kol` の結果を優先してください。

## 5. Chandra と XRISM で重要な情報

### Chandra

- `RA_NOM`、`DEC_NOM`：観測の pointing
- `RA_OBJ`、`DEC_OBJ`：天体座標
- `OBS_ID`、`OBJECT`：観測と天体の同定
- `EXPOSURE`、`TSTART`、`TSTOP`：露出と時間範囲
- `CTYPE*`、`CRVAL*`、`CRPIX*`、`CDELT*`：画像 WCS
- `energy`、`ccd_id`、`x`、`y`：イベントの選別と画像化に関わる列

### XRISM Resolve / Xtend

- `TELESCOP`、`INSTRUME`：望遠鏡と検出器
- `OBS_ID`：観測 ID
- `RA_OBJ`、`DEC_OBJ`：天体座標。実際に使う座標キーワードをヘッダーで確認
- `TSTART`、`TSTOP`、`EXPOSURE`：時間範囲と露出
- `PI`：較正済みの pulse-invariant channel
- `GRADE`、`ITYPE`：イベント選別に使う情報
- `PIXEL`：Resolve の pixel 選択に使う情報
- `STATUS`：イベントの状態フラグ

## 6. 解析でヘッダーを参照する場面

### Chandra の再処理・画像作成

`chandra_repro` 後に event file を使うときは、[Chandra/ACIS の再処理と画像作成](chandra.md)と合わせて、ObsID、検出器、露出時間、event column、WCS を確認します。DS9 で画像を重ねる場合は、WCS と `RA`/`DEC` の情報が必要です。

### XRISM の exposure map と ARF

XRISM の `xaexpmap`、`xaarfgen` を実行するときは、EHK、event file、exposure map、region の座標系、天体の `RA`/`DEC` を対応させます。具体的な手順は [XRISM の再処理と応答作成](xrism.md)を参照し、必要な座標や検出器情報はこのページの方法で確認してください。

### スペクトル解析

PHA、RMF、ARF、背景を XSPEC に読み込む前に、[スペクトル解析と結果の確認](spectra.md)に加えて、`BACKFILE`、`RESPFILE`、`ANCRFILE`、`BACKSCAL`、`EXPOSURE` を確認します。

## 7. 注意点

- `file.fits+1` と `file.fits[EVENTS]` が同じ HDU とは限りません。まず HDU 一覧を確認します。
- `BITPIX` はデータ形式であり、keV、eV、count などの物理単位ではありません。
- `OBJECT` が正しくても、別の観測や別の検出器のデータでないとは限りません。ObsID、時刻、検出器も確認します。
- `.gz` を展開して別名で保存した場合は、原本との対応関係を記録します。
- ヘッダーにキーワードがないことだけで、データが壊れているとは判断しません。HDU、ミッション、ソフトウェアのバージョンによる違いを確認します。
- WCS があっても、異なる画像をそのまま pixel-by-pixel で比較できるとは限りません。pixel grid と再投影の要否を確認します。

!!! note
    FITS ヘッダーは解析の入口です。ヘッダーだけで判断せず、イベント表の列、GTI、region、応答ファイル、CALDB の版を合わせて確認してください。
