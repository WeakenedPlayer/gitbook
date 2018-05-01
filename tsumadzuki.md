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

