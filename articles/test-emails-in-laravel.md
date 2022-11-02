---
title: "Laravelで送信メールをテストするお手軽な2つの方法"
emoji: "📨"
type: "tech" # tech: 技術記事
topics: [Laravel, Test, Email]
published: false
---
# Laravel利用時のお手軽なメールのテスト方法 2選
Laravelでアプリを開発中にLaravelからメール通知を送信する場合があるかと思います。
そんな時に、メールがユーザにどのように見えるのかをお手軽にテストするのに便利なサービスを2つ紹介します✨

## テーブル オブ コンテンツ(目次)
- [mailhogを使うやり方](#mailhogを使うやり方)
- [mailtrapを使うやり方](#mailtrapを使うやり方)

## mailhogを使うやり方
一つ目はmailhogを使うやり方です

- macOSの場合は[Homebrew](https://brew.sh)をお使いだと思うので、brewでmailhogを入れます(未インストール時)

```bash
brew install mailhog
```
> 
> **メモ：Macじゃない人は[こちら](https://www.apple.com/jp/)**

- mailhogを起動します

```bash
mailhog
```
> **メモ：
> (初回はmacOSから「ネットワーク接続を許可する？」とポップアップが出るので許可してください)**

- Laravelは標準の`.env`ファイルにmailhogを使用するサンプルが書かれているので、こいつをちょっといじります

```vim
MAIL_MAILER=smtp
MAIL_HOST=localhost //ここをlocalhostに変えてね
MAIL_PORT=1025
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
MAIL_FROM_ADDRESS="hello@example.com"
MAIL_FROM_NAME="${APP_NAME}"
```

で、Laravelからメール通知を送信してみます📩

- mailhog確認画面にアクセスします（メールが届いてます）

ブラウザを開いて[http://localhost:8025](http://localhost:8025)にアクセスします

![](https://storage.googleapis.com/zenn-user-upload/bb0787932072-20221102.png)

あら、便利💓

## mailtrapを使うやり方
2つ目はMailtrapを使う方法です<br>
(古いバージョンのLaravelではMailtrapが`.env`にサンプルで書かれてたりしました)

- まずはサインアップ

[Mailtrap](https://mailtrap.io)にアクセスしてサインアップします（GitHubアカウント連携とか楽で良いです）
![](https://storage.googleapis.com/zenn-user-upload/9abb8263f1b7-20221102.png)

- メールボックスの作成
Sandboxにある`Setup Inbox`をクリックします
![](https://storage.googleapis.com/zenn-user-upload/bd277dccb77d-20221102.png)

- 右上にある`Add Project`をクリックします
![](https://storage.googleapis.com/zenn-user-upload/8b0c7467f2e2-20221102.png)

- Project Nameを適当に入力（Laravelとか）して`Add`をクリックします
![](https://storage.googleapis.com/zenn-user-upload/daf367087966-20221102.png)

- Add Inboxをクリックしてインボックスを作成します
![](https://storage.googleapis.com/zenn-user-upload/2dc8b8b76698-20221102.png)


![](https://storage.googleapis.com/zenn-user-upload/89514722d1a8-20221102.png)


- 作成したインボックスをクリックします
![](https://storage.googleapis.com/zenn-user-upload/4949121c3dd2-20221102.png)


- Integrationsをクリックして`Laravel 7+`を選択します
![](https://storage.googleapis.com/zenn-user-upload/2a97cf999ea9-20221102.png)


- Laravelの`.env`用の認証情報が表示されます (*下記の情報はサンプルで既に破棄済みです)
![](https://storage.googleapis.com/zenn-user-upload/5fa238cb39e2-20221102.png)

`.env`ファイルの下記をMailtrapの認証情報に合わせて変更します

```vim
MAIL_MAILER=smtp //mailtrapを貼り付け
MAIL_HOST=smtp.mailtrap.io //mailtrapを貼り付け
MAIL_PORT=2525  //mailtrapを貼り付け
MAIL_USERNAME=生成されたUSERNAME  //mailtrapを貼り付け
MAIL_PASSWORD=生成されたPASSWORD  //mailtrapを貼り付け
MAIL_ENCRYPTION=tls  //mailtrapを貼り付け
MAIL_FROM_ADDRESS="hello@example.com"
MAIL_FROM_NAME="${APP_NAME}"
```

- メールを送るとMailtrapのインボックス内に表示されます
![](https://storage.googleapis.com/zenn-user-upload/93f55cca4a7c-20221102.png)

あらあら、便利💓