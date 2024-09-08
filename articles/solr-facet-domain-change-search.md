---
title: "[Solr] Faceted Search: `domain changes` と `wildcard` を使った柔軟な検索"
emoji: "🔎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Solr, Facet, Domain Change, Search, Faceted Search]
published: true
---

# SolrでのFaceted Search: `domain changes` と `wildcard` を使った柔軟な検索

## はじめに

ファセット検索は、ECサイトや大規模なデータベースでユーザーがデータを絞り込むのに便利な機能です。商品検索で「価格帯」「ブランド」「カテゴリ」などのフィルタを使ったことがある人は多いと思いますが、Solrを使ったファセット検索も、同じように簡単に絞り込みができる強力なツールです。

でも、大量のファセット項目がある場合は、もう少し工夫が必要です。私の場合、無限スクロールでファセットを表示していたのですが、ユーザーがその中から検索できるようにする必要がありました。

ここで `facet.prefix` を使うという選択肢もありますが、もう少し柔軟な検索を実現したかったので、Solrの **domain changes** 機能と **wildcard** 検索を組み合わせて対応しました。

## `domain changes` と `wildcard` を活用

### domain changesでファセットにフィルタを追加

`domain changes` は、ファセットごとに独自の検索条件を適用できる機能です。これを使うと、ファセットにだけ特定の条件を追加できて、メインの検索とは別の条件で絞り込むことができます。

たとえば、こんな感じです：

```json
{
  "query": "main_search_term",
  "facet": {
    "categories": {
      "type": "terms",
      "field": "category",
      "domain": {
        "filter": "subcategory:electronics"
      }
    }
  }
}
```

このクエリでは、`category` フィールドのファセットに対して、`subcategory:electronics` というフィルタを適用しています。これで、ファセット結果を「電子機器」だけに絞り込めるようになります。

### wildcardを使って曖昧検索

さらに、ファセットの検索で曖昧検索をしたい場合は、**wildcard** を使うことで実現できます。具体的には、次のようなクエリです。

```json
{
  "query": "main_search_term",
  "facet": {
    "categories": {
      "type": "terms",
      "field": "category",
      "domain": {
        "filter": "category:*electronics*"
      }
    }
  }
}
```

この例では、`*electronics*` という `wildcard` 検索を使っています。これによって、「electronics」という単語を含むすべてのカテゴリがヒットします。カテゴリ名が「Home Electronics」でも「Portable Electronics」でも、どちらも結果に含まれるわけです。

### 大文字・小文字を区別しない検索

Solrはデフォルトで大文字・小文字を区別して検索しますが、`case-insensitive` な検索をするにはスキーマの設定が必要です。`LowerCaseFilterFactory` というフィルターをスキーマに追加すると、大文字小文字を区別せずに検索できるようになります。

```xml
<fieldType name="text_general" class="solr.TextField" positionIncrementGap="100">
  <analyzer type="index">
    <tokenizer class="solr.StandardTokenizerFactory"/>
    <filter class="solr.LowerCaseFilterFactory"/>
  </analyzer>
  <analyzer type="query">
    <tokenizer class="solr.StandardTokenizerFactory"/>
    <filter class="solr.LowerCaseFilterFactory"/>
  </analyzer>
</fieldType>
```

これで、たとえば「Electronics」と「electronics」どちらも同じカテゴリとして検索結果に表示されるようになります。

## まとめ

Solrの **domain changes** や **wildcard** 検索を使えば、ファセット検索に柔軟なフィルタをかけることができます。特に、ユーザーが大量のファセット項目から素早く絞り込めるようにするには、この方法が非常に便利です。また、`case-insensitive` な検索もスキーマを調整することで簡単に実現できます。

これらを組み合わせて使うことで、ファセット検索の精度を上げて、より使いやすい検索機能を提供できるようになります。気になった方は、[公式ドキュメント](https://solr.apache.org/guide/solr/latest/query-guide/json-faceting-domain-changes.html#adding-domain-filters) もぜひ参照してみてください。
