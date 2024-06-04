---
title: "[Rails] AlbaとMobilityを利用してモデルの属性を多言語対応でシリアライズする"
emoji: "🌎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Rails, Alba, Mobility, i18n, Serialize]
published: false
---

## はじめに

この記事では、Mobilityで多言語対応したフィールドをAlbaで簡単にJSONシリアライズする方法について紹介します。

この方法を使えば、言語ごとにフィールドを指定することなく、全ての言語に対応したフィールドを一度にシリアライズできます。

## 実装の背景

フロントエンドをReactで実装している際に、翻訳において下記の問題が発生しました。
それらを解決するために全ての翻訳をAPIで一度に返す方法を採用しました。
1. リソースの翻訳を集約:
    - 各リソースの翻訳をバックエンドに集約したいと考えました。しかし、この方法では、フロントエンドで言語を切り替えるたびに、バックエンドから翻訳済みリソースを再取得する必要がありました。
2. 翻訳の反映の遅延:
    - 上記の問題を解決しようとすると、フロントエンドで言語を切り替えた際に、リソースの翻訳が即座に反映されず、ユーザー体験が損なわれる可能性がありました。
    - また翻訳の反映のために無駄なリクエストや、記述が増えてしまいます。

これを解決するため、全ての翻訳を一度にAPIで返す方法を採用しました。これにより、フロントエンドで言語を切り替えてもAPIの追加リクエストなしでリソースの多言語対応が可能になります。

なお、大量のテキストデータを扱う場合はパフォーマンスの問題が発生する可能性があるため、状況に応じて別の対応を検討してください。

## 出力イメージ

```ruby
post = Post.create(title_ja: "記事を書いてみた", title_en: "I wrote an article", ...)
PostResource.new(post).serializable_hash
=> {
  "id" => 1,
  ...
  "title_ja" => "記事を書いてみた",
  "title_en" => "I wrote an article",
  ...
}
```

## 実際のコード

まず、titleとdescriptionをMobilityで国際化対応しているPostモデルを定義します。

```ruby:app/models/post.rb
class Post < ApplicationRecord
  include Mobility
  translates :title, :sub_title
  ...
end
```

次に、AlbaのresourceのベースクラスApplicationResourceを作成し、ここに`i18n_attributes`メソッドを追加します。このメソッドでは、以下のことを行います：
- 引数として国際化対応する属性の文字列またはシンボルを複数受け取る
- `config/application.rb`で設定されている使用可能な言語に応じてシリアライズする属性を指定する

```ruby:app/resources/application_resource.rb
class ApplicationResource
  include Alba::Resource
  ...
  # Set multiple i18n attributes at once
  #
  # @param attrs [Array<String, Symbol>]
  def self.i18n_attributes(*attrs)
    attrs.each do |attr|
      I18n.available_locales.each do |locale|
        attribute Mobility.normalize_locale_accessor(attr, locale)
      end
    end
  end
  ...
end
```

最後に、PostResourceを作成し、先ほどの`i18n_attributes`を利用します。

```ruby:app/resources/post_resource.rb
class PostResource < ApplicationResource
  attributes :id, ...

  i18n_attributes :title, :sub_title
end
```

## 使用したGem

### Alba

[Alba](https://github.com/okuramasafumi/alba)は、Ruby、JRuby、TruffleRuby用のJSONシリアライザーです。  
（Readmeより引用 & deepl翻訳）

### Mobility

[Mobility](https://github.com/shioyama/mobility?tab=readme-ov-file#mobility)は、翻訳をクラスの属性として保存・取得するためのgemです。これらの翻訳は、ブログ記事の内容、画像のキャプション、ブックマークのタグなど、異なる言語で保存したいものであれば何でも可能です。（Readmeより引用 & deepl翻訳）

## 最後に

誤りや改善点、アドバイスなどがあれば、ぜひコメントで教えてください！
