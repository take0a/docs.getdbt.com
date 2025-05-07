---
title: ウェアハウスにデータをロードするにはどうすればいいですか?
description: "ウェアハウスにデータをロードするためのツールに関する推奨事項"
sidebar_label: 'ウェアハウスにデータを取り込むためのツールに関する推奨事項'
id: loading-data

---
dbt は、<Term id="data-warehouse" /> 内に既にデータのコピーが存在することを前提としています。データをウェアハウスに取り込むには、[Stitch](https://www.stitchdata.com/) や [Fivetran](https://fivetran.com/) などの市販ツールのご利用をお勧めします。

**dbt はデータのロードに使用できますか？**

いいえ、dbt はデータの抽出やロードは行いません。変換ステップのみに特化しています。
