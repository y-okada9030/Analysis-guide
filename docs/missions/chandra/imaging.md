# 画像と領域

公式例の再処理済みイベントを DS9 で開きます。

```sh
ds9 4425/repro/acisf04425_repro_evt2.fits
```

線源を見て領域を描き、DS9 形式で `src.reg` として保存します。背景が必要なら `bkg.reg` も別に作り、重なりやチップ端を確認します。座標系と領域ファイルを解析記録に残してください。

露出補正画像が必要な場合は CIAO の [fluximage](https://cxc.cfa.harvard.edu/ciao/ahelp/fluximage.html) を利用します。次はコマンド形式の入口で、`<EVENT_FILE>` と `<OUTROOT>` を自分のデータに置き換え、エネルギーバンドなどの設定を公式スレッドに合わせて指定します。

```sh
fluximage <EVENT_FILE> <OUTROOT>
```

イベント数画像と露出補正画像は用途が異なります。色や平滑化の強さを変更した画像だけで線源領域を決めず、元イベントと露出の偏りも確認します。

