[![FIWARE Banner](https://fiware.github.io/tutorials.Accessing-Context/img/fiware.png)](https://www.fiware.org/developers)

[![FIWARE Core Context Management](https://nexus.lab.fiware.org/repository/raw/public/badges/chapters/core.svg)](https://www.fiware.org/developers/catalogue/)
[![License: MIT](https://img.shields.io/github/license/fiware/tutorials.Accessing-Context.svg)](https://opensource.org/licenses/MIT)
[![Support badge](https://nexus.lab.fiware.org/repository/raw/public/badges/stackoverflow/fiware.svg)](https://stackoverflow.com/questions/tagged/fiware)
[![NGSI v2](https://img.shields.io/badge/NGSI-v2-blue.svg)](https://fiware-ges.github.io/core.Orion/api/v2/stable/)
<br/>
[![Documentation](https://img.shields.io/readthedocs/fiware-tutorials.svg)](https://fiware-tutorials.rtfd.io)

このチュートリアルでは、FIWARE ユーザにプログラムでコンテキストを変更する方法に
ついて説明しています。チュートリアルでは、以前
の[在庫管理の例](https://github.com/Fiware/tutorials.Context-Providers/)で作成さ
れたエンティティをもとにして、 コンテキスト・データを取得および変更するために
、[NGSI](https://fiware.github.io/specifications/OpenAPI/ngsiv2) 対応 の
[Node.js](https://nodejs.org/) [Express](https://expressjs.com/) アプリケーショ
ンでコードを記述する方法を理解できます。これにより、コマンドラインを使用して
cUrl コマンドを呼び出す必要がなくなります。

このチュートリアルでは、主に Node.js で記述されたコードについて説明しますが、結
果の一部は [cUrl](https://ec.haxx.se/) コマンドを使用して確認できます。同じコマ
ンドの
[Postman マニュアル](https://fiware.github.io/tutorials.Accessing-Context/)も利
用できます。

[![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/fb5f564d9bc65fc3690e)

# 内容

-   [コンテキスト・データへのアクセス](#accessing-the-context-data)
    -   [任意の言語で HTTP リクエストを作成](#making-http-requests-in-the-language-of-your-choice)
    -   [NGSI API クライアントを生成](#generating-ngsi-api-clients)
    -   [このチュートリアルの目標](#the-teaching-goal-of-this-tutorial)
    -   [在庫管理システム内のエンティティ](#entities-within-a-stock-management-system)
-   [アーキテクチャ](#architecture)
-   [前提条件](#prerequisites)
    -   [Docker](#docker)
    -   [Cygwin](#cygwin)
-   [起動](#start-up)
-   [在庫管理フロントエンド](#stock-management-frontend)
    -   [NGSI v2 npm ライブラリ](#ngsi-v2-npm-library)
    -   [コードの分析](#analysing-the-code)
        -   [ライブラリの初期化](#initializing-the-library)
        -   [ストアのデータを読む](#reading-store-data)
        -   [製品と在庫アイテムの集約](#aggregating-products-and-inventory-items)
        -   [コンテキストの更新](#updating-context)
-   [次のステップ](#next-steps)

<a name="accessing-the-context-data"></a>

# コンテキスト・データへのアクセス

一般的なスマートなソリューションでは、CRM システム、ソーシャルネットワーク、モバ
イルアプリ、IoT センサーなどの、さまざまなソースからコンテキスト・データを取得し
、適切なビジネス・ロジックを決定するために、コンテキストをプログラムで分析します
。例えば、在庫管理のデモでは、アプリケーションは、各アイテムに支払われた価格が常
に、**Product** エンティティ内に保持されている現在の価格を反映するようにする必要
があります。動的システムでは、アプリケーションは現在のコンテキストを修正できる必
要もあります。例えば、データの作成または更新、またはセンサーのアキュレートです。

一般的には、以下の 3 つの基本シナリオが定義されています :

-   データの読み込み - 例えば、**Store** エンティティ `urn:ngsi-ld:Store:001` の
    すべてのデータを取得
-   集約 - 例えば Store `urn:ngsi-ld:Store:001` の **InventoryItems** エンティテ
    ィを、販売する **Product** エンティティの名前と価格と組み合わせる
-   コンテキストを変更 - 例えば、製品を販売します :
    -   日々の販売記録を **Product** の価格で更新します
    -   **InventoryItem** エンティティの `shelfCount` を減らします
    -   販売が発生したことを示す、新しいトランザクション・ログレコードを作成しま
        す
    -   販売されているオブジェクトが 10 個未満の場合は、倉庫で警告を発します
    -   など

ご覧のように、コンテキストにアクセス/修正する各リクエストの背後にあるビジネス・
ロジックは、ビジネスニーズに応じて単純なものから複雑なものまでさまざまです。

<a name="making-http-requests-in-the-language-of-your-choice"></a>

## 任意の言語で HTTP リクエストを作成

[NGSI](https://fiware.github.io/specifications/OpenAPI/ngsiv2) の仕様では、HTTP
動詞の標準的な使用法に基づいて、言語に依存しない REST API を定義します。したがっ
て、コンテキスト・データは、HTTP リクエストを行うだけで、どのプログラミング言語
でもアクセスできます。

例えば、[PHP](https://secure.php.net/), [Node.js](https://nodejs.org/),
[Java](https://www.oracle.com/java/) で書かれたものと同じ HTTP リクエストです。

#### PHP (with `HTTPRequest`)

```php
<?php

$request = new HttpRequest();
$request->setUrl('http://localhost:1026/v2/entities/urn:ngsi-ld:Store:001');
$request->setMethod(HTTP_METH_GET);

$request->setQueryData(array(
  'options' => 'keyValues'
));

try {
  $response = $request->send();

  echo $response->getBody();
} catch (HttpException $ex) {
  echo $ex;
}
```

#### Node.js (with `request` library)

```javascript
const request = require("request");

const options = {
    method: "GET",
    url: "http://localhost:1026/v2/entities/urn:ngsi-ld:Store:001",
    qs: { options: "keyValues" }
};

request(options, function(error, response, body) {
    if (error) throw new Error(error);
    console.log(body);
});
```

#### Java (with `CloseableHttpClient` library)

```java
CloseableHttpClient httpclient = HttpClients.createDefault();
try {
    HttpGet httpget = new HttpGet("http://localhost:1026/v2/entities/urn:ngsi-ld:Store:001?options=keyValues");

    ResponseHandler<String> responseHandler = new ResponseHandler<String>() {
        @Override
        public String handleResponse(
                final HttpResponse response) throws ClientProtocolException, IOException {
            int status = response.getStatusLine().getStatusCode();
            if (status >= 200 && status < 300) {
                HttpEntity entity = response.getEntity();
                return entity != null ? EntityUtils.toString(entity) : null;
            } else {
                throw new ClientProtocolException("Unexpected response status: " + status);
            }
        }

    };
    String body = httpclient.execute(httpget, responseHandler);
    System.out.println(body);
} finally {
    httpclient.close();
}
```

<a name="generating-ngsi-api-clients"></a>

## Generating NGSI API Clients

上記の例からわかるように、それぞれは独自のプログラミングパラダイムを使用して次の
ことを行います :

-   適格な URL を作成します
-   HTTP GET リクエストを作成します
-   レスポンスを取得します
-   エラー状態をチェックし、必要に応じて例外をスローします
-   さらなる処理のためにリクエストのボディを返します

このような定型コードは頻繁に再利用されるため、通常はライブラリ内に隠されています
。

[`swagger-codegen`](https://github.com/swagger-api/swagger-codegen) ツールは
、[NGSI v2 Swagger Specification](https://fiware.github.io/specifications/OpenAPI/ngsiv2)
から直接様々なプログラミング言語で定型 API クライアント・ライブラリを生成するこ
とができます。現在、`swagger-codegen` は以下の言語のコードを生成します :

-   ActionScript, Ada, Apex, Bash, C#, C++, Clojure, Dart, Elixir, Elm, Eiffel,
    Erlang, Go, Groovy, Haskell, Java, Kotlin, Lua, Node.js, Objective-C, Perl,
    PHP, PowerShell, Python, R, Ruby, Rust, Scala, Swift, Typescript

たとえば、次のコマンドを実行します :

```console
swagger-codegen generate \
  -l javascript \
  -i https://fiware.github.io/specifications/OpenAPI/ngsiv2/ngsiv2-openapi.json
```

現在の仕様から NGSI v2 用のデフォルト ES5 npm パッケージを直接生成します。

追加情報は実行時に見つけることができます

```console
swagger-codegen help generate
```

実行すると特定の言語で使用できるカスタマイズ・スイッチに関する情報が表示されます

```console
swagger-codegen config-help -l <language-name>
```

<a name="the-teaching-goal-of-this-tutorial"></a>

## このチュートリアルの目標

このチュートリアルの狙いは、一般的なデータアクセスのシナリオをカバーする一連の汎
用コードの例を定義し、議論することによって、コンテキスト・データのプログラムによ
るアクセスについて開発者の理解を向上させることです。この目的のために、簡単な
Node.js Express アプリケーションを作成します。

ここでの意図は、Express でアプリケーションを書く方法をユーザに教えることではなく
、実際にはどの言語を選択してもかまいません。ビジネス・ロジックの目標を達成するた
めに、**任意の**サンプル・プログラミング言語を使用して、どのようにしてコンテキス
トを変更するかを示すだけです。

明らかに、プログラミング言語の選択は、あなた自身のビジネス・ニーズに依存します。
下記のコードを読んで、Node.js を適切な独自のプログラミング言語で置き換えてくださ
い。

<a name="entities-within-a-stock-management-system"></a>

## 在庫管理システム内のエンティティ

エンティティ間の関係は、次のように定義されます :

![](https://fiware.github.io/tutorials.Accessing-Context/img/entities.png)

青色で強調表示された項目は、外部のコンテキスト・プロバイダによって提供されます。

**Store**, **Product** および **InventoryItem** エンティティは、デモ・アプリケー
ションのフロントエンドにデータを表示するために使用されます。

<a name="architecture"></a>

# アーキテクチャ

このアプリケーションは
、[Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/) という
1 つの FIWARE コンポーネントのみを使用します。アプリケーションが _"Powered by
FIWARE"_ と認定するには、Orion Context Broker を使用するだけで十分です。

現在、Orion Context Broker はオープンソースの
[MongoDB](https://www.mongodb.com/) 技術を利用して、コンテキスト・データの永続性
を維持しています。外部ソースからコンテキスト・データをリクエストするために、単純
なコンテキスト・プロバイダ NGSI プロキシも追加されています。コンテキストを視覚化
して操作するために、簡単な Express アプリケーションを追加します。

したがって、アーキテクチャは 4 つの要素で構成されます :

-   [NGSI](https://fiware.github.io/specifications/OpenAPI/ngsiv2) を使用してリ
    クエストを受信する Orion Context Broker サーバ
-   バックエンドの [MongoDB](https://www.mongodb.com/) データベース
    -   Orion Context Broker が、データ・エンティティなどのコンテキスト・データ
        情報、サブスクリプション、登録などを保持するために使用します
-   **コンテキスト・プロバイダ NGSI proxy** は次のようになります :
    -   [NGSI](https://fiware.github.io/specifications/OpenAPI/ngsiv2) を使用し
        てリクエストを受信します
    -   独自の API を独自のフォーマットで使用して、公開されているデータソースへ
        のリクエストを行います
    -   [NGSI](https://fiware.github.io/specifications/OpenAPI/ngsiv2) 形式でコ
        ンテキスト・データを Orion Context Broker に返します
-   **在庫管理フロントエンド**は以下を行います :
    -   ストア情報を表示します
    -   各ストアで購入できる製品を表示します
    -   ユーザが製品を"購入"して、在庫数を減らすことを可能にします

要素間のすべての対話は HTTP リクエストによって開始されるため、エンティティはコン
テナ化され、公開されたポートから実行されます。

![](https://fiware.github.io/tutorials.Accessing-Context/img/architecture.png)

必要な設定情報は、関連する `docker-compose.yml` ファイルの services セクションに
あります。
[以前のチュートリアル](https://github.com/Fiware/tutorials.Context-Providers/)で
説明しました。

<a name="prerequisites"></a>

# 前提条件

<a name="docker"></a>

## Docker

物事を単純にするために、両方のコンポーネントが [Docker](https://www.docker.com)
を使用して実行されます。**Docker** は、さまざまコンポーネントをそれぞれの環境に
分離することを可能にするコンテナ・テクノロジです。

-   Docker を Windows にインストールするには
    、[こちら](https://docs.docker.com/docker-for-windows/)の手順に従ってくださ
    い
-   Docker を Mac にインストールするには
    、[こちら](https://docs.docker.com/docker-for-mac/)の手順に従ってください
-   Docker を Linux にインストールするには
    、[こちら](https://docs.docker.com/install/)の手順に従ってください

**Docker Compose** は、マルチコンテナ Docker アプリケーションを定義して実行する
ためのツールです
。[YAML file](https://raw.githubusercontent.com/Fiware/tutorials.Accessing-Context/master/docker-compose.yml)
ファイルは、アプリケーションのために必要なサービスを構成するために使用します。つ
まり、すべてのコンテナ・サービスは 1 つのコマンドで呼び出すことができます
。Docker Compose は、デフォルトで Docker for Windows と D ocker for Mac の一部と
してインストールされますが、Linux ユーザ
は[ここ](https://docs.docker.com/compose/install/)に記載されている手順に従う必要
があります。

次のコマンドを使用して、現在の **Docker** バージョンと **Docker Compose** バージ
ョンを確認できます :

```console
docker-compose -v
docker version
```

Docker バージョン 18.03 以降と Docker Compose 1.21 以上を使用していることを確認
し、必要に応じてアップグレードしてください。

<a name="cygwin"></a>

## Cygwin

シンプルな bash スクリプトを使用してサービスを開始します。Windows ユーザは
[cygwin](http://www.cygwin.com/) をダウンロードして、Windows 上の Linux ディスト
リビューションと同様のコマンドライン機能を提供する必要があります。

<a name="start-up"></a>

# 起動

リポジトリ内で提供される bash スクリプトを実行すると、コマンドラインからすべての
サービスを初期化できます。リポジトリを複製し、以下のコマンドを実行して必要なイメ
ージを作成してください :

```console
git clone git@github.com:Fiware/tutorials.Accessing-Context.git
cd tutorials.Accessing-Context

./services create; ./services start;
```

このコマンドは、起動時に以前
の[在庫管理の例](https://github.com/Fiware/tutorials.Context-Providers)からシー
ドデータをインポートします。

> :information_source: **注 :** クリーンアップをやり直したい場合は、次のコマンド
> を使用して再起動することができます :
>
> ```console
> ./services stop
> ```

<a name="stock-management-frontend"></a>

# 在庫管理フロントエンド

デモ用の Node.js Express のすべてのコードは、GitHub リポジトリ内の `proxy` フォ
ルダ内にあります
。[在庫管理の例](https://github.com/Fiware/tutorials.Step-by-Step/tree/master/context-provider)。
アプリケーションは次の URLs で実行されます :

-   `http://localhost:3000/app/store/urn:ngsi-ld:Store:001`
-   `http://localhost:3000/app/store/urn:ngsi-ld:Store:002`
-   `http://localhost:3000/app/store/urn:ngsi-ld:Store:003`
-   `http://localhost:3000/app/store/urn:ngsi-ld:Store:004`

> :information_source: **ヒント** : さらに、コンテナのログに従うか、Web ブラウザ
> 上で `localhost:3000/app/monitor` の情報を表示することで、最近のリクエストの状
> 況を自分で見ることができます。
>
> ![FIWARE Monitor](https://fiware.github.io/tutorials.Accessing-Context/img/monitor.png)

<a name="stock-management-frontend"></a>

## NGSI v2 npm ライブラリ

Swagger が生成した NGSI v2 クライアント
[npm ライブラリ](https://github.com/smartsdk/ngsi-sdk-javascript) は
、[SmartSDK](https://www.smartsdk.eu/) チームによって開発されました。これは、低
レベルの HTTP リクエストを処理するために使用されるコールバック・ベースのライブラ
リであり、記述されるコードを単純化します。ライブラリに公開されているメソッド は
、次の名前の NGSI v2
[CRUD 操作](https://github.com/Fiware/tutorials.CRUD-Operations#what-is-crud)に
直接マッピングされます :

| HTTP Verb  |                                                  `/v2/entities`                                                  |                                               `/v2/entities/<entity>`                                                |
| ---------- | :--------------------------------------------------------------------------------------------------------------: | :------------------------------------------------------------------------------------------------------------------: |
| **POST**   | [`createEntity()`](https://github.com/smartsdk/ngsi-sdk-javascript/blob/master/docs/EntitiesApi.md#createEntity) |                                                         :x:                                                          |
| **GET**    | [`listEntities()`](https://github.com/smartsdk/ngsi-sdk-javascript/blob/master/docs/EntitiesApi.md#listEntities) | [`retrieveEntity()`](https://github.com/smartsdk/ngsi-sdk-javascript/blob/master/docs/EntitiesApi.md#retrieveEntity) |
| **PUT**    |                                                       :x:                                                        |                                                         :x:                                                          |
| **PATCH**  |                                                       :x:                                                        |                                                         :x:                                                          |
| **DELETE** |                                                       :x:                                                        |   [`removeEntity()`](https://github.com/smartsdk/ngsi-sdk-javascript/blob/master/docs/EntitiesApi.md#removeEntity)   |

| HTTP Verb   |                                                                     `.../attrs`                                                                      |  `.../attrs/<attribute>`   |                                                     `.../attrs/<attribute>/value`                                                      |
| ----------- | :--------------------------------------------------------------------------------------------------------------------------------------------------: | :------------------------: | :------------------------------------------------------------------------------------------------------------------------------------: |
| **POST**    | [`updateOrAppendEntityAttributes()`](https://github.com/smartsdk/ngsi-sdk-javascript/blob/master/docs/EntitiesApi.md#updateOrAppendEntityAttributes) |            :x:             |                                                                  :x:                                                                   |
| **GET**     |       [`retrieveEntityAttributes()`](https://github.com/smartsdk/ngsi-sdk-javascript/blob/master/docs/EntitiesApi.md#retrieveEntityAttributes)       |            :x:             |    [`getAttributeValue()`](https://github.com/smartsdk/ngsi-sdk-javascript/blob/master/docs/AttributeValueApi.md#getAttributeValue)    |
| **PUT**     |                                                                         :x:                                                                          |            :x:             | [`updateAttributeValue()`](https://github.com/smartsdk/ngsi-sdk-javascript/blob/master/docs/AttributeValueApi.md#updateAttributeValue) |
| **PATCH**   | [`updateExistingEntityAttributes()`](https://github.com/smartsdk/ngsi-sdk-javascript/blob/master/docs/EntitiesApi.md#updateExistingEntityAttributes) |            :x:             |                                                                  :x:                                                                   |
| **DELETE**. |                                                                         :x:                                                                          | `removeASingleAttribute()` |                                                                  :x:                                                                   |

<a name="analysing-the-code"></a>

## コードの分析

説明しているコードは
、[Git リポジトリ](https://github.com/Fiware/tutorials.Step-by-Step/blob/master/context-provider/controllers/store.js)の
`store` コントローラ内にあります。

<a name="initializing-the-library"></a>

### ライブラリの初期化

HTTP アクセスのための不必要な定型コードを書くのに時間を費やしたくなく、再発明し
たくありません。したがって、`ngsi_v2` NPM ライブラリを使用します。これは、図のよ
うにファイルのヘッダに含める必要があります。また、`basePath` を設定する必要があ
ります。これは Orion Context Broker の位置を定義します。

```javascript
const NgsiV2 = require("ngsi_v2");
const defaultClient = NgsiV2.ApiClient.instance;
defaultClient.basePath =
    process.env.CONTEXT_BROKER || "http://localhost:1026/v2";
```

<a name="reading-store-data"></a>

### ストアのデータを読む

この例では、指定された **Store** エンティティのコンテキスト・データを読み取って
結果を画面に表示します。エンティティ・データの読み込みは
、`apiInstance.retrieveEntity()` メソッドを使用して行うことができます。ライブラ
リはコールバックを使用するため、以下のような `Promise` 関数でラップされています
。ライブラリ関数 `apiInstance.retrieveEntity()` は、GET リクエストの URL を記入
し、必要な HTTP 呼び出しを行います :

```javascript
function retrieveEntity(entityId, opts) {
    return new Promise(function(resolve, reject) {
        const apiInstance = new NgsiV2.EntitiesApi();
        apiInstance.retrieveEntity(entityId, opts, (error, data) => {
            return error ? reject(error) : resolve(data);
        });
    });
}
```

これにより、次のように `Promises` の中でリクエストをラップすることができます :

```javascript
function displayStore(req, res) {
    retrieveEntity(req.params.storeId, { options: "keyValues", type: "Store" })
        .then(store => {
            // If a store has been found display it on screen
            return res.render("store", { title: store.name, store });
        })
        .catch(error => {
            debug(error);
            // If no store has been found, display an error screen
            return res.render("store-error", { title: "Error", error });
        });
}
```

これは間接的に HTTP GET リクエストを
`http://localhost:1026/v2/entities/<store-id>?type=Store&options=keyValues` に行
うことです。着信リクエストで Store URN を再利用することに注意してください。

同等の cUrl コマンドは次のようになります :

```console
curl -X GET \
  'http://localhost:1026/v2/entities/urn:ngsi-ld:Store:001?type=Store&options=keyValues'
```

レスポンスは以下のようになります :

```json
{
    "id": "urn:ngsi-ld:Store:001",
    "type": "Store",
    "address": {
        "streetAddress": "Bornholmer Straße 65",
        "addressRegion": "Berlin",
        "addressLocality": "Prenzlauer Berg",
        "postalCode": "10439"
    },
    "location": {
        "type": "Point",
        "coordinates": [13.3986, 52.5547]
    },
    "name": "Bösebrücke Einkauf"
}
```

次に、HTTP レスポンスのボディからのストアのデータが PUG レンダリング・エンジンに
渡され、以下に示すように画面に表示されます :

#### `http://localhost:3000/app/store/urn:ngsi-ld:Store:001`

![Store 1](https://fiware.github.io/tutorials.Accessing-Context/img/store.png)

効率を上げるためには、ネットワーク・トラフィックを削減するためにできるだけ少ない
属性をリクエストすることが重要です。この最適化はまだコードでは行われていません。

コンテキスト・データが使用できない場合、たとえば、ユーザが存在しないストアをクエ
リする場合などには、エラー・ハンドラが必要です。次のようにエラー・ページに転送さ
れます :

#### `http://localhost:3000/app/store/urn:ngsi-ld:Store:005`

![Store 5](https://fiware.github.io/tutorials.Accessing-Context/img/store-error.png)

同等の cUrl コマンドは次のようになります :

```console
curl -X GET \
  'http://localhost:1026/v2/entities/urn:ngsi-ld:Store:005?type=Store&options=keyValues'
```

次のように、レスポンスのステータスは、**404 Not Found** になります :

```json
{
    "error": "NotFound",
    "description": "The requested entity has not been found. Check type and id"
}
```

`catch` メソッド内の `error` オブジェクトは、エラー・レスポンスを保持します。こ
れは、フロントエンドに表示されます。

<a name="aggregating-products-and-inventory-items"></a>

### 製品と在庫アイテムの集約

この例では、指定されたストアの現在の **InventoryItem** エンティティのコンテキス
ト・データを読み取り、その情報を **Product** エンティティの価格と組み合わせます
。結果は手元現金に表示される情報です。

![Till](https://fiware.github.io/tutorials.Accessing-Context/img/till.png)

`Promise` チェーンを作成するか `Promise.all` を使用して、複数のエンティティをリ
クエストして集約できます。ここで、**Product** メソッド および **InventoryItems**
エンティティは、`apiInstance.listEntities()` ライブラリメソッドを使用してリクエ
ストされています。リクエスト内に `q` パラメータが存在すると、受信したエンティテ
ィのリストがフィルタリングされます。

```javascript
function displayTillInfo(req, res) {
    Promise.all([
        listEntities({
            options: "keyValues",
            type: "Product"
        }),
        listEntities({
            q: "refStore==" + req.params.storeId,
            options: "keyValues",
            type: "InventoryItem"
        })
    ])
        .then(values => {
            // If values have been found display it on screen
            return res.render("till", {
                products: values[0],
                inventory: values[1]
            });
        })
        .catch(error => {
            debug(error);
            // An error occurred, return with no results
            return res.render("till", { products: {}, inventory: {} });
        });
}

function listEntities(opts) {
    return new Promise(function(resolve, reject) {
        const apiInstance = new NgsiV2.EntitiesApi();
        apiInstance.listEntities(opts, (error, data) => {
            return error ? reject(error) : resolve(data);
        });
    });
}
```

結果を集約するために使用されるコード(在庫が揃っている各アイテムの製品名を表示)は
、フロントエンドで `mixin` に委託されています。集約されたデータを別のコンポーネ
ントに渡す場合、外部キー集約(`item.refProduct === product.id`)が Node.js コード
に追加されている可能性があります :

```pug
mixin product(item, products)
  each product in products
    if (item.refProduct === product.id)
      span(id=`${product.id}`)
        strong
          | #{product.name}

        | &nbsp; @ #{product.price /100} &euro; each
        | - #{item.shelfCount} in stock
        |
```

Orion Context Broker への HTTP リクエストのいずれかが失敗した場合、空の製品リス
トが返されるようにエラーハンドラが作成されました。

リクエストごとに **Product** エンティティの完全なリストを取得することは効率的で
はありません。キャッシュから製品のリストをロードし、価格が変更された場合にのみリ
ストを更新する方がよいでしょう。これは、後続のチュートリアルの主題である NGSI サ
ブスクリプション・メカニズムを使用して実現できます。

これは、次の cURL コマンド(ビジネス・ロジックを加えたもの)と同等です。

```console
curl -X GET \
  'http://localhost:1026/v2/entities/?type=Product&options=keyValues'
curl -X GET \
  'http://localhost:1026/v2/entities/?q=refStore==urn:ngsi-ld:Store:001&type=InventoryItem&options=keyValues'
```

<a name="updating-context"></a>

### コンテキストの更新

アイテムを購入すると、棚に残されたアイテムの数が減ることになります。この例は 2
つのリンクされたリクエストで構成されています。**InventoryItem** エンティティ・デ
ータの読み取りは、前述のように `apiInstance.retrieveEntity()` メソッドを使用して
行うことができます。`apiInstance.updateExistingEntityAttributes()` メソッドを使
用して Orion Context Broker に送信される前に、データはメモリ内で修正されます。こ
れは事実上、更新される要素を含むボディを持つ
`http://localhost:1026/v2/entities/<inventory-id>?type=InventoryItem` への HTTP
PATCH リクエストを囲むラッパーです。この機能にはエラー処理がありません。ルータの
機能に委ねられています。

```javascript
async function buyItem(req, res) {
    const inventory = await retrieveEntity(req.params.inventoryId, {
        options: "keyValues",
        type: "InventoryItem"
    });
    const count = inventory.shelfCount - 1;
    await updateExistingEntityAttributes(
        req.params.inventoryId,
        { shelfCount: { type: "Integer", value: count } },
        {
            type: "InventoryItem"
        }
    );
    res.redirect(`/app/store/${inventory.refStore}/till`);
}

function updateExistingEntityAttributes(entityId, body, opts) {
    return new Promise(function(resolve, reject) {
        const apiInstance = new NgsiV2.EntitiesApi();
        apiInstance.updateExistingEntityAttributes(
            entityId,
            body,
            opts,
            (error, data) => {
                return error ? reject(error) : resolve(data);
            }
        );
    });
}
```

状況の変化がアトミックに確実に行われるように、コンテキストを修正するときは注意が
必要です。Node.JS ではシングル・スレッドであるため、これは問題ではありません。各
リクエストは 1 つずつリクエストを実行します。しかし、Java などのマルチスレッド環
境では、2 つの購入リクエストを同時に処理することができます。つまり、リクエストが
インターリーブされた場合、`shelfCount` が 1 回だけ削減されます。この問題は、監視
メカニズムを使用することで解決できます。

これは、次の cURL コマンド(ビジネス・ロジックを加えたもの)と同等です。

```console
curl -X GET \
  'http://localhost:1026/v2/entities/urn:ngsi-ld:InventoryItem:001/attrs/shelfCount/value'
curl -iX PATCH \
  'http://localhost:1026/v2/entities/urn:ngsi-ld:InventoryItem:006/attrs' \
  -H 'Content-Type: application/json' \
  -d '{ "shelfCount":
  { "type": "Integer", "value": "13" }
}'
```

<a name="next-steps"></a>

# 次のステップ

高度な機能を追加することで、アプリケーションに複雑さを加える方法を知りたいですか
？このシリーズ
の[他のチュートリアル](https://www.letsfiware.jp/fiware-tutorials)を読むことで見
つけることができます:

---

## License

[MIT](LICENSE) © 2018-2019 FIWARE Foundation e.V.
