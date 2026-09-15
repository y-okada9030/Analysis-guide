# Chandra/ACIS

ACIS の公開 ObsID を例に、[取得・再処理](reprocess.md) → [画像・領域](imaging.md) → [スペクトル・応答](spectra.md) の順に進みます。CIAO 4.18 の [公式 Quick Start](https://cxc.cfa.harvard.edu/ciao/guides/quick_start.html) に沿った入門です。HRC、LETG/HETG グレーティング、混雑した視野、広がった線源には別の公式スレッドが必要です。

ここで使う `4425` は公式チュートリアルの公開例です。自分の ObsID ではファイル名と解析条件を確認して置き換えます。Chandra CALDB は CIAO と合わせて更新し、公開データでも最新校正で `chandra_repro` します。

