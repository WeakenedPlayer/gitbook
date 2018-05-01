# ビルド関係

##  Webpack

### ts-loaderの設定

ターゲットごとに `tsconfig.json` を変更したい場合、`configFile`オプションでtsconfigファイルを切り替えられるようにすると良いが、パスの指定を間違えると見つからずにエラーになる。

```text
ERROR in [tsl] ERROR
      TS18002: The 'files' list in config file 'tsconfig.json' is empty.
```

そうならないよう、`__dirname`を使って`webpack.config.js`の場所を基準としてパスを指定すると良い。以下のようにすれば、`webpack.config.js`と同じ場所にある`tsconfig.node.json`を探しに行くようになる。

```text
const path = require('path');
const webpack = require( 'webpack' );

module.exports = {
    mode: 'development',
    entry: 'index.ts',
    devtool: 'source-map',
    resolve: {
        extensions:['.ts', '.webpack.js', '.web.js', '.js', '.json' ]
    },
    module : {
        rules: [
            {
                test: /\.ts$/,
                loader: 'ts-loader',
                options: {
                	configFile: `${__dirname}/tsconfig.node.json`
                }
            }
        ],
    },
    externals: [],
    plugins:[],
};
```

### webpack-mergeでターゲットごとに分ける

汎用的なモジュールをブラウザとnode.jsの両方で使いたい場合、分けてビルドする。

#### node.js用

```text
const commonConfig = require('./webpack.common.config.js');
const merge = require('webpack-merge');

module.exports = merge( commonConfig, {
    target: 'node',
    output : {
        path: `${__dirname}/../dist/node`,
        filename: '[name].js'
    },
    node: {
        __dirname: false,
        __filename: false,
    }
} );
```

#### ブラウザ用

```text
const commonConfig = require('./webpack.common.config.js');
const merge = require('webpack-merge');

module.exports = merge( commonConfig, {
    target: 'web',
    output : {
        path: `${__dirname}/../dist/web`,
        filename: '[name].js'
    },
} );

```

## npm

### publish

スコープを付けて、ベータ版を発行する場合は、`package.json`を以下のように作っておく。

```text
{
	"scope": "@スコープ",
  	"name": "@スコープ/モジュール名",
	"version": "0.0.0-beta.0",...
}

```

その上で、npm publishを実行する。 ここではベータ版なので`--tag beta`を付けている。

```text
npm publish --tag beta --access public
```

` --access public`が無いとデフォルトでprivate扱いとなるが、privateは有料なのでエラーが起きる。 

```text
npm ERR! publish Failed PUT 402
npm ERR! code E402
npm ERR! You must sign up for private packages : @スコープ/モジュール名
```

## Typescript

### moduleのexport

名前空間の衝突を避けるためにmoduleを使う場合要注意。以下のようにして、MyModule.SubModule1のような階層化を行おうとしてもNGで、SubModule1,2はどちらも見えなくなってしまう。

```text
[root.ts]
import * as MyModule from './my-module';
export { MyModule }

[my-module.ts]
export module SubModule1{
...
}

export module Submodule2 {
...
}
```

理由は調べていないが、以下のようにすると良くなった。\(Eclipseの表示だけ?\)

```text
[my-module.ts]
module SubModule1{
...
}

module Submodule2 {
...
}

export { SubModule1, SubModule2 }
```

