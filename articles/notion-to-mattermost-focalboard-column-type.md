---
title: "Notion to Mattermost focalboard - インポート時のフィールドタイプを動的に切り替える実装"
emoji: "📝"
type: "tech"
topics: ["mattermost", "notion", "import", "board"]
published: true
---

CSVファイルからデータをインポートする際、すべてのカラムを同じフィールドタイプ（例：テキストフィールドやセレクトボックス）として扱うのではなく、カラムによって異なるフィールドタイプを設定したいケースがあります。

この記事では、特定のカラムのみをセレクトボックスとして扱い、それ以外のカラムをテキストフィールドとして扱う実装について解説します。

## 前提条件
具体的な Notion から Mattermost focalboard については[公式のドキュメント](https://github.com/mattermost-community/focalboard/blob/main/docs/import-export-backup-data.md#import-from-notion)をご覧ください。<br>
その中で実行する [importNotion.ts](https://github.com/mattermost/focalboard/blob/de5e5cc4141f14f69bf0e5666383997b81f851d6/import/notion/importNotion.ts#L140-L143) を書き換えて、実行します。<br>

:::message
公式より推奨された方法ではなく、筆者が試したらできたという代物なので、実行に関しては自己責任でお願いします。
:::

## 実装の概要

今回の実装では以下の要件を満たすようにします：

- category, status カラムはセレクトボックスとして扱う
- その他のカラムはテキストフィールドとして扱う

## 全体の実装
下記のGistに全体の実装を記載しています：<br>
https://gist.github.com/kuuumo/dcd09ddf82b31988272a8defde449ae2

## 実装の変更点の説明
### セレクトボックスとして扱うカラムの定義

まず、セレクトボックスとして扱うカラムを定義します：

```typescript
const SELECT_COLUMNS = ['category', 'status']
```

### プロパティタイプの設定

カラムに応じて適切なプロパティタイプを設定します：

```typescript
const cardProperty: IPropertyTemplate = {
    id: Utils.createGuid(),
    name: column, 
    // 変更前
    // type: 'select',
    type: SELECT_COLUMNS.includes(column) ? 'select' : 'text',
    options: []
}
```

### カードのプロパティ値設定

カードのプロパティ値を設定する際、カラムのタイプに応じて異なる処理を行います：

```typescript
// 変更前
// outCard.fields.properties[cardProperty.id] = option.id
if (SELECT_COLUMNS.includes(key)) {
    // セレクトボックスの場合
    outCard.fields.properties[cardProperty.id] = option.id
} else {
    // テキストフィールドの場合
    outCard.fields.properties[cardProperty.id] = value
}
```

## まとめ

CSVインポート時のフィールドタイプを動的に切り替えることで、より柔軟なデータインポートが可能になりました。特定のカラムのみをセレクトボックスとして扱い、残りをテキストフィールドとして扱うことで、データの性質に応じた適切な入力形式を提供できます。

また、この実装は環境変数での設定や自動判定機能の追加など、さらなる拡張の余地も残されています。

## 参考
https://github.com/mattermost-community/focalboard/blob/main/docs/import-export-backup-data.md#import-from-notion
