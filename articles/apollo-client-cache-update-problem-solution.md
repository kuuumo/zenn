---
title: "Apollo Clientのキャッシュで新規作成したデータの更新がUIに反映されない問題の解決法"
emoji: "🐛"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["GraphQL", "Apollo Client", "Cache", "React"]
published: true
---

GraphQLとApollo Clientを使ったフロントエンド開発で、新規作成したリソースの編集機能を実装していた時の話です。

「新しく作成したTodoのタイトルを編集したのに、UIに反映されない...」
「ページをリロードしたら正しく表示される...」
「既存のTodoは編集しても正常に反映されるのに、新規作成したものだけおかしい...」

こんな問題に遭遇して、結構ハマりました。原因を調べてみると、Apollo Clientのキャッシュ管理の理解が足りていなかったことが判明。

今回は、よくあるTodoアプリケーションを例に、この問題の原因と解決策、そしてApollo Clientのキャッシュ管理のベストプラクティスについて詳しく解説していきます。

## 問題の概要

### 発生した問題
- 新規作成したTodoのタイトルを編集しても、UIに反映されない
- ページをリロードすると正しく表示される
- 既存のTodoは編集しても正常に反映される
- 作成直後にApollo Clientのキャッシュに追加する実装を行っている

### 技術スタック
- React + TypeScript
- Apollo Client
- GraphQL

## 問題の原因分析

### 1. キャッシュ構造の不統一

問題は新規作成時のキャッシュ追加方法にありました。

新規作成時に手動でキャッシュに追加していたのですが、その際にApollo Clientがデフォルトで行う`__ref`形式（参照形式）ではなく、Todoのデータを生で直接追加していました。一方、更新時はApollo Clientの自動更新に任せていたため、正規化された`__ref`形式で処理されていました。

この不統一がキャッシュの一貫性を壊していました。

### 2. 正規化キャッシュの理解不足

#### 正規化キャッシュの構造

下記はわかりやすくキャッシュの構造を表現したものです。

```json
{
  "ROOT_QUERY": {
    "todos": [
      // データへの参照が入っている
      {
        "__ref": "Todo:1"
      },
      // 生データが入ってしまっている  ←  ⚠️ これが問題！！！
      {
        "id": "2",
        "title": "Buy groceries",
        "completed": false
      }
    ]
  },
  "Todo:1": {
    "id": "1",
    "title": "Buy groceries",
    "completed": false
  }
}
```

データそのものとデータの配列が別々に管理されており、配列にはデータそのものへの参照が入っているという構造になっています。

※ apollo client の devtools でキャッシュの中身を見ることができます。

Apollo Clientは正規化キャッシュを使っていて、エンティティは`__ref`形式で参照されます。

新規作成時にオブジェクト形式で直接追加すると、更新時にApollo Clientが自動更新しようとしても、正規化された参照を見つけることができず、UIに反映されないという問題が発生します。


## 解決策の実装

問題を解決するために、Apollo Clientのキャッシュ更新についてガイドラインを作成しました。
作成/更新/削除の各操作で ref 形式でキャッシュを管理することを意識しています。

### 作成（Create）パターン

作成時は、`cache.modify()`と`toReference()`を使って参照形式（`__ref`）でキャッシュに追加します。
これにより、更新時にキャッシュが正しく反映されるようになります。

**重要なポイント：**
- `cache.modify()`と`toReference()`を使って参照形式（`__ref`）で追加する
- Fragmentで list 表示する時に必要な全フィールド（id、title、completedなど）を含める
  - GraphQLの良さを殺している気もするので、いい方法があれば教えてください 🙇‍♂️

### 更新（Update）パターン

更新時は、ミューテーションの返り値に`id`と更新されたフィールド（titleなど）を含めて、Apollo Clientの自動更新に任せます。

**重要なポイント：**
- ミューテーションの返り値に`id`と更新されたフィールドを含める
- 手動キャッシュ更新は行わず、Apollo Clientの自動更新に任せる

### 削除（Delete）パターン

削除時は、`cache.evict()`でエンティティを削除し、`cache.gc()`でガベージコレクションを実行します。

## 最後に

Apollo Clientのキャッシュ管理は、正規化キャッシュの理解が重要です。特に、作成・更新・削除の各操作で一貫したパターンを使い、参照形式（`__ref`）を適切に活用することで、UIの一貫性を保つことができます。

本来はこの制御を型やESLintなどで強制したいところですが、既存のものは見当たらず一旦ガイドラインを作成して、Cursor Rules でガイドラインを適用しています。

同じ問題で困っている方がいたら、この記事が参考になれば嬉しいです！

もし認識が間違っている点や、より良い方法があれば、ご指摘いただけると嬉しいです 🙇‍♂️

## 参考資料

- [Apollo Client Cache Field Policies](https://www.apollographql.com/docs/react/caching/cache-field-behavior/)
- [Apollo Client Cache Updates](https://www.apollographql.com/docs/react/caching/cache-interaction/)
- [Apollo Client Normalized Cache](https://www.apollographql.com/docs/react/caching/overview/#normalized-cache) 
