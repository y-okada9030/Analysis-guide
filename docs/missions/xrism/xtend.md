# Xtend: 画像・スペクトル

Xtend は CCD イメージング分光器です。[ABC Guide Xtend 章](https://heasarc.gsfc.nasa.gov/docs/xrism/analysis/abc_guide/Xtend_Data_Analysis.html) に従い、配布イベントの観測モード、追加 screening、flaring、bright source、窓モードを確認します。PI フィルターの数値は Resolve と共通ではありません。

端末で XSELECT を起動します。

```sh
xselect
```

以下は **XSELECT 内** で入力します。ファイルを実際の Xtend cleaned event に替え、必要なら公式の追加 screening を先に適用します。

```text
read events <XRISM_XTEND_CLEANED_EVT> .
set image sky
extract image
save image xtend_sky.img
```

DS9 で画像を見て `source.reg` と `background.reg` を保存します。Xtend の `xaarfgen` は RADEC 座標の領域を扱えますが、XSELECT の region file の形式・座標を一致させてください。CCD の窓、チップ端、他の線源を確認します。

```text
filter region source.reg
extr "image spectrum"
save spec xtend_source.pi
clear region
filter region background.reg
extr "image spectrum"
save spec xtend_background.pi
clear region
```

線源と背景の `BACKSCAL` を見ます。異なる座標系で保存すると面積比が合わないことがあります。

```sh
fkeyprint xtend_source.pi BACKSCAL
fkeyprint xtend_background.pi BACKSCAL
```

Xtend RMF は spectrum を入力に `xtdrmf` で作ります。これは公式例のファイル名を一般化した入力です。

```sh
punlearn xtdrmf
xtdrmf xtend_source.pi xtend_source.rmf
```

ARF は `xaarfgen` で生成しますが、点源では位置と `sourcetype=POINT`、拡散源では画像・輝度分布と領域の設定が必要です。露出マップ、ray-tracing file、RMF に一致するエネルギー格子も指定します。[公式 7.7.2](https://heasarc.gsfc.nasa.gov/docs/xrism/analysis/abc_guide/Xtend_Data_Analysis.html) の条件別の完全な入力を使い、Quick-Start の既知の `xaarfgen` workaround が現行版にも必要か確認してください。

```sh
fhelp xaexpmap
fhelp xaarfgen
```

NXB、pile-up、明るい線源は [公式 7.8–7.9](https://heasarc.gsfc.nasa.gov/docs/xrism/analysis/abc_guide/Xtend_Data_Analysis.html) を参照します。
