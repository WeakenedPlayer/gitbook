---
description: node.jsで作成したサーバを動かすために、OpenShiftを利用しようと思うので、使い方を調べてみた。
---

# OpenShift

## Run Your Nodejs projects on OpenShift in Two Simple Steps

原文「[Run Your Nodejs projects on OpenShift in Two Simple Steps](https://blog.openshift.com/run-your-nodejs-projects-on-openshift-in-two-simple-steps/)」

### Step1: package.json を配置する

Nodejsアプリケーションは、`package.json`をプロジェクトのルートに配置している必要がある。OpenShiftとnpmは、`script.start`と`main`を確認し、サーバを立ち上げるための情報を得る。

```text
"scripts": {
  "start": "node server.js"
},
"main": "server.js",
dependencies: {
  "必要なモジュール": "バージョン", ...
}
```

{% hint style="info" %}
startは起動スクリプト、mainは起動スクリプトで実行する対象と同じものを指定する。
{% endhint %}

また、ビルド時に外部から取得するモジュールが分かるよう、`dependencies` に必要なモジュールが含まれていることも確認すること。

開発時に `npm install`する際、`--save `フラグを付けておけば、常にビルドに必要なモジュールが `package.json` に記述された状態にできる。

### Step2: 環境変数から必要な情報を取得する

OpenShiftのNode.jsカートリッジは、自動的にサーバ接続移管する情報\(ポートやIP\)をアプリケーションの環境変数に追加する。ポートは`OPENSHIFTNODEJS_PORT`、IPは`OPENSHIFT_NODEJS_IP` である。

プロジェクトの移植性のために、以下のように環境が含まれているかどうかを判定するとよいだろう。

```text
var server_port = process.env.OPENSHIFT_NODEJS_PORT || 8080
var server_ip_address = process.env.OPENSHIFT_NODEJS_IP || '127.0.0.1'
 
server.listen(server_port, server_ip_address, function () {
  console.log( "Listening on " + server_ip_address + ", port " + server_port )
});
```

{% hint style="info" %}
「process.env.環境変数名」で、指定した環境変数が存在したらそのIPとポートを使い、しなければデフォルト値として「127.0.0.1: 8080」を使うようにしていることで、ローカル環境とOpenShift環境のどちらでも動作するようになっている。
{% endhint %}

MongoDBを使用している場合、[認証情報を取得](https://blog.openshift.com/10-reasons-openshift-is-the-best-place-to-host-your-nodejs-app#composability)するための[判定が必要](https://blog.openshift.com/getting-started-with-mongodb-on-nodejs-hosting)となるだろう。

```text
//provide a sensible default for local development
mongodb_connection_string = 'mongodb://127.0.0.1:27017/' + db_name;
//take advantage of openshift env vars when available:
if(process.env.OPENSHIFT_MONGODB_DB_URL){
  mongodb_connection_string = process.env.OPENSHIFT_MONGODB_DB_URL + db_name;
}
```

 [config-chain](https://www.npmjs.org/package/config-chain) のようなモジュールにより、単一の変数にこれらの情報をまとめることもできる。これにより、コード全体で情報へのアクセス手段を統一することができる。

ポートやIPだけでなく、アプリケーションキー や Secret も同様に環境変数によりアクセスできるようにすることができる。

```text
rhc env set GA_TRACKER=UA-579081
rhc env set SECRET_KEY=0P3N_S0URC3
rhc env list
```

環境変数に関する情報は、「[RedisCoundのような外部のサービスを利用する](https://blog.openshift.com/how-to-get-easy-access-to-hosted-redis-with-redis-cloud/#env)」を参照すること。

以上。上記の設定ができたなら、もうOpenShift上で動くようになるはずだ。

{% hint style="info" %}
実行環境に依存する IPやポートだけでなく、アプリケーションキーや secret も環境変数から提供すべきなのは、こうした情報をソースコードに書くとまずいから。OpenShiftはGitHubのようなリポジトリに実行するコードが存在することを期待しているので、ソースコードに書くと、公に知られてしまう可能性がある。
{% endhint %}

{% hint style="info" %}
この情報は少し古いらしい。環境変数に直接登録するのは手段の一つだが、「secret」という格納所が用意されたので、その方が便利で安全だと思う。「[OpenShift Applications: Using Object Storage](https://blog.openshift.com/openshift-applications-using-object-storage/)」

ただ、secretは暗号化されていないので、OpenShiftの中の人からは見えてしまう。それすらダメな場合は「[Encrypting Secret Data at Rest](https://blog.openshift.com/encrypting-secret-data-rest/)」のように暗号化する必要があるらしい
{% endhint %}

## OpenShift Applications: Using Object Storage

原文「[OpenShift Applications: Using Object Storage](https://blog.openshift.com/openshift-applications-using-object-storage/)」

### Using Object Storage

OpenShiftでは、Kubernetes の`persistent volume`フレームワークにより、永続化ストレージが利用できるようになっている。これにより、管理者は`pod`の状態によらない永続化ストレージが利用できる。これは、例えばOpenShift上で動作するデータベースの格納や、利用者からアップロードされたファイルの格納などに利用できる。

しかし、アプリケーションを、バックエンドのストレージに依存せず、どのOpenShiftクラスタでも実行したい場合はどうしたらよいか?

そういう場合はObject Storageをバックエンドのストレージとして利用することができる。これは `persistent volume` を必要としない。加えて、OpenShiftのどの環境からもデータを利用することができるようになる。

### Using Secret

secret オブジェクトは、Kubernetes が提供する機密情報\(パスワード、Openshift client config file, プライベートリポジトリへの認証情報 など\)の格納場所である。コンテナにマウントされたら、設定ファイルのように読み出すことができる。

例えば、[Amazon S3ストレージを使ったアップローダのサンプル](https://github.com/christianh814/php-object-store)で使い方を見てみる。

サンプルの`index.php`を見て分かるように、`file_get_contenst`_ 関数を用いて、_S3への接続に必要な情報を取得している。

```text
if (!defined('awsAccessKey')) define('awsAccessKey', file_get_contents('/etc/secret/aws-access-key'));
if (!defined('awsSecretKey')) define('awsSecretKey', file_get_contents('/etc/secret/aws-secret-key'));
```

別途、新しいボリュームをマウントし、アップロードしたファイルを保存するための secret も作成する。\(他にも、サンプルを動かすためには、 ec2-creds ファイルを作成し、環境変数を正しく設定しておく必要がある\)

### Deploying And Application With Secrets

\(以降はサンプルのための設定のため省略\)



めも

当座、環境変数を使う場合

[https://docs.openshift.org/latest/dev\_guide/configmaps.html](https://docs.openshift.org/latest/dev_guide/configmaps.html)

