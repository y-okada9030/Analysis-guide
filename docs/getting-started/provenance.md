# 解析記録

他者が判断を追えるよう、観測と解析条件を残します。とくに点源/拡散源の別、使用した領域、背景の出典、PI の grade、RMF/ARF の作り方は結果を変えます。

端末で科学ツールを実行する前後に、版と日時を記録できます。各ツールが有効な環境で実行してください。

```sh
date -u
command -v chandra_repro
command -v xspec
command -v xselect
```

CIAO と HEASoft の版は、それぞれの公式版表示手順に従って確認します。CALDB とデータ配布時の processing version も解析ノートに記入します。実行ログ、領域ファイル、XSPEC コマンドファイルは解析結果と一緒に保存し、取得した原本は編集しません。
