## AppSync&GraphQL入門


## AppSyncとは？

GraphQLというAPI仕様を用いて「柔軟なAPI」を提供するマネジメントサービス。AWSが提供している

対して、従来の REST API だと AWSは API Gateway を提供しています。

## GraphQLとは？

Facebookが開発しているWeb APIのための規格で、「クエリ言語」と「スキーマ言語」からなる。

REST API は、1URLに対し1つのAPIや情報を提供できるのに対し、

GraphQL は欲しいデータを以下のようなクエリとして発行すると、**欲しいデータを欲しいObject形式で**得ることができます

```
query GetCurrentUser {
  currentUser {
    id
    name
  }
}

```

## AppSyncの仕組み

![AppSyncImage-1024x542](https://user-images.githubusercontent.com/7469495/94530035-773f5300-0275-11eb-8e38-8d1435ec7b44.png)


AppSyncは直接DynamoDBの値を取得・更新・削除することができます。

従来のAPI Gatewayだと、AWS Lambda を書く必要がありましたが、AppSyncはLambda レスでDynamoDBへのアクセスが可能です。

代わりに、 AppSync内のリゾルバーという領域にロジックを記述します。


## AppSyncの料金

他AWSサービスと同様、使用した分だけ課金されます。

### クエリとデータ変更操作

4.00USD ≒ 423.87 円 / クエリおよびデータ変更操作 100 万回あたり

### リアルタイム更新

データが更新された際に、リアルタイムに更新する機能

2.00USD  ≒ 211.94 円 / リアルタイムアップデート 100 万回

最初の12ヶ月の無料利用枠の対象でもあるようなので、登録後12ヶ月は一定回数は無料で使用できます。

詳しくはこちら
https://aws.amazon.com/jp/appsync/pricing/


## 試してみる

実際に、AppSyncを用いてイベントを取得・登録する処理を実装してみます、とても簡単です。

### AppSync API を作成

AWS ログインして、AppSyncページへ移動し、「APIを作成」

サンプルプロジェクトから「イベントアプリ」を選んで「開始」

API 名は `[yourname] App` としてください。

左メニューから「クエリ」ページに移動すると、GraphQL Explorer が表示されます

ここで、GraphQLを試すことが可能です

￼

▶︎ボタンから、実行したいクエリを選択してみると、右側に結果が表示されます

￼


デフォルトで2つのクエリが書かれています

```
mutation CreateEvent {
  createEvent(
    name: "My First Event"
    when: "Today"
    where: "My House"
    description: "Very first event"
  ) {
    id
    name
  }
}
  
query ListEvents {
  listEvents {
    items {
      id
      name
    }
  }
}
```

2つめの `query ListEvents` は、データを取得するためのクエリです

1つめの `mutation CreateEvent` は、データを更新するためのクエリです

`CreateEvent` を何度か実行すると `ListEvents` の結果が変わることがわかります。


### GraphQLには3種類のクエリがある
+ query … データ取得 read+ mutation … データ作成/更新/削除 create/update/delete+ subscription … リアルタイムイベントを受け取れる。データの更新時にその情報を受け取るための、存続期間の長い接続。websocketのようなもの？

サンプルでは query, mutation を使用しています


### フロントエンド環境からAppSyncに接続する

左メニューから、アプリ名をクリックすると、アプリと統合するための手順が表示されます

こちらに従って進めてみましょう
 
￼


Node.js 14系（14.4.0）で試してみます

作業用ディレクトリをつくる

```sh
$ mkdir appSyncSample && appSyncSample
```

次に、AWS Amplify CLI をインストールします

Amplify も AWSのサービスで Hosting server として使ったことがある方がいるかもしれません

どうやら Amplify と AppSync と合わせて使うと、とても簡単にWebサービスが作れるようです

まずは気にせず、進めてみます

```
$ npm install -g @aws-amplify/cli
…
…
省略
….
----------------------------------------
Successfully installed the Amplify CLI
----------------------------------------
```

↑ Successfully と出れば成功

ここで、ターミナルを再起動しておきましょう

```
$ amplify init
```

質問事項が出るので答えながら進めてみましょう

途中で IAMユーザーを作成して、アクセスキーをコピペする箇所が出てくるので注意しましょう

```
$ amplify add codegen --apiId [your API key]
$ amplify codegen
```

これで、graphQLの必要なコードが自動生成されました

あとはここに好きなフレームワークで実装をしていきます

or POSTMAN or axios で実装するとか


------------


PostManで試せる
POST、bodyにqueryを持たせる。Paramsではない

headerに、appsync の api key を持たせるとリクエストできる


------------

HTTPリクエストの仕様

hederって何？　ヘッダー。リクエストの詳細情報があります。
paramsって何？　GETのqueryのこと。PostManで、POST時はqueryとなる挙動をするが実際に使うことはない
bodyって何？　GETとDELETE以外に存在する。POSTで値を渡すときに使う。
graphQLは中身はPOST

## 参考

https://xp-cloud.jp/blog/2020/06/01/7159/