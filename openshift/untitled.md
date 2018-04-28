# reCAPTCHA

ユーザIDを登録しやすくするために、ウェブページを作成したいが、いたずらは防止したいので、reCAPTCHAを導入する。参考ページ「[https://codeforgeek.com/2016/03/google-recaptcha-node-js-tutorial/](https://codeforgeek.com/2016/03/google-recaptcha-node-js-tutorial/)」を調べてみる。

## reCAPTCHAの登録

Googleのアカウントでログインした状態で、「[Google reCAPTCHA](https://www.google.com/recaptcha/intro/android.html)」にアクセスし、Get reCAPTCHA で登録すると、キーをもらえる。

* Site Key: HTTPに埋め込むキー
* Secret Key: サーバで利用するキー

## クライアントの準備

### ng-recaptchaの利用

Angular2の場合、[ng-recaptcha](https://www.npmjs.com/package/ng-recaptcha) というモジュールを使って作ると良さだった。app.module.ts に ng-recaptchaの説明通りに組み込んでおき、コンポーネントで CAPTCHA を配置すればよい。

```text
import { RecaptchaModule } from 'ng-recaptcha';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    HttpModule,
    FormsModule,
    RecaptchaModule.forRoot()   // ここに設定
  ],
  providers: [Location, {provide: LocationStrategy, useClass: PathLocationStrategy}],
  bootstrap: [AppComponent]
})
export class AppModule { }

```

### reCAPTCHAの配置

コンポーネントには、以下のような要素を配置する。これがCAPTCHAになり、ユーザが人だと判断されたら、resolvedイベントに割り当てた関数\(ここではresolved関数\)が呼ばれ、引数にはチェックがうまくいったことを示す文字列が格納される。

```text
 <re-captcha (resolved)="resolved($event)" name="captch" required siteKey="サイトキー"></re-captcha>
```

### スクリプト

例えば、チェックがうまくいったらすぐにリクエストを出す場合は、以下のようにする。色々なデータ\(ここでは name \)と合わせて、チェックがうまくいったことを示す文字列をサーバに送ってあげることで、クライアント側の仕事は完了となる。

```text
    resolved( result: string ) {
        this.http.post( '/submit', {
            name: this.name,
            recaptcha:  result // ここにチェックがうまくいったことを示す文字列が入る
        } ).toPromise().then( ( res: Response ) => {
            console.log( res );
        } );
    }
```

### サーバの準備

サーバは、クライアントから送られてくる「チェックがうまくいったことを示す文字列」が正しいかをGoogleに問い合わせ、OKかどうかを確認する。\([Verifying the user's responseの「API Request」](https://developers.google.com/recaptcha/docs/verify#api-request)\)

例えば以下のようにログを表示したら何が送られてくるか分かる。

```text
let verifyUrl = 'https://www.google.com/recaptcha/api/siteverify?secret='+secret+'&response=' + req.body.recaptcha;
request( verifyUrl, ( err, res, body ) => {
    console.log( JSON.parse( body );
} );
```

結果は以下のようになる。成功していれば`success = true`になっているはずである。応答がどんな形式かは、APIリファレンスの「[API Response](https://developers.google.com/recaptcha/docs/verify#api-response)」の項を参照すること。

```text
{ success: true,
  challenge_ts: '時間',
  hostname: 'localhost' }
```



