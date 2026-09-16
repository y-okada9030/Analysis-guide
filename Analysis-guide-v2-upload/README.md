# 広がったX線源の解析ガイド

Chandra/ACIS と XRISM/Resolve・Xtend の **extended source** 解析に重点を置いた日本語サイトです。超新星残骸や銀河団などを想定し、元データの準備から領域、背景、露出補正、スペクトル、応答、結果確認までを一連の作業として説明します。構成は [CIAO Extended Sources Analysis Guide](https://cxc.cfa.harvard.edu/ciao/guides/esa.html) に倣っています。点源、HRC、グレーティング、時間変動などはサイト内で公式 [CIAO Analysis Guides](https://cxc.cfa.harvard.edu/ciao/guides/) へ案内します。

## 内容

```text
mkdocs.yml                 # サイト設定と7ページの目次
docs/
  index.md                 # 対象と読み方
  preparation.md           # 観測・校正・作業場所
  chandra.md               # ACIS の extended source
  xrism.md                 # Resolve・Xtend の extended source
  spectra.md               # スペクトル解析と確認
  other-analyses.md        # 対象外の解析と将来のミッション
  references.md            # 公式資料
.github/workflows/pages.yml
```

旧版の細分化ページが GitHub 上に残っていても、`exclude_docs` により公開サイトには含めません。将来、旧ページを repository から整理するときは、内容とリンクを確認してから削除します。

入力するコマンドはコピー可能なコードブロックに載せています。ObsID 869 は Chandra 公式 extended-source thread の例です。`<...>` は自分の観測のファイル名に置き換えてください。現在扱わない Suzaku、NuSTAR、XMM-Newton は将来、ミッションごとにページを追加する方針です。

## ローカルプレビュー

Python 3 の仮想環境で実行します。

```sh
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
mkdocs serve
```

ブラウザーで `http://127.0.0.1:8000/` を開きます。公開前に以下を実行します。

```sh
mkdocs build --strict
```

## GitHub Pages

このディレクトリの内容を [`y-okada9030/Analysis-guide`](https://github.com/y-okada9030/Analysis-guide) の repository ルートへ登録します。GitHub の Settings → Pages で Source を **GitHub Actions** に設定します。`main` の更新で `.github/workflows/pages.yml` がサイトを構築・公開します。公開先は [広がったX線源の解析ガイド](https://y-okada9030.github.io/Analysis-guide/) です。

## PDF・Overleaf への派生

Markdown を原本として管理します。PDF が必要なら特定の版を固定し、Pandoc などでコードブロックと公式リンクを保ったまま変換します。Overleaf へは章単位で LaTeX に変換し、紙面向けの図番号や文献を整えます。PDF と LaTeX は派生物として扱い、サイトの原稿は Markdown で更新します。

## 参考資料と権利

本文は CXC/Chandra と HEASARC/XRISM の公式資料を中心に書いています。手元の `Work/N132D` などは作業構造の参考に限り、対象固有の領域・モデル・絶対パスを一般手順には載せません。公式 PDF の図や本文は転載していません。画像を追加する場合は自作を優先し、第三者資料の再利用には出典と権利を確認します。
