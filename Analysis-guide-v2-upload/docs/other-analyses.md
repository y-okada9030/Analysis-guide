# 対象外の解析と今後の追加

このサイトの詳しい手順は、**広がった線源**を対象にしています。点源の flux や PSF 補正、グレーティングの線解析、時間変動、HRC データを同じ手順で扱うと、領域、背景、応答の選び方を誤ることがあります。該当する解析は [CIAO Analysis Guides](https://cxc.cfa.harvard.edu/ciao/guides/) から、目的に合う公式ガイドを選んでください。

| 解析したいこと | 最初に読む公式案内 |
| --- | --- |
| Chandra の点源 | [CIAO Quick Start](https://cxc.cfa.harvard.edu/ciao/guides/quick_start.html) と [pointlike-source spectrum thread](https://cxc.cfa.harvard.edu/ciao/threads/pointlike/) |
| HRC 画像 | [HRC Data Preparation](https://cxc.cfa.harvard.edu/ciao/guides/hrc_data.html) と [HRC Imaging](https://cxc.cfa.harvard.edu/ciao/guides/hrc_imaging.html) |
| HETG・LETG 分光 | [HETG/ACIS-S](https://cxc.cfa.harvard.edu/ciao/guides/gspec_acishetg.html) と [LETG/ACIS-S](https://cxc.cfa.harvard.edu/ciao/guides/gspec_acisletg.html) |
| 点源検出・PSF | [Detection と PSF Central](https://cxc.cfa.harvard.edu/ciao/guides/) |
| 複数 Chandra 観測の位置合わせ | [Merging Central](https://cxc.cfa.harvard.edu/ciao/merging/merge_central.html) |
| XRISM の点源や明るい線源 | [XRISM ABC Guide の Resolve / Xtend 分岐](https://heasarc.gsfc.nasa.gov/docs/xrism/analysis/abc_guide/) |

Suzaku、NuSTAR、XMM-Newton の章は今後追加する予定です。現段階では具体的なコマンドや校正条件を推測して載せません。解析を始める際は、各ミッションの**現在の公式データ解析ガイドとソフト・CALDB の更新情報**を確認してください。追加章はこのサイトの Chandra / XRISM と同じ流れにします：対象となる観測モード、元データの準備、広がった線源の領域と背景、位置依存の応答、結果の点検。新しいミッションごとに独立したページを追加できるため、目次を細かく増やしすぎずに拡張できます。
