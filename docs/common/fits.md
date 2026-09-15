# FITS とイベント

FITS イベント表には時刻、検出器座標、空の座標、エネルギー相当の PI などが入ります。PI のエネルギー換算、grade、座標系は機器ごとに異なり、同じ数値のフィルターを他機器へ移せません。

CIAO で FITS ヘッダーを読む例です。`<EVENT_FILE>` は自分のイベントファイルに置き換えます。

```sh
dmlist '<EVENT_FILE>' header,clean
```

HEASoft 環境ではヘッダーのキーワードを確認できます。

```sh
fkeyprint '<SPECTRUM_FILE>' BACKSCAL
```

スペクトルの PHA/PI とイベント EVT、応答 RMF/ARF、背景スペクトルを取り違えないようにします。値の意味は FITS ヘッダーと各ミッションの公式データ構造の説明に戻って確認してください。

