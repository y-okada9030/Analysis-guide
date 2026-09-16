# mkacispback による粒子起源背景モデリング

このページは [Chandra/ACIS](chandra.md) の「4.2 丁寧な NXB モデリングを行う場合」から参照されています。検出器の粒子起源背景（Particle-Induced Background, NXB）を科学的にモデル化する高度な手法です。

!!! note
    **基本的な解析のほとんどは Blank-sky Background で対応できます。** このページの手法は、特に背景の正確性が重要な場合に検討してください。

## mkacispback とは

[`mkacispback`](https://github.com/hiromasasuzuki/mkacispback) は、観測イベントファイルから Chandra ACIS の**粒子起源 NXB**を XSPEC スペクトルモデルとして直接生成するツールです。

`mkacispback` が表すのは検出器の**粒子起源 NXB**のみです。以下は含みません：

- 宇宙 X 線背景放射
- 銀河系前景放射
- Solar Wind Charge Exchange

スペクトルフィットでは、これらを別成分として加える必要があります。

## 適用される場合

以下のいずれかに当てはまる場合の検討対象です：

- 視野全体に放射が広がり、局所 Background 領域を確保できない
- blank-sky Background の雑音がスペクトル解析を妨害する
- NXB の空間変動や時間変動を詳細に扱う必要がある
- 発表用論文で NXB 処理の透明性が重要

通常の広がった線源解析では、blank-sky または stowed Background で十分です。

## 導入と環境設定

配布元の README が示す主な依存関係：

- CIAO と Chandra CALDB
- HEASoft
- Astropy を利用できる Python 3
- C++11 対応コンパイラ

### リポジトリの取得

```sh
git clone https://github.com/hiromasasuzuki/mkacispback.git /path/to/mkacispback
```

### 環境変数の設定

```sh
export ACISPBACK=/path/to/mkacispback
export ACISPBACK_PYTHON=/path/to/python3
export ACISPBACK_GXX=/path/to/g++
export PATH="$ACISPBACK:$PATH"
```

`ACISPBACK_PYTHON` には `import astropy` が成功する Python を、`ACISPBACK_GXX` には C++11 を扱えるコンパイラを指定します。実行前には HEASoft と CIAO の両方を有効にしておきます。

### 設定確認

```sh
which mkacispback
echo "$CALDB"
"$ACISPBACK_PYTHON" -c 'import astropy; print(astropy.__version__)'
mkacispback --h
```

すべてのコマンドが想定した場所から見つかることを確認します。

### 設定例

手元に `/Users/yamazakipc/mkacispback` がある場合：

```sh
export ACISPBACK=/Users/yamazakipc/mkacispback
export PATH="$ACISPBACK:$PATH"
```

これは個人環境の実例です。自分のパスに置き換えます。

## NXB モデルの生成

### 基本的な実行例

次の例では、再投影済みイベントから `sur2nb.reg` の領域を選び、`21361_sur2nb/` に `pb_nb_a` という NXB モデルを生成します。

```sh
mkacispback \
  "21361_reproj_evt.fits.gz[sky=region(sur2nb.reg)]" \
  outdir=21361_sur2nb \
  name=pb_nb_a
```

引用符は、CIAO の Data Model 式に含まれる角括弧や丸括弧をシェルに解釈させないために必須です。

- `outdir`：生成物の保存先ディレクトリ
- `name`：XSPEC に登録するモデル名（英小文字と `_` のみ、数字は不可）

### 生成物

処理後、以下が生成されます：

- **weight map**：検出器応答のマップ
- **データスペクトル**（`temp_spec.pi`）：9.0–11.5 keV 帯域での粒子イベント
- **NXB 用 RMF**（`temp.rmf`）：NXB スペクトルの応答
- **XSPEC ローカルモデル**：`<outdir>/` ディレクトリ

処理ログに表示される以下を確認します：

- 各 CCD での処理状況
- 9.0–11.5 keV の統計量と gain fit の結果
- C-Statistic と自由度

## XSPEC での利用

### モデルの読み込み

```text
XSPEC> lmod pb_nb_a_pkg ./21361_sur2nb
```

`./` を省略しないでください。モデルディレクトリが相対パスで認識されます。

### スペクトルセットアップ

```text
XSPEC> data 1:1 <source.pi>
XSPEC> response 1:1 <source.rmf>
XSPEC> arf 1:1 <source.arf>
XSPEC> response 2:1 <particle-background.rmf>
XSPEC> model 1:source <source-model>
XSPEC> model 2:pb pb_nb_a
```

重要なポイント：

- **NXB モデル側へ ARF を掛けない**：NXB は望遠鏡で集光された X 線ではないため
- **sky Background は別成分**：source response を通す別モデルとして追加
- **9.0–11.5 keV の注意**：この帯域で sky Background と NXB の重なりに注意

### 背景規格化の判断

S1 と S3 では、観測時期によって 2–6 keV の連続成分を低く予測する場合があります。residual で確認し、追加成分が必要なら理由と判断を記録します。

## 記録と再現性

論文やレポートに含める情報：

- ObsID、検出器（I0–I3、S1–S3）、FAINT/VFAINT
- 抽出領域（region ファイル名）
- CIAO、CALDB、HEASoft、`mkacispback` のバージョンと日付
- 使用した PHA、ARF、RMF
- フィット帯域と統計量
- フィットモデル式（吸収、放射、sky Background、NXB）
- NXB の規格化を固定したか自由にしたか、その判断理由

## 参考資料

- [mkacispback GitHub リポジトリ](https://github.com/hiromasasuzuki/mkacispback)
- [Suzuki et al. 2021, A&A, 665, A116](https://doi.org/10.1051/0004-6361/202141458)：手法と検証
- [CXC ACIS Background Files](https://cxc.cfa.harvard.edu/ciao/threads/acisbackground/)：background 選択の判断
