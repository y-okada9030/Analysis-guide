# XRISM: Resolve と Xtend

XRISM の [ABC Data Reduction Guide v2.0（2026-06-12）](https://heasarc.gsfc.nasa.gov/docs/xrism/analysis/abc_guide/) を主資料とします。Resolve は高分解能分光、Xtend は CCD 画像・分光で、同じ衛星でも PI、領域、grade、応答生成が異なります。公式 [Quick-Start Guide v3.2](https://heasarc.gsfc.nasa.gov/docs/xrism/analysis/quickstart/) は補助資料です。版の新旧と既知の不具合を解析前に照合してください。

1. HEASoft と XRISM CALDB が使える環境を整える。
2. [HEASARC の XRISM データ](https://heasarc.gsfc.nasa.gov/docs/xrism/analysis/abc_guide/XRISM_Data_Analysis.html) と観測ログ、処理ノートを取得する。
3. 配布済み cleaned event、モード、追加スクリーニング、観測の明るさを確認する。
4. [Resolve](resolve.md) または [Xtend](xtend.md) の抽出・応答へ進む。

ABC Guide は N132D を例に使いますが、**その天体の座標、領域、背景、明るさの判断を他の対象に転用しません**。ここでは `<XRISM_CLEANED_EVT>` のようなプレースホルダーを使います。配布ファイルの名前は ObsID、CCD、処理モードで変わります。

解析用に配布パッケージと分けたフォルダーを作り、ログと出力を保存します。再処理が必要か、既存 cleaned event から始めてよいかは [公式 Overview](https://heasarc.gsfc.nasa.gov/docs/xrism/analysis/abc_guide/XRISM_Data_Analysis.html) のソフト・校正更新と観測条件を見て判断します。
