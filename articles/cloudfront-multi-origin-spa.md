---
title: "[AWS CloudFront] 複数のSPAアプリケーションをマルチオリジンで配信する際の注意点"
emoji: "🎏"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [AWS, CloudFront, S3, SPA, Lambda@edge]
published: true
---

## はじめに

AWSのS3とCloudFrontを使用して複数のシングルページアプリケーション（SPA）を配信する際、効率的な構成とキャッシュの適切な管理が重要です。本記事では、複数SPAの配信における問題点とその解決策、さらにLambda@Edgeを使用した配信方法について解説します。

## 基本的な構成と問題点

### 構成概要

1. 複数のS3バケット（例：`bucket-1`、`bucket-2`）にSPAをホスティング
2. 1つのCloudFrontディストリビューションで複数のバケットを配信
3. CloudFrontのビヘイビア設定で各バケットへのルーティングを定義
4. CloudFront Functionを使用して、拡張子なしのリクエストを`/index.html`に変更

### 発生しうる問題

この基本的な構成では、以下のような問題が発生する可能性があります：

1. 異なるパス（例：`/bucket-1/page`と`/page`）へのアクセスが同じ`index.html`を返す
2. キャッシュの不適切な管理による意図しないコンテンツの配信

## 問題の原因：キャッシュキーの理解

CloudFrontのキャッシュキーは通常、以下の要素で構成されます：

- リクエストのホスト名
- リクエストのパス
- クエリ文字列（設定による）
- ヘッダー（設定による）

CloudFront Functionで全てのリクエストを`/index.html`に変更すると、異なるオリジンへのリクエストでも同じキャッシュキーが生成され、意図しないコンテンツが返される可能性があります。

## 解決策：Lambda@Edgeの活用

### Lambda@Edge関数の実装例

```javascript
exports.handler = async (event) => {
    const { request } = event.Records[0].cf;
    const uri = request.uri;

    if (!uri.includes('.')) {
        request.uri = '/index.html';
    } else if (uri.startsWith('/bucket-1')) {
        request.uri = request.uri.replace(/^\/bucket-1/, '');
    }

    return request;
};
```

この実装により、拡張子なしのリクエストの適切な処理と、特定のパスの正確なルーティングが可能になります。

## まとめ

1. CloudFrontで複数のSPAを配信する際は、キャッシュの挙動に注意が必要
2. Lambda@Edgeを活用することで、シンプルなケースにおけるルーティングとキャッシュ問題を解決可能

## 参考資料

- [Lambda@Edgeの使用（AWS公式ドキュメント）](https://docs.aws.amazon.com/ja_jp/AmazonCloudFront/latest/DeveloperGuide/lambda-at-the-edge.html)
- [CloudFrontでのキャッシュの動作を制御する（AWS公式ドキュメント）](https://docs.aws.amazon.com/ja_jp/AmazonCloudFront/latest/DeveloperGuide/controlling-the-cache-key.html)
- [Deploying Multiple SPAs using AWS CloudFront and S3 with Lambda@Edge as Pseudo Reverse Proxy（英語）](https://medium.com/@yatharthagoenka/deploying-multiple-spas-using-aws-cloudfront-and-s3-with-lamda-edge-as-pseudo-reverse-proxy-8f6314531885)
