第8章 - モデルレイヤーの内側 
============================

これまでのところ、ページを作り、リクエストとレスポンスを処理することに多くの検討が行われてきました。しかしながらWebアプリケーションのビジネスロジックの多くはデータモデルに依存しています。symfonyのデフォルトモデルのコンポーネントはPropelのプロジェクト([http://propel.phpdb.org/trac/](http://propel.phpdb.org/trac/))で知られるオブジェクト/リレーショナルマッピング(ORM - Object/Relational Mapping)のレイヤーに基づいています。symfonyのアプリケーションにおいて、アプリケーションの開発者はオブジェクトを通してデータベースに保存されたデータにアクセスし、オブジェクトを通してこれを修正します。決して明確にデータベースにとり組みません。このことによって高い抽象性と移植性が維持されます。

この章では、オブジェクトのデータモデルを作成する方法と、Propelのデータにアクセスして修正する方法を説明します。これはPropelがsymfonyに統合されていることも実証します。

なぜORMと抽象化レイヤーを使うのか？
---------------------------------

データベースはリレーショナルです。一方でPHP 5とsymfonyはオブジェクト指向です。オブジェクト指向のコンテキストでもっとも効果的にデータベースにアクセスするには、オブジェクトをリレーショナルなロジックに変換するインターフェイスが求められます。1章で説明されたように、このインターフェイスはオブジェクトリレーショナルマッピング(ORM - Object-Relational Mapping)と呼ばれ、データにアクセスしてオブジェクトの範囲でビジネスのルールを維持するオブジェクトで構成されます。

ORMを使う主な利点は再利用性です。アプリケーションのさまざまな部分から、異なるアプリケーションからでも、データオブジェクトのメソッドを呼び出すことができます。ORMレイヤーはデータロジックもカプセル化します。たとえば、行われた投稿回数とそれらの投稿がどれだけ人気なのかに基づいてフォーラムのユーザーの評価を計算する方法です。ページがそのようなユーザーの評価を表示する必要があるとき、詳細な計算に悩むことなくsymfonyはデータモデルのメソッドを簡単に呼び出します。計算方法があとで変わった場合、必要なことはモデルの評価メソッドを修正することだけで、アプリケーションの残りの部分はそのままにできます。

レコードの代わりにオブジェクトを使い、テーブルの代わりにクラスを使うことには別の利点があります: これらによって新しいアクセサーをテーブルのカラムにかならずしもマッチしないオブジェクトに追加できます。`client`という名前のテーブルが存在し、これが`first_name`と`last_name`という2つのフィールドを持つ場合、`Name`だけを求めたい場合を考えます。オブジェクト指向の世界において、リスト8-1のように、新しいアクセサーメソッドを`Client`クラスに追加することと同じぐらい簡単です。アプリケーションの観点から、`Client`クラスの`FirstName`、`LastName`、と`Name`属性の間の違いは存在しません。クラス自身がどの属性がデータベースのカラムに対応するのかを決定できます。

リスト8-1 - アクセサーはモデルクラスの実際のテーブル構造を覆い隠す

    [php]
    public function getName()
    {
      return $this->getFirstName().' '.$this->getLastName();
    }

すべての繰り返されるデータアクセス関数とデータのビジネスロジック自身はこのようなオブジェクトのなかに保たれます。`Items`(オブジェクト)を持つ`ShoppingCart`クラスを考えてみましょう。精算のためにショッピングカートの全額を得る方法は、リスト8-2で示されるように、実際の計算をカプセル化するカスタムメソッドを書くことです。

リスト8-2 - アクセサーはデータロジックを覆い隠す

    [php]
    public function getTotal()
    {
      $total = 0;
      foreach ($this->getItems() as $item)
      {
        $total += $item->getPrice() * $item->getQuantity();
      }

      return $total;
    }

データとアクセスの手順を設けるときに考慮すべき別の重要な点があります: データベースベンダーは異なるSQL構文の方言を使います。ほかのデータベースマネジメントシステム(DBMS)に切り替えると以前のDBMSのために設計されたSQLクエリの部分を書き直さなければなりません。データベースから独立した構文を使うクエリを作り、サードパーティのコンポーネントに実際のSQLの翻訳を任せておけば、苦痛をともなわずにデータベースの構文を切り替えることができます。これがデータベースの抽象化レイヤーの目的です。これによってクエリに対して特定の構文を使うことが強制され、DBMSの固有機能に適合してSQLコードを最適化する汚い作業が推進されます。

抽象化レイヤーの主な利点は、移植性です。これによって、プロジェクトの真っ最中でも、別のデータベースに切り替えることができます。アプリケーションに対して迅速にプロトタイプを書く必要があるが、顧客が自身のニーズに最適なデータベースシステムがどれなのかを決断していない場合を考えてみましょう。SQLiteでアプリケーションの開発を始めることが可能であり、たとえば、顧客が決断をする準備ができたときに、、MySQL、PostgreSQL、またはOracleに切り替えます。設定ファイルの一行を変更すれば、アプリケーションは動きます。

symfonyはPropelをORMとして利用し、Propelはデータベースの抽象化のためにCreoleを利用します。これら2つのサードパーティのコンポーネントは、両方ともPropelの開発チームによって開発され、symfonyにシームレスに統合されているので、これらをフレームワークの一部としてみなすことができます。この章で説明しますが、これらの構文と規約はできるかぎりsymfonyのものとは異ならないように採用されました。

>**NOTE**
>symfonyのプロジェクトにおいて、すべてのアプリケーションは同じモデルを共有します。これがプロジェクトレベルの肝心な点: 共通のビジネスルールに依存するアプリケーションを再編することです。モデルがアプリケーションから独立しており、モデルのファイルがプロジェクトのrootの`lib/model/`ディレクトリに保存される理由です。

symfonyのデータベーススキーマ
-----------------------------

symfonyが使うデータオブジェクトモデルを作成するために、データベースが持つリレーショナルモデルはどんなものでもオブジェクトデータモデルに翻訳する必要があります。ORMはマッピングを行うためにリレーショナルモデルの記述が必要です。これを記述するものはスキーマ(schema)と呼ばれます。スキーマにおいて、開発者はテーブル、それらのリレーション、とカラムの特徴を定義します。

スキーマのためのsymfonyの構文はYAMLフォーマットを利用します。`schema.yml`ファイルは`myproject/config/`ディレクトリ内部に設置しなければなりません。

>**NOTE**
>symfonyはこの章の後のほうにある"schema.ymlを越えて: schema.xml"のセッションで説明されるPropelのネイティブなXML形式のスキーマも理解します。

### スキーマの例

データベースの構造をスキーマにどのように変換しますか？具体例は理解するための最良の方法です。2つのテーブル: `blog_article`と`blog_comment`を持つblogのデータベースを想像してください。テーブルの構造は図8-1で示されています。

図8-1 - blogのデータベースのテーブル構造

![blogのデータベースのテーブル構造](/images/book/F0801.png "blogのデータベースのテーブル構造")

関連する`schema.yml`ファイルはリスト8-3のようになります。

リスト8-3 - `schema.yml`のサンプル

    [yml]
    propel:
      blog_article:
        _attributes: { phpName: Article }
        id:
        title:       varchar(255)
        content:     longvarchar
        created_at:
      blog_comment:
        _attributes: { phpName: Comment }
        id:
        article_id:
        author:      varchar(255)
        content:     longvarchar
        created_at:

データベース自身(`blog`)の名前は`schema.yml`には登場しないことに注目してください。代わりに、データベースの内容は接続名(この例では`propel`)の下に記述されます。これは実際の接続設定はアプリケーションが稼働している環境に依存する可能性があるからです。たとえば、開発環境においてアプリケーションを稼働させるとき、開発データベース(たとえば`blog_dev`)にアクセスすることになりますが、運用のデータベースも同じスキーマを使います。接続設定は`databases.yml`ファイルのなかで指定されます。このファイルはこの章の後のほうの"データベースの接続"のセクションで説明します。スキーマは、データベースの抽象化を保つために、詳細な接続情報を設定に含まず、接続名だけを含みます。

### 基本的なスキーマ構文

`schema.yml`ファイルにおいて、最初のキーは接続名を表します。これは、テーブルをいくつか含むことができます。それぞれのテーブルはカラムのセットを持ちます。YAMLの構文に従い、キーはコロンで終わり、構造はインデント(1つか複数のスペース、ただしタブはなし)を通して示されます。

テーブルは`phpName`(生成されるクラスの名前)を含めて、特別な属性を持つことができます。`phpName`がテーブルに記載されていない場合、symfonyはcamelCase(キャメルケース)バージョンの名前でそのテーブルを作ります。

>**TIP**
>camelCaseの規約によれば単語からアンダースコアをとり除き、内部の単語の最初の文字を大文字にします。`blog_article`と`blog_comment`のデフォルトのcamelCaseバージョンは`BlogArticle`と`BlogComment`です。この規約名は長い単語内部の大文字がラクダのコブに見えることから由来しています。

テーブルはカラムを含みます。カラムの値は3つの異なった方法で定義できます:

  * 何も定義していない場合、symfonyはカラムの名前といくつかの規約にしたがってベストな属性を推測します。カラムの名前と規約はこの章の後のほうにある"空のカラム"のセクションで説明されます。たとえば、リスト8-3にある`id`カラムは定義する必要はありません。symfonyはそれを、オートインクリメントの整数型で、テーブルの主キーと定義します。`blog_comment`テーブルの`article_id`は`blog_article`テーブルへの外部キーとして理解されます(`_id`で終わるカラムは外部キーとして見なされ、関連するテーブルはカラム名の最初の部分にしたがって自動的に決定されます)。`created_at`という名前のカラムは自動的に`timestamp`型に設定されます。これらすべてのカラムに対して、型を指定する必要はありません。それが`schema.yml`を書くことがなぜ簡単であるかの理由の1つです。 
  * 1つの属性だけを定義する場合、これはカラム型です。symfonyは通常のカラムの型を理解します: `boolean`、`integer`、`float`、`date`、`varchar(size)`、 `longvarchar`(たとえばMySQLでは`text`に変換されます)などです。256文字を越えたテキストの内容に対しては、サイズを持たない`longvarchar`型(MySQLでは65KBを越えることはできません)を使う必要があります。`date`と`timestamp`型は通常のUnixの日付の制限を持ち、1970年1月1日以前の日付を設定することはできません。古い日付(たとえば誕生日)を設定する必要がある場合、(Unix起源の以前の)日付のフォーマットは`bu_date`と`bu_timestamp`で利用できます。
  * ほかのカラム属性を定義する必要がある場合(デフォルト値、求められた場合、など)、カラム属性を`key: value`のセットとして書きます。この拡張されたスキーマ構文はこの章の後のほうで説明します。

カラムは大文字で始まるバージョンの名前(`Id`、`Title`、`Content`など)である、`phpName`属性を持ち、たいていの場合、オーバーライドする必要はありません。

テーブルは、わずかなデータベース固有の構造の定義と同様に、明示的な外部キーとインデックスを含むことができます。もっと学ぶにはこの章の後のほうにある"拡張されたスキーマ構文"のセクションを参照してください。

モデルクラス
------------

スキーマはORMレイヤーのモデルクラスをビルドするために使われます。実行時間を節約するために、これらのクラスは`propel-build-model`という名前のコマンドラインタスクによって生成されます。

    > symfony propel-build-model

>**TIP**
>モデルをビルドしたあとで、symfonyが新しく生成されたモデルを見つけられるように、`symfony cc`でsymfonyの内部キャッシュをクリアすることを覚えておかなければなりません。

このコマンドを入力することでプロジェクトの`lib/model/om/`ディレクトリのなかでスキーマの解析と基底のデータモデルクラスの生成が実行されます:

  * `BaseArticle.php`
  * `BaseArticlePeer.php`
  * `BaseComment.php`
  * `BaseCommentPeer.php`

さらに、実際のデータモデルクラスは`lib/model/`のなかに作られます:

  * `Article.php`
  * `ArticlePeer.php`
  * `Comment.php`
  * `CommentPeer.php`

2つのテーブルだけを定義したので、8つのファイルで終わります。間違ったことは何もありませんが、いくつかの説明をする必要があります。

### 基底とカスタムクラス

2つのバージョンのデータオブジェクトを2つの異なったディレクトリに保存するのはなぜでしょうか？

おそらくカスタムメソッドとプロパティをモデルのオブジェクトに追加することが必要になります(リスト8-1の`getName()`メソッドを考えてください)。しかし、プロジェクトの開発に関しては、テーブルもしくはカラムも追加することになります。`schema.yml`ファイルを変更するたびに、`propel-build-model`を新しく呼び出してオブジェクトモデルクラスを再生成する必要があります。カスタムメソッドが実際に生成されたクラスのなかに書かれているとしたら、それらはそれぞれが生成された後に削除されます。

`lib/model/om/`ディレクトリのなかに保存される`Base`クラスはスキーマから直接生成されたものです。これらを修正すべきではありません。すべての新しいモデルのビルドによっってこれらのファイルが完全に削除されるからです。

一方で、`lib/model/`ディレクトリのなかに保存される、カスタムオブジェクトクラスは実際には`Base`クラスから継承します。`propel-build-model`タスクが既存のモデルに呼び出されるとき、これらのクラスは修正されません。ですのでここがカスタムメソッドを追加できる場所です。

リスト8-4は`propel-build-model`タスクを最初に呼び出したときに作成されたカスタムモデルクラスの例を示しています。

リスト8-4 - モデルクラスのファイルのサンプル(`lib/model/Article.php`)

    [php]
    class Article extends BaseArticle
    {
    }

これは`BaseArticle`クラスのすべてのメソッドを継承しますが、スキーマ内の修正はこれに影響を与えません。

基底クラスを拡張するカスタムクラスのメカニズムによって、データベースの最終的なリレーショナルモデルを知らなくても、コードを書き始めることができます。関連ファイルの構造によってモデルはカスタマイズ可能で発展性のあるものになります。

### オブジェクトクラスとピアクラス

`Article`と`Comment`はデータベースのなかのレコードを表すオブジェクトクラスです。これらはレコードのカラムと関連するレコードにアクセスできます。リスト8-5で示される例のように、このことは`Article`オブジェクトのメソッドを呼び出すことで、記事のタイトルを知ることができることを意味します。

リスト8-5 - レコードカラムのためのゲッターはオブジェクトクラスで利用できる

    [php]
    $article = new Article();
    ...
    $title = $article->getTitle();

`ArticlePeer`と`CommentPeer`はピアクラスです; すなわち、テーブル上で実行する静的メソッドを含むクラスです。これらはテーブルからレコードを検索する方法を提供します。リスト8-6で示されるように、通常これらのメソッドはオブジェクトもしくは関連するオブジェクトクラスのオブジェクトの集まりを返します。

リスト8-6 -　レコードを検索する静的メソッドはピアクラスのなかで利用できる

    [php]
    $articles = ArticlePeer::retrieveByPks(array(123, 124, 125));
    // $articlesはArticleクラスのオブジェクトの配列

>**NOTE**
>ビューのデータモデルの点から、ピアオブジェクトは存在できません。ピアクラスのメソッドが通常の`->`(インスタンスメソッドの呼び出し)の代わりに`::`(静的メソッドの呼び出し)で呼び出されるのはそういうわけです。

基底とカスタムバージョンのオブジェクトクラスとピアクラスを結合した結果はスキーマのなかに記述されたテーブルごとに生成された4つのクラスになります。実際には、`lib/model/map/`ディレクトリのなかに生成された5番目のクラスが存在します。このディレクトリは実行環境のために必要なテーブルについてのメタデータ情報を含みます。しかしながら、おそらくこのクラスを変更することはないので、忘れてもかまいません。

データにアクセスする
--------------------

symfonyにおいて、データはオブジェクトを通してアクセスされます。リレーショナルモデルとデータを検索し変更するSQLを使うことに慣れていたら、オブジェクトモデルのメソッドは複雑に見えるかもしれません。しかし、ひとたびデータアクセスのためのオブジェクト指向の力を味わえば、おそらくとても好きになるでしょう。

しかし最初は、同じ用語を共有していることを確認してみましょう。リレーショナルデータモデルとオブジェクトデータモデルは似たような概念を使いますが、これらはお互いに独自の命名法を持ちます:

リレーショナル     | オブジェクト指向
------------------ | ----------------
テーブル           | クラス
列、レコード       | オブジェクト
フィールド、カラム | プロパティ

### カラムの値を検索する

symfonyはモデルをビルドするとき、`schema.yml`内で定義されたそれぞれのテーブルに対して1つの基底オブジェクトクラスを作ります。それぞれのクラスはカラム定義に基づいたデフォルトのコンストラクター、アクセサー、ミューテーターを備えています: リスト8-7で示されるように、`new`、`getXXX()`、`setXXX()`メソッドはオブジェクトを作りオブジェクトのプロパティにアクセスすることを助けします。

リスト8-7 - 生成されたオブジェクトクラスのメソッド

    [php]
    $article = new Article();
    $article->setTitle('初めての記事');
    $article->setContent('これは初めての記事です。\n 皆様が楽しんで下さることを祈っています！');

    $title   = $article->getTitle();
    $content = $article->getContent();

>**NOTE**
>生成されたオブジェクトクラスは`Article`と呼ばれ、`blog_article`テーブルに渡される`phpName`です。`phpName`がスキーマで定義されなかった場合、クラスは`BlogArticle`という名前になります。アクセサーとミューテーターはcamelCaseの方言のカラム名を使うので、`getTitle()`メソッドは`title`カラムの値を検索します。

リスト8-8で示されるように、一度にいくつものフィールドを設定するには、それぞれのオブジェクトクラスに対して生成された、`fromArray()`メソッドを使用できます。

リスト8-8 - `fromArray()`メソッドは複数のセッターである

    [php]
    $article->fromArray(array(
      'title'   => '初めての記事',
      'content' => 'これは初めての記事です。\n 皆様が楽しんで下さることを祈っています！',
    ));

### 関連するレコードを検索する

`blog_comment`テーブルの`article_id`カラムは明示的に外部キーを`blog_article`テーブルに定義します。それぞれのコメントは1つの記事に関連し、1つの記事は多くのコメントを持つことができます。生成されたクラスはつぎのようにこのリレーションをオブジェクト指向の方法に翻訳する5つのメソッドを含みます:

  * `$comment->getArticle()`: 関連する`Article`オブジェクトを取得するため
  * `$comment->getArticleId()`: 関連する`Article`オブジェクトのIDを取得するため
  * `$comment->setArticle($article)`: 関連する`Article`オブジェクトを定義するため
  * `$comment->setArticleId($id)`: IDから関連する`Article`オブジェクトを定義するため
  * `$article->getComments()`: 関連する`Comment`オブジェクトを取得するため

`getArticleId()`と`setArticleId()`メソッドは開発者が`article_id`カラムを通常のカラムと見なしてリレーションを手動で設定できることを示します。しかしこれらはあまり面白いものではありません。オブジェクト指向のアプローチの利点はほかの3つのメソッドで大いにあきらかになります。リスト8-9は生成されたセッターを使う方法を示します。

リスト8-9 - 外部キーは特別なセッターに翻訳される

    [php]
    $comment = new Comment();
    $comment->setAuthor('Steve');
    $comment->setContent('うわ～、すごい、感動的だ: 最高の記事だよ！');

    // このコメントを以前の$articleオブジェクトに加える
    $comment->setArticle($article);

    // 代替構文は
    // オブジェクトがすでにデータベースに保存されている場合のみ意味をなす
    $comment->setArticleId($article->getId());

リスト8-10は生成されたゲッターを使う方法を示しています。これはモデルオブジェクトでメソッドチェーンを行うチェーンする方法も示しています。

リスト8-10 - 外部キーは特別なゲッターに翻訳される

    [php]
    // 多対一のリレーション
    echo $comment->getArticle()->getTitle();
     => 初めての記事
    echo $comment->getArticle()->getContent();
     => これは初めての記事です。
        皆様が楽しんで下さることを祈っています！

    // 一対多のリレーション
    $comments = $article->getComments();

`getArticle()`メソッドは`getTitle()`アクセサーから恩恵を受ける、`Article`クラスのオブジェクトを返します。これは開発者自身がJoinを行うよりもベターで、(`$comment->getArticleId()`の呼び出しから始まる)わずかな行のコードしか必要としません。

リスト8-10の`$comments`変数は`Comment`クラスのオブジェクトの配列を含みます。`$comments[0]`で最初のものを表示する、もしくは`foreach($comments as $comment)`によるコレクションを通して繰り返すことができます。

>**NOTE**
>モデルからのオブジェクトは規約によって単数形の名前で定義されるのはなぜなのかこれで理解できます。`blog_comment`テーブルで定義された外部キーによって`getComments()`メソッドが作成されます。`getComments()`メソッドの名前は`Comment`オブジェクトの名前に`s`を追加して名づけられたものです。モデルオブジェクトに複数形の名前をつけると、無意味な`getCommentss()`と命名されたメソッドが生成されることになります。

### データの保存と削除を行う

`new`コンストラクターを呼び出すことで、新しいオブジェクトが作成されましたが、`blog_article`テーブルのなかには実際のレコードが作成されていません。オブジェクトを修正してもデータベースは何も影響を受けません。データをデータベースに保存するために、オブジェクトの`save()`メソッドを呼び出す必要があります。

    [php]
    $article->save();

ORMはオブジェクト間のリレーションを検出するほど賢いので、`$article`オブジェクトを保存することで関連する`$comment`オブジェクトも保存されます。symfonyは保存されたオブジェクトがデータベースのなかに既存の対応部分を持つことも知っているので、`save()`への呼び出しは時々`INSERT`もしくは`UPDATE`によってSQLに翻訳されます。主キーは`save()`メソッドによって自動的に設定されるので、保存した後に、`$article->getId()`によって新しい主キーを検索することができます。

>**TIP**
>`isNew()`を呼び出すことでオブジェクトが新しいかどうかをチェックできます。修正されたオブジェクトを保存すべきかどうか判断がつかないようでしたら、`isModified()`メソッドを呼び出してください。

記事のコメントを読む場合、記事をインターネット上に公開することに関して気が変わることがあります。記事の評論家の皮肉が面白くないのであれば、リスト8-11で示されるように、`delete()`メソッドで簡単にコメントを削除できます。

リスト8-11 - 関連するオブジェクト上の`delete()`メソッドでデータベースからレコードを削除する

    [php]
    foreach ($article->getComments() as $comment)
    {
      $comment->delete();
    }

>**TIP**
>`delete()`メソッドを呼び出したあとでも、リクエストが終了するまでオブジェクトは利用できます。データベースのなかでオブジェクトが削除されることを確認するには、`isDeleted()`メソッドを呼び出してください。

### 主キーでレコードをとり出す

特定のレコードの主キーを知っている場合、関連するオブジェクトを取得するにはピアクラスの`retrieveByPk()`クラスメソッドを使います。

    [php]
    $article = ArticlePeer::retrieveByPk(7);

`schema.yml`ファイルは`id`フィールドを`blog_article`の主キーとして定義します。このステートメントは実際には`id`が7である記事を返します。主キーを使いましたので、あなたは1つのレコードだけが返されることを知っています; `$article`変数は`Article`クラスのオブジェクトを含みます。

いくつかの場合において、主キーは複数のカラムで構成されることがあります。そのような場合において、`retrieveByPK()`メソッドは複数のパラメーターをとり、それぞれの主キーのカラムに対してパラメーターは1つです。

生成された`retrieveByPKs()`メソッドを呼び出すことで、主キーに基づいて複数のオブジェクトを選ぶこともできます。`retireveByPKs()`メソッドはパラメーターとして主キーの配列を必要とします。

### Criteriaでレコードを検索する

複数のレコードを検索したいとき、検索したいオブジェクトに対応するピアクラスの`doSelect()`メソッドを呼び出す必要があります。たとえば、`Article`クラスのオブジェクトを検索するには、`ArticlePeer::doSelect()`を呼び出します。

`doSelect()`メソッドの最初のパラメーターは`Criteria`クラスのオブジェクトです。`Criteria`クラス(訳注：日本語で「基準」を意味する)はデータベースの抽象化のためにSQLなしで定義されたシンプルなクエリの定義クラスです。

空の`Criteria`はすべてのクラスのオブジェクトを返します。たとえば、リスト8-12で示されるコードはすべての記事を検索します。

リスト8-12 - 空のCriteria -- `doSelect()`を持つCriteriaでレコードを検索する

    [php]
    $c = new Criteria();
    $articles = ArticlePeer::doSelect($c);

    // 上記のコードはつぎのSQLクエリになります
    SELECT blog_article.ID, blog_article.TITLE, blog_article.CONTENT,
           blog_article.CREATED_AT
    FROM   blog_article;

>**SIDEBAR**
>ハイドレイティング(hydrating)
>
>`::doSelect()`への呼び出しは実際にはシンプルなSQLクエリよりはるかに強力です。最初に、SQLは選択したDBMSのために最適化されます。2番目に、`Criteria`に渡されるどの値もSQLコードに統合されるまえにエスケープされ、SQLインジェクションのリスクが予防されます。3番目に、メソッドは、結果セットではなく、オブジェクトの配列を返します。ORMはデータベースの結果セットに基づいてオブジェクトを自動的に作成し投入します。このプロセスはハイドレイティング(hydrating)と呼ばれます。

より複雑なオブジェクトを選択するには、`WHERE`、`ORDER BY`、`GROUP BY`、およびほかのSQLステートメントと同等のものが必要です。`Criteria`オブジェクトはこれらすべての条件のためのメソッドとパラメーターを持ちます。たとえば、リスト8-13のように、Steveによって書かれ、日付順に並べられた、すべてのコメントを取得するには、`Criteria`をビルドします。

リスト8-13 - `doSelect()`を持つ`Criteria`によってレコードを検索する -- `Criteria`は条件つき

    [php]
    $c = new Criteria();
    $c->add(CommentPeer::AUTHOR, 'Steve');
    $c->addAscendingOrderByColumn(CommentPeer::CREATED_AT);
    $comments = CommentPeer::doSelect($c);

    // 上記のコードはつぎのようなSQLクエリになる
    SELECT blog_comment.ARTICLE_ID, blog_comment.AUTHOR, blog_comment.CONTENT,
           blog_comment.CREATED_AT
    FROM   blog_comment
    WHERE  blog_comment.author = 'Steve'
    ORDER BY blog_comment.CREATED_AT ASC;

`add()`メソッドへのパラメーターとして渡されるクラスの定数はプロパティ名を参照します。これらの定数はカラム名の大文字バージョンから名づけられます。たとえば、`blog_article`テーブルの`content`カラムを扱うには、`ArticlePeer::CONTENT`クラス定数を使います。

>**NOTE**
>なぜ`blog_comment.AUTHOR`の代わりに`CommentPeer::AUTHOR`を使うのか？ SQLクエリに出力される方法はどちらなのか？データベースの`author`フィールドの名前を`contributor`に変更する必要がある場合を考えてみましょう。`blog_comment.AUTHOR`を使う場合、すべての呼び出しへのモデルを変更しなければなりません。一方で、`CommentPeer::AUTHOR`を使う場合、`schema.yml`内のカラム名を変更し、`phpName`を`AUTHOR`として保存し、モデルをリビルドする必要があるだけです。

テーブル8-1はSQLの構文と`Criteria`オブジェクトの構文を比較します。

テーブル8-1 - SQLの構文と`Criteria`オブジェクトの構文

SQL                                                          | Criteria
------------------------------------------------------------ | -----------------------------------------------
`WHERE column = value`                                       | `->add(column, value);`
`WHERE column <> value`                                      | `->add(column, value, Criteria::NOT_EQUAL);`
**ほかの比較演算子**                                           | 
`> , <`                                                      | `Criteria::GREATER_THAN, Criteria::LESS_THAN`
`>=, <=`                                                     | `Criteria::GREATER_EQUAL, Criteria::LESS_EQUAL`
`IS NULL, IS NOT NULL`                                       | `Criteria::ISNULL, Criteria::ISNOTNULL`
`LIKE, ILIKE`                                                | `Criteria::LIKE, Criteria::ILIKE`
`IN, NOT IN`                                                 | `Criteria::IN, Criteria::NOT_IN`
**ほかのSQLキーワード**                                        | 
`ORDER BY column ASC`                                        | `->addAscendingOrderByColumn(column);`
`ORDER BY column DESC`                                       | `->addDescendingOrderByColumn(column);`
`LIMIT limit`                                                | `->setLimit(limit)`
`OFFSET offset`                                              | `->setOffset(offset) `
`FROM table1, table2 WHERE table1.col1 = table2.col2`        | `->addJoin(col1, col2)`
`FROM table1 LEFT JOIN table2 ON table1.col1 = table2.col2`  | `->addJoin(col1, col2, Criteria::LEFT_JOIN)`
`FROM table1 RIGHT JOIN table2 ON table1.col1 = table2.col2` | `->addJoin(col1, col2, Criteria::RIGHT_JOIN)`

>**TIP**
>生成されたクラスで利用可能なメソッドがどれなのか見つけて理解するためのベストの方法は、生成後に`lib/model/om/`フォルダーのなかの`Base`ファイルを見ることです。メソッドの名前はとても明白ですが、これらに関する詳細なコメントが必要な場合、`config/propel.ini`ファイル内の`propel.builder.addComments`パラメーターを`true`に設定して、モデルをリビルドします。

リスト8-14は複数の条件を持つ`Criteria`のほかの例を示します。日付順に並べ替えられた"enjoy"の単語を含む記事上のSteveによるすべてのコメントを検索します。

リスト8-14 - `doSelect()`を持つ`Criteria`によってレコードを検索する別の例-- `Criteria`は条件つき

    [php]
    $c = new Criteria();
    $c->add(CommentPeer::AUTHOR, 'Steve');
    $c->addJoin(CommentPeer::ARTICLE_ID, ArticlePeer::ID);
    $c->add(ArticlePeer::CONTENT, '%enjoy%', Criteria::LIKE);
    $c->addAscendingOrderByColumn(CommentPeer::CREATED_AT);
    $comments = CommentPeer::doSelect($c);

    // 上記のコードはつぎのようなSQLクエリになる
    SELECT blog_comment.ID, blog_comment.ARTICLE_ID, blog_comment.AUTHOR,
           blog_comment.CONTENT, blog_comment.CREATED_AT
    FROM   blog_comment, blog_article
    WHERE  blog_comment.AUTHOR = 'Steve'
           AND blog_article.CONTENT LIKE '%enjoy%'
           AND blog_comment.ARTICLE_ID = blog_article.ID
    ORDER BY blog_comment.CREATED_AT ASC

SQLはとても複雑なクエリを開発できるシンプルな言語なので、`Criteria`オブジェクトはどんな複雑なレベルの条件を処理できます。しかし、多くの開発者は条件をオブジェクト指向のロジックに翻訳するまえに最初にSQLを考えるので、`Criteria`を最初に把握するのは難しいでしょう。これを理解するベストの方法は具体例とサンプルのアプリケーションから学ぶことです。たとえば、symfonyのプロジェクトのWebサイトは多くの方法であなたを啓発する`Criteria`の開発例で満たされています。

`doSelect()`メソッドに加えて、すべてのピアクラスは`doCount()`メソッドを持ちます。`doCount()`メソッドはパラメーターとして渡された基準を満たすレコードの数をそのままカウントして、カウント数を整数として返します。この場合、返すオブジェクトが存在しないので、ハイドレイティングの処理は行われません。また`doCount()`メソッドは`doSelect()`よりも速いです。

ピアクラスは`Criteria`をパラメーターとして必要とする`doDelete()`、`doInsert()`と`doUpdate()`メソッドも提供します。これらのメソッドによって`DELETE`クエリ、`INSERT`クエリ、と`UPDATE`クエリをデータベースに発行できます。これらのPropelのメソッドの詳細に関しては生成されたモデルのピアクラスを確認してください。

最後に、最初に返されたオブジェクトが欲しい場合、`doSelect()`をすべて`doSelectOne()`呼び出しで置き換えます。これは`Criteria`が1つの結果だけを返すことを知っているときにあてはまる場合で、利点はこのメソッドがオブジェクトの配列ではなくオブジェクトを返すことです。

>**TIP**
>`doSelect()`クエリが多数の結果を返すとき、レスポンスのなかでその部分集合だけを表示したいことがあります。symfonyは結果のパジネーションを自動化する`sfPropelPager`と呼ばれるページャークラスを提供します。詳しい情報と使いかたの例は[http://www.symfony-project.org/cookbook/1_0/pager](http://www.symfony-project.org/cookbook/1_0/pager)を参照してください。

### 生のSQLクエリを使う

時々、オブジェクトを検索する必要はないが、データベースによって算出された総合的な結果だけが欲しいことがあります。たとえば、すべての記事の最新の作成日時を取得するために、すべての記事を検索し、配列でループしても無意味です。結果だけを返すようにデータベースに求めるほうが望ましいです。なぜなら、これはオブジェクトのハイドレイティングの処理をスキップするからです。

一方で、データベース抽象化の利点を失いたくないので、データベース管理のためにPHPのコマンドを直接呼び出したくない場合があります。これはORM(Propel)を回避し、データベースの抽象化(Creole)を回避しないことが必要であることを意味します。

Creoleでデータベースにクエリを行うにはつぎの作業を行う必要があります:

  1. データベースの接続を取得する。
  2. クエリの文字列をビルドする。
  3. それからステートメントを作る。
  4. ステートメントの実行から得られた結果セットをイテレートする

何を言っているのかよくわからないのでしたら、おそらくリスト8-15のコードを見ればより明確になるでしょう。

リスト8-15 - CreoleでカスタムSQLクエリ

    [php]
    $connection = Propel::getConnection();
    $query = 'SELECT MAX(%s) AS max FROM %s';
    $query = sprintf($query, ArticlePeer::CREATED_AT, ArticlePeer::TABLE_NAME);
    $statement = $connection->prepareStatement($query);
    $resultset = $statement->executeQuery();
    $resultset->next();
    $max = $resultset->getInt('max');

PropelのSelect機能と同じように、Creoleのクエリを使い始めたときこれらは扱いにくいです。繰り返しますが、既存のアプリケーションとチュートリアルからの例は正しい方法を示します。

>**CAUTION**
>このプロセスを回避しデータベースに直接アクセスする場合、Creoleによって提供されたセキュリティと抽象化を失うリスクを負うことになります。Creoleの方法は長いですが、パフォーマンス、ポータビリティ、アプリケーションのセキュリティを保証するよい習慣が強制されます。これは信用できないソース(たとえばインターネットのユーザー)からのパラメーターを含むクエリにとりわけあてはまります。Creoleは必要なすべてのエスケープを行い、データベースを安全にします。データベースに直接アクセスすることはSQLインジェクション攻撃のリスクが存在する状態に晒されることを意味します。

### 特別な日付カラムを使う

通常、テーブルが`created_at`と呼ばれるカラムを持つとき、レコードの作成日時のタイムスタンプを保存するためにこのカラムは使われます。同じことが`updated_at`カラムにもあてはまります。レコード自身が更新されるたびに現在の時間の値に更新されます。

吉報はsymfonyがこれらのカラムを認識し更新を扱うことです。`created_at`カラムと`updated_at`カラムを手動で設定する必要はありません; リスト8-16で示されるように、これらは自動的に更新されます。同じことが`created_on`と`updated_on`カラムにもあてはまります。

リスト8-16 - `created_at`と`updated_at`カラムは自動的に処理される

    [php]
    $comment = new Comment();
    $comment->setAuthor('Steve');
    $comment->save();

    // 作成時点の日付を表示する
    echo $comment->getCreatedAt();
      => [date of the database INSERT operation]

加えて、日付カラムのためのゲッターは日付フォーマットを引数として受けとります:

    [php]
    echo $comment->getCreatedAt('Y-m-d');

>**SIDEBAR**
>**データレイヤーへのリファクタリング**
>
>symfonyを開発しているとき、アクションのドメインロジックのコードを書くことが始まるのがよくあります。しかしながらデータベースクエリとモデル操作のコードはコントローラーレイヤーに保存すべきではなく、データに関連するすべてのロジックはモデルレイヤーに移動させるべきです。アクションの複数の場所で同じリクエストを行う必要があるときは、関連コードをモデルに移動させることを考えてください。この作業を行うことでアクションのコードを短くて読みやすい状態に保つための助けになります。
>
>たとえば、blogで(リクエストパラメーターとして渡される)任意のタグに対してもっとも人気のある記事を検索するために必要なコードを想像してください。このコードはアクションのなかには存在しませんが、モデルのなかに存在します。実際、テンプレートのなかでこの記事の一覧を表示する必要がある場合、アクションはつぎのようなシンプルなものになります:
>
>     [php]
>     public function executeShowPopularArticlesForTag()
>     {
>       $tag = TagPeer::retrieveByName($this->getRequestParameter('tag'));
>       $this->foward404Unless($tag);
>       $this->articles = $tag->getPopularArticles(10);
>     }
>
>アクションはリクエストパラメーターから`Tag`クラスのオブジェクトを作ります。それからデータベースにクエリを行うために必要なすべてのコードはこのクラスの`getPopularArticles()`メソッドに設置されます。これによってアクションはより読みやすくなり、モデルのコードは別のアクションのなかで簡単に再利用できます。
>
>コードをより適切な場所に移動させることはリファクタリングの技術の1つです。頻繁にこの作業を行えば、コードは維持しやすくほかの開発者にわかりやすくなります。データレイヤーにリファクタリングを行うときのよい経験則はアクションのコードに含まれるPHPのほとんどのコードが10行を越えないことです。

データベースの接続
------------------

データモデルは使うデータベースから独立していますが、最終的にはデータベースを使うことになります。プロジェクトのデータベースにリクエストを送るためにsymfonyから求められる最小限の情報は名前、アクセスコードとデータベースのタイプです。これらの接続設定は`config/`ディレクトリに設置された`databases.yml`に入力されます。リスト8-17はこのようなファイルの例を示します。

リスト8-17 - データベースの接続設定のサンプル(`myproject/config/databases.yml`)

    [yml]
    prod:
      propel:
        param:
          hostspec:           mydataserver
          username:           myusername
          password:           xxxxxxxxxx

    all:
      propel:
        class:                sfPropelDatabase
        param:
          phptype:            mysql     # デフォルトのベンダー
          hostspec:           localhost
          database:           blog
          username:           login
          password:           passwd
          port:               80
          encoding:           utf8      # テーブルに対するデフォルトの文字集合
          persistent:         true      # 永続的接続を使う

接続設定は環境に依存します。アプリケーションにおいて`prod`、`dev`、と `test`環境、そのほかの環境などに対して相異なる設定を定義できます。`apps/myapp/config/databases.yml`などのアプリケーション固有のファイルで異なる設定を行うことで、この設定をアプリケーションごとにオーバーライドできます。たとえば、フロントエンドとバックエンドのアプリケーションに対して異なるセキュリティ方針を持たせるために、またはデータベースを扱うためにデータベースで異なる権限を持ったデータベースのユーザーをいくつか定義するために、このアプローチを利用できます。

それぞれの環境に対して、多くの接続を定義できます。それぞれの接続は同じ名前がラベルづけされたスキーマを参照します。リスト8-17の例において、propelの接続はリスト8-3の`propel`スキーマを参照します。

`phptype`パラメーターの認められる値はCreoleによってサポートされるデータベースシステムです:

  * `mysql`
  * `mssql`
  * `pgsql`
  * `sqlite`
  * `oracle`

`hostspec`、`database`、`username`、`password`はデータベースの通常の設定です。これらはデータソースネーム(DSN - Data Source Name)としてより短い記法で書くこともできます。リスト8-18はリスト8-17の`all:`セクションと同等です。

リスト8-18 - 省略記法によるデータベースの接続設定

    [yml]
    all:
      propel:
        class:          sfPropelDatabase
        param:
          dsn:          mysql://login:passwd@localhost/blog

SQLiteデータベースを使う場合、`hostspec`パラメーターはデータベースファイルのパスに設定しなければなりません。たとえば、blogデータベースを`data/blog.db`に保存する場合、`databases.yml`ファイルはリスト8-19のようになります。

リスト8-19 - SQliteのためのデータベースの接続設定はファイルパスをホストとして使う

    [yml]
    all:
      propel:
        class:          sfPropelDatabase
          param:
            phptype:  sqlite
            database: %SF_DATA_DIR%/blog.db

モデルを拡張する
----------------

生成されたモデルメソッドはすばらしいものですが、十分ではないことはよくあることです。独自のビジネスロジックを実装すると同時に、新しいメソッドを追加するか既存のメソッドをオーバーライドすることで、ビジネスロジックを拡張する必要があります。

### 新しいメソッドを追加する

新しいメソッドを`lib/model/`ディレクトリのなかに生成された空のモデルクラスに追加できます。現在のオブジェクトのメソッドを呼び出すには`$this`を使い、現在のクラスの静的メソッドを呼び出すには`self::`を使います。カスタムクラスが`lib/model/om/`ディレクトリのなかに設置された`Base`クラスからメソッドを継承することを覚えておいてください。

たとえば、リスト8-20で示されるように、リスト8-3に基づいて生成された`Article`オブジェクトに対して、`Article`クラスのオブジェクトをechoすることでタイトルを表示できるように、`__toString()`マジックメソッドを追加できます。

リスト8-20 - モデルをカスタマイズする(`lib/model/Article.php`)

    [php]
    class Article extends BaseArticle
    {
      public function __toString()
      {
        return $this->getTitle();  // getTitle()はBaseArticleから継承される
      }
    }

ピアクラスを拡張することもできます。たとえば、リスト8-21で示されるように、記事作成の日付順で並べられたすべての記事を検索するにはメソッドを追加します。

リスト8-21 - モデルをカスタマイズする(`lib/model/ArticlePeer.php`)

    [php]
    class ArticlePeer extends BaseArticlePeer
    {
      public static function getAllOrderedByDate()
      {
        $c = new Criteria();
        $c->addAscendingOrderByColumn(self::CREATED_AT);
        return self::doSelect($c);

      }
    }

リスト8-22で示されるように、新しいメソッドは生成されたメソッドと同じ方法で利用できます。

リスト8-22 -カスタムモデルメソッドを利用することは生成されたメソッドを利用することと似ている

    [php]
    foreach (ArticlePeer::getAllOrderedByDate() as $article)
    {
      echo $article;      // __toString()マジックメソッドを呼び出す
    }

### 既存のメソッドをオーバーライドする

`Baseクラス`内部の生成されたいくつかのメソッドがあなたの要件に合わない場合、これらのメソッドをカスタムクラスでオーバーライドすることもできます。同じメソッドのシグネイチャ(すなわち、同じ数の引数)を使っていることを確認してください。

たとえば、`$article->getComments()`メソッドは`Comment`オブジェクトの配列を順不同で返します。最新のコメントが一番最初になるように作成時の日付順でコメントを並べたい場合、リスト8-23で示されるように`getComments()`メソッドをオーバーライドします。オリジナルの`getComments()`メソッド(`lib/model/om/BaseArticle.php`で見つかる)はパラメーターとして基準の値と接続の値が必要なので、あなたの関数が同じことを行わなければならないことに注意してください。

リスト8-23 - 既存のモデルメソッドをオーバーライドする(`lib/model/Article.php`)

    [php]
    public function getComments($criteria = null, $con = null)
    {
      if (is_null($criteria))
      {
        $criteria = new Criteria();
      }
      else
      {
        // PHP 5ではオブジェクトは参照で渡されるので、オリジナルを修正することを避けるには、cloneしなければならない
        $criteria = clone $criteria;
      }
      $criteria->addDescendingOrderByColumn(CommentPeer::CREATED_AT);

      return parent::getComments($criteria, $con);
    }

カスタムメソッドは最終的に親の`Base`クラスの1つを呼び出します。これはよい習慣です。しかしながら、完全にそれを回避し、望む結果を返すことができます。

### モデルのビヘイビアーを使う

いくつかのモデルを修正したものは一般的で再利用できます。たとえば、モデルオブジェクトをソート可能にしてオブジェクトの保存が同時に起きることを防止する楽観的ロック(オプティミスティックロック)にすることは多くのクラスに追加できる一般的な拡張機能です。

symfonyはこれらの拡張機能をビヘイビアーにまとめます。ビヘイビアー(behavior)とは追加メソッドをモデルクラスに提供する外部クラスです。モデルクラスはすでにフックを含み、symfonyは`sfMixer`(詳細は17章を参照)の方法によってビヘイビアーを拡張する方法を知っています。

モデルクラスのビヘイビアーを有効にするには、`config/propel.ini`ファイルの設定の1つを修正しなければなりません:

    propel.builder.AddBehaviors = true     // デフォルト値はfalse

symfonyにデフォルトで搭載されているビヘイビアーは存在しませんが、それらはプラグインを通してインストールできます。いったんビヘイビアーのプラグインがインストールされると、クラスを1行でビヘイビアーに割り当てることができます。たとえば、`sfPropelParanoidBehaviorPlugin`をアプリケーションにインストールする場合、`Article.class.php`の最後の行につぎのコードを追加すればこのビヘイビアーを持つ`Article`クラスを拡張できます:

    [php]
    sfPropelBehavior::add('Article', array(
      'paranoid' => array('column' => 'deleted_at')
    ));

モデルをリビルドしたあとで、`sfPropelParanoidBehavior::disable()`でビヘイビアーを一時的に無効にしないかぎり、削除された`Article`オブジェクトはORMを使うクエリには見えないだけで、データベースに保存されたままになります。

ビヘイビアーを見つけるには公式サイトのwikiにあるプラグインのリストを確認してください([http://trac.symfony-project.org/wiki/SymfonyPlugins#Behaviors](http://trac.symfony-project.org/wiki/SymfonyPlugins#Behaviors))。それぞれのプラグインには独自のドキュメントとインストールガイドがあります。

スキーマの拡張構文
------------------

リスト8-3で示されるように、`schema.yml`ファイルをシンプルにすることができます。しかしながらリレーショナルモデルは複雑であることがよくあります。それがスキーマがほとんどすべての場合を扱うことができる拡張された構文を持つ理由です。

### 属性

リスト8-24で示されるように、接続とテーブルは固有の属性を持つことができます。これらは`_attributes`キーの下で設定します。

リスト8-24 - 接続とテーブルのための属性

    [yml]
    propel:
      _attributes:   { noXsd: false, defaultIdMethod: none, package: lib.model }
      blog_article:
        _attributes: { phpName: Article }

コード生成が行われるまえにスキーマを検証したい場合を考えます。これを行うには、接続に対して`noXSD`属性を無効にします。接続は`defaultIdMethod`属性もサポートします。何も提供されない場合、IDを生成するデータベースのネイティブなメソッドが使われます。たとえば、MySQLに対しては`autoincrement`、PostgreSQLに対しては`sequences`です。ほかのとりうる値は`none`です。

`package`属性は名前空間のようなものです; これは生成されたクラスが保存される場所のパスを決めます。デフォルト値は`lib/model/`ですが、サブパッケージのモデルを編成するために変更できます。たとえば、コアのビジネスクラスとデータベースに保存された統計エンジンを定義するクラスを同じディレクトリのなかで混在させたくない場合、`lib.model.business`パッケージと`lib.model.stats`パッケージで2つのスキーマを定義してください。

テーブルをマッピングする生成クラスの名前を設定するために使われる、`phpName`テーブル属性はすでに見ました。

リスト8-25で示されるように、ローカライズされた内容を含むテーブル(すなわち、国際化のために、関連するテーブルのなかに存在する、複数のバージョンの内容)も2つの追加属性をとります(詳細は13章を参照)。

リスト8-25 - 国際化テーブルのための属性

    [yml]
    propel:
      blog_article:
        _attributes: { isI18N: true, i18nTable: db_group_i18n }

>**SIDEBAR**
>**複数のスキーマを扱う**
>
>アプリケーションごとに複数のスキーマを持つことができます。symfonyは`config/`フォルダーの`schema.yml`もしくは`schema.yml`で終わるすべてのファイルを考慮に入れます。アプリケーションが多くのテーブルを持つ場合、もしくはテーブルが同じ接続を共有しない場合、このアプローチがとても便利であることがわかります。
>
>つぎの2つのスキーマを考えてください:
>
>
>      // config/business-schema.ymlにおいて
>      propel:
>        blog_article:
>          _attributes: { phpName: Article }
>        id:
>        title: varchar(50)
>
>      // config/stats-schema.ymlにおいて
>      propel:
>        stats_hit:
>          _attributes: { phpName: Hit }
>        id:
>        resource: varchar(100)
>        created_at:
>
>
>同じ接続を共有する両方のスキーマ(`propel`)と、`Article`クラスと`Hit`クラスは同じ`lib/model/`ディレクトリのもとで生成されます。あたかも1つだけのスキーマを書いたようにすべての物事が行われます。
>
>異なる接続(たとえば、`databases.yml`のなかで定義される`propel`と`propel_bis`)を使う異なるスキーマを持つことが可能で生成クラスをサブディレクトリに分類できます。
>
>      [yml]
>      // config/business-schema.ymlにおいて
>      propel:
>        blog_article:
>          _attributes: { phpName: Article, package: lib.model.business }
>        id:
>        title: varchar(50)
>
>      // config/stats-schema.ymlにおいて
>      propel_bis:
>        stats_hit:
>          _attributes: { phpName: Hit, package: lib.model.stat }
>        id:
>        resource: varchar(100)
>        created_at:
>
>
>多くのアプリケーションは複数のスキーマを使います。とりわけ、プラグインのなかにはアプリケーション独自のクラスに干渉しないようにプラグイン独自のスキーマとパッケージを持つものがあります(詳細は17章を参照)。

### カラムの詳細

基本構文は選択肢を2つ与えてくれます; (空の値を渡すことで)symfonyに名前からカラムの特徴を推測させるか、1つの`type`キーワードで型を定義するかです。リスト8-26はこれらの選択肢のお手本を示しています。

リスト8-26 - 基本的なカラム属性

    [yml]
    propel:
      blog_article:
        id:                 # symfonyに仕事を任せる
        title: varchar(50)  # あなた自身が型を指定する

しかしながら、カラムに対してもっと多くのことを定義できます。もし行う場合、リスト8-27で示されるように、カラムの設定を連想配列として定義する必要があります。

リスト8-27 - 複雑なカラム属性

    [yml]
    propel:
      blog_article:
        id:       { type: integer, required: true, primaryKey: true, autoIncrement: true }
        name:     { type: varchar(50), default: foobar, index: true }
        group_id: { type: integer, foreignTable: db_group, foreignReference: id, onDelete: cascade }

カラムのパラメーターはつぎのとおりです:

  * `type`: カラムの型。選択肢は`boolean`、`tinyint`、`smallint`、`integer`、`bigint`、`double`、`float`、`real`、`decimal`、`char`、`varchar(size)`、`longbarchar`、`date`、`time`、`timestamp`、`bu_date`、`bu_timestamp`、`blob`、と`clob`です。
  * `required`: ブール値。カラムをrequiredにしたい場合これを`true`に設定します。
  * `size`: 型がサポートするフィールドのサイズもしくは長さ
  * `scale`: decimalデータ型使用のための小数位(sizeも指定しなければなりません)
  * `default`: デフォルト値。
  * `primaryKey`: ブール値。主キーに対してこれを`true`に設定します。
  * `autoIncrement`: ブール値。オートインクリメントされた値を取る必要のある`integer`型のカラムに対してこれを`true`に設定します。
  * `sequence`: `autoIncrement`カラムに対してシーケンスを使うデータベース(たとえばPostgreSQL、Oracle)のためのシーケンス名。
  * `index`: ブール値。シンプルなインデックスが欲しい場合は`true`に、カラムにユニークなインデックスを作りたい場合は`unique`に設定します。
  * `foreignTable`: 別のテーブルに外部キーを作るために使われる、テーブル名。
  * `foreignReference`: `foreingTable`経由で外部キーが定義された場合の関連カラムの名前。
  * `onDelete`: 関連テーブルに存在するレコードが削除されたときにアクションを起動させるために指定します。`setnull`に設定したとき、外部キーのカラムは`null`に設定されます。`cascade`に設定したとき、レコードは削除されます。データベースエンジンがsetのビヘイビアーをサポートしない場合、ORMがエミュレートします。これは`foreignTable`と`foreingReference`を持つカラムだけに該当します。
  * `isCulture`: ブール値。ローカライズされた内容テーブルにあるcultureのカラムに対してこれを`true`に設定してください(13章を参照)。

### 外部キー

`foreignTable`と`foreignReference`カラム属性の代わりに、外部キーをテーブルの`_foreignKeys:`キーの下に追加できます。リスト8-28のスキーマは`blog_user`テーブルの`id`カラムにマッチする、`user_id`カラムの上に外部キーを作ります

リスト8-28 - 外部キーの代替構文

    [yml]
    propel:
      blog_article:
        id:
        title:   varchar(50)
        user_id: { type: integer }
        _foreignKeys:
          -
            foreignTable: blog_user
            onDelete:     cascade
            references:
              - { local: user_id, foreign: id }

リスト8-29で示されるように、この代替構文は複数参照を持つ外部キーに対して外部キーに名前を与えるために役立ちます。

リスト8-29 - 複数参照の外部キーに適用された外部キーの代替構文

        _foreignKeys:
          my_foreign_key:
            foreignTable:  db_user
            onDelete:      cascade
            references:
              - { local: user_id, foreign: id }
              - { local: post_id, foreign: id }

### インデックス

`index`カラム属性の代わりに、テーブル内の`_indexes:`キーの下にインデックスを追加できます。ユニークインデックスを定義したい場合、`_uniques:`ヘッダーを代わりに使わなければなりません。リスト8-30はインデックスのための代替構文を示しています。

リスト8-30 - インデックスとユニークインデックスの代替構文

    [yml]
    propel:
      blog_article:
        id:
        title:            varchar(50)
        created_at:
        _indexes:
          my_index:       [title(10), user_id]
        _uniques:
          my_other_index: [created_at]

代替構文は複数のカラムで構築されたインデックスに対してのみ役立ちます。

### 空のカラム

値を持たないカラムに遭遇するとき、symfonyはいくつかの手品を行い、それ自身の値を追加します。空のカラムに追加された詳細内容に関してリスト8-31をご覧ください。

リスト8-31 - カラムの名前から推定されたカラムの詳細内容

    // idという名前で空のカラムは主キーと見なされる
    id:         { type: integer, required: true, primaryKey: true, autoIncrement: true }

    // XXX_idという名前で空のカラムは外部キーと見なされる
    foobar_id:  { type: integer, foreignTable: db_foobar, foreignReference: id }

    // created_at、updated at、created_onとupdated_onという名前を持つ空のカラムは
    // 日付と見なされ自動的にtimestamp型をとる
    created_at: { type: timestamp }
    updated_at: { type: timestamp }

外部キーに対して、symfonyはカラムの名前の始めで同じ`phpName`を持つテーブルを探し、1つが見つかったら、このテーブルの名前を`foreignTable`としてとります。

### 国際化テーブル

symfonyは関連テーブル内で内容の国際化機能のサポートをします。このことは、内容の題目を国際化するとき、2つのテーブルに個別に保存されることを意味します: 1つは変わらないカラムでもう1つが国際化されたカラムです。

`schema.yml`ファイルにおいて、テーブルを`footbar_i18n`と名づけたときにすべてが暗黙のうちに行われます。たとえば、国際化した内容のメカニズムが働くようにリスト8-32で示されるスキーマはカラムとテーブル属性を自動的に備えています。内部では、symfonyはあたかもリスト8-33のように書かれたものとして理解します。13章で国際化に関して詳しい説明が行われます。

リスト8-32 - 暗黙的な国際化のメカニズム

    [yml]
    propel:
      db_group:
        id:
        created_at:

      db_group_i18n:
        name:        varchar(50)

リスト8-33 - 明示的な国際化のメカニズム

    [yml]
    propel:
      db_group:
        _attributes: { isI18N: true, i18nTable: db_group_i18n }
        id:
        created_at:

      db_group_i18n:
        id:       { type: integer, required: true, primaryKey: true,foreignTable: db_group, foreignReference: id, onDelete: cascade }
        culture:  { isCulture: true, type: varchar(7), required: true,primaryKey: true }
        name:     varchar(50)

### schema.ymlを越えて: schema.xml

実際のところ、`schema.yml`フォーマットはsymfonyの内部に存在します。`propel-command`を呼び出すとき、symfonyは実際にこのファイルを`generated-schema.xml`ファイルに翻訳します。このXMLファイルは実際にはモデル上のタスクを実行するためにPropelによって求められるタイプのファイルです。

`schema.xml`ファイルはYAMLの同等のものとして同じ情報を含みます。たとえば、リスト8-3はリスト8-34で示されるようなXMLファイルに変換されます。

リスト8-34 - リスト8-3に対応する`schema.yml`のサンプル

    [xml]
    <?xml version="1.0" encoding="UTF-8"?>
     <database name="propel" defaultIdMethod="native" noXsd="true" package="lib.model">
        <table name="blog_article" phpName="Article">
          <column name="id" type="integer" required="true" primaryKey="true"autoIncrement="true" />
          <column name="title" type="varchar" size="255" />
          <column name="content" type="longvarchar" />
          <column name="created_at" type="timestamp" />
        </table>
        <table name="blog_comment" phpName="Comment">
          <column name="id" type="integer" required="true" primaryKey="true"autoIncrement="true" />
          <column name="article_id" type="integer" />
          <foreign-key foreignTable="blog_article">
            <reference local="article_id" foreign="id"/>
          </foreign-key>
          <column name="author" type="varchar" size="255" />
          <column name="content" type="longvarchar" />
          <column name="created_at" type="timestamp" />
        </table>
     </database>

`schema.xml`フォーマットの記述方法はPropelプロジェクトWebサイト([http://propel.phpdb.org/docs/user_guide/chapters/appendices/AppendixB-SchemaReference.html](http://propel.phpdb.org/docs/user_guide/chapters/appendices/AppendixB-SchemaReference.html))ドキュメントと"Getting Started"のセクションで見ることができます。

YAMLフォーマットはスキーマの読み書きをシンプルに保つために設計されましたが、 トレードオフはもっとも複雑なスキーマを`schema.yml`ファイルで記述できないことです。一方で、XMLフォーマットは、どんなに複雑なものであれ、データベースのベンダー固有の設定、テーブル、継承などを含めて、完全なスキーマ構文を記述できます。

実際にはsymfonyはXMLフォーマットで書かれたスキーマを理解します。あなたのスキーマがYAMLの構文で記述するには複雑すぎる場合、既存のXMLスキーマを持つ場合、もしくはすでにPropelのXMLフォーマットに慣れ親しんでいる場合、symfonyのYAML構文に切り替える必要はありません。`schema.yml`をプロジェクトの`config/`ディレクトリに設置し、モデルをビルドします。簡単でしょ。

>**SIDEBAR**
>**symfonyにおけるPropel**
>
>この章で説明されたすべての内容はsymfony固有のものではなく、むしろPropelのものです。Propelはsymfonyに対して優先されるオブジェクト/リレーショナル抽象化レイヤーですが、代わりのものを選ぶことができます。しかしながら、つぎの理由から、symfonyはPropelとよりシームレスに連携します:
>
>すべてのオブジェクトデータモデルクラスと`Criteria`クラスはオートロードクラスです。これらを使うと同時に、symfonyは正しいファイルをインクルードするので、ファイルをインクルードするステートメントを手動で追加する必要はありません。symfonyにおいて、Propelを起動したり、初期化する必要もありません。オブジェクトがPropelを利用するとき、ライブラリは自分自身で初期化を行います。symfonyのヘルパーはハイレベルなタスク(たとえばパジネーションもしくはフィルタリング)を実現するためにPropelのオブジェクトをパラメーターとして使います。Propelのオブジェクトによってアプリケーションのための素早いプロトタイピングとバックエンドの生成が可能です(14章で詳細な説明をします)。スキーマは`schema.yml`ファイルを通して速く書けます。
>
>Propelがデータベースに対して独立していることと同様に、symfonyもPropelに対して独立しています。

同じモデルを2回作らない
-----------------------

ORMを使う場合のトレードオフはデータ構造を2回定義しなければならないことです: 1回目はデータベースに対して、2回目はオブジェクトモデルに対してです。幸いにも、symfonyは一方に基づいてもう一方を生成するコマンドラインツールを提供するので、重複作業を回避できます。

### 既存のスキーマに基づいてSQLのデータベース構造をビルドする

`schema.yml`ファイルを書くことでアプリケーションを始める場合、symfonyはYAMLデータモデルから直接テーブルを作成するSQLクエリを生成できます。クエリを使うために、プロジェクトのrootに移動し、つぎのコマンドを入力します:

    > symfony propel-build-sql

`lib.model.schema.sql`ファイルは`myproject/data/sql/`に作られます。生成されたSQLコードが`propel.ini`ファイルの`phptype`パラメーターで定義されたデータベースシステムに対して最適化されることを覚えておいてください。

テーブルを直接ビルドするために`schema.yml`ファイルを利用できます。たとえば、MySQLでは、つぎのコマンドを入力します:

    > mysqladmin -u root -p create blog
    > mysql -u root -p blog < data/sql/lib.model.schema.sql

生成されたSQLもほかの環境のデータベースのリビルド、もしくはほかのDBMSに変更するために役立ちます。接続設定が`propel.ini`で適切に定義される場合、これを自動的に行うsymfonyの`propel-insert-sql`を利用することもできます。

>**TIP**
>コマンドラインはテキストファイルに基づいたデータをデータベースに投入するタスクも提供します。`propel-load-data`タスクとYAMLフィクスチャファイルの詳細な情報は16章をご覧ください。

### 既存のデータベースからYAMLのデータモデルを生成する

イントロスペクション(introspection データベースが影響を与えるテーブルの構造を決定するデータベースの機能)のおかげで、symfonyは既存のデータベースから`schema.yml`ファイルを生成するためにCreoleデータベースアクセスレイヤーを使うことができます。これはリバースエンジニアリングを行うとき、もしくはオブジェクトモデルよりもデータベースにとり組みたい場合に役立ちます。

これを行うために、プロジェクトの`propel.ini`ファイルが正しいデータベースを指し示しすべての接続設定を含んでいることを確認する必要があります。それから`propel-build-schema`コマンドを呼び出します:

    > symfony propel-build-schema

データベースの構造からビルドされた真新しい`schema.yml`ファイルは`config/`ディレクトリのなかに生成されます。このスキーマに基づいてモデルをビルドできます。

スキーマ生成のコマンドはとても強力でデータベースに依存する多くの情報をスキーマに追加できます。YAMLフォーマットはこの種のベンダーの情報を扱うことができないので、この情報を利用するにはXMLフォーマットを生成する必要があります。`xml`の引数を`build-schema`タスクに追加することでこれを簡単に行うことができます:

    > symfony propel-build-schema xml

`schema.yml`ファイルを生成する代わりに、これは、Propelと十分に互換性を持ち、すべてのベンダーの情報を含む`schema.xml`ファイルを作ります。しかし、生成されたXMLスキーマは読むにはとても冗長で難しいことを念頭に置いてください。

>**SIDEBAR**
>propel.iniの設定
>
>`propel-build-sql`と`propel-build-schema`タスクは`databases.yml`ファイルで定義された接続設定を使いません。むしろ、`propel.ini`という名前の別のファイルの接続設定を使います。`propel.ini`はプロジェクトの`config/`ディレクトリに保存されています:
>
>
>      propel.database.createUrl = mysql://login:passwd@localhost
>      propel.database.url       = mysql://login:passwd@localhost/blog
>
>
>このファイルは生成されたモデルクラスをsymfonyと互換性のあるものにするPropelジェネレーターを設定するために使われるほかの設定を含みます。ごく一部を除いて、多くの設定は内部に関するもので、ユーザーにとっては面白くないものです:
>
>
>      // Baseクラスはsymfonyでオートロードされる
>      // 代わりにinclude_onceステートメントを使うためにこれをtrueに設定する
>      // (パフォーマンスに対してわずかながら負な影響がある)
>      propel.builder.addIncludes = false
>
>      // 生成されたクラスはデフォルトでコメントされない
>      // コメントをBaseクラスに追加するためにこれをtrueに設定する
>      // (パフォーマンスに小さな負の影響がある)
>      propel.builder.addComments = false
>
>      // ビヘイビアーはデフォルトで扱われない
>      // これらを扱うことができるようにするにはつぎの項目をtrueに設定する
>      propel.builder.AddBehaviors = false
>
>
>`propel.ini`設定ファイルの修正を行った後に、変更が反映されるようにモデルをリビルドすることを忘れないでください。

まとめ
----

symfonyはPropelをオブジェクトリレーショナルマッピング(ORM - Object-Relational Mapping)として、Creoleをデータベース抽象化レイヤーとして利用します。これはオブジェクトモデルクラスを生成するまえに、最初にYAMLフォーマットでデータベースのリレーショナルスキーマを記述しなければならないことを意味します。それから、実行時において、オブジェクトのメソッドとレコードもしくはレコードのセットについての情報をとり出すためにピアクラスを使います。接続設定は複数の接続をサポートする`databases.yml`ファイルで定義されます。そして、コマンドラインは重複して構造を定義しないようにする特別なタスクを含みます。

モデルレイヤーはsymfonyフレームワークのなかでもっとも複雑です。複雑である理由の1つはデータ操作が込み入った問題であるからです。関連するセキュリティ問題はWebサイトにとって重大で、無視できません。ほかの理由はsymfonyが中規模から大規模のアプリケーションにもっとも適しているからです。このようなアプリケーションにおいて、symfonyのモデルによって提供された自動化は本当に時間を節約するので、内部構造を学ぶ価値はあります。

ですので、モデルオブジェクトとメソッドを十分に理解するにはこれらをテストすることに時間を費やすことを躊躇しないでください。アプリケーションの堅牢性とスケーラビリティが大きな報酬として得られます。
