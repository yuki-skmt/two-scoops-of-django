# Two Scoops of Django 3.x

## 3. Djangoプロジェクトのレイアウト
- プロジェクトのレイアウトは、Djangoのコアな開発者の間でも、何がベストプラクティスかについて意見が分かれる分野の一つである。
- [PACKAGE TIP] Djangoプロジェクトテンプレート
  - 本章で説明するパターンに沿っていて、かつDjangoプロジェクトを促進させるのに役立つプロジェクトテンプレートは数多くある。
  - ここでは、プロジェクトを新規に作成する際に役立ちそうなリンクをいくつか紹介する。
- github.com/pydanny/cookiecutter-django
  - 本章で紹介されている。
- github.com/grantmcconnaughey/cookiecutter-django-vue-graphql-aws
  - 本章で紹介されている。
- djangopackages.org/grids/g/cookiecutters/ 
  - cookiecutter テンプレートの代替になるリスト。

### 3.1 Django 3 のデフォルトプロジェクトのレイアウト
- `startproject` や `startapp` を実行したときに作成される、デフォルトのプロジェクトレイアウトを見ていく。

```terminal
django-admin startproject mysite
cd mysite
django-admin startapp my_app
```

- プロジェクトのレイアウトは以下のようになる。

```
mysite/
├── manage.py
├─┬ my_app
│ ├── __init__.py
│ ├── admin.py
│ ├── apps.py
│ ├─┬ migrations
│ │ └── __init__.py
│ ├── models.py
│ ├── tests.py
│ └── views.py
├── mysite
├── __init__.py
├── asgi.py
├── settings.py
├── urls.py
└── wsgi.py
```

- Django のデフォルトのプロジェクトレイアウトにはいくつかの問題がある。
  - チュートリアルには便利だが、実践的なプロジェクト開発においては、 それほど便利ではない。
  - 本章の残りでその理由を解説する。

### 3.2 本書で推奨されるプロジェクトレイアウト
- `startproject` コマンドで生成されたものをベースとした、改良された二層構造のアプローチを採用している。
- 最上位レベルのレイアウトは以下のようになる。

```
<repository_root>/
├── <configuration_root>/
└── <django_project_root>/
```

#### 3.2.1 第1層：リポジトリのルート
- \<repository_root>ディレクトリは、プロジェクトの絶対的なルートディレクトリである。
- \<django_project_root> と \<configuration_root> に加えて、README.md、docsディレクトリ、manage.py、.gitignore、requirements.txt ファイルなどのコンポーネントや、プロジェクトのデプロイや実行に必要な高レベルのファイルを含む。

- [TIP] 一般的な慣習との差異
  - 開発者の中には、\<django_project_root>をプロジェクトの\<repository_root>にまとめることを好む人もいる。

#### 3.2.2 第2層：Djangoプロジェクトルート
- \<django_project_root>/ ディレクトリは、実際のDjangoプロジェクトのルート。
- 設定以外のPythonコードファイルは、このディレクトリ以下に配置する。
- `startproject` を使う場合は、リポジトリのルートからコマンドを実行する。
- このコマンドで生成された Djangoプロジェクトがプロジェクトルートになる。

#### 3.2.3 第2層：設定ルート
- \<configuration_root>/ ディレクトリには、settings モジュールとベースの URLConf (urls.py) が置かれる。
- これは \_\_init\_\_.py モジュールを含む有効なPythonパッケージである必要がある。
  - Python 3 でも、\_\_init\_\_.py が含まれていないと、 \<configuration_root>/ は python パッケージとして認識されない。
- `startproject` を使った場合、設定ルートは最初、Django プロジェクトルートの中にあるので、リポジトリルートに移動する必要がある。
  - 設定ルートにあるファイルは django-admin startproject コマンドで生成されるものの一部。

### 3.3 プロジェクトのレイアウト例
- よくある例として、シンプルな評価サイトを考えてみる。
  - アイスクリームのブランドやフレーバーを評価するウェブアプリケーション「Ice Cream Ratings」を作成するとする。
  - このようなプロジェクトでは、次のようにレイアウトする。

```
icecreamratings_project
├─┬ config/
│ ├── settings/
│ ├── __init__.py
│ ├── asgi.py
│ ├── urls.py
│ └── wsgi.py
├── docs/
├─┬ icecreamratings/
│ ├── media/ # Development only!
│ ├── products/
│ ├── profiles/
│ ├── ratings/
│ ├── static/
│ └── templates/
├── .gitignore
├── Makefile
├── README.md
├── manage.py
└── requirements.txt
```

- \<repository_root>に相当する`icecreamratings_project`ディレクトリには、以下の表にまとめたようなファイルやディレクトリが存在する。

|ファイル・ディレクトリ名|目的|
|:--|:--|
|.gitignore|Gitが無視すべきファイルやディレクトリを列挙する。|
|config/|プロジェクトの\<configuration_root>で、プロジェクト全体の設定、urls.py、wsgi.py 等のモジュールが置かれる|
|Makefile|簡単なデプロイタスクとマクロを含む。より複雑なデプロイには、Invoke、Paver、Fabricなどのツールを使用するとよい。|
|manage.py|これを入れたままにする場合、内容を変更してはいけない。|
|README.md and docs/|開発者向けのプロジェクトドキュメント。|
|requirements.txt|プロジェクトに必要になるパッケージのリスト|
|icecreamratings/|\<django_project_root>に相当する。|

このプロジェクトレイアウトでは、プロジェクトを俯瞰的に見ることができ、他の開発者や、開発者ではない人とも簡単に作業ができる。
例えば、デザイナーに特化したディレクトリをルートディレクトリに作ることもできる。

\<django_project_root>に相当する`icecreamratings_project/icecreamratings`ディレクトリには、以下の表にまとめたようなファイルやディレクトリを配置する。

|ファイル・ディレクトリ名|目的|
|---|:--|
|media/|開発用：ユーザがアップロードした写真など、ユーザが作成した静的メディアファイル (※)|
|products/|アイスクリームのブランドを管理・表示するアプリ|
|profiles/|ユーザーのプロフィールを管理・表示するアプリ|
|ratings/|ユーザーの評価を管理するアプリ|
|static/|CSS、JavaScript、画像ファイルなど、ユーザが作成したものではない静的メディアファイル (※)|
|templates/|サイト全体のDjangoテンプレートを配置する|

- (※)大規模プロジェクトの場合は、別の静的メディアサーバでホスティングされる。

- [TIP] 静的メディアのディレクトリ名に関する規約
  - 上記の例では、Djangoの公式ドキュメントの規定に従って、 (ユーザが生成しない) 静的メディアのディレクトリに static/ を使っている。
  - 混乱するようであれば、assets/ や site_assets/ としても問題ない。
  - ただし、STATICFILES_DIRS の設定を適切に更新すること。

### 3.4 Virtualenvは何処へ？
- プロジェクトディレクトリやサブディレクトリのどこにもvirtualenv ディレクトリが含まれていないことに注意すること(これは完全に意図的なもの)。
- 別のディレクトリに、すべてのPythonプロジェクトのためのすべてのvirtualenvを保管する場所を作成すること。
  - すべての環境を1つのディレクトリに置き、すべてのプロジェクトを別のディレクトリに置くことが好ましい。

- Mac や Linux の例

```terminal
~/projects/icecreamratings_project/
~/.envs/icecreamratings/
```

- Windowsの例

```terminal
c:projects\icecreamratings_project\
c:envs\icecreamratings\
```

- virtualenvwrapper (Mac/Linux) またはvirtualenvwrapper-win (Windows) を使用している例
  - ディレクトリはデフォルトで「~/.virtualenvs/」となり、以下のように配置される。

```terminal
~/.virtualenvs/icecreamratings/
```

- 仮想環境 (virtualenv) の内容をバージョン管理する必要はない。
  - なぜなら、仮想環境内のソースコードを直接編集することはなく、仮想環境はすでにrequirements.txtですべての依存関係を把握いるからである。
  - ただし、requirements.txtはバージョン管理下に置いておく必要があることに注意。
- [警告] 環境ディレクトリを公開リポジトリにアップロードしないこと。
  - venv等の環境ディレクトリをGitHubの公開リポジトリに含めてはならない。
  - .node_modules についても同様。
  - このような失敗をしないために、すべてのプロジェクトのルートディレクトリに .gitignoreファイルを配置すること。これにより、gitに追加するべきでないファイルやディレクトリがコミットに含まれることがなくなる。

#### 3.4.1 現在の依存関係のリストアップ
- virtualenvで使用している依存関係のバージョンを判断するのが難しい場合、コマンドラインで次のように入力して依存関係をリストアップすることができる。

```terminal
$ pip freeze
```

MacやLinuxでは、これをパイプでrequirements.txtファイルに書き出すことができる。

```terminal
$ pip freeze > requirements.txt
```

### 3.5 'startproject' を超えていけ
- Django の`startproject` コマンドでは、シンプルな Django プロジェクトのテンプレートを作成することができる。
  - しかし時間が経つにつれ、プロジェクト周りの制御 (デプロイやフロントエンドなど) は複雑になっていく。
  - 多くの人は、`startproject`の限界にすぐぶつかり、より強力なプロジェクトテンプレート作成ツールを求めるようになる。
  - そこで著者は、Djangoプロジェクトのボイラープレートを生成するための高度なプロジェクトテンプレートツールとして、Cookiecutter を使用することにした。

#### 3.5.1 Cookiecutterによるプロジェクト・ボイラープレートの生成
- Cookiecutterの仕組みを以下に記す。
  1. まず、一連の値 (例: project_name の値) を入力するように求められる。
  2. 次に、入力された値に基づいて、すべてのボイラープレートプロジェクトファイルを生成する。

- まず、Cookiecutter の公式ドキュメントに従い、Cookiecutter をインストールすること。

#### 3.5.2 Cookiecutter Django で開始プロジェクトを生成する
- ここでは、Cookiecutter DjangoからDjango 3のボイラープレートを生成する方法を紹介する。

```
cookiecutter https://github.com/pydanny/cookiecutter-django
You've downloaded /home/quique/.cookiecutters/cookiecutter-django
,→ before. Is it okay to delete and re-download it? [yes]: no
Do you want to re-use the existing version? [yes]: yes
project_name [My Awesome Project]: icecreamratings
project_slug [icecreamratings]:
description [Behold My Awesome Project!]: Support your Ice Cream
,→ Flavour!
author_name [Daniel Feldroy]:
\<snip for brevity>
```

- 上記のように全ての値を入力すると、Cookiecutterを起動したディレクトリにプロジェクト用のディレクトリが作成される。
  - 上記の例に沿って値を入力した場合、ディレクトリの名前は`icecreamratings`になります。
  - 出来上がったプロジェクトファイルは、前述のレイアウト例とほぼ同じものになる。
  - このプロジェクトには、設定、requirementsファイル、スタータードキュメント、スターターテストスイートなどが含まれる。

- [TIP] その他のファイルについて
  - Cookiecutter Django は、この章の前半で説明した基本的なプロジェクトレイアウトよりもはるかに優れている。
  - これは著者がプロジェクトに使用する究極のDjangoプロジェクトテンプレートであり、Djangoが提供するデフォルトの startproject テンプレートよりもずっと多機能である。

### 3.6 'startproject' の代替になる他の方法
- 前述したように、正しい方法はひとつではない。
  - プロジェクトが本書のレイアウトと異なることは問題ではないが、適切に階層化されていること、プロジェクトの各要素 (ドキュメント、テンプレート、アプリ、設定など) の場所がルートのREADME.md に記述されていることが必須条件となる。
  - Cookiecutter Django のforkして調査したり、他の Cookiecutter を搭載した Django プロジェクトテンプレートをオンラインで検索したりすることを推奨する。
  - 他の人のプロジェクトテンプレートを研究することで、様々なトリックを学ぶことができる。
- [TIP] Cookiecutter-django-vue-graphql-aws
  - Cookiecutter テンプレートの中で優れたものとして、Grant Mc-Connaughey 氏の `cookiecutter-django-vue-graphql-aws` が挙げられる。
  - このテンプレートは、我々にとって馴染み深い技術 (Django、GraphQL、Vue、AWS Lambda + Zappaなど) を簡潔にまとめている。
  - 文書化されていないものは、明示的に命名されている。
  - [ここ](github.com/grantmcconnaughey/cookiecutter-django-vue-graphql-aws)から参照できる。

### 3.7 まとめ
- 本章では、基本的なDjangoプロジェクトのレイアウトに対するアプローチを紹介した。
- プロジェクトレイアウトは、Djangoの分野の中でも、開発者やグループごとに方法が大きく異なる部分である。
- 小規模なチームではうまくいく方法でも、リソースが分散している大規模なチームではうまくいくとは限らないことに留意されたし。
- いずれにせよ、プロジェクトレイアウトについては明確に文書化する必要がある。
