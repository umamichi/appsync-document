## AppSync&GraphQL入門


## AppSyncとは？

GraphQL というAPI仕様を用いて「柔軟なAPI」を提供するAWSのマネジメントサービス。

対して、従来の REST API だと AWSは API Gateway を提供しています。

## GraphQLとは？

Facebookが開発しているWeb APIのための規格。「クエリ言語」と「スキーマ言語」からなる。

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

従来のAPI Gatewayだと、AWS Lambda が間に必要でしたが、

AppSyncは **Lambda レス** でDynamoDBへのアクセスが可能です。

代わりに、AppSync内のリゾルバーという領域にロジックを記述します。

## AppSyncの料金

使用した分だけ課金されます

### クエリとデータ変更操作

4.00USD ≒ 423.87 円 / クエリおよびデータ変更操作 100 万回あたり

### リアルタイム更新

データが更新された際に、リアルタイムに更新する機能

2.00USD ≒ 211.94 円 / リアルタイムアップデート 100 万回

最初の12ヶ月の無料利用枠の対象でもあるようなので、登録後12ヶ月は一定回数は無料で使用できます。

[料金の詳細はこちら](https://aws.amazon.com/jp/appsync/pricing/)

## 試してみる

実際に、AppSyncを用いてイベントを取得・登録する処理を実装してみます、とても簡単です

### AppSync API を作成

AWS ログインして、AppSyncページへ移動し、「APIを作成」

サンプルプロジェクトから「イベントアプリ」を選んで「開始」

API 名は `[yourname] App` としてください。

左メニューから「クエリ」ページに移動すると、GraphQL Explorer が表示されます

ここで、GraphQLを試すことが可能です

![image](https://user-images.githubusercontent.com/7469495/94671324-b42b4880-034e-11eb-8cbf-d5b6b5870770.png)

▶︎ボタンから、実行したいクエリを選択してみると、右側に結果が表示されます

￼![image](https://user-images.githubusercontent.com/7469495/94671244-9b229780-034e-11eb-941a-3d4ee808f1c4.png)



デフォルトで2つのクエリが用意されています。

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

1つめの `mutation CreateEvent` は、新たな Event のデータを作成するための mutation です

2つめの `query ListEvents` は、DBに登録されている Event のデータを取得するための query です


`CreateEvent` を何度か実行すると `ListEvents` の結果が変わることがわかります


### GraphQLには3種類のクエリがある

| 名前 | 説明 |
| -- | -- |
| query | データ取得 （read） |
| mutation | データ作成/更新/削除 （create / update / delete） |
| subscription | リアルタイムイベントを受け取れる。内部的にはwebsocketが使われている |


先ほどのサンプルでは query と mutation を使用しています

<!-- ### フロントエンド環境からAppSyncに接続する

左メニューから、アプリ名をクリックすると、アプリと統合するための手順が表示されます

こちらに従って進めてみましょう -->
 

### フロントエンドからGraphQLを使ってリクエストしてみる

Axios を使って試してみます

[こちらにサンプルコードを用意しました](https://codepen.io/umamichi/pen/wvGqGvj)

URL と API KEY を AppSyncコンソールの設定ページから見つけて、セットしてみてください

上手くいけば、GraphQLのresultが、consoleに表示されます


```javascript
  const data = await axios.post(
    API_URL,
    {
      query: `
      // ここにqueryをかく    
      `
    },
    {
      headers: {
        // header に APIキーを渡す。 appSync設定画面から取得
        "x-api-key": ""
      }
    }
  ); 
```

`request body` にクエリを記述、`request header` に `x-api-key` として API KEY を持たせることで認証されます、とても簡単ですね


### 認証方法

appSyncでは、4つの認証方法が用意されています

| 名前 | 概要 | ユースケース |
| -- | -- | -- |
| API_KEY | 今回使ったもの。最大 365 日間有効に設定可能で、該当日からさらに最大 365 日、既存の有効期限を延長可 | パブリック API の公開が安全であるユースケース、または開発目的での使用が推奨 |
| AWS_IAM | IAMポリシーを紐づけて使用 | IAMロールごとに、特定の機能のみに制限したい場合 |
| OPENID_CONNECT | OpenID Connect (OIDC) トークンを適用 | OpenID Connectを使いたい場合（未調査） | 
| AMAZON_COGNITO_USER_POOLS | Amazon Cognito ユーザープールによって提供される OIDC トークンが使用されます | Amazon Cognito ユーザープールによって提供されるOIDCトークンを使いたい場合（未調査） |


基本的には、上2つ `API_KEY` と `AWS_IAM` を使うパターンが多いでしょう


[認証についての詳細（公式ページ）](https://docs.aws.amazon.com/ja_jp/appsync/latest/devguide/security.html)

### データベースの中身を見てみる

左メニューから、 `データソース` を選択すると、`DynamoDB` へのリンクがあります

`AppSync` が自動生成してくれたテーブルが、ここに表示されています

![image](https://user-images.githubusercontent.com/7469495/94674333-fb1b3d00-0352-11eb-9b56-5052469777db.png)



### APIを追加してみる





------

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