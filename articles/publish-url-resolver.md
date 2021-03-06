---
title: "「伸び悩んでいる3年目Webエンジニアのための、Python Webアプリケーション自作入門」を更新しました"
emoji: "🚶"
type: "tech"
topics: [python, web, http]
published: true
---

# 本を更新しました

[チャプター「URLパラメータを受け取れるようにする」](https://zenn.dev/bigen1925/books/introduction-to-web-application-with-python/viewer/url-resolver) を更新しました。

続きを読みたい方は、ぜひBookの「いいね」か「筆者フォロー」をお願いします ;-)

----

以下、書籍の内容の抜粋です。

------
# 中〜大規模なWebサービスで必要となる機能
もうかなりフレームワークっぽくなってきて、現在ある機能だけでもそれなりのWebサービスが構築できると思います。
もうWebアプリケーションの基本的な仕組みが理解できて、満足してきた人もいるでしょう。

しかし、サービスが少し大きくなってくると共通して欲しくなる機能がまだ不足しています。
これ以降の章はそういった
*「小さなサービスでは今すぐに必要ではないが、世間ではよく使われる機能」*
について扱っていくことになります。

具体的な例として、

- URLパラメータ
- テンプレートエンジン
- Cookie
- Session
- Middleware
- Database連携

などがあります。

手始めに、本章では**URLパラメータ**を扱えるようにしていきます。

# URLパラメータを扱う
「URLパラメータを扱う」とは、URLの中に現れる文字列（= URLパラメータ）を変数としてアプリケーション内で扱うことを意味します。

言葉で説明すると難しいですが、よくある例として、
`/user/132/profile`
のようなpathにアクセスすると、IDが132のユーザーの詳細情報が表示される、などのWebページを見たことはないでしょうか。

私達の今のアプリケーションでは上記の`123`のような文字列を変数としては扱えないため、
```python
URL_VIEW = {
    '/user/1/profile': views.user1,
    '/user/2/profile': views.user2,
    '/user/3/profile': views.user3,
    # ...
}
```
などと1つずつpathを追加していくしかありません。
しかも、ユーザーの数が増えたり減ったりした状況には対応できません。

コレに対しては、例えば
```python
URL_VIEW = {
    '/user/<user_id>/profile': views.user
}
```
などと設定しておけば`/user/(数字)/profile`のpathが全てviews.userに紐付いて、view関数の中から`(数字)`の値にアクセスできるようになっていると便利です。

このような機能は、一般的なWebフレームワークであれば必ず備えている機能ですので、私達も実装してみましょう。

-----

URLパターンの扱い方は、

- URLパラメータのパターン指定の方法
→　`/user/<user_id>/profile`と書くのか、`/user/:user_id/profile`と書くのか
- パラメータへのアクセスの仕方
→　`request.params["user_id]`でアクセスできるのか、view関数の第2引数として渡されてくる(`def user(request, user_id):`となる)のか

などの観点でフレームワークごとに特色があります。

私たちのフレームワークでは、比較的簡単な下記の仕様で実装していこうと思います。

- **ルーティング設定には`/user/<user_id>/profile`のように、`<変数名>`の形式で指定する**
- **パラメータは、リクエストオブジェクトのparams属性(`request.params`)に`dict型`で格納/アクセスする**

## ソースコード
では、まずは上記の機能を実装したソースコードを見てみましょう

**`study/urls.py`**
https://github.com/bigen1925/introduction-to-web-application-with-python/blob/main/codes/chapter18/urls.py

**`study/views.py`**
https://github.com/bigen1925/introduction-to-web-application-with-python/blob/main/codes/chapter18/views.py

**`study/henango/http/request.py`**
https://github.com/bigen1925/introduction-to-web-application-with-python/blob/main/codes/chapter18/henango/http/request.py

**`study/henango/server/worker.py`**
https://github.com/bigen1925/introduction-to-web-application-with-python/blob/main/codes/chapter18/henango/server/worker.py

## 解説

### `study/urls.py`
#### 8行目
```python
URL_VIEW = {
    # ...
    "/user/<user_id>/profile": views.user_profile,
}
```
動作確認用のサンプルとして、URLパラメータを含むのパスを追加しました

### `study/views.py`
#### 80-94行目
```python
def user_profile(request: HTTPRequest) -> HTTPResponse:
    user_id = request.params["user_id"]
    html = f"""\
        <html>
        <body>
            <h1>プロフィール</h1>
            <p>ID: {user_id}
        </body>
        </html>
    """
    body = textwrap.dedent(html).encode()
    content_type = "text/html; charset=UTF-8"
    status_code = 200

    return HTTPResponse(body=body, content_type=content_type, status_code=status_code)
```
URLパラメータを扱うviewを追加しました。
```python
user_id = request.params["user_id"]
```
この部分で、`HTTPRequest`オブジェクトからURLパラメータを取得しているところが注目ポイントです。

画面としては、シンプルにユーザーIDのみが表示されるのみです。

### `study/henango/http/request.py`
```python
class HTTPRequest:
    # ...
    params: dict

    def __init__(
        # ...
        params: dict = None,
    ):
        # ...
        if params is None:
            params = {}

        # ...
        self.params = params

```
クエリパラメータをセットするため、`HTTPRequest`クラスに`params`属性を追加しました。

### `study/henango/server/worker.py`
#### 58-67行目

```python
            # pathにマッチするurl_patternを探し、見つかればviewからレスポンスを生成する
            for url_pattern, view in URL_VIEW.items():
                match = self.url_match(url_pattern, request.path)
                if match:
                    request.params.update(match.groupdict())
                    response = view(request)
                    break

            # pathにマッチするurl_patternが見つからなければ、静的ファイルからレスポンスを生成する
            else:
                # ...
```

今までは単純に`URL_VIEW`辞書のキーとpathが文字列として一致するかどうかでviewを検索していましたが、今回からは判定ロジックが複雑になるため`self.url_match()`というメソッドを新しく作成してその中で判定するようにしました。

`url_match()`の詳細は次に説明しますが、この関数はurl_patternがpathにマッチした場合は正規表現でお馴染みの`re`の`Match`オブジェクトを返します。
URLパラメータが含まれている場合は`.groupdict()`メソッドを実行することでパラメータが辞書で取得できます。

マッチしなかった場合は`None`を返すので、if文は実行されません。


また、`python`以外が母国語の方は馴染みがないかもしれませんが、ここでは`for-else`文を使っています。
`for`句が最後まで実行された（＝途中でbreakが呼ばれなかった）場合のみ、`else`句が実行されます。

### 174-178行目
```python
    def url_match(self, url_pattern: str, path: str) -> Optional[Match]:
        # URLパターンを正規表現パターンに変換する
        # ex) '/user/<user_id>/profile' => '/user/(?P<user_id>[^/]+)/profile'
        re_pattern = re.sub(r"<(.+?)>", r"(?P<\1>[^/]+)", url_pattern)
        return re.match(re_pattern, path)
```

ここでは、`/user/<user_id>/profile`のような私達独自のURL表現のことを、本プログラムでは`URLパターン`と呼ぶことにしています。
URLパターンは私達独自の記法ですので、そのままではpythonはpathとマッチするかどうかを判定できません。

ですので、まずはURLパターンを正規表現パターンに変換してpythonで扱えるようにしてから、マッチング判定をしています。
`(?P<name>pattern)`は名前付きグループといって、マッチしたグループを後から`.groupdict()`などを使って名前で取り出すことができます。

参考）https://docs.python.org/ja/3/howto/regex.html#non-capturing-and-named-groups

## 動作確認
では、まずは動作確認してみましょう。

サーバーを再起動したら、Chromeで`http://localhost:8080/user/123/profile`へアクセスしてみましょう。

次のような画面が表示されれば成功です。

![](https://storage.googleapis.com/zenn-user-upload/povdssdupnaeue6duipr7ka98ddh)

うまく表示された人は、`123`の部分を変えてみて、URLに合わせて画面表示が変わることも確認しておきましょう。

# リファクタリング STEP1 URLマッチング判定
最近リファクタリングしたばっかりですが、`worker.py`の責務がまた増えて複雑になってしまったので、またリファクタリングしておきましょう。
これぐらいの規模になると、機能追加とリファクタリングはセットになってきますね。

pathから目当てのviewを取得する処理（＝**URL解決**）だけでかなりのボリュームをしめているので、これを外部モジュールに切り出してスリムにしていきましょう。

まずは、pathとURLパターンのマッチング判定を外部へ切り出します。
色々なやり方がありますが、本書ではpathとviewの組み合わせの情報をオブジェクト化し、そのオブジェクト内にマッチング判定機能も持たせてしまうことにします。

------

# 続きはBookで！

[チャプター「URLパラメータを受け取れるようにする」](https://zenn.dev/bigen1925/books/introduction-to-web-application-with-python/viewer/url-resolver)  
