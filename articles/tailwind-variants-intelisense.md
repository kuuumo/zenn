---
title: "tailwind-variantsでサジェストを有効にする"
emoji: "💨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["tailwind", "ide", "tailwind-variants", "intellij", "vscode", "neovim"]
published: true
---

[本家のDocs](https://www.tailwind-variants.org/docs/getting-started#intellisense-setup-optional)に記載のあることですが備忘録的に書き記しておきます。

## やり方

お使いのエディタごとに正しい箇所に設定してください。（vscode, neovim, intellij がドキュメントにて紹介されています。）

僕の場合は Intellij を利用しているので`settings -> Languages & Frameworks -> Style Sheets -> Tailwind CSS` で下記の設定を加えます。

```json:settings -> Languages & Frameworks -> Style Sheets -> Tailwind CSS
"experimental": {
  "configFile": null,
  "classRegex": [
    ["tv\\((([^()]*|\\([^()]*\\))*)\\)", "[\"'`]([^\"'`]*).*?[\"'`]"]
  ]
}
```

## 感想
ドキュメントにはたくさん有用な情報が記載されているので、これからもちゃんと参照しよう！
