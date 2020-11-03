---
title: "へなちょこWebサーバーを作る"
---

# Webサーバーを作る手順
いきなり「Webサーバーとはこういうものです」と解説を始めると、急に難解な本になってしまいます。

なので、できるだけ説明と勉強を少なくするために、実際にブラウザとWebサーバーがやりとりしている通信を盗み見て、それを最小限に再現する真似っ子サーバーを作ることにしましょう。
意味を理解するのは後です。

**実際にプログラムをインストールしたりコードを書いたりするのは次章以降にして、この章ではその手順のみを説明します。**

1. ChromeとApacheで通信してみる
2. Chromeと自作サーバーで通信してみる
3. 自作クライアントとApacheで通信してみる
4. 自作サーバーを進化させる


# STEP1: ChromeとApacheで通信してみる
シェアは減少傾向とはいえ、現在でも世界的によく使われているWebサーバーに`Apache`というものがあります。
まずは、このApacheというWebサーバーを自分のPC内で起動して、自分のPC内で起動したブラウザからアクセスしてみることにします。

「自分のマシンで動いているブラウザというプログラム」と、
「自分のマシンで動いているApacheというプログラム」で通信をする、
ということです。

ブラウザはなんとなくChromeを使うことにしました。特に理由はありません。

これで、普通にやりとりするブラウザとWebサーバーが用意できます。

![](https://storage.googleapis.com/zenn-user-upload/baachuvryp4tm31nji2yvy2kobdr)



# STEP2: ブラウザと自作サーバーで通信してみる
「じゃあ早速ChromeとApacheの通信を覗き見て、Apacheの代わりになるような真似っ子Webサーバーを作ってやるぜ！」
とやりたいところですが、他人が作ったプログラムの通信を傍受するのは結構大変です。

なのでまずは、**通信の受け取りだけ**ができる簡単なサーバーを自作して、Apacheと置き換えてみます。

この自作のへなちょこサーバーは通信の受け取りだけはできるので、ブラウザが送ってきた通信内容を見ることができます。
本当はApacheが返すような返事をブラウザに返すことができればいいのですが、私たちはまだApacheがどのような返事を返しているのかを知りません。
ですので、このサーバーはブラウザに返事を返すことはしないことにします。
ブラウザにはエラーが表示されるでしょうが、最初はそれで十分です。

しかしこれでとりあえず、 **「ブラウザがWebサーバーにどんなリクエストを送っているか」** を知ることが出来ます。

![](https://storage.googleapis.com/zenn-user-upload/7d5ic67are7kx9x6sb43p1aqkbgf)



# STEP3: 自作クライアントとApacheで通信してみる
今度は、ブラウザの代わりになるようなクライアント^[サーバーと通信するプログラムを、総称してクライアントと呼びます]を自作してみます。
この自作のへなちょこクライアントは、ブラウザのように色々なページを色々なやり方でサーバーに要求することはできません。

しかし、STEP2でChromeがWebサーバーへ送っている内容は分かっていますので、その内容をApacheへそのまま送りつけることはできます。
すると、ApacheはまともなWebサーバーですので、まともな返事をしてくれます。
自作クライアントではこれを受け取れば **「Webサーバーがどのようなレスポンス^[リクエストに対する応答をレスポンスといいます。] を返すのか」** が分かるという寸法です。

![](https://storage.googleapis.com/zenn-user-upload/qctbfjd6qevkzl4qr7r2stglv335)


# STEP4: 自作サーバーを進化させる
STEP3で、世間一般のWebサーバーはどんなレスポンスを返すのかが分かるはずなので、自作サーバーがきちんとしたレスポンスを返せるように進化させましょう。

自作サーバーがそれなりにまともなレスポンスを返せるようになると、ブラウザにはエラー画面ではなくWebページっぽいのが表示されるようになります。

ここまでできると、「へなちょこWebサーバー」の完成です。

![](https://storage.googleapis.com/zenn-user-upload/1vn37rowyibflzqikq84m0p8nsvy)


それでは、次章をお楽しみに！