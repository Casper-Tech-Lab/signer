# Casper Signer

Casper Signerは、Casper Networkのトランザクションへ署名するブラウザープラグインです。

最新バージョンは、[Chrome Web Store](https://chrome.google.com/webstore/detail/casperlabs-signer/djhndpllfiibmcdbnmaaahkhchcoijce)にあります。使い方やヘルプは、[Casper Signer ユーザーガイド](https://casper.blockchaintechlab.jp/casper-network/documentation/how-to/casper-signer%e3%83%a6%e3%83%bc%e3%82%b6%e3%83%bc%e3%82%ac%e3%82%a4%e3%83%89/)をご確認ください。

**Note:** Casper Signerは、Google ChromeとBraveの様なChromiumベースのブラウザーで利用できます。最新バージョンのブラウザーの使用を推奨しています。

## Casper Signerとの接続

Mainnet上でCasper Signerをご利用のウェブサイトに接続する際は、[承認プロセス](https://github.com/casper-ecosystem/signer/wiki/Casper-Signer-Whitelisting-Request-Guide)を踏んでください。

## Architecture 

Casper Signerの拡張機能が持っている4つのコンポーネントの概要は下記のとおりとなります。

### 1. ユーザーインターフェイス

src/popupのディレクトリーには、Reactで書かれたブラウザーのユーザーインターフェイス（UI）があります。ユーザーがSignerをクローズする時に、状態を保存しない為、次のステップにあるサーバーとして動作するバックグラウンドスクリプトを使用する必要があります。 

RPCを介したバックグラウンドへのUIによる呼び出しによるアップデートリクエストの送信にて、その状態の同期と修正を行います。

### 2. バックグラウンドスクリプト

2つめのコンポーネントは、UIをアップデートするRPCメソッドを提供するバックエンドサーバーとしてバックグランドで動作するスクリプトです。関連するソースコードは、src/backgroundにあります。

### 3. コンテントスクリプト

コンテントスクリプトは、Casper Signer拡張機能の一部であり、対象のウェブページで機能します。`manifest.json/content_scripts`にルールの追加をし、どのウェブページにコンテントスクリプトを実行させるかを指定できます。 

ブロックエクスプローラ[CSPR.live](https://cspr.live/)の様なウェブページにSignerを紐づけた際、ウェブ拡張機能が新しい*ContentScript*をウェブページのコンテキスト内に作成します。ソースコードは、`src/content/contentscript.ts`にあります。 

よって、質問にて各ページのコンテントスクリプトをセットアップし、`inject.ts`のインジェクトスクリプトを何もロードをしない内にDOMへ呼び出す必要があります。その時点で、コンテントスクリプトは、注入されたスクリプトからリクエストを受け取るプロキシをセットアップし、バックグラウンドスクリプトにそれを転送します。

### 4. スクリプトの注入

4つ目のコンポーネントは、スクリプトの注入（`inject.ts`）です。それは、Casper Signerと紐づいているウェブページのコンテキスト内で実行します。コンテントスクリプトが、前述した注入スクリプトを呼び出すステップを記憶し、`CasperLabsPluginHelper`のグローバルインスタンスを作成すると、ウェブページが`window.casperlabsHelper`を呼び出せるようになります。

## ローカルでの開発

Casper Signerをビルドし実行する為の役立つコマンドをいくつか提供しています。

### 前提条件

まず初めに、全ての依存関係をセットアップする際に使う`npm`をインストールする必要があります。

```bash

npm install

```

### Casper Signerのビルド

次のコマンドは、Casper Signerをビルドし、`build`ディレクトリにファイル達を出力します。このコマンドは、ユーザーインターフェイスの再構築に役立ちますが、変更の反映を確認する際はSignerのポップアップを再度開くかリフレッシュしてください。

```bash

npm run watch

```

### スクリプトの実行

下記のコマンドは、バックグラウンドとコンテント、注入スクリプトを*watch*モードでビルドします。このコマンドを使用する際は、Chromeの拡張機能の管理ページ上で拡張機能をリロードしなくてはなりません。

```bash

npm run scripts_watch

```

### ビルドと公開

変更のビルドと公開には、下記コマンドを実行してください。`artifacts`ディレクトリにビルドされたパッケージが作成されます。

```bash

npm run complete

```

