# Two Scoops of Django 3.x

## 6. モデルに関するベストプラクティス
- モデルはDjangoプロジェクトの基盤である。
- よく考えずに Django のモデルを書こうとすると、将来的に問題が発生する可能性が高まる。
- 私たち開発者は、自分のやっていることの影響を考えずに、モデルの追加や修正を急ぐことがままある。
- 手っ取り早く修正したり、一時的に杜撰な判断をしてコーディングすると、数ヶ月後、数年後に、おかしな回避策を余儀なくされたり、既存のデータを破壊されたりして、痛い目を見ることになる。
- Djangoで新しいモデルを追加したり、既存のモデルを修正したりするときは、このことを念頭に置くこと。
- 時間をかけて考え、可能な限り強固で健全な基盤を設計すること。
- [PACKAGE TIP] モデルを扱う上でおすすめのDjangoパッケージ
  - django-model-utilsでは、 `TimeStampedModel`のような有名なパターンを扱うことができる。
  - django-extensionsには `shell_plus`という強力な管理コマンドがあり、 インストールされている全てのアプリのモデルクラスを自動読み込みできる。
    - 多くの機能を読み込んでしまうために、小さく単一責任なアプリを作るという志向から外れてしまう欠点がある。

### 6.1 基本
#### 6.1.1 モデル数の多いアプリを分割する
- 1つのアプリに20以上のモデルが含まれる場合、アプリを小さく分割することを考慮すべきである。
  - アプリ内のモデルは5～10個に収めたい。

#### 6.1.2 モデルの継承に注意する
- Djangoにはモデル継承を行うための3つの方法がある。
  - 抽象基底クラス
  - 複数テーブル継承
  - プロキシモデル
- [警告] Django の抽象基底クラスとPythonの抽象基底クラスは異なる
  - Djangoの抽象基底クラスとPython標準ライブラリのabcモジュールの抽象基底クラスを混同しないこと。(目的や動作が異なる)
- ここでは、3つの継承スタイルの長所と短所を説明する。
  - 比較のため、モデルを全く継承しない場合も含める。
- **継承なしの場合**
  - 共通のフィールドがある場合、全てのモデルにそのフィールドを付与する。
  - 長所
    - モデルがDBのテーブルにどのように対応しているか一目で理解しやすくなる。
  - 短所
    - モデル間で重複するフィールドが多いと、メンテナンスに手間がかかる。
- **抽象基底クラスの場合**
  - テーブルは派生モデルに対してのみ作成される。
  - 長所
    - 共通のフィールドを抽象クラスに持たせるので、フィールドが重複しない。
    - 複数テーブル継承(後述)を採用した場合に発生する、余分なテーブルや結合のオーバーヘッドが発生しない。
  - 短所
    - 親クラスを単独で使用することができない。
- **複数テーブル継承の場合** (非推奨)
  - 親と子 (基底と派生) の両方にテーブルが作成される。
    - 暗黙の`OneToOneField`が親と子をリンクする。
  - 長所
    - 各モデルに独自のテーブルを与えるので、親と子のどちらにも問い合わせができる。
    - また、親オブジェクトから子オブジェクトを取得することができる。
  - 短所
    - 子テーブルへのクエリはすべての親テーブルとの結合を必要とするため、かなりのオーバーヘッドが発生する。
    - 複数テーブルの継承を使用しないことを強く推奨する (下記の[警告]を参照)。
- **プロキシモデルの場合**
  - テーブルは元のモデルに対してのみ作成される。
  - 長所
    - 異なるPythonの振る舞いを持つモデルのエイリアスを持つことができる。
  - 短所
    - モデルのフィールドは変更できない。
- [警告] 複数テーブルの継承を避ける
  - 複数テーブルの継承 (「具象継承」とも呼ばれる) は、多くの開発者にとって悪いことだと考えられており、使用しないことを強く推奨する (後に詳述)。
- どの継承タイプをいつ使用すべきか知るための経験則を紹介する。
- **モデル間の重複が少ない場合** 
  - 例えば、1～2個の同名フィールドを共有するモデルがいくつかあるだけの場合。
  - 継承は必要ないことが多い。
- **モデル間の重複が少ない場合** 
  - モデル間に十分な重複があり、各モデルの重複するフィールドのメンテナンスが混乱やミスの原因になるような場合。
  - 共通のフィールドを抽象基底クラスに配置する必要がある。
- プロキシモデルは時折役に立つ便利な機能ですが、他の2つの継承スタイルとは大きく異なる。
- 複数テーブル継承は、混乱と大きなオーバーヘッドを招くため、絶対に避けるべき。
  - 複数テーブル継承の代わりにモデル間で明示的な`OneToOneFields`と`ForeignKey`を使用することで、結合処理によるオーバーヘッドが増大するタイミングを制御することができる。
  - 著者の経験上、複数テーブル継承はトラブルの原因になったことしかない。

#### 6.1.3 実践的なモデルの継承例：TimeStampedModel
- Djangoプロジェクトでは、すべてのモデルに`created`と`modified`のtimestampフィールドを含めることがある。
- これらのフィールドを一つ一つのモデルに手動で追加することもできるが、作業量が多く、ヒューマンエラーの危険性もある。
- `TimeStampedModel`を用いることでモデルの実装がより簡単になる。

```python:core/models.py
from django.db import models

class TimeStampedModel(models.Model):
    """
    An abstract base class model that provides selfupdating
    ``created`` and ``modified`` fields.
    """
    created = models.DateTimeField(auto_now_add=True)
    modified = models.DateTimeField(auto_now=True)
    
    class Meta:
        abstract = True
```

- サンプルコードの最後の2行に注意すること。
  - この記述により、`TimeStampedModel`クラスは抽象基底クラスになる。
- `TimeStampedModel`を抽象基底クラスとして定義し、新しいクラスを定義する際にそれを継承することで、`migrate`の実行時に`core_timestampedmodel`テーブルが作成されなくなる。

```python:flavors/models.py
# flavors/models.py
from django.db import models

from core.models import TimeStampedModel

class Flavor(TimeStampedModel):
    title = models.CharField(max_length=200)
```

- 上記の例では、`flavors_flavor`という名のテーブルしか作成されない。
- 一方、`TimeStampedModel`が抽象基底クラスではない (つまり具象基底クラス) 場合は…
  - `core_timestampedmodel`テーブルも作成される。
  - `Flavor`を含むすべてのサブクラスはフィールドを持たず、作成・変更時のタイムスタンプを処理するために`TimeStampedModel`への暗黙の外部キーを持つことになる。
  - `Flavor`を参照して`TimeStampedModel`を読み書きすると、2つのテーブルに影響を与えることになる。
- 以上のことから、具象継承は、パフォーマンスのボトルネックになる可能性があることがわかる。
- 具象モデルクラスを何度もサブクラス化すると、さらにその傾向が強くなる。
- 詳細は[モデルの継承](docs.djangoproject.com/en/3.2/topics/db/models/#model-inheritance)

### 6.2 データベースマイグレーション
- Djangoには、マイグレーションと呼ばれるデータベースの変更を反映するための強力なライブラリがある。

#### 6.2.1 マイグレーションに関するヒント
- 新しいアプリやモデルを作成したら、すぐに最初のマイグレーションを作成すること。
  - ターミナルに `python manage.py makemigrations` と入力するだけでよい。
- 複雑な変更を伴う場合は、マイグレーション実行前によく吟味すること。
  - 生成されたマイグレーションコードを検討する。
  - `sqlmigrate`コマンドで実行予定のSQLを確認する。
- Djangoスタイルのマイグレート手段を持たないサードパーティアプリに対しては`MIGRATION_MODULES`設定を使い、マイグレーションを書いて管理すること。
- 作成されるマイグレーションの数は気になくてよい。
  - マイグレーションが増えすぎて扱いにくくなったら、`squashmigrations`コマンドを使うこと (migrationsファイルを1ファイルにまとめることができる)。
- マイグレーションを実行する前に、必ずデータをバックアップすること。

#### 6.2.2 マイグレーションにPython関数とカスタム SQLを追加する
- プロジェクト内のデータや相互作用する外部コンポーネントなどの複雑な変更をマイグレーションは予測できない。
- そのような場合にはPythonコードやカスタムSQLを用いてマイグレーションを補助することができる。
- `RunPython`または`RunSQL`クラスの使用を検討すること。
  - [RunPythonクラス](docs.djangoproject.com/en/3.2/ref/migration-operations/#runpython)
  - [RunSQLクラス](docs.djangoproject.com/en/3.2/ref/migration-operations/#runsql)

### 6.3 RunPythonの一般的な問題点について
- `RunPython`の関数を書いていると、いくつかの問題点に遭遇する。

#### 6.3.1 カスタムモデルマネージャのメソッドへのアクセス
- カスタムモデルマネージャのメソッドを使って、レコードのフィルタリング、 除外、作成、変更などをしたいことがある。
  - しかし、Djangoのマイグレーション機能はデフォルトではこれらのコンポーネントを除外する。
  - カスタムマネージャに`use_in_migrations = True`を追加することで、この挙動をオーバーライドすることができる。
  - [参考](docs.djangoproject.com/en/3.2/topics/migrations/#model-managers)

#### 6.3.2 カスタムモデルメソッドへのアクセス
- Djangoのマイグレーション機能はモデルをシリアライズするため、マイグレーション中にカスタムメソッドを呼び出すことはできない。
  - [参考](docs.djangoproject.com/en/3.2/topics/migrations/#historical-models)
- [警告] カスタマイズされた保存・削除メソッドについての注意点
  - モデルのsaveとdeleteメソッドをオーバーライドすると`RunPython`から呼ばれなくなる。これは致命的な問題を引き起こす可能性もあるため注意すること。

#### 6.3.3 何もしないことを表すRunPython.noop
- マイグレーションの可逆性を保つためには、RunPythonの引数`reverse_code`に処理を渡し、ロールバック時に実行される処理を定義しなければならない。
- しかし、既存のデータを新しく追加されたフィールドに結合するなど、処理の中には冪等 (訳者註: 状態を持たず、特定の入力に対する出力が常に一意に定まること) なものもあり、これらの関数のために`reverse_code`を書くことは不可能か、無意味である。
- このような場合は、`RunPython.noop`を`reverse_code`として使用する。

```python:runpython_noop.py
from django.db import migrations, models

def add_cones(apps, schema_editor):
    Scoop = apps.get_model('scoop', 'Scoop')
    Cone = apps.get_model('cone', 'Cone')

    for scoop in Scoop.objects.all():
        Cone.objects.create(
            scoop=scoop,
            style='sugar'
        )

class Migration(migrations.Migration):

    initial = True

    dependencies = [
        ('scoop', '0051_auto_20670724'),
    ]

    operations = [
        migrations.CreateModel(
        name='Cone',
        fields=[
            ('id', models.AutoField(auto_created=True, 
                primary_key=True,
                serialize=False, verbose_name='ID')),
            ('style', models.CharField(max_length=10),
                choices=[('sugar', 'Sugar'), ('waffle', 'Waffle')]),
            ('scoop', models.OneToOneField(null=True,
                to='scoop.Scoop'
                on_delete=django.db.models.deletion.SET_NULL,
                    )),
        ],
    ),
    # RunPython.noop does nothing but allows reverse migrations to occur
    migrations.RunPython(add_cones, migrations.RunPython.noop)
]
```

#### 6.3.4 デプロイ時の注意点やマイグレーションの管理について
- マイグレーション実行前に、データを必ずバックアップすること。
- デプロイ前に、ロールバック可能であることを必ず確認すること。
  - 可逆性を常に保証することはできないが、以前の状態にロールバックできないと、バグの追跡に支障をきたすし、大規模なプロジェクトではデプロイできないこともある。
- 数百万行のテーブルを持つプロジェクトでは、本番サーバーでマイグレーションを実行する前に、ステージングサーバーで同規模のデータに対してテストを行うこと。
  - 実運用データのもとでのマイグレーションは、予想よりもはるかに時間がかかることがある。
- MySQLを使用している場合の注意点
  - スキーマを変更する前には、絶対にDBをバックアップしなければならない。MySQLはスキーマ変更時のトランザクションをサポートしていないため、ロールバックが不可能。
  - 可能であれば、変更を実行する前にプロジェクトを読み取り専用モードにする。
  - 注意しないと、データ件数の多いテーブルのスキーマ変更には長い時間がかかる。これには数時間かかることもある。
- [TIP] マイグレーションコードを必ずバージョン管理下に置く。
  - バージョン管理システムにはマイグレーションコードを必ず含めること。
  - 設定ファイルと同様、バージョン管理せずとも開発はできるが、マシンを切り替えたり、他のメンバーをプロジェクトに参加させたりするときに問題が顕在化する。

### 6.4 Djangoのモデル設計
- 優れたDjangoモデルの設計する方法は最も難しく、最も注目されていないテーマの一つでもある。
- 時期尚早な最適化に陥ることなくパフォーマンスを担保するにはどのようにすればいいのか、ここではいくつかの方法を見ていく。

#### 6.4.1 「正規化」から始める
- データベースの正規化に精通することを推奨する。
- Djangoでモデルを扱う上で、正規化の知識は必須となるので、もし馴染みがないのであれば理解を深めておくこと。
- このテーマについての詳細な説明は本書の範囲外なので、以下のリンクをお勧めする。
  - [Database Normalization (Wikipedia)](en.wikipedia.org/wiki/Database_normalization)
  - [Relational Database Design/Normalization](en.wikibooks.org/wiki/Relational_Database_Design/Normalization)
- Djangoのモデル設計は、必ず正規化から始めること。
- どのモデルにも、他のモデルに既に格納されているデータが含まれないように、時間をかけて確認すること。
- この段階では、関係フィールド (訳者註: 外部キーなどのこと？) を自由に使ってよい。
- 早々に非正規化してはいけない。
- データ構造をよく理解しておく必要がある。

#### 6.4.2 非正規化する前のキャッシュ
- 適切な場所にキャッシュを設定することで、非正規化を行わずに済むことがよくある。
  - キャッシュについては、第26章で詳述する。

#### 6.4.3 絶対に必要な場合のみ非正規化する
- 安易な非正規化はやめるべきである。
  - これはプロジェクトを複雑にし、データ損失のリスクを大幅に高めるプロセスである。
  - 非正規化の前にキャッシングを検討すること。
- 第26章で説明する、ボトルネックの発見と削減におけるあらゆる方策を試し、それでも限界に達した場合のみ、データベースの非正規化の検討を始めるべき。

#### 6.4.4 NullとBlankを使用するタイミング
- モデルフィールドを定義する際に、`Null=True`と`Blank=True`のオプションを設定することができる。
  - デフォルトでは、これらはFalseとなっている。
- これらのオプションをいつ使用べきかは、開発者にとって悩ましい問題の一つである。
- これらのモデルフィールド引数の標準的な使用方法を、以下の表に示す。

|フィールドタイプ|null=True|blank=True|
|:--|:--|:--|
|CharField<br>TextField<br>SlugField<br>EmailField<br>CommaSeparated-IntegerField<br>UUIDField|`unique=True`と`blank=True`の両方を設定している場合OK。この場合、空の値を持つ複数のオブジェクトを保存する際の一意制約違反を避けるために、`null=True`が必要。|対応するフォームウィジェットが空の値を許容する場合、設定可。`null=True`と`unique=True`も設定されている場合、空の値はデータベースにNULLとして保存される。それ以外の場合は、空の文字列として保存される。|
|FileField<br>ImageField|設定してはいけない。Djangoは`MEDIA_ROOT`からファイルや画像のパスをCharFieldに格納しているので、FileFieldsも同じパターンになる。|CharFieldと同じ。|
|BooleanField|設定可。|デフォルトは`blank=True`。|
|IntegerField<br>FloatField<br>DecimalField<br>DurationField, etc|NULLを許容する場合、設定可。|対応するフォームウィジェットが空の値を許容する場合、設定可。その場合、`null=True`を設定する必要がある。|
|DateTimeField<br>DateField<br>TimeField, etc.|NULLを許容する場合、設定可。|対応するフォームウィジェットが空の値を許容する場合、設定可。(`null=True`を設定する必要がある)<br>`auto_now`や`auto_now_add`を使用している場合にも適している。|
|ForeignKey<br>OneToOneField|NULLを許容する場合、設定可。|対応するフォームウィジェット (セレクトボックスなど) が空の値を許容する場合、設定可。|
|ManyToManyField|NULLは意味をなさない。|対応するフォームウィジェット (セレクトボックスなど) が空の値を許容する場合、設定可。その場合、`null=True`を設定する必要がある。|
|GenericIPAddressField|NULLを許容する場合、設定可。|対応するフォームウィジェットが空の値を許容する場合、設定可。その場合、`null=True`を設定する必要がある。|
|JSONField|設定可。|設定可。|

#### 6.4.5 BinaryField を使用する場合
- BinaryFieldには、生のバイナリデータやバイト列を保存することができる。
- BinaryFieldでは、filterやexcludeなどのSQLアクションを実行することはできない。
- 例えば、以下のようなものを保持するのに使用する。
  - MessagePackでフォーマットされたコンテンツ。
  - センサー等から取得した生データ。
  - 圧縮データ
    - 例えばSentryがBLOBとして保存しており、base64エンコードが必要なデータなど。
- 様々な用途があるが、バイナリデータは巨大なチャンクで送られてくることがあり、データベースの速度を低下させる可能性があることに注意すること。
- これがボトルネックになっている場合は、バイナリデータをファイルに保存し、FileFieldで参照することが解決策になることもある。
- [警告] BinaryFieldにファイルを保持してはならない。
  - データベースのフィールドにファイルを保持することは、絶対にあってはならない。
  - データベースをファイルストアとして使用することの問題点について、PostgreSQLのエキスパートであるFrank Wiles氏の意見を要約する。
    - DBへの読み書きは常にファイルシステムよりも遅い。
    - DBのバックアップが膨大になり、処理に時間がかかるようになる。
    - ファイルへのアクセスのために、アプリ(Django)とDBのレイヤーを経由する必要がある。
    - 詳細は revsys.com/blog/2012/may/01/three-things-you-should-never-putyour-database/

#### 6.4.6 ジェネリックなリレーションを避ける
- 「ジェネリックなリレーション」とは、制約のない外部キー (GenericForeignKey) を使って、あるテーブルを別のテーブルと結合するというもの。
- RDBにジェネリクスを持ち込んだり、`models.field.GenericForeignKey`を使用したりすることは推奨されない。
- 筆者は価値以上に問題が多いと考えており、ジェネリクスに頼っている場合、非合理な近道や間違った解決策を取っていることを示唆しているケースが多い。
- 外部キー制約がないNoSQLデータストアを、本当は外部キー制約を使うべきプロジェクトの基盤として使うことに似ている。
- 「ジェネリックなリレーション」により、以下のようなことが起こる。
  - モデル間のインデックスがないため、クエリの速度が低下する。
  - 存在しないレコードに対してテーブルが別のテーブルを参照できるため、データが破損する危険性がある。
- 利点としては、アプリを簡単に構築できる点が挙げられる。ただし作成した多くのモデル間に相互作用が生じることに注意すること。
- GenericForeignKeyの使用を避けることで、多少の開発の手間はかかるものの、処理速度と整合性というメリットが得られる。
- プロジェクトの主要データを定義する方法としてGenericForeignKeyが使用される場合、本当に厄介なものになる。
- まとめると、
  - 「ジェネリックなリレーション」やGenericForeignKeyはなるべく避ける。
  - もし、「ジェネリックなリレーション」が必要だと思うのであれば、より良いモデル設計やPostgreSQLの新しいフィールドによって問題が解決できるかどうかを検討する。
  - どうしても使用を避けられない場合は、既存のサードパーティ製アプリの使用を検討する。 
    - サードパーティ製のアプリが提供する分離は、データをよりクリーンに保つのに役立ちます。
  - [参考 (Avoid Django GenericForeignKey)]( lukeplant.me.uk/blog/posts/avoid-django-genericforeignkey)

#### 6.4.7 選択肢のモデル定数化
- タプルで定義された構造体としてのモデルに、選択肢をプロパティとして追加するのは良い方法である。
- これらはモデルに結びついた定数なのでどこからでも簡単にアクセスでき、開発が容易になる。
- このテクニックは以下で説明されている。
  - docs.djangoproject.com/en/3.2/ref/models/fields/#django.db.models.Field.channels 
- アイスクリームをベースにした例に変換すると、こうなります。

```python:choice_model.py
# orders/models.py
from django.db import models

class IceCreamOrder(models.Model):
    FLAVOR_CHOCOLATE = 'ch'
    FLAVOR_VANILLA = 'vn'
    FLAVOR_STRAWBERRY = 'st'
    FLAVOR_CHUNKY_MUNKY = 'cm'
    
    FLAVOR_CHOICES = (
        (FLAVOR_CHOCOLATE, 'Chocolate'),
        (FLAVOR_VANILLA, 'Vanilla'),
        (FLAVOR_STRAWBERRY, 'Strawberry'),
        (FLAVOR_CHUNKY_MUNKY, 'Chunky Munky')
    )
    
    flavor = models.CharField(
        max_length=2,
        choices=FLAVOR_CHOICES
    )
```

- このモデルを使うと次のようになる。

```
>>> from orders.models import IceCreamOrder
>>> IceCreamOrder.objects.filter(flavor=IceCreamOrder.FLAVOR_CHOCOLATE)
[<icecreamorder: 35>, <icecreamorder: 42>, <icecreamorder: 49>]
```

- これでPythonコード上でもテンプレートでも使えるし、この属性はクラスでもインスタンス化されたモデルオブジェクトでもアクセスできる。

#### 6.4.8 選択肢に列挙型を使う
- 選択肢にDjangoの列挙型を使うことを推奨している開発者もいる。

```python:choice_enum_model.py
from django.db import models

class IceCreamOrder(models.Model):
    class Flavors(models.TextChoices):
        CHOCOLATE = 'ch', 'Chocolate'
        VANILLA = 'vn', 'Vanilla'
        STRAWBERRY = 'st', 'Strawberry'
        CHUNKY_MUNKY = 'cm', 'Chunky Munky'

    flavor = models.CharField(
        max_length=2,
        choices=Flavors.choices
    )
```

```
>>> from orders.models import IceCreamOrder
>>> IceCreamOrder.objects.filter(flavor=IceCreamOrder.Flavors.CHOCOLATE)
[<icecreamorder: 35>, <icecreamorder: 42>, <icecreamorder: 49>]
```

- 列挙型を使ったアプローチにはいくつかの欠点がある。
  - 名前付きのグループは列挙型では使用できません。
    - つまり、選択肢の中にカテゴリーを持たせたい場合は、従来のタプルベースのアプローチを使わなければならない。
  - また、strやint以外の型が必要な場合、自分で定義しなければならない。
- APIの使い勝手を考慮すると、列挙型のフィールドで選択肢を構築することは望ましい。
- 上記のような欠点が顕在化するまでは、列挙型を使うことを推奨する。

#### 6.4.9 PostgreSQL固有のフィールド: NullとBlankを使用するタイミング

|フィールドタイプ|null=True|blank=True|
|:--|:--|:--|
|ArrayField|OK|OK|
|HStoreField|OK|OK|
|IntegerRangeField<br>BigIntegerRangeField<br>FloatRangeField|NULLを許容する場合、設定可。|対応するフォームウィジェットが空の値を許容する場合、設定可。その場合、`null=True`を設定する必要がある。|
|DatetimeRangeField<br>DateRangeField|NULLを許容する場合、設定可。|対応するフォームウィジェットが空の値を許容する場合、設定可。(`null=True`を設定する必要がある)<br>`auto_now`や`auto_now_add`を使用している場合にも適している。|

### 6.5 \_meta API
- \_meta APIは、以下のような特徴がある。
  - 接頭辞が「\_」であるにもかかわらず公開されており、ドキュメント化されたAPIである。
- Djangoの他の「\_」接頭辞付きコンポーネントとは異なり、 \_metaはフレームワークの他の部分と同じ非推奨パターンに従っている。
  - その理由は、 Django 1.8以前のモデル \_metaのAPIは非公式で、意図的に文書化されていなかったからである。
- \_metaの本来の目的は、単にDjangoが自分で使うためにモデルの追加情報を保存することでしたが、その便利さが証明されたので、今ではドキュメント化されたAPIになっている。
- 主な用途は、以下のような場合である。
  - モデルのフィールドのリストを取得する。
  - モデルの特定のフィールドのクラスや、その継承元などに関連する情報を取得する。
  - 情報を取得する方法を、将来のDjangoのバージョンでも変わらないように定義する。
- 上記が必要になる具体的な例
  - Djangoモデルのイントロスペクションツール (オブジェクト情報を参照して変更を加えるツール) の構築。
  - カスタムDjangoフォームライブラリの構築。
  - Djangoモデルのデータを編集、操作するための管理者ツールの作成。
    - 例えば、"foo" で始まるフィールドの情報だけを分析するといった、視覚化や分析用のライブラリ。
- 参考文献: [\_meta のドキュメント](docs.djangoproject.com/en/3.2/ref/models/meta/)

### 6.6 モデルマネージャ
- DjangoのORMを使ってモデルに問い合わせをするときはいつも、**モデルマネージャ**と呼ばれるインタフェースを経由してデータベースとやりとりしている。
- モデルマネージャは、可能な限り全てのインスタンスのフルセット (つまり、テーブル内の全てのデータ) に作用してしまうので扱いづらい、とよく言われている。
- Djangoはモデルクラスごとにデフォルトのモデルマネージャを用意していますが、自分で定義することもできる。
- ここでは、カスタムモデルマネージャの簡単な例を示す。

```python:custom_model_manager.py
from django.db import models
from django.utils import timezone

class PublishedManager(models.Manager):

    def published(self, **kwargs):
        return self.filter(pub_date__lte=timezone.now(), **kwargs)

class FlavorReview(models.Model):
    review = models.CharField(max_length=255)
    pub_date = models.DateTimeField()
    
    # add our custom model manager
    objects = PublishedManager()
```

- 上記モデルで、最初にアイスクリームのフレーバーのレビューの全件数を表示し、次に公開された件数だけを表示したい場合は、次のようにする。

```
>>> from reviews.models import FlavorReview
>>> FlavorReview.objects.count()
35
>>> FlavorReview.objects.published().count()
31
```

- モデルマネージャを追加すれば、更に便利だと考えるかもしれない。そうすれば、次のようになる。

```
>>> from reviews.models import FlavorReview
>>> FlavorReview.objects.filter().count()
35
>>> FlavorReview.published.filter().count()
31
```

- しかし残念ながら、実際のプロジェクト開発でこの方法を使うときは非常に注意が必要である。なぜか？
  - まず、モデル継承を使用する場合、抽象基底クラスの子は親のモデルマネージャーを継承することができるが、具象基底クラスの子は継承できない。
  - 次に、モデルクラスに最初に適用されるマネージャは、Djangoデフォルトのものである。これはPythonの通常のパターンとは大きく異なり、QuerySetsの結果が予測できないものになる可能性がある。
  - これらを念頭に置いて、モデルクラスでは、`objects = models.Manager()` をカスタムモデルマネージャの上に手動で定義すること。
- [警告] モデルマネージャの操作順序を知る
  - `objects = models.Manager()`は、全てのカスタムモデルマネージャの上に必ず配置すべきである。
  - このルールは破ることができるが、高度すぎるのでほとんどのプロジェクトでは推奨できない。

- [追加情報](docs.djangoproject.com/en/3.2/topics/db/managers/)

### 6.7 Fat Modelについて
- "Fat Model" のコンセプトは、データに関連するコードをビューやテンプレートに入れるのではなく、モデルメソッド、クラスメソッド、プロパティ、マネージャメソッドにロジックを内包することを良しとする考え方のことである。
- これにより、どのViewやタスクでも同じロジックを使用することができる。
- 例えば、アイスクリームのレビューを表すモデルがあるとしたら、次のようなメソッドを付加する。
  - Review.create_review(cls, user, rating, title, description)
    - レビューを作成するためのクラスメソッドで、HTMLやRESTからモデルクラス自体が呼び出されるほか、スプレッドシートを読み込むインポートツールもある。
  - Review.product_average
    - 商品の平均評価を返す`Review`クラスのインスタンスプロパティ。
    - レビューの詳細表示で使用され、読者はページを離れることなくユーザの感想を知ることができる。
  - Review.found_useful(self, user, yes)
    - 読者がレビューを有用だと感じたかどうかを設定するメソッド。
    - HTML と REST の両方の実装で、詳細ビューとリストビューで使用される。
- 上記を見れば推測できるように、Fat Modelはプロジェクト全体でコードの再利用を向上させる優れた方法である。
- 実際、ロジックをビューやテンプレートからモデルに移すという習慣は、何年も前からプロジェクトやフレームワーク、言語を問わず増え続けている。
- ただし、必ずしも良いことばかりとは限らない。
- すべてのロジックをモデルに入れることの問題点は、モデルのコードサイズが爆発的に増大し、「神オブジェクト (god object)」と呼ばれるようになることである。
- このアンチパターンのもとでは、モデルクラスは数百、数千、酷い時には数万行のコードを持つようなクラスになる。
- 神オブジェクトはその大きさと複雑さのために理解が難しく、テストやメンテナンスも困難になる。
- ロジックをモデルに移行する際には、オブジェクト指向プログラミングの基本的な考え方の一つである「大きな問題は、小さな問題に分割すると解決しやすい」ということを忘れないようにすること。
- モデルのサイズが肥大し扱いにくくなってきたら、他のモデルで再利用できるコードや、複雑さを管理する必要のあるコードを分離していくこと。
- メソッドやクラスメソッド、プロパティはそのままに、それらに含まれるロジックを**モデルビヘイビア**や**ステートレスヘルパー関数**に移動させる。
- 以下では、この2つの手法について解説する。

#### 6.7.1 モデルビヘイビア (別名ミックスイン)
- ミックスインを使用することで、コンポジション (訳者註：委譲や合成とも呼ばれる。オブジェクトをまとめて、新しい機能を実装する手法。OOPにおいて継承の代替策になる。) やカプセル化の概念を取り入れる手法をモデルビヘイビアと呼ぶ。
- モデルは、抽象モデルからロジックを継承する。
- 詳しくは、[コンポジションの使用に関するKevin Stone氏の記事](blog.kevinastone.com/django-model-behaviors.html)を参照。

#### 6.7.2 ステートレスヘルパー関数
- ロジックをモデルからユーティリティー関数に移すことで、ロジックはより分離され、ロジックのテストが書きやすくなる。
- しかし、この関数はステートレス (状態を持たない) なので、すべての引数を渡さなければならないという欠点があります。
  - 31章で詳述する。

#### 6.7.3 モデルビヘイビア vs ヘルパー関数
- これらのテクニックはどちらも単独では完璧ではないが、適切に使用すれば効果的なものである。
- どちらが優れているか、という議論に関しても絶えず変化し続けるものである。
- モデルを構成する要素に対してテストを設けることは重要である。

### 6.8 その他の記事
- この章で説明した内容を基にした記事をいくつか紹介する。
- [Djangoモデルを取り扱う上でのルールに関するHaki Benita氏の記事](hakibenita.com/bullet-proofing-django-models)
- [Andrew BrooksのDjangoのORMに対する考察](spellbookpress.com/books/temple-of-django-database-performance/)は、モデル設計のパフォーマンスを気にしている人には一読の価値がある。

### 6.9 まとめ
- モデルはほとんどのDjangoプロジェクトの基礎となるものなので、時間をかけて慎重に設計すること。
- 正規化から始め、それを逸脱するのは他の選択肢を十分に検討した後にすること。
- 生SQLを確認することで遅くて複雑なクエリを単純化できることがある。
- キャッシュを適切に配置することでパフォーマンスの問題を解決できることがある。
- モデルを継承する場合は、具象モデルではなく抽象基底クラスを親とすることで、不必要に暗黙の結合を処理する等の混乱を避けることができる。
- `null=True`および`blank=True`のフィールドオプションを使用する上での問題点を把握しておくこと。
- django-model-utils や django-extensionsなどの有用なパッケージの使用を検討すること。
- Fat Modelはロジックをモデルにカプセル化する際の考え方だが、「神オブジェクト」になってしまわないように注意を払うこと。
