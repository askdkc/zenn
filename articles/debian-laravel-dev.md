---
title: "Debianに一瞬でLaravel開発環境を構築"
emoji: "🏎️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["linux","laravel","php","debian"]
published: false
---
# Debianに一瞬でLaravel開発環境を構築
## これは何？
Windowsが重たくて使わなくなった余ってるPCにLinuxを突っ込むと生き返るというゾンビのような手段が昔（90年代？00年代？）は流行っておりましたが、Debianとかを突っ込んだPCで最近流行りのLaravelを開発したいじゃない？したいよね？じゃあしようよ！みたいなお話です

## 必要なもの
PCに新規にDebian 12 Bookwormをインストールしたもの

## やり方
1. Terminalを起動します

2. 以下のスクリプトを実行します

```bash
sudo apt install -y curl

curl -L https://gist.githubusercontent.com/askdkc/a687a1c171e4141f529a26704f1dd5cc/raw/6ed03303e88f1d5f55bd406668c5e648b93ba3c5/deb12laravel-setup.sh | bash
```

3. 新規にLaravelプロジェクトを作ります
```bash
cd ~/Sites

laravel new myproj
```

4. プロジェクト名.testなURLにアクセスします

http://myproj.test/

ハイ便利〜！！

