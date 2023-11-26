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
- WSL2（Ubuntu22.04）をインストールしている
- GitHubにリポジトリをpushできる

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


### CLIをインストールする
Zennのコンテンツを管理したいディレクトリで、以下のコマンドを実行します。
```bash:workspace下で実行
npm init --yes # プロジェクトをデフォルト設定で初期化
npm install zenn-cli # zenn-cliを導入
```
これでディレクトリにCLIがインストールされます。

### Zenn用のセットアップを行う
続いて以下の`npx`コマンドを実行します。
```bash:workspace下で実行
npx zenn init
```
以上のコマンドを順に実行すると、ディレクトリ構成が以下のようになります。
```bash=
workspace/
├── node_modules/
├── articles/
├── books/
├── package-lock.json
├── package.json
└── README.md
```
`articles/` ディレクトリに`.md` ファイルを配置すると、記事を作成することができます。

### CLIをアップデートする
Zenn CLIの表示がzenn.devと異なるときやCLI利用時に更新通知が表示されたときは下記のコマンドでアップデートを行ってください。
```bash:workspace下で実行
npx install zenn-cli@latest
```

### ローカルで記事を閲覧する
```bash:workspace下で実行
npx zenn preview
```
で記事を閲覧することができます。コマンドを実行すると、

```
👀 Preview: http://localhost:8000
```
と出てきます。webブラウザで`http://localhost:8000`にアクセスすると、記事を閲覧することができます。


# 3. VSCodeの拡張機能Zenn Editorを使ってCWLを編集する環境を整える
[Zennの執筆を支援するVSCode拡張Zenn Editor](https://zenn.dev/negokaz/articles/aa4e12b76d516597a00e)に詳しく掲載されています。これを用いると、VSCode上でZennの記事をプレビューすることができます。こちらのプレビュー機能は、おそらくZennのサイトで公開するときと同じように表示されます。

:::message
Zenn Editorをアクティブにするためには、CLIをインストールしたディレクトリ（先ほどの`workspace\`）を開いている必要があります。
:::


# 4. 作成した記事をホスティングサービスにアップロードする
作成した記事を非公開のままレビューを貰いたい場合、ホスティングサービスにアップロードする必要があります。ここでは、netlifyを用いてアップロードする方法を紹介します。

netlifyは無料で使うことができ、GitHubにpushすると勝手に更新してくれるため便利です。（URLは都度変わりますが。）

## netlifyのアカウントを作成する
[netlify](https://www.netlify.com/)にアクセスして、アカウントを作成します。githubアカウントで作成するのがおすすめです。

## ホスティングサービスにアップロードするための準備
[zennでも限定公開がしたい！](https://zenn.dev/cumet04/articles/zenn-private-preview#3.-netlify%E3%81%AB%E4%B8%8A%E3%81%92%E3%81%A6%E3%81%BF%E3%82%8B)を参考にしました。
`preview.js` と `package.json` を`workspace/` に作成します。すなわち、ディレクトリ構造は以下のようになり、`zenn-article.md` は記事のファイル名です。

```bash=
workspace/
├── node_modules/
├── articles/
        └── zenn-article.md
├── books/
├── preview.js
├── package.json
├── package-lock.json
└── README.md
```

それぞれのファイルの中身は次のように編集します。
::::details preview.js
```js:preview.js
const fs = require("fs");
const path = require("path");
const markdownToHtml = require("zenn-markdown-html");
const matter = require("gray-matter");

const outDir = "./preview";

function main() {
  listArticles()
    .filter((article) => !article.published)
    .forEach((article) => {
      const html = buildArticle(article);

      const out = path
        .resolve(outDir, article.filepath)
        .replace(/\.md$/, ".html");
      fs.mkdirSync(path.dirname(out), { recursive: true });
      fs.writeFileSync(out, html);
    });

  fs.mkdirSync(outDir, { recursive: true });
  fs.copyFileSync(
    "./node_modules/zenn-content-css/lib/index.css",
    path.resolve(outDir, "zenn-content-css.css")
  );

  // TODO: プレビューしない記事のものも含めて全コピーしているのを何とかする？
  const imageDir = path.resolve(outDir, "images");
  fs.mkdirSync(imageDir, { recursive: true });
  fs.cpSync("./images", imageDir, { recursive: true });
}

function listArticles() {
  return fs.readdirSync("./articles").map((file) => {
    const filepath = path.relative(__dirname, path.resolve("articles", file));
    const raw = fs.readFileSync(filepath, { encoding: "utf-8" });
    const {
      data: { title, published },
      content,
    } = matter(raw);

    return {
      filepath,
      title,
      published,
      content,
    };
  });
}

function buildArticle({ title, content }) {
  const article = markdownToHtml.default(content);

  return `
<html>
<head>
  <meta name="robots" content="noindex">
  <script src="https://embed.zenn.studio/js/listen-embed-event.js" ></script>
  <link rel="stylesheet" href="../zenn-content-css.css" />
  <style>
    body {
      background-color: #edf2f7;
    }
    header {
      margin-top: 50px;
      height: 100px;
      display: grid;
      place-items: center;
    }
    article {
      max-width: 790px;
      padding: 40px;
      margin: 0 auto;
      border-radius: 12px;
      background-color: white;
    }
  </style>
</head>

<body>
  <header>
    <h1 class="title">${title}</h1>
  </header>
  <main>
    <article class="znc">${article}</article>
  </main>
</body>
</html>
  `;
}

main();
```
::::
::::details package.json
```json:package.json
{
  "name": "your_project_name",
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
::::

必要なパッケージが入っていない場合、npmでインストールします。
```bash=
npm install zenn-markdown-html gray-matter
```

ファイルを作成し、GitHubにpushします。

```bash=
git add .
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/your_account/your_repository.git
git push -u origin main
```

:::message
ローカルでbuildを行うと、`preview/` というディレクトリが作成されます。このディレクトリにはmdファイルを元にして変換されたhtmlファイルが格納されます。netlifyでホスティングするときは、ローカルでbuildを行う必要はありません。
:::

## netlifyでビルドする
netlifyのサイトにアクセスして、`Add new site`から`Import an existing project` をクリックします。初めての場合は少し表示が違うかもしれません。

Let's deploy your projectと表示されるので、Deploy with GitHubをクリックします。GitHubのアカウントを選択し、先ほどのリポジトリを選択します。

- Site configurationにあるBuild settingsのBuild commandに `npm run build:preview` を入力します。
- Build settingsのPublish directoryに `preview` を入力します。

![netlify](/images/netlify.png =800x)
設定が終わったら、Deployボタンをクリックします。以下のように表示されれば成功です。
![netlify_deploy](/images/deploy_log.png =1000x)

## プレビューを確認する
Deploy logから`Preview` を取得すると、`https://656313fbea7b154f6ef3ca28--famous-lamington-80d36e.netlify.app/`のようなURLが表示されます。このURLがプレビューのためのURLです。

しかし、このURLにアクセスすると404エラーが表示されます。先ほど取得したURLの末尾に`articles/zenn-article` をURLの末尾に追加すると、記事が表示されます。このURLをコピーして、レビューを依頼する人に記事を共有することができます。

:::message
`articles/`下にある`.md`ファイルが複数ある場合でも、`articles/zenn-article2`のようにURLを変更するだけで表示することができます。
:::

:::message alert
netlifyでホスティングしたサイトは、ところどころ不完全なところがあります。例えば、`mermaid` で作成した図や、`KaTeX`の数式がレンダリングされません。これは、Zennのカスタムエレメント`zenn-embed-elements`がnetlifyで動作していないためです。この問題の解決方法は模索中です。
:::

### 不具合の例
```mermaid
graph LR;
A --> B --> C;
```

$$
e^{i\pi} + 1 = 0
$$