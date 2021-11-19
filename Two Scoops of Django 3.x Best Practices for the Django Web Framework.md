# Two Scoops of Django 3.x

## 序章

### 本書の目的

Djangoについての明文化されていないヒント、トリック、そして一般的なノウハウのすべてを記述すること。

### 推奨事項
Djangoの公式ドキュメントとは異なり、この本では特定のコーディングスタイル、パターン、およびライブラリの選択を推奨する。以下は数年のDjangoの経験をもとに蓄積した「個人的」な推奨事項である。

- ベストプラクティス
- 特定のツールやライブラリに対する好み
- アンチパターンと見なされる一般的な慣行

### 『Two Scoops of Django』の意味
- 二すくいのDjango
  - 著者がアイスクリーム好き。
- コードサンプルにアイスクリームのアナロジーが使われている。

### 始める前に
- この本はチュートリアルではなく、初めてDjangoを使用する者には難易度の高い部分も多い。
- Pythonへの理解も必須となる。
- 以下のいずれかを修了していることを推奨する。
  - [公式のDjangoチュートリアル](https://docs.djangoproject.com/en/3.2/intro/tutorial01/)
  - Djangoクラッシュコースチュートリアル (404)
  - 類似の初学者向けコンテンツ
- OOP (オブジェクト指向プログラミング)の経験もあるとよい。

#### 対象バージョン
- Django3.xおよびPython3.8または3.9を対象としている。
  - Django2.2などでは機能しない。
- 機能互換性については保証していないが、少なくとも本書の一般的なアプローチのほとんどは、Django1.0以降のすべてのバージョンに対応している。
- Pythonのバージョンに関しては、Python3.8で動作確認されている。
  - コード例の大半は、Python3.7.xおよび3.6.xで動作する。

#### 各章は独立している
- 本書では、各章が独立するように意図的に構成されている。
  - 特定のトピックに関する章を簡単に参照できる。
  - さまざまなコーディングシナリオを説明および支援する、有用で分離されたスニペットと考えること。

### 本書で使用されている規則
- コード例に関しては灰色背景に黒字で記述する。
  - コードをコンパクトに保つため、PEP8のコメントや行間隔に関する規則に違反することがある。
- [コード例はここで入手できる。](github.com/feldroy/two-scoops-of-django-3.x.)
- 避けるべきコード例 (アンチパターン)に関しては黒背景に白字で記述する。
- その他の表記規則
  - コード、コマンド等の表記には固定幅フォントを使用する。
  - ファイル名には斜体 (イタリック)を使用する。
  - 新しい用語、重要な用語には太字を使用する。
- TIPs、警告、推奨パッケージなどの記述には固有のボックスを使用する。

### コアコンセプト
Djangoでの開発において、常に念頭に置くべき考え方。

#### Keep It Simple, Stupid
- 不必要な複雑性によって、新しい機能を追加したり、古い機能を維持したりすることが難しくなる。
- 最も単純な解決策を試みること。ただし、誤った仮定に基づき過度に単純化された実装をしないように注意すること。

#### Fat Models, Utility Modules, Thin Views, Stupid Templates
太ったモデルとモジュール、薄いビュー、愚かなテンプレート
- ロジックをどこに記述するか迷った際に念頭に置くべきアプローチ。
  - ViewとTemplateは極力簡潔に保つべき。
  - Template TagやFilterに関しても、可能な限り最小限のロジックにするべき。
- コードがより明確になり、自己文書化が進み、重複が少なくなり、再利用性が向上する。

#### デフォルトから始める
- サードパーティのテンプレートエンジン、ORM、非リレーションDBなどで代替することを検討する前に、まず標準のDjangoコンポーネントを使用した実装を試みること。
- 標準Djangoコンポーネントによる実装で問題が発生した場合は、コンポーネントを交換する前にすべての可能性を探ること。

#### Djangoの設計思想に精通する
- Djangoのドキュメントを定期的に読み設計思想を理解することで、Djangoが特定の制約やツールを提供する理由を理解することができる。
- Djangoは単にビューを提供するだけの手段ではない。短時間で保守性の高いプロジェクトをまとめあげることを目指して設計されている。
- [参考文献](https://docs.djangoproject.com/en/3.2/misc/design-philosophies/)

#### The Twelve-Factor App
- Webアプリケーション設計をする際の包括的なアプローチをまとめたもの。
  - デプロイしやすく、スケールしやすいアプリケーションを構築するための方法論。
  - 本書のアプローチとも通底している。
- [参考文献](https://12factor.net/)

### 本書執筆上のコンセプト
- 省略。

## 1. コーディングスタイル
### 1.1 コードをリーダブルにすることの重要性
> プログラムは、人が読むために書かれたものでなければならず、機械が実行するために書かれたものであってはならない。

- コードは書かれるよりも読まれることの方が多い。
- 個々のコードブロックは、書くのは一瞬でも、デバッグするのに数分～数時間かかる。
- 理解しやすいコードは、矛盾を解決するための精神的な負担を軽減し、あらゆる規模におけるプロジェクトの保守・改善の容易性を高める。
- 可能な限り読みやすいコードにするために、以下のような努力を払うべき。
  - 変数名を省略しない。
    - e.g.) `bsd`や`bal_s_d`よりも、`balance_sheet_decrease`が望ましい。
  - 関数の引数名を書き出す。
  - クラスやメソッドを文書化する。
  - コードにコメントをつける。
  - 繰り返されるコードを再利用可能な関数やメソッドにリファクタリングする。
  - 関数やメソッドは短くまとめる。
    - 関数やメソッド全体を読むのにスクロールしない程度であることが望ましい。

### 1.2 PEP 8
- Python の公式スタイルガイド。
- PEP 8 を詳しく読み、PEP 8 のコーディング規約に従うことを推奨する。
  - [PEP 8](python.org/dev/peps/pep-0008/)
- PEP 8 のガイドラインを覚えるのが難しい場合は、コーディング時に自動でチェックしてくれるエディタのプラグインを導入する。
- [警告] 既存のプロジェクトの慣習を変えないこと。
  - PEP 8 のスタイルは、新しいDjangoプロジェクトにのみ適用される。
  - PEP 8 とは異なる慣習に従っている既存のDjangoプロジェクトに持ち込まれた場合は、既存の慣習に従うこと。
  - PEP 8 の "A Foolish Consistency is the Hobgoblin of Little Minds" (愚かな一貫性は狭量なる心の表れ) の項を参照。
    - [A Foolish Consistency is the Hobgoblin of Little Minds](python.org/dev/peps/pep-0008/#a-foolish-consistency-is-thehobgoblin-of-little-minds)
- [PACKAGE TIP] **Black**: Pythonコードの整形。
  - [Black](github.com/psf/black)
- [PACKAGE TIP] **Flake8**: Pythonコードの品質チェック。
  - プロジェクトのコーディングスタイル、品質、ロジックエラーをチェックするコマンドラインツール。
  - ローカルでの開発時や、継続的インテグレーション (CI)のコンポーネントとして使用する。
  - [Flake8](github.com/PyCQA/flake8)

#### 1.2.1 79文字の制限について
- PEP 8 によると、1行あたりのテキストの制限は79文字。
  - ほとんどの開発チームにとって、コードの理解度を損なわない安全な値という位置づけ。
  - 排他的なチームプロジェクトの場合、この制限を99文字まで緩和するという規定もある。
    - 「排他的な」というのは、OSSプロジェクトでないことを指す。(OSSプロジェクトでは79文字を厳守すべき)

> "コードを79文字に収めることは、変数や関数、クラスの名前を悪くする正当な理由にはなりません。
> 30年前のハードウェアの恣意的な制限に合わせるよりも、読みやすい変数名を持つことの方がずっと重要なのです。"
> ー Aymeric Augustin (Djangoのコア開発者)

### 1.3 インポート
- PEP 8では、importは次のような順序でグループ化することを提案している。
  1. 標準ライブラリのインポート
  2. 関連するサードパーティのインポート
  3. ローカルアプリケーションまたはライブラリ固有のインポート


- 良いimportの例

```python:good_import.py
# Stdlib imports
from math import sqrt
from os.path import abspath

# Core Django imports
from django.db import models
from django.utils.translation import gettext_lazy as _

# Third-party app imports
from django_extensions.db.models import TimeStampedModel

# Imports from your apps
from splits.models import BananaSplit
```

- Django プロジェクトでのインポートの順序
  1. 標準ライブラリからのインポート
  2. コアDjango からのインポート
  3. Djangoと無関係なものを含む、サードパーティアプリからのインポート
  4. Djangoプロジェクトの一部として作成したアプリからのインポート

- [PACKAGE TIP] **isort** 
  - インポートをアルファベット順にソートするツール。
  - インポートをセクションやタイプ別に自動的に分類する。

### 1.4 明示的な相対インポートを理解する
- 明示的な相対インポートを使う習慣を身につける。
  - 個々のモジュールが周囲のアーキテクチャと密結合しないようにする強力なツールである。

- 相対インポートの例

```python:relative_import.py
# cones/views.py
from django.views.generic import CreateView

# Relative imports of the 'cones' package
from .models import WaffleCone
from .forms import WaffleConeForm

# absolute import from the 'core' package
from core.views import FoodMixin

class WaffleConeCreateView(FoodMixin, CreateView):
    model = WaffleCone
    form_class = WaffleConeForm
```

- 絶対インポートと明示的相対インポートの違いを理解することで、ローカル(内部インポート)とグローバル(外部インポート)を即座に見分けることができ、コード単位としてのPythonパッケージを強調することができる。
- 以下は、DjangoプロジェクトにおけるPythonのimport タイプの違いと、それを使うタイミングを表にしたもの。

|コード例|インポートタイプ|使い方|
|:--|:--|:--|
|from core.views import FoodMixin|絶対的なインポート|現在のアプリ外からのインポート|
|from .models import WaffleCone|明示的な相対インポート|現在のアプリ内の他モジュールからのインポート|

### 1.5 import * を使わない
- 各モジュールを明示的にインポートし、決して`import *`を使わないこと。
- 良い例

```python:good_import.py
from django import forms
from django.db import models
```

- 悪い例

```python:bad_import.py
# ANTI-PATTERN: Don't do this!
from django.forms import *
from django.db.models import *
```

- 使ってはいけない理由は、他のPythonモジュールの全てのローカルを、現在のモジュールの名前空間に暗黙のうちに読み込んでしまうことを避けるため。
- 上記「悪い例」では、`django.forms` と `django.models` の両ライブラリに `CharField` というクラスが含まれる。
  - 両方のライブラリを暗黙的にロードすることで、models ライブラリが forms バージョンのクラスを上書きしてしまう。

#### 1.5.1 その他のPythonの名前付けの衝突
- 以下のように、同名のモジュールを複数インポートしようすると、同様の問題に遭遇する。

```python:module_collisions.py
# ANTI-PATTERN: Don't do this!
from django.db.models import CharField
from django.forms import CharField
```

- 名前の衝突を避ける必要がある場合は、エイリアス (別名)を使用する。

```python:aliases.py
# ANTI-PATTERN: Don't do this!
from django.db.models import CharField as ModelCharField
from django.forms import CharField as FormCharField
```

### 1.6 Djangoのコーディングスタイル
- 公式ガイドラインと、非公式だが一般的に受け入れられているDjangoの慣習の両方を扱う。

#### 1.6.1 Django のコーディングスタイルガイドラインを考える
- 一般的なDjangoのスタイル規約を知っておくと良いことは言うまでもない。
- 実際、DjangoにはPEP 8 を拡張した独自のスタイルガイドラインがある。
  - [Coding style](docs.djangoproject.com/en/3.2/internals/contributing/writing-code/coding-style/)

#### 1.6.2 URLパターン名にはダッシュではなくアンダースコアを使う
- URL設計においては、ダッシュよりもアンダースコアを使う。
- これは単にPythonicな(Pythonらしい)だけではなく、より多くのIDEやテキストエディタと親和性がある。
- ここでは、ブラウザに入力された実際のURLではなく、url()のname引数を参照していることに注意する。

- URL名にダッシュを使う間違った例。

```python:bad_url_pattern.py
patterns = [
    path(route='add/',
        view=views.add_topping,
        name='add-topping'),
    ]
```

- 良いURLパターン名の例。

```python:good_url_pattern.py
patterns = [
    path(route='add/',
        view=views.add_topping,
        name='toppings:add_topping'),
    ]
```

- 実際に入力されるURLにダッシュが入っていても問題ない。
  - (e.g.) route='add-topping/'

#### 1.6.3 テンプレートブロック名にはダッシュではなくアンダースコアを使う
- URLパターン名と同様の理由で、テンプレートブロック名にアンダースコアを使うことを推奨する。
  - よりPythonらしく、より編集しやすい名前になる。

### 1.7 JS、HTML、CSSのスタイルガイドを選ぶ

#### 1.7.1 JavaScriptのスタイルガイド
- 公式なスタイルガイドがあるPythonとは異なり、JavaScriptには公式なスタイルガイドがない。その代わりに、様々な個人や企業によって非公式なJSスタイルガイドが作成されている。
  - [Standard combined JavaScript and Node.js Style Guide](github.com/standard/standard)
  - [idiomatic.js (一貫性のある慣用的なJavaScriptを書くための原則)](github.com/rwaldron/idiomatic.js) 
  - [AirbnbのJavaScriptスタイルガイド](github.com/airbnb/javascript)

- Django や JavaScript のコミュニティでは、これらのうちのどれか1つについてコンセンサスがあるわけではないので、好きなものを選び、それを守るようにする。
- ただし、独自のスタイルガイドを持つ JavaScript フレームワークを使用している場合は、そのガイドを使用する必要がある。

- [PACKAGE TIP] **ESLint** : JavaScriptとJSXのためのプラグイン式リンティングユーティリティ
  - JavaScriptとJSXのコードスタイルをチェックするツール。
  - 上に挙げたスタイルガイドを含む、いくつかのスタイルガイドのJSスタイルルールのプリセットを備えている。
  - また、様々なエディタ用のESLintプラグインや、Webpack、Gulp、GruntなどのさまざまなJavaScriptツール用のESLintタスクもある。

#### 1.7.2 HTMLとCSSのスタイルガイド
- [@mdoによるHTMLとCSSのコードガイド](codeguide.co)
- [idomatic-css: Principles of Writing Consistent, Idiomatic CSS:](github.com/necolas/idiomatic-css)
- [PACKAGE TIP] **stylelint**: CSS用のコーディングスタイルフォーマッター。
  - 設定したルールとの整合性チェック。
  - CSSプロパティのソート順チェック。

### 1.8 特定のIDEやテキストエディタに向けたコードを書かない
- 特定のIDEの機能を理由として、プロジェクトのレイアウトや実装を決定してはならない。
  - 開発者とツールが一致しない人にとって、コードの理解を著しく困難にする。
  - 自分以外の開発者がそれぞれのツールを使いたがることを常に想定し、コードやプロジェクトのレイアウトを透明性のあるものにすること。
- 例えば、テンプレートタグのソースを発見することは、限定的なIDEを使用している開発者以外には難しく、時間がかかる。
  - そのため、一般的に使用されている `<app_name>_tags.py` というネーミングパターンを採用する。

### 1.9 まとめ
- 上記で採用しているコーディングスタイルに従わない場合でも、一貫したコーディングスタイルを守ること。
- 多様なスタイルが同居するプロジェクトはメンテナンスが非常に難しく、開発に時間がかかり、開発者のミスを誘発する。

## 2. 最適なDjangoの開発環境を準備する

### 2.1 どこでも同じデータベースエンジンを使う
- ローカルの開発ではSQLite3を使い、本番環境ではPostgreSQL (またはMySQL)を使う、というのはよく見られるケースだが、これが落とし穴となる。
- このセクションでは、開発用と本番用で異なるデータベースエンジンを使用した場合に遭遇する問題を紹介する。

#### 2.1.1 本番データの正確なコピーをローカルで調べることができない
- 本番用DBがローカルの開発用DBと異なる場合、本番用DBの正確なコピーを取得してローカルでデータを検証することはできない。
  - 本番環境からSQLダンプを生成してローカルにインポートすることはできるが、エクスポートとインポートの後に正確なコピーが得られるとは限らない。

#### 2.1.2 データベースによってフィールドの種類や制約が違う
- フィールドデータの型キャストの扱いが異なる。
- 例えば、ローカル開発でSQLite3を、本番環境でPostgreSQLを使った場合。
  - SQLite3は動的な弱い型付けをしており、PostgreSQLは強い型付けを使用しているので、最終的に問題が発生する。
  - DjangoのORMにはSQLite3をより強い型付けで操作する機能があるが、開発中のフォームやモデルの検証ミスは、本番サーバで実行されるまで(テストでも)検出されない。
  - 本番環境では、PostgreSQLやMySQLが、ローカルでは見たことのない制約エラーを投げることがある。そしてそれは、ローカルで同じ環境をセットアップしない限り再現することが難しい。
  - ほとんどの問題は、型付けの強いDB(PostgreSQLやMySQLなど)でプロジェクトを実行するまで発見できない。
- [TIP] Django + PostgreSQLの優れた連携性
  - 多くのDjango開発者が、開発、ステージング、QA、本番システムなどのあらゆる環境でPostgreSQLを使うことを好んでいる。

### 2.1.3 フィクスチャは魔法の解決策ではない
- フィクスチャは、シンプルなテストデータセットを作成するのに適している。
  - 特にプロジェクトの初期段階で、開発中に仮データをDBに入れておく必要がある場合などに効果的。
- 一方でフィクスチャは、DBに依存しない方法で、大規模なデータセットをあるDBから別のDBに移行するための信頼できる手段ではない。
- [警告] 本番環境のDjangoでSQLite3を使用しないこと。
  - 2人以上のユーザを想定していたり、軽い並行性以上のものを必要とするWebプロジェクトにとって、SQLite3 は悪夢のような存在である。
  - 問題発生後にデータをSQLite3から並行処理用に設計されたもの (例えば、PostgreSQL)に移行することは非常に難しい。

### 2.2 pipとVirtualenv (または venv) の使用
- pip と virtualenv の両方に精通することを強く推奨する。
  - これらはDjangoプロジェクトのデファクトスタンダードであり、Djangoを使っているほとんどの企業がこれらのツールに依存している。
- pip は Python Package Index とそのミラーからPythonパッケージを取得するツールである。
  - Pythonパッケージの管理とインストールに使用され、Linux以外のほとんどのPythonインストーラに付属されている。
- Virtualenvは、パッケージの依存関係を維持するために独立したPython環境を作成するためのツールである。
  - 同時に複数のプロジェクトで作業していて、プロジェクト間で使用しているライブラリのバージョンが異なるような状況で最適である。

- [PACKAGE TIP] **Conda**: Virtualenvの代替。
  - Windows、Mac、Linuxで動作するオープンソースのパッケージ管理システムおよび環境管理システム。
  - コンパイルされたバイナリをインストールしたいWindowsプラットフォームのユーザにとっては最も使いやすいものとなっている。
  - また、Condaは複数のPythonのバージョンを簡単に扱うことができる。
  - [Conda](docs.conda.io/)

- [PACKAGE TIP] **Poetry**および**Pipenv**: Pip+Virtualenvの代替。
  - Poetryは、Pythonプロジェクトの依存関係の宣言、管理、インストールを支援し、あらゆる場所で正しいスタックを確保することができる。
    - 直感的なCLIで多くの機能をカプセル化している点が評価されている。
    - 人気があり、安定している。
    - [Poetry](python-poetry.org)

  - Pipenvは、pipとvirtualenvを1つのインターフェースにまとめたツール。
    - 環境の作成などのプロセスを自動化し、再現性のあるビルドを可能にするファイルであるPipfile.lockを導入しています。
    - [再現性のあるビルド](https://ja.wikipedia.org/wiki/%E5%86%8D%E7%8F%BE%E6%80%A7%E3%81%AE%E3%81%82%E3%82%8B%E3%83%93%E3%83%AB%E3%83%89)
    - [Pipenv](pipenv.pypa.io/)
- [警告] Linuxディストリビューションでは、Pipは必ずしもデフォルトでインストールされていない。
  - 多くのLinuxディストリビューションでは、メンテナがいくつかのモジュールを標準的なPythonライブラリから削除し、それらを個別のインストール可能なパッケージにしている。
  - 例えば、ubuntuではpython3-pip, python3-setuptools, python3-wheel, python3-distutilsをインストールする必要がある。
  - Windows 10では近い将来、WSL2 (Linux on Windows) の安定版が提供される予定なので、同様にこれらのユーザーに影響を与える可能性がある。

#### 2.2.1 virtualenvwrapper
- MacやLinux用のvirtualenvwrapperや，Windows用のvirtualenvwrapper-winの使用を推奨する。

- virtualenvwrapperを使用していない場合
  - コマンドが冗長。

```command:
$ source ~/.virtualenvs/twoscoops/bin/activate
```

- virtualenvwrapperを使用した場合
  - コマンドが簡潔。

```command:
$ workon twoscoops
```

### 2.3 Pip による Django とその他の依存関係のインストール
- Djangoをインストールする方法は複数あるが、pip とrequirementsファイルを使ったインストール方法を推奨する。
  - requirementsファイルには各パッケージの名前と、必要に応じて適切なバージョン範囲が記載されている。pip を使って、このリストからパッケージを仮想環境にインストールする。
- [TIP] PYTHONPATHの設定
  - コマンドラインや環境変数を理解している場合は、仮想環境のPYTHONPATHを設定することで、django-adminコマンドを使用してサイトのサービスやその他のタスクを実行することができる。
  - また、virtualenvのPYTHONPATHに、最新バージョンのpipが入っているカレントディレクトリを設定することもできる。
  - プロジェクトのルートディレクトリで`pip install -e` を実行すれば、カレントディレクトリがパッケージとしてインストールされ、その場で編集できるようになる。





