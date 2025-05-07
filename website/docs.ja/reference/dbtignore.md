---
title: .dbtignore
---

[dbt プロジェクト](/docs/build/projects) のルートに `.dbtignore` ファイルを作成すると、dbt によって **完全に** 無視されるファイルを指定できます。このファイルは [`.gitignore` ファイルと同様に動作し、同じ構文を使用します](https://git-scm.com/docs/gitignore)。このパターンに一致するファイルとサブディレクトリは、存在しないかのように、dbt によって読み込まれたり、解析されたり、その他の方法で検出されたりすることはありません。

**Examples**

<File name=".dbtignore">

```md
# .dbtignore

# ignore individual .py files
not-a-dbt-model.py
another-non-dbt-model.py

# ignore all .py files
**.py

# ignore all .py files with "codegen" in the filename
*codegen*.py

# ignore all folders in a directory
path/to/folders/**

# ignore some folders in a directory
path/to/folders/subfolder/**

```

</File>
