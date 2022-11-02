---
title: "Laravelで送信メールをテストするお手軽な2つの方法"
emoji: "📨"
type: "tech" # tech: 技術記事
topics: [Laravel, Test, Email]
published: false
---
# Laravel利用時のお手軽なメールのテスト方法 2選
Laravelでアプリを開発中にLaravelから送信するメールがユーザにどのように見えるのかをお手軽にテストするのに便利なサービスを2つ紹介します

### mailhogを使うやり方
一つ目はmailhogを使うやり方です

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

- macOSの場合は[Homebrew](https://brew.sh)をお使いだと思うので、brewでmailhogを入れます(まだ入れてないなら)

```bash
brew install mailhog
```
> **メモ：Macじゃない人は[こちら](https://www.apple.com/jp/)**
<br>

- mailhogを起動します

```bash
mailhog
```
> **メモ：(初回はmacOSから「ネットワーク接続を許可する？」とポップアップが出るので許可してください)**
<br>

- mailhog確認画面にアクセスします

ブラウザを開いて[http://localhost:8025](http://localhost:8025)にアクセスします

<img width="1170" alt="image" src="https://user-images.githubusercontent.com/7894265/198750115-30c28afe-b239-4ff3-b844-5c96238f18c0.png">

あら、便利💓

### mailtrapを使うやり方
2つ目はMailtrapを使う方法です<br>
(古いバージョンのLaravelではMailtrapが`.env`にサンプルで書かれてたりしました)

- まずはサインアップ

[Mailtrap](https://mailtrap.io)にアクセスしてサインアップします（GitHubアカウント連携とか楽で良いです）
<img width="640" alt="image" src="https://user-images.githubusercontent.com/7894265/198857616-cd1ca19c-51a3-4144-9586-97cdffa8925b.png">

- メールボックスの作成
Sandboxにある`Setup Inbox`をクリックします
<img width="640" alt="image" src="https://user-images.githubusercontent.com/7894265/198857638-3b6a1d66-6f0d-43ef-a714-94ac40b2dc20.png">

- 右上にある`Add Project`をクリックします
<img width="640" alt="image" src="https://user-images.githubusercontent.com/7894265/198857675-85948f68-80ee-4081-ab8a-099b6cfb4ec6.png">

- Project Nameを適当に入力（Laravelとか）して`Add`をクリックします
<img width="640" alt="image" src="https://user-images.githubusercontent.com/7894265/198857688-333cebbc-e87c-484e-bbf0-bb99e698eb3b.png">

- Add Inboxをクリックしてインボックスを作成します
<img width="640" alt="image" src="https://user-images.githubusercontent.com/7894265/198857709-37adc06b-916e-4a2b-b81c-8f6c0dd56b3e.png">

<img width="640" alt="image" src="https://user-images.githubusercontent.com/7894265/198857714-2d36405c-1ef8-473c-996c-b17dd4e7cff8.png">

- 作成したインボックスをクリックします
<img width="640" alt="image" src="https://user-images.githubusercontent.com/7894265/198857735-26a32e37-7cff-4af5-92e2-530db944149e.png">

- Integrationsをクリックして`Laravel 7+`を選択します
<img width="640" alt="image" src="https://user-images.githubusercontent.com/7894265/198857783-b8e70965-eb66-4d3a-999f-c361062f281a.png">

- Laravelの`.env`用の認証情報が表示されます (*下記の情報はサンプルで既に破棄済みです)
<img width="640" alt="image" src="https://user-images.githubusercontent.com/7894265/198857811-77dbd6f7-fa84-4cbc-af83-c344c84ddc25.png">

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
<img width="640" alt="image" src="https://user-images.githubusercontent.com/7894265/198858006-31db2c44-8fab-4b16-af08-7e28ba0c1b02.png">

あらあら、便利💓