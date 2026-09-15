# 公開データの取得と再処理

CIAO を有効にした端末で作業します。公式チュートリアルの ObsID 4425 を、現在のディレクトリへ取得し再処理する例です。

```sh
download_chandra_obsid 4425
chandra_repro 4425 outdir=
```

`download_chandra_obsid` は公開済みデータのみを取得します。非公開データにはアーカイブの適切な認証付き経路を使います。`chandra_repro` は出力先を確認してから実行し、CIAO・CALDB の版を解析記録へ残してください。公式の [Quick Start](https://cxc.cfa.harvard.edu/ciao/guides/quick_start.html) と [アーカイブ取得スレッド](https://cxc.cfa.harvard.edu/ciao/threads/archivedownload/) が手順の根拠です。

実データでは ObsID を `<OBSID>` に置換し、出力済みフォルダーを上書きしない作業場所で実行します。

```sh
download_chandra_obsid <OBSID>
chandra_repro <OBSID> outdir=
```

再処理後は `repro/` 内の `evt2` イベントファイル、処理ログ、bad pixel と GTI を確認します。ACIS の特殊な観測条件については `chandra_repro` の公式ヘルプと関連スレッドへ進みます。

