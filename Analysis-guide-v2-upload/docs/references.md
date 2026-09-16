# 公式資料

各リンクは解析手順と校正の一次資料です。公式ページは更新されるため、解析時に版・更新日・当てはまる caveat を確認してください。このガイドは必要な作業の順番を説明し、観測固有の完全な設定は公式スレッドへ案内します。

## Chandra / CIAO

- [CIAO Analysis Guides](https://cxc.cfa.harvard.edu/ciao/guides/)：機器・解析目的から Science Thread を選ぶ入口。
- [Extended Sources Analysis Guide](https://cxc.cfa.harvard.edu/ciao/guides/esa.html)：背景、点源除外、露出補正、広がった線源のスペクトルをつなぐ公式の道案内。
- [ACIS Data Preparation](https://cxc.cfa.harvard.edu/ciao/guides/acis_data.html)：再処理・フィルタの判断。
- [Extended-source spectrum thread](https://cxc.cfa.harvard.edu/ciao/threads/extended/)：`specextract` と weighted response の具体例。
- [ACIS background files](https://cxc.cfa.harvard.edu/ciao/threads/acisbackground/)：blank-sky の適用と注意点。
- [mkacispback](https://github.com/hiromasasuzuki/mkacispback)：ACIS の粒子起源 Background をスペクトルモデルとして生成する第三者製ツール。README の版、要件、制約を確認して使用する。
- [Suzuki et al. 2021, A&A, 665, A116](https://doi.org/10.1051/0004-6361/202141458)：`mkacispback` の手法と検証。
- [CIAO Data Caveats](https://cxc.cfa.harvard.edu/ciao/caveats/) と [CIAO download](https://cxc.cfa.harvard.edu/ciao/download/)：既知の問題と対応版。

## XRISM / HEASoft

- [XRISM ABC Data Reduction Guide v2.0](https://heasarc.gsfc.nasa.gov/docs/xrism/analysis/abc_guide/)：データの構造、screening、Resolve と Xtend の抽出・応答・NXB。
- [HEASoft](https://heasarc.gsfc.nasa.gov/docs/software/lheasoft/) と [XSPEC Manual](https://heasarc.gsfc.nasa.gov/docs/software/xspec/manual/)：ソフトとスペクトル解析。

本文中の ObsID 869 とツール名は、公式例を**手順の説明に必要な範囲だけ**参照しています。公式ページの図や長い本文は転載していません。
