# 領域・背景・応答

## 領域

DS9 で線源領域と背景領域を作り、座標系・形・大きさを記録します。点源は PSF と周辺線源を考え、拡散源は抽出領域に加えて **空での輝度分布** を応答生成に反映する場合があります。地域の見かけの空白が適切な背景とは限りません。

## 背景

背景には機器由来の non X-ray background (NXB)、空の diffuse background、観測内の局所背景などがあります。解析対象が拡散している場合、局所背景にも線源の光子が混ざり得ます。Xtend のスペクトルでは線源と背景領域を同じ座標系で保存し、`BACKSCAL` を確認します。Resolve の NXB は単純な隣接ピクセルの空領域に置き換えず、[公式手順](https://heasarc.gsfc.nasa.gov/docs/xrism/analysis/abc_guide/Resolve_Data_Analysis.html) を使います。

## 応答

RMF は入射光子エネルギーと検出チャンネルの対応、ARF は有効面積を表します。スペクトルと同じ観測・grade・領域・時間帯で作る必要があります。Chandra の `specextract` は PHA と対応する応答を一緒に作ります。XRISM Resolve は `rslmkrmf` + `xaarfgen` または `rslmkrsp`、Xtend は `xtdrmf` + `xaarfgen` を使います。応答の線源形状は点源と拡散源で変更します。

XSPEC に入れる前に、スペクトルの `RESPFILE`、`ANCRFILE`、`BACKFILE` などの参照先が妥当か確認してください。必要なら XSPEC 内で対応するファイルを明示します。応答作成の完全なコマンド例は観測ごとの入力が多いため、XRISM [Resolve](../missions/xrism/resolve.md) と [Xtend](../missions/xrism/xtend.md) の章では公式例の入口と必須判断を示します。

