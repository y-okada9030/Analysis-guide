# DS9 画像解析でよくある間違い

このページは [DS9 を用いた Chandra SNR 画像解析](ds9.md) から参照されています。

## Energy の単位を取り違える

ACIS event file の `energy` column は通常 eV です。

```text
正：energy=500:7000
誤：energy=0.5:7.0
```

一方、`fluximage` の `bands` parameter は keV を使います。

## RGB channel の画像範囲が異なる

画像の配列サイズ、sky grid、pixel size が異なると、色ずれや端の人工構造が生じます。同じ event file から作る場合でも、3 band で同じ bin expression を使います。

## PNG や JPEG を天球座標で重ねようとする

通常の PNG、JPEG は天文 WCS を保持しません。FITS を用いるか、別途 astrometric calibration を行います。

## WCS match を再投影とみなす

`Match WCS` と `Lock WCS` は表示同期であり、pixel array を同じ grid に変換しません。ratio map や pixel-by-pixel comparison の前には再投影が必要です。

## Display smoothing を解析済み画像とみなす

DS9 smoothing は元 FITS を変更しません。論文図の処理条件を残す場合は smoothing parameter を記録し、必要なら処理済み FITS を別名で保存します。

## Contour の低い level を採用しすぎる

background fluctuation や画像端を構造として表示する危険があります。background rms、significance、exposure threshold を確認します。

## Exposure map の端を実構造とみなす

exposure の低い視野端では補正値が不安定になります。counts image、exposure map、thresholded image を併せて確認してください。

## 色を物理量に直接対応させる

RGB の色は、energy band、吸収、温度、電離状態、元素組成、non-thermal emission、background、display scale の組み合わせで決まります。色だけから単一の物理量を断定しないでください。

## チェックリスト

- 入力が reprocessed event file である
- event filter と `fluximage` で energy の単位を取り違えていない
- 3 band の sky range、pixel size、WCS が揃っている
- counts image と exposure-corrected image を混在させていない
- scale function と limits を記録した
- smoothing の種類と scale を記録した
- contour level、単位、入力画像、smoothing を記録した
- WCS alignment を point source などで確認した
- region を sky coordinates で保存した
- PNG/TIFF、region、contour、DS9 backup、設定メモを保存した
- 視野端、chip gap、pile-up、readout streak、低 exposure 領域を確認した
