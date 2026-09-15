# データとディレクトリ

元データを保存し、派生物を別の場所へ出します。次は **一例** で、ユーザーの `Work/N132D` のフォルダー分けから発想を得つつ、観測 ID と機器を先に置いて第三者に探しやすくした構造です。実在のユーザーパスは固定しません。

```text
xray-project/
  README.md
  data/
    chandra/<obsid>/archive/   # 取得した原本
    xrism/<obsid>/archive/
  analysis/
    chandra/<obsid>/repro/
    chandra/<obsid>/regions/
    chandra/<obsid>/spectra/
    xrism/<obsid>/resolve/
    xrism/<obsid>/xtend/
  scripts/
  logs/
  figures/
```

公式ツールが作るファイル配置とぶつからないよう、解析用ディレクトリの中でそのツールを実行します。Chandra `download_chandra_obsid` は現在のディレクトリに ObsID 名のフォルダーを作ります。XRISM 配布パッケージの内部構造は壊さず、別の `analysis/` で処理します。

Mac での作業場所はホーム以下など書き込み可能な場所を選びます。空白や日本語が混じるパスは、古い天文ツールで問題になることがあるため、解析用フォルダー名は簡潔な ASCII にすると管理しやすくなります。

