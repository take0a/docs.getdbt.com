---
title: "バージョン管理されたSQLモデルのユニットテスト"
sidebar_label: "Versions"
---

モデルに複数のバージョンがある場合、デフォルトのユニットテストはモデルの_すべての_バージョンに対して実行されます。ユニットテストを実行するモデルのバージョンを指定するには、モデルバージョン設定で、必要なバージョンに対して `include` または `exclude` を使用してください:

```yaml

# my test_is_valid_email_address unit test will run on all versions of my_model
unit_tests:
  - name: test_is_valid_email_address
    model: my_model
    ...
            
# my test_is_valid_email_address unit test will run on ONLY version 2 of my_model
unit_tests:
  - name: test_is_valid_email_address 
    model: my_model 
    versions:
      include: 
        - 2
    ...
            
# my test_is_valid_email_address unit test will run on all versions EXCEPT 1 of my_model
unit_tests:
  - name: test_is_valid_email_address
    model: my_model 
    versions:
      exclude: 
        - 1
    ...

```
