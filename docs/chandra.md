# Chandra/ACIS：広がった線源

このセクションでは、Chandra ACIS の imaging 観測データから、広がった線源の画像とスペクトルを作成する手順を説明します。CXC の [ACIS Data Preparation](https://cxc.cfa.harvard.edu/ciao/guides/acis_data.html) と [Extended Sources Analysis Guide](https://cxc.cfa.harvard.edu/ciao/guides/esa.html) を並行して参照することを強く推奨します。

## 1. 公開データを取得し、再処理する

CXC の公式 extended-source スレッドでは、ACIS-S の ObsID `869` を例に使用しています。CIAO を有効にした端末で、作業用ディレクトリから以下を実行します。

```sh
download_chandra_obsid 869
cd 869
punlearn chandra_repro
chandra_repro mode=h
```

- `download_chandra_obsid`：公開アーカイブからデータを取得
- `chandra_repro`：現在の校正に基づき Level 2 イベントファイルを `repro/` に生成

自分の観測では `869` を該当する ObsID に置き換えます。

## 2. イベントファイルを視認して、解析領域を決める

イベントファイルを DS9 で開き、観測領域を確認します。以下は ObsID 869 の公式例です。自分のデータでは `repro/` に実在する evt2 ファイルを指定してください。

```sh
ds9 repro/acisf00869_repro_evt2.fits
```

以下の点を確認します：
- 広がった放射の空間的な広がりと境界
- CCD チップ間の継ぎ目
- 露出が低い領域
- 線源に重なった点源

線源領域と背景領域をそれぞれ別々の領域ファイルに保存し、DS9 で再度表示して重なりを確認します。

## 3. 露出補正画像を解釈する

観測画像のカウント数は、天体の真の明るさだけでなく以下の要素を反映します：
- 露出時間の分布
- 検出器の有効面積の変動
- CCD 間のすき間
- 光学系の周辺減光

天体の形や表面輝度分布を正確に理解するには、露出補正が必須です。

```sh
punlearn fluximage
fluximage repro/acisf00869_repro_evt2.fits diffuse
```

これは `fluximage` ツールを試す最小限の例です。実際の解析では、エネルギー帯、画素サイズ、複数 CCD の扱い方を [fluximage のマニュアル](https://cxc.cfa.harvard.edu/ciao/tool_usage/index.html) と公式スレッドで確認してください。

## 4. 背景データを選択する

線源が観測視野の一部に限定されていれば、線源や他の点源が混入しない局所背景領域の利用を検討します。視野全体に放射が広がっている場合は、局所背景領域を無理に作るべきではありません。背景推定の複数の方法（blank-sky、stowed、モデル）それぞれの利点と制限を理解した上で選択します。

### 4.1 Blank-sky 背景を使う（推奨）

**ほとんどの解析では、これで十分です。** Chandra は blank-sky observations（宇宙線背景を優先的に受ける視野での定期観測）を実施しており、その処理済みデータが公開されています。

Blank-sky ファイルを取得し、自分の観測領域に合わせた上で背景スペクトルを抽出します。

```sh
cd repro
# Blank-sky ファイルを別途ダウンロード、または CALDB から指定

# 自分の観測の背景領域から PHA を抽出
punlearn xselect
xselect
```

`xselect` の対話環境で、以下を実行します。`<EVT2>` と `<BKG_REG>` は自分のファイルに置き換えます。

```text
read events <EVT2> .
filter region <BKG_REG>
extract spectrum
save spec background.pi
```

Blank-sky スペクトルも同様に抽出し、スペクトル解析で対応づけます。詳細は [ACIS background files](https://cxc.cfa.harvard.edu/ciao/threads/acisbackground/) を参照してください。

### 4.2 丁寧な検出器背景モデリングが必要な場合

視野全体に放射が広がり、局所背景領域を確保できない、または検出器背景の精密なモデリングが必要な場合は、[別ページ「mkacispback による粒子起源背景のモデリング」](nxb-modeling-mkacispback.md) を参照してください。

これは高度な手法で、以下の場合に検討します：
- 科学目標上、粒子起源背景（NXB）の精度が重要
- 大型で均一な拡張天体の解析
- 複数 CCD の背景の空間変動を詳細に扱う必要がある

基本的な広がった線源解析のほとんどは、上記の blank-sky 背景で対応できます。

## 5. 広がった線源のスペクトルと応答行列を作成する

CXC の [広がった線源のスペクトル抽出スレッド](https://cxc.cfa.harvard.edu/ciao/threads/extended/) では、ObsID 869 の `simple.reg` と `simple_bkg.reg` を用いて `specextract` を実行しています。

```sh
cd repro
punlearn specextract
specextract 'acisf00869_repro_evt2.fits[sky=region(simple.reg)]' simple \
  bkgfile='acisf00869_repro_evt2.fits[sky=region(simple_bkg.reg)]'
```

`specextract` の `weight=yes`（既定値）は、広がった線源に適した weighted ARF を自動生成します。点源解析用の `correctpsf=yes` と `weight=no` を機械的に追加しないでください。

抽出完了後は、以下を確認します：
- PHA ヘッダーの `BACKFILE`、`RESPFILE`��`ANCRFILE`、`BACKSCAL`、`EXPOSURE`
- 対応する ARF/RMF ファイルが存在すること

スペクトルのファイル名だけで判断せず、ヘッダーの参照が正しいか必ず検証してください。

## 6. スペクトル解析の実例（今後追加予定）

スペクトル図と XSPEC の具体例は、今後このセクションに追加できます。ページ URL やサイト階層は変わりません。実例に含めるべき情報：

- ObsID、検出器（I0–I3、S1–S3）、データモード（FAINT/VFAINT）、抽出領域
- 使用した CIAO、CALDB、HEASoft のバージョン
- 使用した PHA、ARF、RMF ファイルと適用したフィット帯域
- 統計手法（C-統計量、binning 方法）
- 吸収、放射、sky Background のモデル式
- 背景の規格化を固定したか自由にしたか、その根拠
