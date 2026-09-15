# HEASoft と XSPEC

HEASoft の XSPEC は機器に依存しないスペクトルフィットの道具です。Chandra のイベント再処理は CIAO、XRISM のイベントと応答は各機器の公式手順で作ってから、生成したスペクトルを XSPEC へ渡します。環境設定は [環境構築ガイドへの案内](../getting-started/environment.md) を参照してください。

```sh
xspec
```

XSPEC 内で入力する最小例です。`<SOURCE.pha>`、`<BACKGROUND.pha>`、`<RESPONSE.rmf>`、`<AREA.arf>` は解析から得たファイルです。背景や応答が PHA ヘッダーに正しく埋め込まれている場合は対応する行を省略できます。

```text
data <SOURCE.pha>
backgrnd <BACKGROUND.pha>
response <RESPONSE.rmf>
arf <AREA.arf>
plot data
```

モデル、使用エネルギー帯、統計量、グルーピングは信号量と研究目的で決めます。次の `powerlaw` は **操作の例** で、熱的プラズマや吸収を持つ天体の物理モデルとして一般化しません。

```text
model powerlaw
fit
plot ldata delchi
```

使った統計量、パラメーターの固定値と初期値、無視したエネルギー帯、残差を記録します。Resolve の高分解能スペクトルでは安易な粗いグルーピングで線情報を失わないようにします。XSPEC の最新版の [公式マニュアル](https://heasarc.gsfc.nasa.gov/docs/software/xspec/manual/) に、各コマンドと統計量の定義があります。
