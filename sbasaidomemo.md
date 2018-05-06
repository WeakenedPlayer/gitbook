# サーバサイドメモ

## 提供する機能

Outfitの設立者が不在となり、募集ページが使えなくなった際の救済策として、Commanderによるinviteを効率的に行うための仕組みを提供する。

### ログイン通知

Outfit参加希望者が自身のキャラクタIDを登録すると、そのキャラクタがログインしたタイミングでOutfitに関連付けられたDiscordチャネルに通知が届くようになる。それをOutfitメンバーが確認してログイン中にoutfit inviteすれば、参加させることができるようになる。

1.  Googleアカウントでログインする
2. キャラクタを検索する
3. 仮登録
4. Googleアカウントにメールが届く
5. URLを開くと登録される

### Outfit参加希望者募集

募集者が自身のキャラクタが参加するアウトフィットに対して、募集を追加する。

1.  Googleアカウントでログインする
2. キャラクタを検索する
3. 仮登録
4. メールが届く
5. URLを開くと登録される

## Outfit Recruit API

{% api-method method="post" host="" path="/v1/recruit-reg/" %}
{% api-method-summary %}
募集仮追加
{% endapi-method-summary %}

{% api-method-description %}

{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-body-parameters %}
{% api-method-parameter name="id\_token" type="object" required=true %}
Google Identification 
{% endapi-method-parameter %}
{% endapi-method-body-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```

```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

{% api-method method="get" host="" path="/v1/recruit/outfit" %}
{% api-method-summary %}
Get Outfits
{% endapi-method-summary %}

{% api-method-description %}
募集中のアウトフィット一覧を取得する。
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}
アウトフィット一覧を返す。
{% endapi-method-response-example-description %}

```javascript
    "outfit_list": [
        {
            outfit_id: string;
            faction_id: string;
            world_id: string;
        }
    ],
    "returned": number;
}
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

{% api-method method="post" host="" path="/v1/reg/outfit/:id" %}
{% api-method-summary %}
Add Outfit
{% endapi-method-summary %}

{% api-method-description %}
アウトフィットを追加する。\(事前に認証が必要\)
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-path-parameters %}
{% api-method-parameter name="id" type="string" required=true %}
アウトフィットID
{% endapi-method-parameter %}
{% endapi-method-path-parameters %}

{% api-method-body-parameters %}
{% api-method-parameter name="channel" type="string" required=true %}
連絡先のDiscord Channel
{% endapi-method-parameter %}

{% api-method-parameter name="captcha" type="string" required=true %}
reCAPTCHAの結果
{% endapi-method-parameter %}
{% endapi-method-body-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```

```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

{% api-method method="post" host="" path="/v1/reg/character" %}
{% api-method-summary %}

{% endapi-method-summary %}

{% api-method-description %}
暫定的に、ログインを一切必要としないインターフェースとしている。アウトフィットとキャラクタが存在し、サーバと勢力の条件が合致すれば登録される。
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-body-parameters %}
{% api-method-parameter name="outfit\_id" type="string" required=true %}
参加したいアウトフィットのID
{% endapi-method-parameter %}

{% api-method-parameter name="character\_id" type="string" required=true %}
参加させるキャラクタのID
{% endapi-method-parameter %}
{% endapi-method-body-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```

```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}



