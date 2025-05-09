---
title: "Record timing info"
id: "record-timing-info"
---

`-r` または `--record-timing-info` フラグは、パフォーマンスプロファイリング情報をファイルに保存します。このファイルは `snakeviz` で視覚化でき、dbt 呼び出しのパフォーマンス特性を把握できます。

<File name='Usage'>

```text
$ dbt -r timing.txt run
...

$ snakeviz timing.txt
```

</File>

あるいは、[`py-spy`](https://github.com/benfred/py-spy) を使用して、次のように dbt コマンドの [speedscope](https://github.com/jlfwong/speedscope) プロファイルを収集することもできます。

```shell
python -m pip install py-spy
sudo py-spy record -s -f speedscope -- dbt parse
```
