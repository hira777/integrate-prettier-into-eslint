# Integrate Prettier into ESLint

Sample Code integrating Prettier into ESLint.

## Installation

```
npm install
```

## Usage

### ESLint(with Prettier)

You can format, lint, auto-fix your code using Prettier and ESLint when you run command below.

```
npm run lint
```

### Pre-commit Hook

You can use ESLint(with Prettier) with a pre-commit tool. This can run `npm run lint` to your files that are marked as "staged" via `git add` before you commit.

# はじめに

Prettier（v.2.1.1） に関しての備忘録です。

- 「Prettier の何が便利なのかよくわからない」
- 「ESLint と何が違うのかよくわからない」
- 「`eslint --fix`でコード整形ができるから Prettier の必要性が感じられない」

といった人達向けに書いた記事であり

- Prettier の基本情報、利用目的、利用方法を理解する
- 何故 ESLint だけではなく Prettier も利用するのかを理解する
- Prettier と ESLint の併用方法を理解する

ことを目的としています。

ESLint 自体の説明は記載しておりませんので、ご注意ください。

解説に利用しているコードの最終形態は GitHub 上にあります。

[hira777/integrate-prettier-into-eslint](https://github.com/hira777/integrate-prettier-into-eslint)（2020/09/08 更新）

# Prettier とは？

コードフォーマッター（ソースコードを整形してくれるツール）のこと。読み方はプリティア。

以下のような様々な形式にサポートしている。

- JavaScript
- JSX
- Flow
- TypeScript
- JSON
- HTML
- Vue
- Angular
- CSS
- Less
- SCSS
- styled-components 💅
- styled-jsx
- GraphQL
- Markdown
- YAML

## 何故コードフォーマッタを利用するのか（コードを整形するのか）

- ソースコードの品質を保つため（コードのスタイルの一貫性を保つため）
- コードレビュー時に、設計や命名などの重要な箇所に集中するため（コードのスタイルの指摘に時間を割くのを防ぐため）
- 複数のメンバーが各自の整形ルールを適用し、更新する度に余計な差分が発生することを防ぐため
- ソースコードを綺麗にするための労力（スタイル定義の議論や時間）を費やさなくて済むため
- ツールに任せられることはツールに任せてしまった方が今後楽になるため

# Prettier を利用してみる

Prettier を利用してソースコードが整形されたファイルを出力してみる。

## ディレクトリ構成

今回 Prettier を利用するディレクトリ構成は以下を前提とする。

```
integrate-prettier-into-eslint
├── package.json
└── app.js
```

### `package.json`

パッケージをローカルインストールするため、`package.json`は以下のコマンドで生成する。

```shell

npm init -y
```

or

```shell
yarn init -y
```

### `app.js`

ソースコードを整形するファイル。

スペースの数が統一されておらず、無駄な改行が存在したりとコードが汚い状態。

```js:整形前のapp.js
class Person{
  constructor(name) {
    this.name =   name;
  }

}


const profile = {
  name: 'soarflat', sex: 'male', location: 'Tokyo'

};

const hoge = (message)=>{
  console.log(message);
}

const fooBar = (a, b, c) =>{
    console.log(a);
  console.log(b)
   console.log(c);
};

fooBar(111, 
    {
        hoge: 'hoge!'
    },profile
)
```

## Prettier のインストール

`npm`か`yarn`でローカルインストールする。

```shell
npm install prettier@2.1.1 -D
```

or

```shell
yarn add prettier@2.1.1 -D
```

### ローカルインストールした Prettier を実行するために PATH を通す

`prettier`コマンドを実行できるようにするため、以下のように PATH を通す。

```sh
export PATH=$PATH:./node_modules/.bin
```

※「PATH を通す」が不明な方は以下をご覧ください。

- [PATH を通すとは？ (Mac OS X)](http://qiita.com/soarflat/items/09be6ab9cd91d366bf71)
- [PATH を通すために環境変数の設定を理解する (Mac OS X)](http://qiita.com/soarflat/items/d5015bec37f8a8254380)

## Prettier の実行

`app.js`に対して`prettier`コマンドを実行する。ファイルを上書きするために`--write`オプションをつける。

```
prettier app.js --write
```

コマンドを実行すれば、整形されたファイルが出力される。

```js:整形後のapp.js
class Person {
  constructor(name) {
    this.name = name;
  }
}

const profile = {
  name: "soarflat",
  sex: "male",
  location: "Tokyo"
};

const hoge = message => {
  console.log(message);
};

// hoge(new Person('Person').name);

const fooBar = (a, b, c) => {
  console.log(a);
  console.log(b);
  console.log(c);
};

fooBar(
  111,
  {
    hoge: "hoge!"
  },
  profile
);
```

### 整形されたファイルを出力できたが

ESLint でもコード整形はできるし、結局 Prettier の何が良いのかわからないため、Prettierは ESLint より何が優れており、何故利用（併用）すべきかを説明していく。

# 何故 Prettier を利用（ESLint と併用）するのか

ESLint でも`eslint --fix`でコード整形ができるが、Prettier の方がコード整形が優れているから。

具体的には以下の点が優れている。

- ESLint では整形できないコードを整形できる
- ESLint と比べて手軽で確実に整形できる

## ESLint では整形できないコードを整形できる

以下のような一行の文字数が長いコードは ESLint でエラーが出力されるが、整形はされない。

```js
foo(reallyLongArg(), omgSoManyParameters(), IShouldRefactorThis(), isThereSeriouslyAnotherOne());
```

一方、Prettier では以下のように整形される。

```js
foo(
  reallyLongArg(),
  omgSoManyParameters(),
  IShouldRefactorThis(),
  isThereSeriouslyAnotherOne()
);
```

上記のような整形をしてくれれば、複数のメンバーがエラーに対する独自の修正を適用し、更新するたびに差分が出てしまうような事態を防ぐことができる。

そのため、Prettier を利用した方がコードのスタイルの一貫性は保たれる。

## ESLint と比べて手軽で確実に整形できる

`eslint --fix`は設定に該当したエラーのみを整形する。そのため、**設定次第ではコードを整形しきれない**。

整形を厳密に行うためには、多くの設定を指定する必要があり、設定自体が複雑になる（設定ファイルがすごい行数になる）可能性がある。

一方、Prettier はデフォルトのスタイル（整形ルール）が存在するため、`prettier`コマンドを実行するだけでコードが整形される。

スタイルの変更も可能だが、こだわりがなければデフォルトのままで問題ないし、確実に整形してくれる。

また、Prettier は良くも悪くも設定できる項目が少ないため、設定が複雑になる可能性が低い。

そのため、Prettier を利用した方が手軽で確実に整形できる。

## Prettier は ESLint よりもコード整形が優れているが

上記の理由で Prettier を利用すべきだが、Prettier はコードフォーマッタのため、ESLint のような構文チェック機能はない。

そのため、コードの整形は Prettier が行い、コードの構文チェックは ESLint が行うように併用する。

# ESLint との併用

併用方法はいくつかあるが、今回は Prettier の設定を ESLint で利用できるようにする。

そのため、`eslint --fix`を実行時に、Prettier の整形ルールでコードを整形し、整形したコードに対して構文チェックが実行されるようにする。

この方法は以下のメリットがある。

- `eslint --fix`だけでコードの整形と構文チェックができる
- 既存の ESLint の設定ファイル（`.eslintrc`など）に、少し設定を記述するだけで利用できる

## 併用に必要なパッケージのインストール

上記の併用方法で必要なパッケージをインストールする。

```
npm install eslint@7.8.1 eslint-config-prettier@6.11.0 eslint-plugin-prettier@3.1.4 -D
```

or

```
yarn add eslint@7.8.1 eslint-config-prettier@6.11.0 eslint-plugin-prettier@3.1.4 -D
```

それぞれのパッケージの詳細は以下の通り。

- `eslint`（ESLint 本体）
- `eslint-config-prettier`（ESLint のフォーマット関連のルールを全て無効にする、要は Prettierが整形した箇所に関してエラーを出さなくなる）
- `eslint-plugin-prettier`（Prettier を ESLint 上で実行する）

## `.eslintrc`の設定

先ほど Prettier を実行したディレクトリに`.eslintrc`（ESLint の設定ファイル）を作成する。

そのため、ディレクトリ構成は以下のようになる。

```
.
├── .eslintrc
├── package.json
└── app.js
```

`.eslintrc`の内容は以下の通り。

```json:.eslintrc
{
  "env": {
    "browser": true,
    "es6": true
  },
  "extends": ["eslint:recommended", "plugin:prettier/recommended"],
  "rules": {
    "prettier/prettier": [
      "error",
      {
        "singleQuote": true,
        "trailingComma": "es5"
      }
    ]
  }
}
```

Prettier に関連する記述は

```json
"plugin:prettier/recommended"
```

と

```json
"rules": {
  "prettier/prettier": [
    "error",
    {
      "singleQuote": true,
      "trailingComma": "es5"
    }
  ]
}
```

である。`"singleQuote": true`などのオプションは以下のオプションを指定できる。

[Options · Prettier](https://prettier.io/docs/en/options.html)

また、`prettier`に関連する extends は必ず`extends`配列内の最後に記述する。

以下のような記述だとうまく動作しないことがあるため注意。

```json
{
  "extends": ["plugin:prettier/recommended", "eslint:recommended"]
}
```

## Prettier と ESLint を併用した状態で`eslint --fix`を実行してみる

<img src="https://qiita-image-store.s3.amazonaws.com/0/69667/1cf78e8a-f449-af48-dca6-895095f995e2.gif" width=500px>

コードの整形、構文チェックができた。

# `git commit`時に`eslint --fix`が実行されるようにする

上記の方法でコード整形と構文チェックが可能になったが、`eslint --fix`を実行しない限りコードは整形されないため、整形されていないコードがコミットされてしまう可能性がある。

これを防ぐために、エディタと連携してファイル保存時に`eslint --fix`を実行する手段がある。

個人での開発ではそれで問題ないが、複数人開発の場合、全員がそれぞれのエディタプラグインを導入するコストがかかる。

そのため、整形したコードのみを確実にコミットしたいのであれば`git commit`時に`eslint --fix`が実行されるようにする。

Git にはコミット前に指定のスクリプトを実行できる`pre-commit`フックと言う仕組みがあるため、それを利用する。

## `pre-commit`フックに必要なパッケージをインストール

```
npm install lint-staged@10.3.0 husky@4.3.0 -D
```

or

```
yarn add lint-staged@10.3.0 husky@4.3.0 -D
```

## `package.json`に`pre-commit`フックの設定を記述する

`package.json`に以下の記述を追加する。

```json
{
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  },
  "lint-staged": {
    "*.js": "eslint --fix"
  }
}
```

記述後の`package.json`は以下のようになる。

```json:package.json
{
  "name": "integrate-prettier-into-eslint",
  "version": "1.0.0",
  "scripts": {
    "lint": "eslint --fix ."
  },
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  },
  "lint-staged": {
    "*.js": "eslint --fix"
  },
  "devDependencies": {
    "eslint": "^7.8.1",
    "eslint-config-prettier": "^6.11.0",
    "eslint-plugin-prettier": "^3.1.4",
    "husky": "^4.3.0",
    "lint-staged": "^10.3.0",
    "prettier": "^2.1.1"
  },
  "license": "MIT"
}
```

これで`git commit`時に`eslint --fix`が実行されるようになった。

## `git commit`時に`eslint --fix`が実行されるか確認する

![precommit.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/69667/39ad464b-c37e-1dcb-e175-64aa0a58181f.gif)

`git commit`時に`eslint --fix`が実行され、エラーを解消しなければコミットできなくなった。

# 終わり

コードは綺麗に越したことはないし、手軽に導入できるので少しでも魅力に感じたら利用してみましょう。

`pre-commit`フックまで利用するのは堅苦しく感じる方もいると思いますので、とりあえずはコマンドで整形するかエディタと連携させれば良いと思います。