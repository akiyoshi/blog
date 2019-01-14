---
title: "Hugoで生成した静的サイトをAzure Storage Blobでホスティングする手順"
date: 2019-01-13T19:30:32+09:00
draft: false
categories:
  - Azure
tags:
  - Azure Blob storage

---

先月、[Azure Blob Storageでの静的サイトのホスティングがGAしました](https://azure.microsoft.com/en-us/blog/static-websites-on-azure-storage-now-generally-available/)。
今回は、これを使って、[Hugo](https://gohugo.io/)で生成したサイトをホスティングする手順をまとめます。

<!--more-->

### 検証環境

- OS: Mac 10.14.2
- Homebrew 1.8.6
- Hugo: 0.52
- azure-cli :2.0.52

# 環境構築

### Homebrewのインストール

Mac OSの場合は、[Homebrew](https://brew.sh/index_ja.html)を使うのが簡単です。[公式サイト](https://brew.sh/index_ja.html)からインストールしてください。

インストールが完了すると以下のターミナルでバージョンが確認できます。

```
brew -v
```

### Hugoのインストール

[公式ドキュメント](https://gohugo.io/getting-started/installing/)に各OSごとの説明があるので参照してください。
Mac OSの場合は以下のコマンドでインストールできます。

```
brew install hugo
```

### Azure CLIのインストール

Blobに生成したファイルをアップロードするために、[Azure CLI](https://docs.microsoft.com/ja-jp/cli/azure/install-azure-cli?view=azure-cli-latest)を使用します。Mac以外のOSを使用している場合は、[公式ドキュメント](https://docs.microsoft.com/ja-jp/cli/azure/install-azure-cli?view=azure-cli-latest)を参照してください。

```
brew install azure-cli
```

# Hugoによるサイトの作成

Hugo[公式のクイックスタート](https://gohugo.io/getting-started/quick-start/)でサンプルサイトを作成します。既に作成済みのサイトがある場合はスキップしてください。

```
# サンプルサイトの作成
hugo new site quickstart

# gitの初期化とテーマのインストール
cd quickstart

git init
git submodule add https://github.com/budparr/gohugo-theme-ananke.git themes/ananke

echo 'theme = "ananke"' >> config.toml

# 新規投稿を追加
hugo new posts/my-first-post.md
```

以下のコマンドにhugo サーバーを起動し、http://localhost:1313/ にアクセスして確認します。
```
hugo server -D
```

# 静的サイトの有効化

細かい手順は[公式のクィックスタート](https://docs.microsoft.com/ja-jp/azure/storage/blobs/storage-blob-static-website#quickstart)が正確ですが、以下の通りの手順で設定します。

1. (ない場合は)[Storage アカウントを作成します。](https://docs.microsoft.com/ja-jp/azure/storage/common/storage-quickstart-create-account?tabs=azure-portal)
2. ポータルから作成したアカウントの設定から「静的なWebサイト」を選択
3. 「有効」を選択して保存します。
4. インデックスドキュメント(index.html)と、エラードキュメント(404.html)をそれぞれ設定します。

設定が完了すると、以下のようになります。

![portal-for-static-website](/images/portal-for-static-website.png)



プライマリエンドポイント(ex. https://<アカウント名>.z11.web.core.windows.net)が作成したサイトのURLになります。

また、ストレージのコンテナ名は **$web** になります。

# サイトのデプロイ

以上で準備ができたので、ファイルをアップロードしてデプロイします。
最新の手順は[公式ドキュメント](https://docs.microsoft.com/ja-jp/azure/storage/blobs/storage-blob-static-website#azure-cli)を参照してください。

```
# ログイン
az login

# サブスクリプションの選択
az account set --subscription <SUBSCRIPTION_ID>

# ファイルのアップロード
# <SOURCE_PATH>はHugoで生成したファイルパス
# 例えば/public フォルダです
az storage blob upload-batch -s <SOURCE_PATH> -d \$web --account-name <ACCOUNT_NAME>
```

https://<アカウント名>.z11.web.core.windows.net にアクセスすると、作成されたサイトが表示されます。

**この段階だとHugo側の設定が不十分なため、cssや画像がうまく機能しません**

# 調整

**config.toml** を開き、baseUrlを以下のように変更します。

```
baseurl = "https://<アカウント名>.z11.web.core.windows.net"
```

これでファイルのパスが修正されるので、変更を反映し、ファイルをアップロードします

```
# サイトを再ビルド
hugo

# ファイルをアップロード
az storage blob upload-batch -s <SOURCE_PATH> -d \$web --account-name <ACCOUNT_NAME>
```

サイトにアクセスして、ローカルと同じように表示されたら完了です。