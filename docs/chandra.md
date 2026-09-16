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

### 4.1 `mkacispback` で粒子起源 NXB をモデル化する

視野を満たす広がった線源では、線源のない局所 Background 領域を確保できないことがあります。[`mkacispback`](https://github.com/hiromasasuzuki/mkacispback) は、観測のイベントから検出器の粒子起源背景（NXB）をスペクトルモデルとして生成します。

`mkacispback` が表すのは検出器の**粒子起源 NXB**です。宇宙 X 線背景放射、銀河系前景放射、Solar Wind Charge Exchange などの sky Background は含みません。スペクトルフィットではこれらを別成分として加える必要があります。

#### 導入と環境設定

配布元の README が示す主な依存関係は、CIAO、HEASoft、Astropy を利用できる Python 3、C++11 対応コンパイラです。配布元を任意の場所へ取得し、実際の環境に合わせて設定します。

```sh
git clone https://github.com/hiromasasuzuki/mkacispback.git /path/to/mkacispback

export ACISPBACK=/path/to/mkacispback
export ACISPBACK_PYTHON=/path/to/python3
export ACISPBACK_GXX=/path/to/g++
export PATH="$ACISPBACK:$PATH"
```

`ACISPBACK_PYTHON` には `import astropy` が成功する Python を、`ACISPBACK_GXX` には C++11 を扱えるコンパイラを指定します。実行前には HEASoft と CIAO の両方を有効にしておきます。

設定後、以下のコマンドで確認します。

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

次の例では、再投影済みイベントから `sur2nb.reg` の領域を選び、`21361_sur2nb/` に `pb_nb_a` という NXB モデルを生成します。イベントファイルと region ファイルの位置に合わせて実行します。

```sh
mkacispback \
  "21361_reproj_evt.fits.gz[sky=region(sur2nb.reg)]" \
  outdir=21361_sur2nb \
  name=pb_nb_a
```

引用符は、CIAO の Data Model 式に含まれる角括弧や丸括弧をシェルに解釈させないために必要です。`outdir` は生成物の保存先、`name` は XSPEC に登録するモデル名です。

通常は、選択領域の weight map、データスペクトル、NXB 用 RMF、XSPEC のローカルモデルが生成されます。処理後はログに表示される CCD、9.0–11.5 keV の統計量、gain fit の結果を確認します。

#### XSPEC で線源成分と同時にフィットする

配布元の例に従い、生成したローカルモデルを `lmod` で読み込みます。`./` を省略しないでください。線源放射には通常の ARF/RMF を、粒子起源 NXB には別の RMF を指定します。

```text
XSPEC> lmod pb_nb_a_pkg ./21361_sur2nb
XSPEC> data 1:1 <source.pi>
XSPEC> response 1:1 <source.rmf>
XSPEC> arf 1:1 <source.arf>
XSPEC> response 2:1 <particle-background.rmf>
XSPEC> model 1:source <source-model>
XSPEC> model 2:pb pb_nb_a
```

NXB は望遠鏡で集光された X 線ではないため、NXB モデル側へ source ARF を掛けません。sky Background は source response を通る別成分として加えます。9.0–11.5 keV の帯域では、sky Background が NXB モデルの規格化に影響するため、NXB の規格化を固定するか自由にするかを慎重に判断します。

S1 と S3 では、観測時期によって 2–6 keV の連続成分を低く予測する場合があります。必要性を residual で確認し、追加成分を使った場合は理由と判断根拠を記録します。

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
- CIAO、CALDB、HEASoft、`mkacispback` の版
- 使用した PHA、ARF、RMF とフィット帯域
- 統計量、binning、吸収・放射・sky Background・NXB のモデル式
- NXB の規格化を固定したか自由にしたか、その判断理由
