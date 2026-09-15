# Resolve: Hp スペクトル

Resolve は Hp grade を基準とする入門です。ABC Guide は、暗い対象の Hp を良く校正された高分解能事象として説明します。明るい対象、Mp を含める解析、特殊なピクセルや時間帯は別の選択と応答が必要です。開始前に [Resolve 章の issues・screening](https://heasarc.gsfc.nasa.gov/docs/xrism/analysis/abc_guide/Resolve_Data_Analysis.html) を確認してください。

## cleaned event を読む

端末で XSELECT を起動します。

```sh
xselect
```

次は **XSELECT 内** の入力です。読み込むファイルは実際に配布された Resolve cleaned event に置き換えます。XSELECT がミッション確認を求めたら XRISM を確認します。

```text
read events <XRISM_RESOLVE_CLEANED_EVT> .
filter GRADE 0:0
extr spectrum
save spectrum resolve_hp.pi
clear grade
```

公式例では全観測・Hp のスペクトルを抽出します。点源が中心にあり混入が少ない場合は全ピクセルを使えることがあります。拡散源や周辺線源のある視野では DET ピクセル選択と空での混合を検討してください。grade を取り除いた後のイベントを応答用に使う点にも注意します。

## 応答を作る前の確認

Resolve の RMF/RSP は grade 分岐率、ピクセル、時間、CALDB に依存します。現在の ABC Guide には `rslmkrmf`+`xaarfgen` と `rslmkrsp` の二つの経路があります。後者はエネルギー依存の grade 分岐率を考慮します。点源・拡散源、明暗、FOV 外からの寄与により `sourcetype` と `imgfile` が変わります。

公式の Hp・全ピクセル RMF コマンドの骨格です。`<ALL_GRADE_CLEANED_EVT>` は**Hp だけに絞ったファイルではなく**、公式 6.7.4.1 に従って応答用に用意した event です。線源スペクトルと同じピクセル、GTI、`resolist=0` を指定します。例の `whichrmf=L` は公式の特定目的用で、必要な行列サイズ・品質は解析目的に合わせて決めます。

```sh
punlearn rslmkrmf
rslmkrmf infile=<ALL_GRADE_CLEANED_EVT> outfileroot=resolve_hp \
  regmode=DET whichrmf=L resolist=0 regionfile=ALLPIX
```

ARF の `xaarfgen` には露出マップ、望遠鏡レイトレース、線源位置と形、対象と抽出領域、CALDB 入力が必要です。値を埋めるだけで得られる汎用の安全なコマンドはないため、[公式 6.7.3–6.7.5](https://heasarc.gsfc.nasa.gov/docs/xrism/analysis/abc_guide/Resolve_Data_Analysis.html) の点源または拡散源例を観測に合わせて使用します。まずツールの現行パラメーターを調べる入力です。

```sh
fhelp xaexpmap
fhelp xaarfgen
fhelp rslmkrsp
```

Resolve の NXB は [ABC Guide 6.8](https://heasarc.gsfc.nasa.gov/docs/xrism/analysis/abc_guide/Resolve_Data_Analysis.html) に従って生成・評価します。銀河団や超新星残骸のような拡散源では PSF による [spatial-spectral mixing](https://heasarc.gsfc.nasa.gov/docs/xrism/analysis/abc_guide/Resolve_Data_Analysis.html) を考慮し、単一の均一な線源を仮定しません。
