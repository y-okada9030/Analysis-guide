# X線データ解析ガイド

公開 X線観測データから再現可能な解析結果を作るための入門です。対象は第三者の研究者・学生で、macOS を含む Unix 系の作業環境を想定します。**Chandra/ACIS** と **XRISM/Resolve・Xtend** の現行手順を説明します。Suzaku・NuSTAR・XMM-Newton は入口を用意し、詳細手順は今後追加します。

## 読み方

1. [全体の流れ](getting-started/workflow.md) と [データ管理](getting-started/data-layout.md) を読む。
2. 対象機器の [Chandra](missions/chandra/index.md) または [XRISM](missions/xrism/index.md) へ進む。
3. [応答と背景](common/regions-response.md)、[XSPEC](common/xspec.md)、[品質確認](common/quality.md) を使う。

解析環境の導入には [環境構築ガイドへの案内](getting-started/environment.md) を使います。

## コマンドの約束

本文の入力コマンドはコピー可能なコードブロックに載せます。`sh` は通常の端末、`text` で XSELECT と説明したブロックは XSELECT 内、XSPEC と説明したブロックは XSPEC 内で入力します。対話ツールのプロンプトはブロックに含めません。`<FILE>` のような記号は置換する変数で、そのまま実行しません。公式文書の ObsID・ファイル名を使う例は実データ用の設定と区別します。

機器・観測モード・線源の広がりにより、同じ入力でも適切な応答や背景が変わります。各章の「確認点」を省略せず、[公式資料](appendices/references.md) の更新情報も参照してください。
