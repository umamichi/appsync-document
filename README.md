## AppSync & GraphQL 入門

![AppSync](https://user-images.githubusercontent.com/7469495/105184322-9179a580-5b72-11eb-8f73-5bbe20a686ec.png)

## AppSync とは？

GraphQL というAPI仕様を用いて「柔軟なAPI」を提供するAWSのマネジメントサービス

ちなみに、従来の REST API 形式だと AWSは API Gateway を提供している

## GraphQL とは？

Facebookが開発しているWeb APIのための規格

「クエリ言語」 と 「スキーマ言語」 からなる

REST API は、1URLに対し1つのAPIや情報を提供できるのに対し、

GraphQL は欲しいデータを以下のようなクエリとして発行すると、**欲しいデータを欲しいObject形式で**得ることができます

```javascript
// リクエスト
query GetCurrentUser {
  currentUser {
    id
    name
  }
}

```

↓

```javascript
// レスポンス
{
  id: 'hoge',
  name: 'yamada'
}
```

## AppSyncの仕組み

![AppSyncImage-1024x542](https://user-images.githubusercontent.com/7469495/94530035-773f5300-0275-11eb-8e38-8d1435ec7b44.png)

AppSyncは直接DynamoDBの値を取得・更新・削除することができます

従来のAPI Gatewayだと、AWS Lambda が間に必要でしたが、

AppSyncは **Lambda レス** でDynamoDBへのアクセスが可能です

代わりに、AppSync内のリゾルバーという領域にロジックを記述します

### クエリ

実行されるGraphQlのこと

### スキーマ

どの型の値をどこで使うかを定義する設計書

### リゾルバー

関数のこと。ロジックを記述する。

リゾルバーは、`リクエストマッピングテンプレート` と `レスポンスマッピングテンプレート` で構成さる

`リクエストマッピングテンプレート` は、「変換」と「実行」のロジックが含まれている

### リソース

データベースのこと。AppSync では `AWS DynamoDB` に自動的に接続される

## AppSyncの料金

使用した分だけ課金されます

### クエリとデータ変更操作

4.00USD ≒ 423.87 円 / クエリおよびデータ変更操作 100 万回あたり

### リアルタイム更新

データが更新された際に、リアルタイムに更新する機能

2.00USD ≒ 211.94 円 / リアルタイムアップデート 100 万回

最初の12ヶ月の無料利用枠の対象でもあるようなので、登録後12ヶ月は一定回数は無料で使用できます

[料金の詳細はこちら](https://aws.amazon.com/jp/appsync/pricing/)

## 試してみる

実際に、AppSyncを用いてイベントを取得・登録する処理を実装してみます、とても簡単です

### AppSync API を作成

AWS ログインして、AppSyncページへ移動し、「APIを作成」

サンプルプロジェクトから「イベントアプリ」を選んで「開始」

API 名は `[yourname] App` としてください

左メニューから「クエリ」ページに移動すると、GraphQL Explorer が表示されます

ここで、GraphQLを試すことが可能です

![image](https://user-images.githubusercontent.com/7469495/94671324-b42b4880-034e-11eb-8cbf-d5b6b5870770.png)

▶︎ボタンから、実行したいクエリを選択してみると、右側に結果が表示されます

￼![image](https://user-images.githubusercontent.com/7469495/94671244-9b229780-034e-11eb-941a-3d4ee808f1c4.png)


デフォルトで2つのクエリが用意されています

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
 

### 実際に Javascript で GraphQL を使ってみる

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



### APIを変更・追加してみる


#### createEvent に `who` という項目を追加してみる

AWSコンソールの左メニューから `スキーマ` を選択すると、定義されている Schema が表示されます

この中から、 `Mutation` の下にある `createEvent` を見つけ、引数に `who` を追加してみます

![スクリーンショット 2020-10-28 19 28 00](https://user-images.githubusercontent.com/7469495/97424426-0e222c80-1954-11eb-99c2-76ea52ab1f5d.png)

右上から　`スキーマを保存` します

`クエリ` ページから、who に適当な値を追加して、実行してみます

![image](https://user-images.githubusercontent.com/7469495/97554741-8b17d980-1a1a-11eb-883b-df4e39437046.png)


これで、dynamoDBに `who` が追加されたか確認してみましょう

左メニューから `データソース` -> `AppSyncEventTable` のリソースを開きます

あれ、項目 `who` が追加されていると思いましたが、追加されていません😢

![image](https://user-images.githubusercontent.com/7469495/97555381-6708c800-1a1b-11eb-8ecc-917b5e80cb44.png)

理由は簡単です、 `リゾルバー` も変更する必要があります💡

`リゾルバー` とは、このページの冒頭で表示した図にあるように、ロジックを記述する領域です

`リゾルバー` の変更は、 `スキーマ` ページの右カラムから可能です

`createEvent` を見つけましょう⤵︎

![image](https://user-images.githubusercontent.com/7469495/97555776-e4343d00-1a1b-11eb-955c-1aa84a2b914f.png)

`リクエストマッピングテンプレート` を以下のようにして、 `who` を追記します


```json
{
    "version": "2017-02-28",
    "operation": "PutItem",
    "key": {
      "id": { "S": "$util.autoId()"}
    },
    "attributeValues": {
      "name": { "S": "$context.arguments.name" },
      "where": { "S": "$context.arguments.where" },
      "when": { "S": "$context.arguments.when" },
      "who": { "S": "$context.arguments.who" },
      "description": { "S": "$context.arguments.description" }
    }
}
```

リゾルバーを保存して、実行すると、`who` 項目が追加されていることが確認できました🎉

![image](https://user-images.githubusercontent.com/7469495/97556176-6fadce00-1a1c-11eb-95ee-f465e07f5ef8.png)


今回は内部の挙動を理解するために、

ブラウザからAWSコンソールを通じてスキーマやリゾルバーの変更を行いましたが、

実際には `AWS Cloudformation` や `Amplify Framework` などを用いると良いそうです

## まとめ

### メリット

+ GraphQL は REST API に**比べて欲しいデータを欲しい形式で**得ることが可能
  
+ GraphQL により、画面や機能ごとに、**個別にAPIを定義するコストが削減**される

+ AppSync を使えばリソース（DynamoDB）との連携を楽に行うことができる
  
+ 既存の REST API を AppSync でラップして、GraphQL を導入することも可能らしい（未調査）


### デメリット

+ GraphQL, Appsync の学習コストがかかる

+ フロントエンドの都合の良いように、値を返す必要があるため、リゾルバーのロジックが複雑になる

+ 効率的にデータを処理できないので、パフォーマンスが低下し、N+1問題が発生する

※ N+1問題・・・ループ処理の中で都度SQLを発行してしまい、大量のSQLが発行されてパフォーマンスが低下してしまう問題のこと


## 実際にAppSyncで実装してみたページ

dynamoDB に入っているニュースのデータを AppSync を使って表示しています

https://umamichi.com/news/


## これから調べる

`AWS_IAM` を使った AppSync 認証方法。 `Cognito` を使うらしいです

## 参考

https://docs.aws.amazon.com/ja_jp/appsync/latest/devguide/welcome.html

https://xp-cloud.jp/blog/2020/06/01/7159/

