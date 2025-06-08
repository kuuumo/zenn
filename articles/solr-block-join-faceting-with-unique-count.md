---
title: "SolrのBlock Join Faceting入門：親の検索条件を引き継ぎ子からファセット取得"
emoji: "🌞"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Solr", "Facet", "Block Join"]
published: true
---

# Solr の Block Join Faceting 入門：親の検索条件を引き継ぎ子からファセット取得

Apache Solr を使って、検索対象が親ドキュメント（例：記事）でありながら、ファセットはその子ドキュメント（例：タグ）に対して取得したい場面は少なくありません。本記事では、Solr の`blockChildren`ドメイン機能を活用し、親の検索条件を引き継ぎつつ、子ドキュメントに対する正確なファセット集計を行う方法を紹介します。

---

## 背景：親子ドキュメント構造とファセットの課題

たとえば、`article`が親ドキュメント、`tags`がその子ドキュメントという構造があり、以下のような要件を満たしたいとします：

- 記事を `title` や `category` で検索条件にかける（`fq`で絞る）
- その記事に紐づいたタグ（子ドキュメント）の `name` をファセットとして取得したい
- 無料ユーザーには `tags.theme_type:free` に限定したファセットを表示したい

一見シンプルに見えるこの要件も、Solr の内部構造を知らないと正確なカウントが得られなかったり、検索結果とファセットの整合性が取れなかったりします。

---

## 基本構文：blockChildren による子ドキュメントへのドメイン変換

```json
{
  "q": "*:*",
  "fq": ["title:検索条件", "category:カテゴリ条件"],
  "json.facet": {
    "tags": {
      "type": "terms",
      "field": "tags.name",
      "domain": {
        "blockChildren": "type_s:article"
      }
    }
  }
}
```

この構文で、親ドキュメント（type_s=article）を検索条件に使いつつ、子ドキュメント（tags）に対するファセットを取得できます。

---

## 応用：無料ユーザーに限定したタグのファセット

無料ユーザーには `tags.theme_type:free` で絞る必要があります。以下のように `domain.filter` を使います：

```json
{
  "json.facet": {
    "tags_theme": {
      "type": "terms",
      "field": "tags.theme",
      "domain": {
        "blockChildren": "type_s:article",
        "filter": "tags.theme_type:free"
      }
    }
  }
}
```

これで、記事を検索した上で、無料テーマに絞ったタグのファセットを取得可能です。

---

## 注意：ファセットのカウントが検索結果と一致しないケース

Solr では、`blockChildren` を使用すると、検索対象（親ドキュメント）とファセット対象（子ドキュメント）の数が一致しないことがあります。たとえば：

- 3 件の記事（親）
- 各記事に 4 つのタグ（子）がある場合、ファセットのカウントは最大で 12 になります

この設計は意図的です。Solr は親子構造を持つドキュメントにおいて、ファセットをより柔軟に集計するためにこのアプローチを採用しています。

---

## 解決策：uniqueBlock で親単位のユニークカウントを取得する

ファセットごとに親ドキュメント単位でのユニークな件数を知りたい場合は `uniqueBlock(_root_)` を使います：

```json
"facet": {
  "tags": {
    "type": "terms",
    "field": "tags.name",
    "domain": {
      "blockChildren": "type_s:article"
    },
    "facet": {
      "unique_articles": "uniqueBlock(_root_)"
    }
  }
}
```

この設定で、各タグがいくつの親（記事）に登場したかをユニークにカウントできます。

---

## まとめ

- `blockChildren` は親 → 子の変換、`blockParent` は子 → 親の変換
- `domain.filter` を使えば、ファセット側にも親の検索条件を反映できる
- `uniqueBlock()` を使うことで、子ドキュメントの重複による過剰カウントを避けられる

Solr の JSON Facet API は柔軟ですが、その分構造の理解が必要です。特にブロックジョインを使う場合、正確なドキュメントセットの把握と条件の伝播制御が重要です。

---

## 参考リンク

- [Apache Solr GitHub](https://github.com/apache/solr)
- [TestJsonFacets.java](https://github.com/apache/solr/blob/6f7b8b08/solr/core/src/test/org/apache/solr/search/facet/TestJsonFacets.java)
- [FacetRequest.java](https://github.com/apache/solr/blob/6f7b8b08/solr/core/src/java/org/apache/solr/search/facet/FacetRequest.java)

---

Solr のファセット実装に悩む方の助けになれば幸いです。
