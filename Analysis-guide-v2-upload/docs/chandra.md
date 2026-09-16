# Chandra/ACIS：広がった線源

この章は ACIS の imaging 観測を対象にします。CXC の [ACIS Data Preparation](https://cxc.cfa.harvard.edu/ciao/guides/acis_data.html) と [Extended Sources Analysis Guide](https://cxc.cfa.harvard.edu/ciao/guides/esa.html) の順序に沿い、**最新校正でイベントを準備する → 画像と領域を確かめる → Background と応答を含むスペクトルを作る**流れを説明します。公式ガイドは各作業に対応する Science Thread への道案内です。ここでも観測固有の設定を一つの万能コマンドにしません。

## 1. 公開データを取得し、再処理する

公式 extended-source スレッドでは、ACIS-S の ObsID `869` を例にしています。CIAO を有効にした端末で、作業用ディレクトリから次を実行します。

```sh
download_chandra_obsid 869
cd 869
punlearn chandra_repro
chandra_repro mode=h
```

`download_chandra_obsid` は公開データを取得し、`chandra_repro` は現在の校正に基づく Level 2 イベントを `repro/` に作ります。自分の観測では `869` を ObsID に置き換え、既存の出力を上書きしない作業場所を選びます。再処理が終わったら、ログに重大なエラーがないか、`repro/` に `evt2` と必要な補助ファイルがあるかを確認します。アスペクト補正、flare 除去、観測時の ACIS 温度などの判断は [ACIS Data Preparation](https://cxc.cfa.harvard.edu/ciao/guides/acis_data.html) に戻って行います。複数観測を使うときも、ここまでは ObsID ごとに行います。

## 2. 元イベントを見て、解析領域を決める

イベントファイルを DS9 で開きます。以下のファイル名は ObsID 869 の**公式例**です。自分の観測では `repro/` に実在する `evt2` を指定します。

```sh
ds9 repro/acisf00869_repro_evt2.fits
```

広がった放射の境界、チップの継ぎ目、露出の低い場所、重なった点源を確認します。線源領域と Background 領域を別々に保存し、DS9 で再表示して座標系を確かめます。同じチップ内に Background 領域を取れる場合でも、そこに対象の淡い背景放射が含まれていないかを画像と既知の天体情報から検討します。**点源の除外領域**を作る場合は、明るい点源を検出した理由と除外範囲を記録します。点源検出には [CXC の source detection threads](https://cxc.cfa.harvard.edu/ciao/guides/esa.html) が案内する `wavdetect` などを使えます。単に明るい画素を消すと、対象の小さな構造も消しかねません。

## 3. 露出補正画像を解釈する

画像のカウント数には、天体の明るさだけでなく、露出時間、有効面積、チップ間のすき間、望遠鏡の周辺減光も反映されます。形や表面輝度を比較する際は [露出マップの公式手順](https://cxc.cfa.harvard.edu/ciao/guides/esa.html) を参照し、対象のスペクトルに適したエネルギー帯と重みを選びます。

```sh
punlearn fluximage
fluximage repro/acisf00869_repro_evt2.fits diffuse
```

上はツールを試すための入口です。実際の解析では、エネルギー帯、画素サイズ、複数チップの扱いを [fluximage のヘルプ](https://cxc.cfa.harvard.edu/ciao/ahelp/fluximage.html) に合わせて指定します。特に線スペクトルが強い対象や場所ごとにスペクトルが違う対象では、単一エネルギーの露出マップで得た値を精密な光子フラックスとして扱えません。平滑化や点源の穴埋めをした図は表示には有用ですが、測定には元イベントと露出マップを使います。

## 4. Background を決める

対象がチップの一部だけなら、線源の放射と他の点源が入らない局所 Background を検討します。視野全体に放射が広がるなら、局所 Background を無理に設定せず、同種の別チップや [ACIS blank-sky background](https://cxc.cfa.harvard.edu/ciao/threads/acisbackground/) を検討します。blank-sky は観測に合わせたフィルタ、再投影、規格化が必要です。局所 Background と同じように `bkg.reg` を描くだけでは使えません。また、blank-sky に含まれる背景放射が対象方向の空の背景放射と一致するとは限りません。Background を差し引く方法と、Background を別成分としてモデル化する方法のどちらを使うかを解析ノートに書きます。

### 4.1 `mkacispback` で粒子起源 NXB をモデル化する

視野を満たす広がった線源では、線源のない局所 Background 領域を確保できないことがあります。[`mkacispback`](https://github.com/hiromasasuzuki/mkacispback) は、そのような解析で Chandra/ACIS の**粒子起源 Background（NXB）をスペクトルモデルとして生成する第三者製ツール**です。ACIS-I0〜I3 と ACIS-S1〜S3 の FAINT/VFAINT データに対応します。blank-sky イベントを線源スペクトルから差し引く手順とは異なり、抽出領域、CCD 上の位置、観測時期とデータモードに応じた NXB モデルを作り、XSPEC で線源成分と同時にフィットします。

`mkacispback` が表すのは検出器の**粒子起源 NXB**です。宇宙 X 線背景放射、銀河系前景放射、Solar Wind Charge Exchange などの sky Background は含みません。これらが無視できない場合は、別の天体成分としてモデル化し、必要に応じて線源外の観測や ROSAT などの情報で制約します。

#### 導入と環境設定

配布元の README が示す主な依存関係は、CIAO、HEASoft、Astropy を利用できる Python 3、C++11 対応コンパイラです。配布元を任意の場所へ取得し、実際の場所に合わせて次を設定します。

```sh
git clone https://github.com/hiromasasuzuki/mkacispback.git /path/to/mkacispback

export ACISPBACK=/path/to/mkacispback
export ACISPBACK_PYTHON=/path/to/python3
export ACISPBACK_GXX=/path/to/g++
export PATH="$ACISPBACK:$PATH"
```

`ACISPBACK_PYTHON` には `import astropy` が成功する Python を、`ACISPBACK_GXX` には C++11 を扱えるコンパイラを指定します。実行前には HEASoft と CIAO の両方を初期化し、`CALDB` が **CIAO CALDB** を指していることを確認します。このガイドの共存設定を使っている場合も、端末を開いた直後に両者が正しく有効かを確認してください。

```sh
which mkacispback
echo "$CALDB"
"$ACISPBACK_PYTHON" -c 'import astropy; print(astropy.__version__)'
mkacispback --h
```

手元に `/Users/yamazakipc/mkacispback` がある場合の設定例は次のとおりです。これは個人環境の実例なので、第三者は自分のパスへ置き換えます。

```sh
export ACISPBACK=/Users/yamazakipc/mkacispback
export PATH="$ACISPBACK:$PATH"
```

#### ObsID 21361 の領域に対して生成する

次の例では、再投影済みイベントから `sur2nb.reg` の領域を選び、`21361_sur2nb/` に `pb_nb_a` という NXB モデルを生成します。イベントファイルと region ファイルは、コマンドを実行するディレクトリから参照できる必要があります。

```sh
mkacispback \
  "21361_reproj_evt.fits.gz[sky=region(sur2nb.reg)]" \
  outdir=21361_sur2nb \
  name=pb_nb_a
```

引用符は、CIAO の Data Model 式に含まれる角括弧や丸括弧をシェルに解釈させないために必要です。`outdir` は生成物の保存先、`name` は XSPEC に登録するモデル名です。`name` には小文字と `_` だけを使い、数字や大文字を含めません。既存出力を意図して更新するときだけ `clobber=yes` を追加します。

通常は、選択領域の weight map、データスペクトル、NXB 用 RMF、XSPEC のローカルモデルが生成されます。処理後はログに表示される CCD、9.0–11.5 keV のカウント数、effective exposure、領域面積、規格化係数、gain、fit statistic を保存します。高エネルギー側のカウントが少ない場合は規格化の不確かさが大きくなるため、結果を機械的に採用しません。

#### XSPEC で線源成分と同時にフィットする

配布元の例に従い、生成したローカルモデルを `lmod` で読み込みます。`./` を省略しないでください。線源放射には通常の ARF/RMF を、粒子起源 NXB には `mkacispback` が作った RMF を割り当てます。次は構造を示す例なので、実際のファイル名は出力ディレクトリ内を確認して置き換えます。

```text
XSPEC> lmod pb_nb_a_pkg ./21361_sur2nb
XSPEC> data 1:1 <source.pi>
XSPEC> response 1:1 <source.rmf>
XSPEC> arf 1:1 <source.arf>
XSPEC> response 2:1 <particle-background.rmf>
XSPEC> model 1:source <source-model>
XSPEC> model 2:pb pb_nb_a
```

NXB は望遠鏡で集光された X 線ではないため、NXB モデル側へ source ARF を掛けません。sky Background は source response を通る別成分として加えます。9.0–11.5 keV に sky emission が混入する可能性がある場合、配布元は NXB モデルの規格化を自由パラメータにすることを推奨しています。

S1 と S3 では、観測時期によって 2–6 keV の連続成分を低く予測する場合があります。必要性を residual で確認し、追加成分を使った場合は理由と影響を記録します。また、一つの領域が複数 CCD にまたがると差が大きくなることがあります。その場合は CCD ごとにモデルを作り、スペクトルを同時フィットします。再現可能性のため、`mkacispback` の版、CIAO・CALDB・HEASoft の版、実行コマンド、region ファイルを解析記録に残します。

## 5. 広がった線源のスペクトルと応答を作る

CXC の [広がった線源のスペクトル抽出スレッド](https://cxc.cfa.harvard.edu/ciao/threads/extended/) は ObsID 869 の `simple.reg` と `simple_bkg.reg` を使い、`specextract` で線源・背景の PHA と応答を作ります。**以下の領域ファイルは公式例を自分で作成し、`repro/` に保存した後に使います。** 公式ページの領域座標を別の観測に転用しないでください。

```sh
cd repro
punlearn specextract
specextract 'acisf00869_repro_evt2.fits[sky=region(simple.reg)]' simple \
  bkgfile='acisf00869_repro_evt2.fits[sky=region(simple_bkg.reg)]'
```

`specextract` の `weight=yes` は広がった線源に適した weighted ARF を既定で作ります。点源用の `correctpsf=yes` と `weight=no` を機械的に足さないでください。応答は抽出領域の位置や、領域内でどこに光子が分布するかに影響されます。背景を単純に差し引くだけなら、背景側の応答を作らない `bkgresp=no` も選択肢です。一方、背景をモデル化するならその応答が必要になり得ます。`energy_wmap`、複数領域、チップ端、暖かい ACIS データは [公式スレッドの caveats](https://cxc.cfa.harvard.edu/ciao/threads/extended/) を確認します。

抽出後は PHA の `BACKFILE`、`RESPFILE`、`ANCRFILE`、`BACKSCAL`、露出時間と、対応する ARF/RMF の存在を確認します。スペクトルのファイル名だけを見て終わりにせず、**選んだ領域と Background に合う応答ができたか**を確かめます。複数 ObsID の画像を合わせる場合も、スペクトルと応答は観測ごとに作ってから、同時フィットや [CXC の Merging Central](https://cxc.cfa.harvard.edu/ciao/merging/merge_central.html) に従った結合を選びます。

## 6. スペクトル実例を後から追加する

スペクトル図と XSPEC の具体例は、この節へ後から追加できます。ページの URL とサイト階層を変える必要はありません。実例には、データと総モデル、線源成分、粒子起源 NXB、sky Background、残差を区別した図を載せ、次の情報を併記します。

- ObsID、検出器、FAINT/VFAINT、抽出 region
- CIAO、CALDB、HEASoft、`mkacispback` の版
- 使用した PHA、ARF、RMF とフィット帯域
- 統計量、binning、吸収・放射・sky Background・NXB のモデル式
- NXB の規格化を固定したか自由にしたか、その判断理由
