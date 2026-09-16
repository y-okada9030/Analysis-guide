# 改稿時の確認（2026-09-15）

- 目次を22ページから7ページへ整理。古い細分化ページはローカル原稿から削除し、GitHub 上に残った場合も `exclude_docs` で公開サイトから除外する。
- CIAO Analysis Guides、Extended Sources、ACIS Data Preparation、extended-source spectrum thread と XRISM ABC Guide v2.0 を確認して章の順序と判断事項を改稿した。
- Chandra の点源例を主手順から外し、公式 extended-source の ObsID 869 を説明例にした。
- 点源、HRC、グレーティングなどの対象外解析は公式ガイドへ直接リンクした。
- 7ページすべてが `mkdocs.yml` の目次にあり、内部リンクとコードブロックは静的検査で問題なし（16ブロック）。
- ローカル MkDocs パッケージが使えず、ネットワーク制限で導入できなかったため、`mkdocs build --strict` は今回未実行。GitHub 更新後の Actions 実行で最終確認が必要。
- 観測データを使う科学コマンドは実行していない。観測固有の結果は未検証。
