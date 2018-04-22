# Census API

## Census API のイベント通知

Census API にはWebsocketを介してイベント通知を受けることができる機能がある。これにより、リアルタイムにユーザのログインや経験値の取得を知ることができるようになっていて、今話題のAmmo PaddingやHeal Paddingの監視にも使える面白い機能である。

### イベント通知を受けるために

Websocket でどんな通知を受けたいかをCensusAPIに伝えることで、指定したイベント通知を受けられるようになる。また、イベント通知が始まった後でもイベントを追加したり削除したり、フィルタ条件を変更したりできるようになっている。

### Subscribeコマンド

WebsocketでCensus APIへ以下のようなJSON形式のコマンドを送ることで、イベント通知を受けることができるようになる。

```text
{
	"service":"event",          // このデータがコマンドであることを示す
	"action":"subscribe",       // コマンドの種類がsubscribeであることを示す
	"eventNames":["Death"],     // イベントの種類
	"characters":["5428010618015189713"],  // フィルタ
}
```

#### イベントの種類

イベントの種類は `eventNames: string[ ]`で複数指定することができる。使用可能なイベントは以下のようなものがある。なお、経験値取得イベントは頻繁に発生するため、種類\(experience\_id\)ごとに細かく指定することができるようになっている。

* キャラクタ関係
  * AchievementEarned
  * BattleRankUp
  * Death
  * ItemAdded
  * SkillAdded 
  * VehicleDestroy
  * GainExperience
  * GainExperience\_experience\_id\_\*\*\*
  * PlayerFacilityCapture
  * PlayerFacilityDefend
* サーバ関係
  * ContinentLock
  * ContinentUnlock
  * FacilityControl
  * MetagameEvent
* キャラクタ・サーバ関係
  * PlayerLogin
  * PlayerLogout

#### イベントのフィルタ

イベントはそのままだと多すぎる場合があるため、フィルタにより対象を絞ることができるようになっている。フィルタには、キャラクタID `characters: string[ ]` とサーバ `worlds: string[ ]` の2種類がある。

| イベント | charactersフィルタ | worldsフィルタ |
| --- | --- | --- | --- |
| キャラクタ関係 | 〇 | 〇 \(OR/AND選択可\) |
| サーバ関係 | × | 〇 |
| キャラクタ・サーバ関係 | 〇 | 〇 \(OR/AND選択可\) |

#### フィルタの論理AND

上表に示すように、キャラクタ関係およびキャラクタ・サーバ関係のイベントには、characters と worlds  両方のフィルタが適用できる。両方のフィルタAND したければ以下のように `logicalAndCharactesrWithWorlds` を `true`に設定する。

> ただ、論理ANDを使うことは滅多ないと思う。キャラクタはどこかのサーバに所属しているから、charactesrs フィルタで指定しておきながらworlds で論理AND で除外するのなら、初めから characters に入れない方が良い。

以下の例は、サーバ: Connery かつ全てのキャラクタの論理AND = ConneryのプレイヤーのDeathイベントを取得できるようになる。

```text
{
    "service":"event",
    "action":"subscribe",
    "eventNames":["Death"],
    "characters":["all"],
    "worlds":["1"],
    "logicalAndCharactersWithWorlds":true
}
```

フィルタのイメージは以下のようになる。

| キャラクタフィルタ | サーバフィルタ | AND: false時 | AND: true時 |
| --- | --- | --- | --- | --- |
| 〇 | 〇 | 〇 | 〇 |
| 〇 |  × | 〇 | × |
| × | 〇 | ×  | × |
| × |  × | × | × |

#### subscribe変更時の応答

subscribeコマンドと、後述するclearSubscribeコマンドを送ると、以下のような応答が返ってくる。子の応答から、現在何を購読しているかが分かるようになっている。

```text
{
	"subscription":{
		"characterCount": charactersフィルタで指定したキャラクタ数,
		"eventNames":[ 購読中のイベント, ... ],
		"logicalAndCharactersWithWorlds": 論理ANDするかどうか,
		"worlds":[ 対象のWorld ]
	}
}
```

### clearSubscribe コマンド

`eventNames: string[ ]` で指定したイベントの通知を止めるコマンド。フィルタは再度指定しなおす必要がある。以下では、`characters`と`worlds`フィルタをそれぞれ指定しなおしている。

```text
{
	"service":"event",
	"action":"clearSubscribe",
	"eventNames":[
		"BattleRankUp",
		"Death",
	],
	"characters":[
		"1",
		"2"
	],
	"worlds":[
		"1",
		"2"
	]
}
```

全てのイベントの通知を止める場合、以下のように all を指定する。

```text
{
    "action":"clearSubscribe",
    "all":"true",
    "service":"event"
}
```

### recentCharacterIds コマンド

現在ログインしているユーザIDすべて返す。

```text
{
    "action":"recentCharacterIds",
    "service":"event"
}
```

一度だけ以下のような応答が返ってくる。

```text
{
  "service":"event",
  "type":"serviceMessage",
  "payload": {
    "recent_character_id_count":3127,"
    "recent_character_id_list":[ "ID", ... ] 
  }
}

```

### recentCharacterIdsCountコマンド

現在ロゴグインしているユーザID数を返す。

```text
{
    "action":"recentCharacterIdsCount",
    "service":"event"
}
```

一度だけ以下のような応答が返ってくる。

```text
{
	"service":"event",
	"type":"serviceMessage",
	"payload":{
		"recent_character_id_count":3181
	}
}
```

### echoコマンド

各サーバ\(Connery, Emerald, ...\)から、送ったデータがそのまま送り返されてくる。

```text
{
    "service":"event"
    "action":"echo",
    "payload":{
      "test":"test"
    }
}
```

### helpコマンド

ヘルプコマンドの説明。応答は「http://census.daybreakgames.com/」の説明がそのまま出てくるのであまり意味はない。

```text
{
    "service":"event",
    "action":"help"
}
```

## イベント

イベント通知とは別に、ハートビートや接続時のメッセージ等、Census APIがメッセージが送られるようになっている。

### Websocket接続

Websocketに接続できたら送られる。

```text
{
	"service":"push",
	"type":"connectionStateChanged",
	"connected":"true"
}
```

### イベントサーバ接続

Websocketの接続後、各イベントサーバにも接続できたら送られる。

```text
{ 
    "service":"event", 
    "type":"serviceStateChanged",
    "detail":"EventServerEndpoint_Connery_1",
    "online":"true"
}
```

### ハートビート

Census APIのイベントを通知するサーバが生きていることが分かるよう定期的に送られる。

```text
{
	"service":"event",
	"type":"heartbeat",
	"online":{
		"EventServerEndpoint_Connery_1":"true",
		"EventServerEndpoint_Miller_10":"true",
		"EventServerEndpoint_Cobalt_13":"true",
		"EventServerEndpoint_Emerald_17":"true",
		"EventServerEndpoint_Jaeger_19":"true",
		"EventServerEndpoint_Briggs_25":"true"
	}
}

```





