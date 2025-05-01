---
title: エンティティ
id: entities
description: "Entities are real-world concepts that correspond to key parts of your business, such as customers, transactions, and ad campaigns."
sidebar_label: "Entities"
tags: [Metrics, Semantic Layer]
---

エンティティとは、顧客、取引、広告キャンペーンなど、ビジネスにおける現実世界の概念です。
私たちは多くの場合、顧客離脱や年間経常収益モデリングなど、特定のエンティティを中心に分析を行います。
セマンティックモデルでは、エンティティをID列で表します。ID列は、セマンティックグラフ内の他のセマンティックモデルへの結合キーとして機能します。

セマンティックグラフ内では、エンティティに必要なパラメータは「name」と「type」です。
「name」は、基になるデータテーブルのキー列名を参照するか、「expr」パラメータで参照される列名のエイリアスとして機能する場合があります。
エンティティの「name」は、セマンティックモデル内で一意である必要があり、同じモデル内の既存の「measure」または「dimension」と同じにすることはできません。

エンティティは、1つの列または複数の列で指定できます。セマンティックモデル内のエンティティ（結合キー）は、名前で識別されます。
各エンティティ名はセマンティック モデル内では一意である必要がありますが、異なるセマンティック モデル間では一意である必要はありません。

エンティティタイプには次の4種類があります:
- [Primary](#primary) - テーブルの各行にレコードが1つだけ含まれ、データプラットフォーム内のすべてのレコードが含まれます。
このキーはテーブル内の各レコードを一意に識別します。
- [Unique](#unique) - テーブルの各行にレコードが1つだけ含まれ、null値が許容されます。
データウェアハウス内にレコードのサブセットが存在する場合があります。
- [Foreign](#foreign) - あるテーブル内のフィールド（またはフィールドセット）で、別のテーブルの行を一意に識別します。
このキーはテーブル間のリンクを確立します。
- [Natural](#natural) - 実際のデータに基づいてレコードを一意に識別する、テーブル内の列または列の組み合わせ。
このキーは実際のデータ属性から派生します。

:::tip エンティティをディメンションとして使用する
エンティティをディメンションとして使用することもできます。これにより、エンティティの粒度でメトリックを集計できます。
:::

## Entity types

MetricFlow の結合ロジックは、使用するエンティティ `type` に依存し、セマンティックモデルの結合方法を決定します。結合の構築方法の詳細については、[結合](/docs/build/join-logic) を参照してください。

### Primary

主キーは、テーブルの各行に1つのレコードのみを持ち、データプラットフォーム内のすべてのレコードを含みます。主キーは一意の値を持つ必要があり、null値を含めることはできません。主キーを使用することで、テーブル内の各レコードが一意かつ識別可能であることを保証します。

<Expandable alt_header="Primary key example">

たとえば、次の列を持つ従業員のテーブルを考えます:

```sql
employee_id (primary key)
first_name
last_name
```
この場合、`employee_id` が主キーです。各 `employee_id` は一意であり、特定の従業員を表します。
`employee_id` は重複することはできず、null にすることもできません。

</Expandable>

### Unique

ユニークキーは、テーブル内の行ごとに1つのレコードのみを含みますが、データウェアハウス内のレコードのサブセットを含む場合があります。ただし、主キーとは異なり、ユニークキーではNULL値が許容されます。ユニークキーは、NULL値を除き、列の値が一意であることを保証します。

<Expandable alt_header="Unique key example">

たとえば、次の列を持つ学生のテーブルを考えます:

```sql
student_id (primary key)
email (unique key)
first_name
last_name
```

この例では、「email」が一意キーとして定義されています。各メールアドレスは一意である必要がありますが、複数の学生がnullのメールアドレスを持つことができます。これは、一意キー制約により1つ以上のnull値が許容される一方で、null以外の値は一意でなければならないためです。これにより、一意のメールアドレス（null以外）を持つレコードセットが作成され、これはすべての学生を含むテーブル全体のサブセットとなる可能性があります。

</Expandable>

### Foreign

外部キーとは、あるテーブル内のフィールド（またはフィールドセット）で、別のテーブルの行を一意に識別するものです。外部キーは、2つのテーブルのデータ間のリンクを確立します。外部キーには、同じレコードのインスタンスが0個、1個、または複数個含まれる場合があります。また、NULL値を含めることもできます。

<Expandable alt_header="Foreign key example">

たとえば、`customers` と `orders` という 2 つのテーブルがあるとします。

customers table:

```sql
customer_id (primary key)
customer_name
```

orders table:

```sql
order_id (primary key)
order_date
customer_id (foreign key)
```

この例では、`orders` テーブルの `customer_id` は、`customers` テーブルの `customer_id` を参照する外部キーです。このリンクは、各注文が特定の顧客に関連付けられていることを意味します。ただし、すべての注文に顧客が関連付けられる必要はありません。orders テーブルの `customer_id` は、複数の注文で null または同じ `customer_id` を持つことができます。

</Expandable>

### Natural

自然キーとは、テーブル内の列または列の組み合わせであり、実世界のデータに基づいてレコードを一意に識別します。例えば、「sales_person_department」ディメンションテーブルの場合、「sales_person_id」を自然キーとして使用できます。自然キーは[SCDタイプIIディメンション](/docs/build/dimensions#scd-type-ii)でのみ使用できます。

## Entities configuration

以下はエンティティの完全な仕様です:

<VersionBlock firstVersion="1.9">

```yaml
semantic_models:
  - name: semantic_model_name
   ..rest of the semantic model config
    entities:
      - name: entity_name  ## Required
        type: Primary, natural, foreign, or unique ## Required
        description: A description of the field or role the entity takes in this table  ## Optional
        expr: The field that denotes that entity (transaction_id).  ## Optional
              Defaults to name if unspecified.  
        [config](/reference/resource-properties/config): Specify configurations for entity.  ## Optional
          [meta](/reference/resource-configs/meta): {<dictionary>} Set metadata for a resource and organize resources. Accepts plain text, spaces, and quotes.  ## Optional
```
</VersionBlock>

<VersionBlock lastVersion="1.8">

```yaml
semantic_models:
  - name: semantic_model_name
   ..rest of the semantic model config
    entities:
      - name: entity_name     ## Required
        type: Primary, or natural, or foreign, or unique ## Required
        description: A description of the field or role the entity takes in this table ## Optional
        expr: The field that denotes that entity (transaction_id).  ## Optional
              Defaults to name if unspecified.
```
</VersionBlock>

セマンティック モデルでエンティティを定義する方法の例を次に示します:

<VersionBlock firstVersion="1.9"> 

```yaml
entities:
  - name: transaction
    type: primary
    expr: id_transaction
  - name: order
    type: foreign
    expr: id_order
  - name: user
    type: foreign
    expr: substring(id_order from 2)
    entities:
  - name: transaction
    type: 
    description: A description of the field or role the entity takes in this table ## Optional
    expr: The field that denotes that entity (transaction_id).  
          Defaults to name if unspecified.
    [config](/reference/resource-properties/config):
      [meta](/reference/resource-configs/meta):
        data_owner: "Finance team"
```
</VersionBlock>

<VersionBlock lastVersion="1.8"> 

```yaml
entities:
  - name: transaction
    type: primary
    expr: id_transaction
  - name: order
    type: foreign
    expr: id_order
  - name: user
    type: foreign
    expr: substring(id_order from 2)
    entities:
  - name: transaction
    type: 
    description: A description of the field or role the entity takes in this table ## Optional
    expr: The field that denotes that entity (transaction_id).  
          Defaults to name if unspecified.
```
</VersionBlock>

## 列をキーで結合する

テーブルにキー（主キーなど）がない場合は、_代理キーの組み合わせ_を使用して、2つの列を組み合わせることでレコードを識別するためのキーを作成します。これは、すべての[エンティティタイプ](/docs/build/entities#entity-types)に適用されます。たとえば、`raw_brand_target_weekly`テーブルの`date_key`と`brand_code`を組み合わせて_代理キー_を作成できます。次の例では、`date_key`と`brand_code`をパイプ（`|`）で区切って結合し、代理キーを作成しています。

```yaml

entities:
  - name: brand_target_key # Entity name or identified.
    type: foreign # This can be any entity type key. 
    expr: date_key || '|' || brand_code # Defines the expression for linking fields to form the surrogate key.
```
