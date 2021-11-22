# Two Scoops of Django 3.x

## 4. Djangoアプリ設計の基礎知識
- Django開発の初心者は、Djangoにおける「アプリ」という用語の理解に苦しむことがよくある。
  - そこで、Djangoアプリの設計に入る前にいくつかの定義を確認する。
- **Djangoプロジェクト**とは、 Djangoのフレームワークを使ったWebアプリケーションのことである。
- **Djangoアプリ**とは、プロジェクトの一機能として設計されたライブラリのことである。
- Djangoプロジェクトは、複数のDjangoアプリで構成されている。
- Djangoアプリの中には、プロジェクトの内部にあって再利用されないものもあれば、サードパーティのDjangoパッケージもある。
- **INSTALLED_APPS**とは、Djangoプロジェクトが利用することのできるDjangoアプリの一覧のことである。
- **サードパーティのDjangoパッケージ**とは、パッケージ化された、プラグインとして利用可能 (再利用可能)なDjangoアプリのことである。

### 4.1 Djangoアプリ設計の黄金律
> "優れたDjangoアプリを開発し、維持していく秘訣は、Douglas McIlroyによる次のUnix哲学に従うことだ。「一つのことをするプログラムを書き、それをうまくやる」"
> ーJames Bennett (Django のコア開発者)

- それぞれのアプリは自分自身のタスクに集中しなければならない。
  - アプリについて、(適切な長さの)一文で説明できない場合や、説明の過程で「and (そして…)」を何度も言わなければならない場合は、アプリが大きすぎて分割すべきだと言える。

#### 4.1.1 アプリの実例
- 架空のアイスクリームショップ「Two Scoops」のWebアプリケーションを作っていると仮定する。
- 一例として、プロジェクト内のアプリは以下のようになる。
  - アイスクリームのフレーバーを記録し、ウェブサイトに掲載するフレーバーアプリ。
  - Two Scoopsの公式ブログ用のブログアプリ。
  - お店のイベント情報をウェブサイトに掲載するイベントアプリ。
- これらのアプリは、それぞれがひとつの役割を担っている。
- アプリ同士に関連性はあるものの、1つのアプリで全てをまかなうよりも、3つの機能に特化したアプリがある方が望ましいことがわかる。
- 将来的には、こんなアプリでサイトを拡張していくかもしれない。
  - アイスクリームの通信販売を行うためのショップアプリ。
  - アイスクリーム食べ放題のプレミアムイベントのチケットを販売するチケットアプリ。
    - イベントアプリとチケットアプリを分割していることに注目してほしい。
    - イベントアプリを拡張してチケットを販売するのではなく、チケットを必要としないイベントがほとんどであることや、イベントカレンダーやチケット販売は、サイトの成長に伴い複雑なロジックを含む可能性があることから、別のチケットアプリを作成した。

### 4.2 Djangoアプリの命名について
- 退屈でつまらなく、わかりやすい名前を使うことを推奨する。
- 具体的には、flavors (フレーバー/味)、animals (動物)、blog (ブログ)、polls (投票)、estimates (見積もり)、finances (財務) など…
  - 可能な限り一語であることが望ましい。
- 覚えやすい名前にすることで、プロジェクトのメンテナンスが容易になる。
- 「アプリの名前はアプリのメインモデルの複数形でなければならない」というのが一般的なルールとしてあるが、このルールには多くの例外がある。(blogはその代表例)
- 命名の際には、「URLをどのように表示させたいか」も考慮する必要がある。
  - サイトのブログが http://www.example.com/weblog/ に表示されるようにしたい場合は、メインモデルが Post であっても、アプリの名前を blog、posts、blogposts ではなく weblog とすることを検討すべき。
  - これにより、どのアプリがサイトのどの部分に対応しているかがわかりやすくなる。
- PEP 8のパッケージ名に準拠した命名を使用することを推奨する。
  - 数字、ダッシュ、ピリオド、スペース、特殊文字を含まない、短い、すべて小文字の名前。
  - 可読性を保つために必要であれば、アンダースコアを使って単語を区切ってもよい (が、推奨されない)。

### 4.3 迷ったらアプリを小さくする
- アプリのデザインを完璧にしようと考えすぎる必要はない。時にはアプリを作り直したり、分割し直したりしなければならないこともある。
- アプリは極力小さくしておくべき。
  - 巨大なアプリがいくつかあるより、小さなアプリがたくさんあるほうがいい。

### 4.4 アプリに含まれるモジュールについて
- このセクションでは、アプリに含まれるPythonモジュールについて、一般的な例と一般的でない例について解説する。

#### 4.4.1 一般的なアプリのモジュール
- 世のDjangoアプリの99%で見られる、一般的な例。
- スラッシュ (/) で終わるモジュールはPythonパッケージを表し、1つ以上のモジュールを含むことができる。

```
# Common modules
scoops/
├── __init__.py
├── admin.py
├── forms.py
├── management/
├── migrations/
├── models.py
├── templatetags/
├── tests/
├── urls.py
└── views.py
```

- アプリを構築する際、Djangoアプリにおけるモジュール名の慣習に従うことで、自分自身や他の人の振る舞いを規定し、コードの読解を容易にすることができる。
- PythonやDjangoは十分に柔軟性があるので、この慣例に違反する名前を付けることもできるが、そうした場合には問題が生じることになる。
  - 技術的な観点での問題がすぐに起こるわけではなく、しばらくしてから標準的でないモジュール名が不快な経験をもたらす原因となるだろう。

#### 4.4.2 一般的でないアプリのモジュール

```
# Common modules
scoops/
├── api/
├── behaviors.py
├── constants.py
├── context_processors.py
├── decorators.py
├── db/
├── exceptions.py
├── fields.py
├── factories.py
├── helpers.py
├── managers.py
├── middleware.py
├── schema.py
├── signals.py
├── utils.py
└── viewmixins.py
```

|ファイル・ディレクトリ名|目的|
|---|:--|
|api/|apiを作成する際に必要な様々なモジュールを配置する|
|behaviors.py|モデルミックスインを配置する。(6.7.1 参照)|
|constants.py|アプリ単位の設定値を配置する。設定値が多い場合はさらに分割するとプロジェクトの見通しを良くなる。|
|decorators.py|デコレーターを配置する。(9.3 参照)|
|db/|カスタムモデルのフィールドやコンポーネントを配置する。|
|fields.py|一般的にフォームフィールドに使用される。db/パッケージを作成するほどのフィールドがない場合、モデルに使用されることもある。|
|factories.py|テストデータのファクトリを配置する。(24.3.5 参照)|
|helpers.py|ヘルパー関数を配置する。ViewやModelは肥大化する傾向にあるので、それらから切り出して軽量化する。utils.pyと命名する場合もある。|
|managers.py|(Modelが大きくなりすぎた場合)カスタムモデルマネージャーをこのモジュールに配置する。|
|schema.py|GraphQL APIのビハインドコードを配置する。|
|signals.py|カスタムシグナルを配置する。(非推奨)|
|utils.py|helpers.pyと同義。|
|viewmixins.py|View Mixinを配置する。(Viewを軽量化できる)|

- 上記のモジュールはすべて「アプリレベル」にフォーカスしたものである。

### 4.5 Ruby on Railsスタイルのアプローチ
- Ruby on Rails (略してRails) は、Djangoとほぼ同年代に成功した、Rubyを使ったアプリケーションフレームワークである。
  - 有名なRailsプロジェクトには、GitHub、GitLab、Coinbase、Stripe、Squareなどがある。
- DjangoとRails、そしてPythonとRubyの間には多くの共通点があり、そのアプローチを検討する価値がある。

#### 4.5.1 サービスレイヤ
- Django でコーディングするとき、ビジネスロジックをどこに置くべきか悩むのは初心者にありがちなこと。
  - 典型的な例は、複数のモデルやアプリにまたがるユーザオブジェクトや関連データを作成すること。
- テーマパーク「Icecreamlandia」のチケット販売システムの一例。
  1. `create_user()`というメソッドでユーザーレコードを作成する。
  2. `create_user()`内部で、ユーザーの写真がファイルホスティングシステムにアップロードされる。
  3. さらに、`create_user()`から`create_ticket()`というメソッドが呼ばれ、チケット・オブジェクトが生成される。
  4. `create_ticket()` は、サードパーティのサービスであるゲスト用の「Icecreamlandia」チェックインアプリにAPI呼び出しを行う。

- これらのアクションの典型的な配置は、UserとTicketに割り当てられたマネージャのメソッドに分散している。
- また、代わりにクラスメソッドを使用する場合もある。

```python:typical_business_logic_placement.py
class UserManager(BaseUserManager):
    """In users/managers.py"""
    def create_user(self, email=None, password=None, avatar_url=None):
        user = self.model(
            email=email,
            is_active=True,
            last_login=timezone.now(),
            registered_at=timezone.now(),
            avatar_url=avatar_url
        )
        resize_avatar(avatar_url)
        Ticket.objects.create_ticket(user)
        return user

class TicketManager(models.manager):
    """In tasks/managers.py"""
    def create_ticket(self, user: User):
        ticket = self.model(user=user)
        send_ticket_to_guest_checkin(ticket)
        return ticket
```

- 上記方法は有効であるものの、ユーザに関するコードの中にチケットに関するコードが埋め込まれており、二つのドメインが密結合しているという問題を抱えている。
- そこで、関心の分離を実現するために新しいレイヤを追加し、これをサービスレイヤと呼ぶ。

```
# Service layer example
scoops/
├── api/
├── models.py
├── services.py # Service layer location for business logic
├── selectors.py # Service layer location for queries
└── tests/
```

- サービスレイヤのコードは service.py と selectors.py に配置する。

```python:service_layer_business_logic_placement.py
# In users/services.py
from .models import User
from tickets.models import Ticket, send_ticket_to_guest_checkin

def create_user(email: str, password: str, avatar_url: str) -> User:
    user = User(
        email=email,
        password=password,
        avatar_url=avatar_url
    )
    user.full_clean()
    user.resize_avatar()
    user.save()

    ticket = Ticket(user=user)
    send_ticket_to_guest_checkin(ticket)
    return user
```

- サービスレイヤのメリット
  1. 処理が短くなる。(サンプルコードでは17行→12行)
  2. 各モデル (ユーザーやチケット) が疎結合なので、修正が容易にできる。
  3. 関心の分離により、個々のコンポーネントの機能テストが容易になる。
  4. 型アノテーションの追加が典型的アプローチよりも容易になる。

- サービスレイヤのデメリット
  1. 導入にコストがある。単純な置き換えで済むプロジェクトは非常に稀である。
  2. 小規模なプロジェクトでは、不必要に複雑化してしまう危険性が高い。
  3. 複雑なプロジェクトでは、サービスレイヤのコードが肥大化する傾向にある。
  4. selectors.py をどこでも使えるようにすると、シンプルなORMクエリを作るたびに余分なステップが必要になる。
  5. Djangoは、モデル自体にインポート可能なビジネスロジックを配置することでその永続性を担保しているが、サービスレイヤはその機能を排除する。

- サービスレイヤへのさらなる反論や、従来のDjangoツールでビジネスロジックを整理する方法の説明は、長年Djangoのメンテナを務めている James Bennett 氏のブログや、Django REST Frameworkの創始者である Tom Christie 氏の記事で読むことができる。
  - b-list.org/weblog/2020/mar/16/no-service/
  - b-list.org/weblog/2020/mar/23/still-no-service/
  - dabapps.com/blog/django-models-and-encapsulation/

- 本書の著者にとって、サービスレイヤは新しいものではなく長年見てきたものであり、その経験から言うとサービスレイヤを使ったプロジェクトでの失敗がやや多くなってきている。
- このアプローチにメリットがないというわけではないが、抽象度を上げる努力に見合うメリットが得られないことも多い。

#### 4.5.2 大規模な単一アプリのプロジェクト
- 全てのコードを単一のアプリにまとめる手法。
  - 小規模なプロジェクトでは一般的だが、大規模なプロジェクトでは一般的にこのパターンをとらない。
- マイグレーションが容易になり、テーブル名がシンプルなパターンになる。
  - Rails などの他のフレームワークはこの手法を採用し、成功している。
    - ただ、Rails等はこのパターンに従うよう設計されているが、 Djangoはこの設計に最適化されていない。
    - Djangoでこの手法を使う場合には、ドキュメントにほとんど記述されていないようなパターンの経験や専門知識が必要となる。

- 全てを巨大な models.py, views.py, tests.py, urls.py モジュールにまとめるため、最終的にはプロジェクトが複雑化して崩壊するか、モデルをドメインごとに分割するかの二択を迫られる。
  - e.g.) ユーザに関するものはmodels/users.pyに、チケットに関するものはmodels/tickets.pyに配置する。
- このアプローチは慎重に行うべき。
  - これは、いくつかのプロジェクトを経験しDjangoに熟練した者のみ手を出すべきである。

- [警告] Djangoに独自色の強いアーキテクチャを持ち込むことの危険性
  - 独自色の強すぎるアーキテクチャを採用することにはデメリットがある。
  - 新しい開発者が参加するたびに新しいパラダイムに学習する必要があるという点で、保守が難しいプロジェクトになってしまう。
  - Djangoの魅力は、慣習に基づく部分も大きいのだが、奇抜な設計はそれを壊してしまう。

### 4.6 まとめ
- この章では、Djangoアプリの設計技術について解説した。
- 具体的には、Djangoアプリはシンプルで覚えやすい名前を持つべきで、かつ自らのタスクにしっかりとフォーカスするような設計にすべきである。
  - アプリが複雑すぎるようであれば、小さなアプリに分割すべき。
  - 設計を正しく行うには鍛錬と努力が必要だが、その価値は十分にある。
- また、アプリを構築するための別のアプローチや、アプリ内でのロジックの呼び出し方についても解説した。