# Chandra/ACIS：広がった線源

この章は ACIS の imaging 観測を対象にします。CXC の [ACIS Data Preparation](https://cxc.cfa.harvard.edu/ciao/guides/acis_data.html) と [Extended Sources Analysis Guide](https://cxc.cfa.harvard.edu/ciao/guides/esa.html) を並行して参照してください。

## 1. 公開データを取得し、再処理する

公式 extended-source スレッドでは、ACIS-S の ObsID `869` を例にしています。CIAO を有効にした端末で、作業用ディレクトリから次を実行します。

```sh
download_chandra_obsid 869
cd 869
punlearn chandra_repro
chandra_repro mode=h
```

`download_chandra_obsid` は公開データを取得し、`chandra_repro` は現在の校正に基づく Level 2 イベントを `repro/` に作ります。自分の観測では `869` を ObsID に置き換えます。

## 2. 元イベントを見て、解析領域を決める

イベントファイルを DS9 で開きます。以下のファイル名は ObsID 869 の**公式例**です。自分の観測では `repro/` に実在する `evt2` を指定します。

```sh
ds9 repro/acisf00869_repro_evt2.fits
```

広がった放射の境界、チップの継ぎ目、露出の低い場所、重なった点源を確認します。線源領域と Background 領域を別々に保存し、DS9 で再表示して重なりを点検します。

## 3. 露出補正画像を解釈する

画像のカウント数には、天体の明るさだけでなく、露出時間、有効面積、チップ間のすき間、望遠鏡の周辺減光も反映されます。形や表面輝度を見るには、露出補正が必須です。

```sh
punlearn fluximage
fluximage repro/acisf00869_repro_evt2.fits diffuse
```

上はツールを試すための入口です。実際の解析では、エネルギー帯、画素サイズ、複数チップの扱いを [fluximage のヘルプ](https://cxc.cfa.harvard.edu/ciao/tool_usage/index.html) と公式スレッドで確認します。

## 4. Background を決める

対象がチップの一部だけなら、線源の放射と他の点源が入らない局所 Background を検討します。視野全体に放射が広がるなら、局所 Background を無理に作るべきではありません。複数の Background 推定法（blank-sky、stowed、モデル）の長所と限界を理解して選びます。

### 4.1 Blank-sky Background を使う（基本）

**多くの場合、これで十分です。** Chandra は blank-sky observations（宇宙線背景を優先的に受ける視野での観測）を定期的に実施しており、その処理済みデータが公開されています。

Blank-sky を取得し、自分の観測領域にマッチさせた上で背景スペクトルを抽出します。

```sh
cd repro
# Blank-sky ファイルを別途ダウンロード、または CALDB から指定

# 自分の観測の Background 領域から PHA を抽出
punlearn xselect
xselect
```

`xselect` 内で以下を実行します。`<EVT2>` と `<BKG_REG>` は自分のファイルに置き換えます。

```text
read events <EVT2> .
filter region <BKG_REG>
extract spectrum
save spec background.pi
```

Blank-sky スペクトルも同様に抽出し、スペクトル解析で対応づけます。詳しくは [ACIS background files](https://cxc.cfa.harvard.edu/ciao/threads/acisbackground/) を参照してください。

### 4.2 丁寧な NXB モデリングを行う場合

視野全体に放射が広がり、局所 Background を確保できない場合、または検出器背景の正確なモデリングが必要な場合は、[別ページ「mkacispback による粒子起源背景モデリング」](nxb-modeling-mkacispback.md) をご覧ください。

この方法は高度な手法で、以下の場合に検討します：

- 科学的に NXB の正確さが重要
- 大型で均一な拡張天体の解析
- 複数 CCD の背景の空間変動を詳細に扱う必要がある

基本的な解析のほとんどは、上記の blank-sky Background で十分対応できます。

## 5. 広がった線源のスペクトルと応答を作る

CXC の [広がった線源のスペクトル抽出スレッド](https://cxc.cfa.harvard.edu/ciao/threads/extended/) は ObsID 869 の `simple.reg` と `simple_bkg.reg` を使い、`specextract` を実行します。

```sh
cd repro
punlearn specextract
specextract 'acisf00869_repro_evt2.fits[sky=region(simple.reg)]' simple \
  bkgfile='acisf00869_repro_evt2.fits[sky=region(simple_bkg.reg)]'
```

`specextract` の `weight=yes` は広がった線源に適した weighted ARF を既定で作ります。点源用の `correctpsf=yes` と `weight=no` を機械的に足さないでください。

抽出後は PHA の `BACKFILE`、`RESPFILE`、`ANCRFILE`、`BACKSCAL`、露出時間と、対応する ARF/RMF の存在を確認します。スペクトルのファイル名だけを見て終わらず、ヘッダーの参照が正しいかを検証します。

## 6. スペクトル実例を後から追加する

スペクトル図と XSPEC の具体例は、この節へ後から追加できます。ページの URL とサイト階層を変える必要はありません。実例には、データと総モデルについて以下を含めてください：

- ObsID、検出器、FAINT/VFAINT、抽出 region
- CIAO、CALDB、HEASoft の版
- 使用した PHA、ARF、RMF とフィット帯域
- 統計量、binning、吸収・放射・sky Background のモデル式
- Background の規格化を固定したか自由にしたか、その判断理由
