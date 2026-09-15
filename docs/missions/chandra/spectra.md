# ACIS スペクトルと応答

CIAO Quick Start の点状線源例です。`src.reg` は前章で保存した領域です。

```sh
specextract '4425/repro/acisf04425_repro_evt2.fits[sky=region(src.reg)]' mysrc
```

`specextract` は線源 PHA と対応する ARF/RMF を生成します。上の Quick Start は操作確認用です。**点源として有効面積を評価する場合**、CIAO の [点源スレッド](https://cxc.cfa.harvard.edu/ciao/threads/pointlike/) は `weight=no` と `correctpsf=yes` を指定します。背景 `bkg.reg` がある場合の入力形式は次です。観測固有の PSF、領域の中心、背景の必要性を確認して使います。

```sh
punlearn specextract
specextract '4425/repro/acisf04425_repro_evt2.fits[sky=region(src.reg)]' mysrc \
  bkgfile='4425/repro/acisf04425_repro_evt2.fits[sky=region(bkg.reg)]' \
  weight=no correctpsf=yes
```

抽出後の確認項目は、PHA に対応する背景と応答の参照先、`BACKSCAL`、露出時間、領域、異なる ObsID を混ぜていないかです。拡散源では [extended source の公式スレッド](https://cxc.cfa.harvard.edu/ciao/threads/extended/) を優先し、応答に必要な `weight` などを判断します。

複数 ObsID を同じ対象に使う場合でも、まず各観測で再処理・抽出を行います。結合は [combine_spectra](https://cxc.cfa.harvard.edu/ciao/ahelp/combine_spectra.html) または XSPEC の同時フィットを目的に応じて選び、応答と背景を単純なコピーで共有しません。
