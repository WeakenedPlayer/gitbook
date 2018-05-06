# つまづき

## ERROR &lt;unavailable&gt; {#error-less-than-unavailable-greater-than}

Angularのhttpクライアント機能を使ってCensus API のデータをロードしようとしたところ、Firefoxのログにこのエラーが表示されて、うまくいかなかった。始めはCORS関係の制約に引っかかっているのかと思ったが、Chromeでは動作したことから、ブラウザ側の問題の様だった。

調べてみると[StackOverflow「Object 'unavailable' in Firefox console」](https://stackoverflow.com/questions/45373290/object-unavailable-in-firefox-console)に答えがあり、エラーというよりは console.log の問題らしかった。表示はできなくても裏で動作はしているらしかった。ログを表示したければ`console.log(JSON.stringify( object ))` のように加工すると良いらしい。

数時間を無駄にしてしまった。

## NPM

### postinstall-build

\([npmの説明ページ](https://www.npmjs.com/package/postinstall-build#what-does-it-do)\) モジュールをインストールした際にビルドしてもらうための仕組み。`article`とは、ビルドしたものが格納される場所を指す。`npm install` した時に、空だったら再度ビルドされる。ビルドするときだけ、devDependenciesをインストールし、終わったら消す。

```text
 postinstall-build <article> ビルドコマンド(デフォルトでnpm run build)
```

typescriptやwebpackはビルド時にインストールする必要があるものの、既に提供されていることも想定されるので、buildDependenciesにだけ書くと良い。インストール時の環境に存在しなければ警告が表示される。

ビルド時に必要なもの\(型情報とか\)であれば、devDependencies/buildDependencies両方に追加する。

```text
{
 	 "scripts": {
 	 	"build": "tsc -p tsconfig.json",
  	  	"postinstall": "postinstall-build dist"
	},
 	 "dependencies": {
  	 	"es6-error": "^4.1.1",
  	 	"postinstall-build": "^3.0.0"
  	},
  	"devDependencies": {
        ここにはtypescriptを書かない
    },
    "buildDependencies": {
		"typescript": "*"←ビルド時に既に提供されていることを期待するもの
    }
}

```

#### .npmignoreに注意する

ビルドに必要なファイルをうっかり `.npmignore` に入れてしまわないようする必要がある。ビルドに必要な `tsconfig.json` を` .npmignore` に入れてしまい、ファイルが無いと怒られてしまうと以下のようになる。

```text
 error TS5058: The specified path does not exist: 'tsconfig.json'.
```

逆に、上記の `article` を`.npmignore`に入れわすれ、npmにアップロードすると、postinstall-buildが動かなくなってしまう。古いファイルが入ったままだと、「なぜか最新が反映されない」という事態に陥る。どちらも気づきにくい嵌まりどころなので要注意。

### package.json

 mainにモジュールの利用者から見た実行コードの入り口を設定し、typesに型情報\(d.tsファイル: tsconfig.jsonのdeclarationオプションをtrueにしていると生成される\) を設定する。

これを忘れると、モジュールが見つからなかったり、型情報が参照されずにコードコンプリートが表示されないなどの問題が起こる。

```text
{
 	"main": "dist/index.js",
    "types": "dist/index.d.ts"
}
```

## rxjs

### 思った値が出てこない {#ttagatekonai}

Observableの種類を間違えている可能性がある。Subscribeするタイミングが、値が生成されるタイミングより後の場合、次の値が来るまで何も得られない。もし、最後の値を使いたい場合、replayや、shareReplayを使うことで、Subscribeしたところで最後の値が取得できるようにしたい。

### 最新版と以前のバージョンの互換性

最新版\(v6\)は、map, flatMapのようなオペレータがなくなり、代わりにpipe関数を使って、map, flatMap等の関数をつなぐような仕様になっている。色々違うので、そのままでは古いバージョンと新しいバージョンの共存ができない。

最新と旧バージョンの差については[MIGRATION.mdを参照するとよい](https://github.com/ReactiveX/rxjs/blob/master/MIGRATION.md)。大体の場合はrxjs-compatという後方互換性パッケージがあるのでそれを利用するらしい。

## 外部のJavascriptライブラリを使う

Google Sign-inで使用する「Google Platform Library」のように、NPMで提供されないライブラリを利用する場合、Index.htmlでライブラリをロードしておく必要がある。\(ローカルにコピーするのも手だが\)

動的にロードした方が安全かもしれないので、動的にロードし準備ができるのを待つことにする。

> 参考: [How to load external scripts dynamically in Angular?](https://stackoverflow.com/questions/34489916/how-to-load-external-scripts-dynamically-in-angular)

```text
import { Injectable, Optional } from '@angular/core';
import { BehaviorSubject, Observable } from 'rxjs';

const url = 'https://apis.google.com/js/platform.js';
declare let gapi: any;

export class GoogleSigninServiceConfig {
    clientId: string;
} 

@Injectable()
export class GoogleSigninService {
    private _loaded: BehaviorSubject<boolean> = new BehaviorSubject( false );
    get loaded$(): Observable<boolean> {
        return this._loaded;
    }

    constructor( private config: GoogleSigninServiceConfig ) {
        if( !config ) {
            throw new Error( '[Error] Google Sign-in Service requires configuration.' );
        }
        this.loadScript();
    }

    private loadScript() {
        let script = document.createElement( 'script' );
        script.src = url;
        script.type = 'text/javascript';
        script.async = true;
        script.charset = 'utf-8';
        script.onload = () => { this.onLoad() };
        document.getElementsByTagName( 'head' )[ 0 ].appendChild( script );
    }
    
    private onLoad() {
        gapi.load('auth2', () => {
            gapi.auth2.init( { 'client_id': this.config.clientId } );
            this._loaded.next( true );
        } );
    }
    
    render( element: any, options?: any ) {
        if( !options ) {
            options = {};
        }
        gapi.signin2.render( element, {
            ...options,
            'accesstype': 'online',
            'onsuccess': ( user ) => { 
                console.log( user );
            },
            'onfailure': ( user ) => { console.log( 'failed' ) }
        } );
    }
}
```

forRootで、ClientIDも設定できるようにした。

```text
import { NgModule, ModuleWithProviders, Optional, SkipSelf } from '@angular/core';
import { GoogleSigninButtonComponent } from './button.component';
import { GoogleSigninService, GoogleSigninServiceConfig } from './service';

@NgModule( {
    declarations: [ GoogleSigninButtonComponent ],
    exports:      [ GoogleSigninButtonComponent ],
    imports: [],
    providers: []
} )
export class GoogleSigninModule {
    static forRoot( config: GoogleSigninServiceConfig ): ModuleWithProviders {
        return {
            ngModule: GoogleSigninModule,
            providers: [ { provide: GoogleSigninServiceConfig, useValue: config } ]
        };
    }
}

```

