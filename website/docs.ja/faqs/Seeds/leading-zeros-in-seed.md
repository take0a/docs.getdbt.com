---
title: シードの先頭のゼロを保持するにはどうすればよいですか?
description: "列タイプを使用してシードの先頭にゼロを含める"
sidebar_label: 'Include leading zeroes in your seed file'
id: leading-zeros-in-seed

---

先頭のゼロを保持する必要がある場合 (たとえば、郵便番号や携帯電話番号)、シード ファイルに先頭のゼロを含め、正しい長さの varchar データ型で `column_types` [構成](reference/resource-configs/column_types.md) を使用します。
