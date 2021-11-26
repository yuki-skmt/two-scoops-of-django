# Two Scoops of Django 3.x

## 9. FBVのベストプラクティス
- 関数ベースのビューは世界中の開発者によく使用されている。
- クラスベースのビューよりもシンプルに実装できる点が魅力である。

### 9.1 FBVの利点
- FBVのシンプルさは、コードの再利用を犠牲にしている。FBVには、スーパークラスを継承する機能がない。
- FBVはより関数的な性質を持っている。
- FBVを書くときには、以下のガイドラインに従うこと。
  - ビューのコードは少ない方が良い。
  - ビューの中でコードを繰り返してはいけない。
  - ビューには表示に関わるロジックのみ書く。
    - 訳者註: 原文は"handle presentation logic" (プレゼンテーションロジックを扱う)となっている。ビジネスロジックを書かないということが主旨と思われる。

- ビジネスロジックは可能な限りモデルの中に置くようにする。可能な限りモデルに、必要に応じてフォームに置くようにすること。
  - ビューはシンプルにしましょう。
  - ビューはカスタマイズされた403、404、500エラーのハンドラを書くことに使う。
  - 複雑なネストされたifブロックは避ける。

### 9.2 HttpRequestオブジェクトの渡し方
- ビューの中でコードを再利用したいが、ミドルウェアやコンテキストプロセッサのようなグローバルなアクションには結び付けたくない場合がある。
  - 訳者註: コンテキストプロセッサとは、全てのテンプレートで使う変数を定義できるDjangoの機能のこと。
- 本書の冒頭では、プロジェクト全体で使用できるユーティリティ関数を作成することを推奨している。
- 多くのユーティリティ関数では、 `django.http.HttpRequest`オブジェクトから属性を受け取り、データを収集したり操作を行ったりする。
- リクエストオブジェクトそのものを第一引数とすることで、より多くのメソッドの引数をシンプルにできるということである。
- つまり、HttpRequestオブジェクトを渡すだけでよくなり、関数やメソッドの引数を管理するための認知的な負荷が少なくなる。

```python
from django.core.exceptions import PermissionDenied
from django.http import HttpRequest

def check_sprinkle_rights(request: HttpRequest) -> HttpRequest:
    if request.user.can_sprinkle or request.user.is_staff:
        return request

    # Return a HTTP 403 back to the user
    raise PermissionDenied
```

- check_sprinkle_rights() 関数は、ユーザの権限を簡単にチェックして `django.core.exceptions.PermissionDenied` 例外を発生させ、カスタムHTTP403 ビューをトリガーする。
- 任意の値やNoneオブジェクトではなく、HttpRequestオブジェクトを返していることに注意すること。
- Pythonは動的型付け言語であるため、以下のようにHttpRequestに追加の属性を付けることができるメリットがある。

```python
from django.core.exceptions import PermissionDenied
from django.http import HttpRequest, HttpResponse

def check_sprinkles(request: HttpRequest) -> HttpRequest:
    if request.user.can_sprinkle or request.user.is_staff:
        # By adding this value here it means our display templates
        # can be more generic. We don't need to have
        # {% if request.user.can_sprinkle or request.user.is_staff %}
        # instead just using {% if request.can_sprinkle %}
        request.can_sprinkle = True
        return request
    
    # Return a HTTP 403 back to the user
    raise PermissionDenied
```

- 以下のように使用する。

```python
# sprinkles/views.py
from django.shortcuts import get_object_or_404
from django.shortcuts import render
from django.http import HttpRequest, HttpResponse

from .models import Sprinkle
from .utils import check_sprinkles

def sprinkle_list(request: HttpRequest) -> HttpResponse:
    """Standard list view"""

    request = check_sprinkles(request)

    return render(request,
        "sprinkles/sprinkle_list.html",
        {"sprinkles": Sprinkle.objects.all()})

def sprinkle_detail(request: HttpRequest, pk: int) -> HttpResponse:
    """Standard detail view"""
    
    request = check_sprinkles(request)

    sprinkle = get_object_or_404(Sprinkle, pk=pk)

    return render(request, "sprinkles/sprinkle_detail.html",
        {"sprinkle": sprinkle})

def sprinkle_preview(request: HttpRequest) -> HttpResponse:
    """Preview of new sprinkle, but without the
    check_sprinkles function being used.
    """

    sprinkle = Sprinkle.objects.all()

    return render(request,
        "sprinkles/sprinkle_preview.html",
        {"sprinkle": sprinkle})
```

- この方法の更なる利点は、CBVに簡単に統合できること。

```python
from django.views.generic import DetailView

from .models import Sprinkle
from .utils import check_sprinkles

class SprinkleDetail(DetailView):
    """Standard detail view"""

    model = Sprinkle

    def dispatch(self, request, *args, **kwargs):
        request = check_sprinkles(request)
        return super().dispatch(request, *args, **kwargs)
```

- [TIP] CBV でのリクエストオブジェクトの受け渡し
  - 引数を単一にすることの欠点は、`'pk'`、 `'flavor'`、`'text'`などの具体的な引数を使う場合に比べて、関数の目的を一目で理解しづらくなることである。
  - よって、このテクニックはできるだけ一般的な振る舞いをする関数に適用すること。

### 9.3 デコレータは素晴らしい
- コンピュータサイエンスの用語で、シンタックスシュガー(糖衣構文)とは、読みやすさや表現しやすさのためにプログラミング言語に追加される構文のことである。
- Pythonにおいて、デコレータは必要に迫られて追加された機能ではなく、人間が読みやすく、コードをすっきりさせるために追加された機能である。
- シンプルな関数の力とデコレータの構文的な美しさを組み合わせると、 `django.contrib.auth.decorators.login_required`デコレータのような、非常に便利で再利用可能なツールができる。
- ここでは、FBVで使用するデコレータのテンプレートのサンプルを紹介する。

```python
import functools

def decorator(view_func):
    @functools.wraps(view_func)
    def new_view_func(request, *args, **kwargs):
        # You can modify the request (HttpRequest) object here.
        response = view_func(request, *args, **kwargs)
        # You can modify the response (HttpResponse) object here.
        return response
    return new_view_func
```

- インラインコメントをつけながら詳述する。
- まず、上記のデコレータを我々の必要に応じて変更する。
  - 訳者註: このサンプルに限り、コメントを例外的に翻訳する。

```python
# sprinkles/decorators.py
import functools

from . import utils

# 先述のサンプルをベースとするデコレータ
def check_sprinkles(view_func):
    """Check if a user can add sprinkles"""
    @functools.wraps(view_func)
    def new_view_func(request, *args, **kwargs):
        # リクエストオブジェクトを操作する
        request = utils.can_sprinkle(request)

        # ビューファンクションを呼ぶ
        response = view_func(request, *args, **kwargs)

        # HttpResponseオブジェクトを返す
        return response
    return new_view_func
```

- そして、それをこのように関数にアタッチする。

```python
# sprinkles/views.py
from django.shortcuts import get_object_or_404, render

from .decorators import check_sprinkles
from .models import Sprinkle

# デコレータをビューにアタッチする
@check_sprinkles
def sprinkle_detail(request: HttpRequest, pk: int) -> HttpResponse:
    """Standard detail view"""
    
    sprinkle = get_object_or_404(Sprinkle, pk=pk)

    return render(request, "sprinkles/sprinkle_detail.html",
        {"sprinkle": sprinkle})
```

- [TIP] `functools.wraps()`とは？
  - 賢明な読者は、デコレータのサンプルでPython標準ライブラリの functools.wraps() デコレータ関数が使われていることに気づいたかもしれない。
  - これは、docstringのような重要な情報を含むメタデータを、新しくデコレーションされた関数にコピーする便利なツールである。
  - 必須ではないが、プロジェクトのメンテナンスをより簡単にしてくれる。

#### 9.3.1 デコレータの使用は慎重に
- デコレータは使い方を間違えると大変なことになる。
  - デコレータの数が多すぎるとそれ自体が難読化してしまい、複雑なCBVにも可読性の面で劣るようになる。
  - デコレータを使用する際には、ビューに設定できるデコレータの数に制限を設け、それを守るようにすること。
  - [関連動画](pyvideo.org/pycon-us-2011/pycon-2011--how-to-write-obfuscated-python.html)

#### 9.3.2 デコレータに関する追加情報
- [デコレータの説明](jeffknupp.com/blog/2013/11/29/improve-your-python-decorators-explained/)

- [デコレータのチートシート (作: Daniel Feldroy氏)](daniel.feldroy.com/python-decorator-cheatsheet.html)

### 9.4 HttpResponseオブジェクトの渡し方
- HttpRequestオブジェクトと同様に、HttpResponseオブジェクトも関数から関数へと受け渡すことができる。
- 詳細は、[Process Template Response](docs.djangoproject.com/en/3.2/topics/http/middleware/#process-template-response)。
- デコレータを使ってこのテクニックを行うことができる。

### 9.5 FBVに関するその他のリソース
- Luke Plant (Djangoのコア開発者、熱烈なFBV支持派)の記事。
  - spookylukey.github.io/django-views-the-right-way/

### 9.6 まとめ
- FBVはDjangoの世界ではまだ健在である。
- すべての関数はHttpRequestオブジェクトを受け取り、HttpResponseオブジェクトを返すことを覚えておけば、それを使うことができる。
- HttpRequestとHttpResponseを変更する関数を活用して、デコレータを構築することもできる。
- 本章で学んだFBVについての知見は次の章で説明するCBVにも適用できることを確認し、本章を終わる。






