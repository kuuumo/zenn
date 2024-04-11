---
title: "【Rails】Config Gem で細かい環境ごとに設定を変更できるようになっていた"
emoji: "🌟"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Rails", "config", "Ruby"]
published: true
---

## Rails Config Gem とは
環境固有の定数を管理するための便利なgemです。
https://github.com/rubyconfig/config

本題から逸れるので導入などは他の記事を参照いただけると幸いです。

## 本題
下記のように設定するだけで、開発/ステージング/本番/テスト/ローカル それぞれの環境で別のSettingsを参照することができます。

今までも initializer に数行書けば同じようなことが実現できましたが、その必要がなくなりました。

```ruby:config/initializers/config.rb
config.environment = ENV.fetch('ENVIRONMENT', :development)
```

設定ファイルは下記のように `config/settings/hogehoge` と `ENVIRONMENT` に合わせたファイル名にして、各環境の定数を記述していくだけです
```text
config/settings
├── staging.yml
├── local.yml
├── production.yml
└── test.yml
```

変更PR: https://github.com/rubyconfig/config/pull/356
リリースバージョン: 5.4.0

## どんな時に嬉しいの
ステージング環境と本番環境を Rails.env = production で運用しているが、それぞれの環境で別のAPIを呼びたい時にわざわざ AWS Parameter Store などを使わずに、コードベースで環境変数を管理できる

:::message
・上述の通り、別途環境変数 `ENVIRONMENT` (環境変数名は任意) を環境ごとに変更する必要あります。
・また、秘匿情報は環境変数などを適切に利用し、git上に残さないよう気をつけてください
:::

## 終わりに
便利なのでみなさんもぜひ使ってみてください
