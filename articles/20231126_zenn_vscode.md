---
title: "WSL2とVSCodeを用いてローカル環境でのzenn執筆環境を整える"
emoji: "🙌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [VSCode,zenn]
published: false
Author: yuuma
---

# はじめに

__本記事の対象となる方__

■ WSL2とVSCodeを用いてZennの執筆環境を整えたい人


この記事では､以下の3点をご紹介します｡
__(1) WSL2のディストリビューション（Ubuntu）にZenn CLIをインストールする__
__(2) VSCodeの拡張機能Zenn Editorを使ってCWLを編集する環境を整える__
__(3) ホスティングサービスを利用して、記事の限定公開を実現する__

&nbsp;

# 前提
WSL2（Ubuntu22.04）をインストールすること

# 1. Ubuntuにnode.jsとnpmを入れる
初めに、Ubuntuにnode.jsとnpmを入れます。npmとはNode.jsのパッケージを管理するシステムのことです。これを用いてZenn CLIをインストールします。__[【2023年4月版】Ubuntu に node.js と npm を入れたい（バージョン管理も）](https://qiita.com/nouernet/items/d6ad4d5f4f08857644de)__ の手順をそのまま掲載しています。

### aptのパッケージを更新
まずは、aptのパッケージを最新にします。`sudo apt update` でaptが管理しているパッケージリストを更新します。次に、 `sudo apt upgrade -y` でパッケージをアップデートします。 

```bash=
sudo apt update
sudo apt upgrade -y
```


### aptを使ってとりあえずのnode.jsとnpmをインストール
次に､node.jsとnpmをaptでインストールします。
```bash=
sudo apt install -y nodejs npm
```

### n（バージョン管理システム）をインストール
```bash=
sudo npm install n -g
```

### 最新のnode.jsとnpmをインストール
```bash=
sudo n stable
```

### 最初に入れたnode.jsとnpmをアンインストール
```bash=
sudo apt purge -y nodejs npm
sudo apt autoremove -y
```


&nbsp;

# 2. Zenn CLIを導入して、ローカルで記事を編集する
npmをインストールしたら、Zenn CLIを導入します。Zenn CLIを用いると、ローカルの好きなエディターで投稿コンテンツの作成・編集ができるようになります。
参考：[Zenn CLIをインストールする](https://zenn.dev/zenn/articles/install-zenn-cli)


## CLIをインストールする
Zennのコンテンツを管理したいディレクトリで、以下のコマンドを実行します。
```bash=
npm init --yes # プロジェクトをデフォルト設定で初期化
npm install zenn-cli # zenn-cliを導入
```
これでディレクトリにCLIがインストールされます。

## Zenn用のセットアップを行う
続いて以下の`npx`コマンドを実行します。
```bash=
npx zenn init
```
## CLIをアップデートする
Zenn CLIの表示がzenn.devと異なるときやCLI利用時に更新通知が表示されたときは下記のコマンドでアップデートを行ってください。
```bash=
npx install zenn-cli@latest
```

## ローカルで記事を閲覧する
```bash=
npx zenn preview
```
で記事を閲覧することができます。


# 3. VSCodeの拡張機能Zenn Editorを使ってCWLを編集する環境を整える

# 4. 作成した記事をホスティングサービスにアップロードする
作成した記事を非公開のままレビューを貰いたい場合、ホスティングサービスにアップロードする必要があります。ここでは、netlifyを用いてアップロードする方法を紹介します。

## netlifyのアカウントを作成する
[netlify](https://www.netlify.com/)にアクセスして、アカウントを作成します。githubアカウントで作成するのがおすすめ。

## ホスティングサービスにアップロードするための準備
[zennでも限定公開がしたい！](https://zenn.dev/cumet04/articles/zenn-private-preview#3.-netlify%E3%81%AB%E4%B8%8A%E3%81%92%E3%81%A6%E3%81%BF%E3%82%8B)を参考にした。
`preview.js` と `package.json` をダウンロードして追加する。
```js:preview.js
const fs = require("fs");
const path = require("path");
const markdownToHtml = require("zenn-markdown-html");
const matter = require("gray-matter");
```
これが必要なパッケージなので、npmでインストールする。
```bash=
npm install zenn-markdown-html gray-matter
```
`package.json` を以下のように編集する
```json:package.json
{
  "name": "latex_install",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "build:preview": "node preview.js"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "gray-matter": "^4.0.3",
    "zenn-cli": "^0.1.149",
    "zenn-content-css": "^0.1.134",
    "zenn-markdown-html": "^0.1.149"
  }
}
```

## netlifyでビルドする
netlifyのサイトにアクセスして、`Import an existing project` をクリックする。Build settingsのBuild commandに `npm run build:preview` を入力する。
![netlify](/images/netlify.png =1000x)

## プレビューを確認する
Site overviewから、`Published` となっているものをクリックする。Deploy logから`Preview` を取得すると、
`https://656313fbea7b154f6ef3ca28--famous-lamington-80d36e.netlify.app/`
のようなURLが表示される。このURLにアクセスすると、404エラーが表示される。しかし、
`articles/20231126_texlive` をURLの末尾に追加すると、記事が表示される。