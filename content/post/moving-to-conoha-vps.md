---
title: "ConoHaのVPSにブログを引っ越しました"
date: 2021-10-18T09:20:01+09:00
draft: false
tags: ['conoha', 'blog', 'vps']
summary: "ConoHaのVPS色々わかりやすくていいですね"
---

ブログ以外にもアプリをデプロイする場所が欲しかったので、ConoHaのVPSを契約してnetlifyからお引越ししました。VPSのプランは一番スペックの低いサーバーで、以下のようなスペックになっています。
- CPU 1Core
- メモリ 512MB
- SSD 30GB

契約初日になぜかsshがタイムアウトするなどトラブルはありましたが(なぜか解決した)なんとかブログを公開することができました。また、引っ越しに伴い`kmgk.dev`から`blog.kmgk.site`へドメインを変更しました。

ブログを公開するまでにやったことは
- VPS契約
- Google ドメインでドメインを取得
- DNS設定
- ユーザ作ったり公開鍵認証でssh接続
- nginxのインストール＆設定
- Let's encryptで証明書取得

こんな感じです。

せっかく契約したので色々アプリを公開していきたいですね。