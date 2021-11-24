# Two Scoops of Django 3.x

## 6. モデルに関するベストプラクティス
- モデルはDjangoプロジェクトの基盤である。
- よく考えずに Django のモデルを書こうとすると、将来的に問題が発生する可能性が高まる。
- 私たち開発者は、自分のやっていることの影響を考えずに、モデルの追加や修正を急ぐことがままある。
- 手っ取り早く修正したり、杜撰な「一時的な」設計判断をしてコーディングすると、数ヶ月後、数年後に、おかしな回避策を余儀なくされたり、既存のデータを破壊されたりして、痛い目を見ることになる。
- Djangoで新しいモデルを追加したり、既存のモデルを修正したりするときは、このことを念頭に置くこと。
- 時間をかけて考え、可能な限り強固で健全な基盤を設計すること。
- [PACKAGE TIP] モデルを扱う上でおすすめのDjangoパッケージ
  - django-model-utilsでは、 `TimeStampedModel`のような有名なパターンを扱うことができる。
  - django-extensionsには `shell_plus`という強力な管理コマンドがあり、 インストールされている全てのアプリのModelクラスをオートロードできる。
    - 多くの機能が含まれるあまりに、小さく単一責任なアプリを作りたいという志向から外れてしまうことが欠点である。

### 6.1 基本
#### 6.1.1 モデル数の多いアプリを分割する
- 1つのアプリに20以上のModelが含まれる場合、アプリを小さく分割することを考慮すべき。
  - アプリ内のModelは5～10個に収めたい。

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
    - モデル間で重複するフィールドが多いと、メンテナンスが大変になる。
- **抽象基底クラスの場合**
  - テーブルは派生モデルに対してのみ作成される。
  - 長所
    - 共通のフィールドを抽象クラスに持たせるので、フィールドが重複しない。
    - 複数テーブル継承(後述)を採用した場合に発生する、余分なテーブルや結合のオーバーヘッドが発生しない。
  - 短所
    - 親クラスを単独で使用することはできない。
- **複数テーブル継承の場合**
  - 親と子(基底と派生)の両方にテーブルが作成される。
    - 暗黙の`OneToOneField`が親と子をリンクする。
  - 長所
    - 各モデルに独自のテーブルを与えるので、親と子のどちらにも問い合わせができる。
    - また、親オブジェクトから子オブジェクトを取得することができる。
  - 短所
    - 子テーブルへのクエリはすべての親テーブルとの結合を必要とするため、かなりのオーバーヘッドが発生する。
    - 複数テーブルの継承を使用しないことを強く推奨する(下記の[警告]を参照)。
- **プロキシモデルの場合**
  - テーブルは元のモデルに対してのみ作成される。
  - 長所
    - 異なるPythonの振る舞いを持つモデルのエイリアスを持つことができる。
  - 短所
    - モデルのフィールドは変更できない。
- [警告] 複数テーブルの継承を避ける
  - 複数テーブルの継承 (「具象継承」とも呼ばれる) は、多くの開発者にとって悪いことだと考えられており、使用しないことを強く推奨する (後に詳述する)。
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

#### 6.1.3 実践的なModel継承例：TimeStampedModel
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
- `TimeStampedModel`を抽象基底クラスとして定義し、新しいクラスを定義する際にそれを継承することで、Djangoは`migrate`の実行時に`core_timestampedmodel`テーブルを作成しない。

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
- 詳細は docs.djangoproject.com/en/3.2/topics/db/models/#model-inheritance

### 6.2 データベースマイグレーション
- Djangoには、「migrations」と呼ばれるデータベースの変更を反映するための強力なライブラリが付属している。

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
- マイグレーションは、プロジェクト内のデータや相互作用する外部コンポーネントなどの複雑な変更を予測することはできない。
- そのような場合にはPythonコードやカスタムSQLなどでマイグレートの実行を支援することができる。
- `RunPython`または`RunSQL`クラスの使用を検討すること。
  - [RunPythonクラス](docs.djangoproject.com/en/3.2/ref/migration-operations/#runpython)
  - [RunSQLクラス](docs.djangoproject.com/en/3.2/ref/migration-operations/#runsql)

### 6.3 RunPythonの一般的な問題点について
- `RunPython`の関数を書いていると、いくつかの問題点に遭遇する。

#### 6.3.1 カスタムモデルマネージャのメソッドへのアクセス
- カスタムモデルマネージャのメソッドを使って、レコードをフィルタリングしたり、 除外したり、作成したり、変更したりしたいことがある。
  - しかし、Djangoのマイグレーション機能はデフォルトではこれらのコンポーネントを除外する。
  - 幸いなことに、カスタムマネージャに`use_in_migrations = True`を追加することで、この挙動をオーバーライドすることができる。
  - [参考](docs.djangoproject.com/en/3.2/topics/migrations/#model-managers)

#### 6.3.2 カスタムモデルメソッドへのアクセス
- Djangoのマイグレーション機能はモデルをシリアライズするため、マイグレーション中にカスタムメソッドを呼び出すことはできない。
  - [参考](docs.djangoproject.com/en/3.2/topics/migrations/#historical-models)
- [警告] カスタマイズされた保存・削除メソッドについての注意点
  - モデルのsaveとdeleteメソッドをオーバーライドした場合、`RunPython`からそれらは呼ばれなくなる。致命的な問題を引き起こす可能性もあるため注意すること。

#### 6.3.3 何もしないことを表すRunPython.noop
- マイグレーションの可逆性を保つためには、RunPythonの引数`reverse_code`に処理を渡し、ロールバック時に実行される処理を定義なければならない。
- しかし、既存のデータを新しく追加されたフィールドに結合するなど、処理の中には冪等なものもあり、これらの関数のために`reverse_code`を書くことは不可能か、無意味である。
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

### 6.5 モデルの\_meta API
- \_meta APIは、以下のような特徴がある。
  - 接頭辞が「\_」であるにもかかわらず、公開されており、ドキュメント化されたAPIです。
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

### 6.6 モデルのマネージャ
- DjangoのORMを使ってモデルに問い合わせをするときはいつも、モデルマネージャと呼ばれるインタフェースを使ってデータベースとやりとりしている。

- モデルマネージャは、このモデルクラスの可能な全てのインスタンスのフルセット (テーブル内の全てのデータ) に作用して、扱いたいものを制限すると言われています。

Django はモデルクラスごとにデフォルトのモデルマネージャを用意していますが、自分で定義することもできます。
ここでは、カスタムモデルマネージャの簡単な例を示します。

さて、最初にアイスクリームのフレーバーのレビューをすべてカウントして表示し、次に公開されたものだけをカウントして表示したい場合は、次のようにします。

簡単でしょう？でも、2つ目のモデルマネージャを追加した方が、より意味があると思いませんか？
そうすれば、次のようになります。

表面的には、デフォルトのモデルマネージャーを置き換えることは、賢明なことのように見えます。
しかし残念ながら、実際のプロジェクト開発の経験から、この方法を使うときは非常に注意が必要です。
なぜでしょうか？まず、モデル継承を使用する場合、抽象ベースクラスの子供は親のモデルマネージャーを受け取り、具象ベースクラスの子供は受け取りません。

第二に、モデルクラスに最初に適用されたマネージャは、 Django がデフォルトとして扱うものです。
これは Python の通常のパターンとは大きく異なり、 QuerySets の結果が予測できないものになる可能性があります。
このことを念頭に置いて、モデルクラスでは、objects = models.Manager() をカスタムモデルマネージャの上に手動で定義してください。

警告: モデルマネージャーの操作順序を知る
objects = models.Manager()は、新しい名前のカスタムモデルマネージャーの上に必ず設定してください。
このルールは破ることができますが、高度なテクニックなので、ほとんどのプロジェクトではお勧めできません。


追加情報: docs.djangoproject.com/en/3.2/topics/db/managers/




