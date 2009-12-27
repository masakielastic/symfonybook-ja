第14章 - ジェネレーター
=====================

多くのアプリケーションはデータベースに保存されたデータに基づいており、それにアクセスするためのインターフェイスを提供します。symfonyはPropelオブジェクトに基づいたデータ操作機能を提供するモジュール作成の反復タスクを自動化します。オブジェクトモデルが適切に定義したのであれば、symfonyはサイト全体の管理機能(administration)も自動的に生成します。この章では、symfonyに搭載された2つのジェネレーター: scaffoldingジェネレーターとadministrationジェネレーターをお教えします。後者は完全な構文による特別な設定ファイルに依存するので、この章の多くはadministrationジェネレーターのさまざまな可能性について説明します。

モデルをもとにコードを生成する
-----------------------------

Webアプリケーションにおいて、データアクセスオペレーションはつぎのように分類できます:

  * レコードの作成
  * レコードの検索
  * レコードの更新(とカラムの修正)
  * レコードの削除

これらのオペレーションは共通なので、頭文字をとった専用の略語であるCRUD(Create、Retrieval、Update、Deletion)が存在します。多くのページはこれらの1つに還元できます。たとえばフォーラムのアプリケーションにおいて、最新投稿のリストは検索オペレーション(retrieve)で、投稿への返答は作成オペレーション(create)に対応します。

任意のテーブルに対するCRUDオペレーションを実装する基本的なアクションとテンプレートはWebアプリケーション内部で繰り返し作られます。symfonyにおいて、バックエンドインターフェイスの初期開発を加速するために、モデルレイヤーはCRUDオペレーションを生成できるようにするための十分な情報を持ちます。

### データモデルの例

この章全体を通して、一覧表示機能はシンプルな例に基づいたsymfonyのadminジェネレーターの機能を示します。これによって8章を思い出すでしょう。これは2つの`BlogArticle`クラスと`BlogComment`クラスを含む、blogアプリケーションのよく知られた例です。図14-1で描かれているように、リスト14-1はスキーマを示します。

リスト14-1 - blogログアプリケーションの`schema.yml`の例

    [yml]
    propel:
      blog_article:
        id:          ~
        title:       varchar(255)
        content:     longvarchar
        created_at:  ~
      blog_comment:
        id:               ~
        blog_article_id:  ~
        author:           varchar(255)
        content:          longvarchar
        created_at:       ~

図14-1 データモデルの例

![データモデルの例](http://github.com/masakielastic/symfonybook-ja/raw/master/images/F1401.png "データモデルの例")

コード生成を可能にするためにスキーマ作成の期間で従わなければならない特別なルールは存在しません。symfonyはスキーマをそのまま使い、administrationを生成するためにスキーマの属性を解釈します。

>**TIP**
>この章を最大限に活用するには、例の内容を実際に行う必要があります。リストで説明されたすべてのステップを眺めるのであれば、symfonyが何を生成し、生成されたコードで何が行われるのかということをより理解できるようになります。以まえに説明されたようなデータ構造を作成したり、`blog_article`と`blog_comment`テーブルをともなうデータベースを作成したり、サンプルデータをこのデータベースに投入したくなるでしょう。

administration
--------------

symfonyは、バックエンドアプリケーションのために、`schema.yml`ファイルからのモデルクラス定義に基づいて、モジュールを作成できます。生成されたadministrationモジュールだけを用いてサイト全体のadministrationを作成できます。このセクションの例において`backend`アプリケーションに追加されるadministraionモジュールを説明します。プロジェクトが`backend`アプリケーションを持たない場合、`generate:app`タスクを呼び出してスケルトンを作成します:

    > php symfony generate:app backend

administrationモジュールは`generator.yml`という名前の特別な設定ファイルを通してモデルを解釈します。すべての生成コンポーネントとモジュールの外見を拡張するために`generator.yml`ファイルを変更できます。このようなモジュールは通常のモジュールメカニズムからの恩恵を受けます(レイアウト、ルーティング、カスタム設定、オートロードなど)。独自機能を生成されたadministrationに統合するために、生成されたアクションもしくはテンプレートを上書きすることもできますが、`generator.yml`はもっとも共通の要件を考慮して、PHPコードの使いかたを限定します。

>**NOTE**
>たいていの要件は`generator.yml`設定ファイルでカバーされますが、この章の後のほうで見るように、設定クラスを通してadministrationモジュールを設定することもできます。

### administrationモジュールを初期化する

モデルを基本単位としてsymfonyコマンドでadministrationをビルドします。モジュールは`propel:generate-admin`タスクを利用するPropelオブジェクトに基づいて生成されます:

    > php symfony propel:generate-admin backend BlogArticle --module=article

この呼び出しだけで`backend`アプリケーションのなかに`BlogArticle`クラスの定義に基づく`article`モジュールが作成されるので、つぎのURLからアクセスできます:

    http://localhost/backend.php/article

図14-5、図14-6で描かれている生成モジュールの外見は商用アプリケーションとしてそのまま利用できるほど十分に洗練されています。

>**NOTE**
>administrationモジュールはRESTアーキテクチャに基づいています。`propel:generate-admin`タスクは`routing.yml`設定ファイルにルートなどを自動的に追加します:
>
>     [yml]
>     # apps/backend/config/routing.yml
>     article:
>       class: sfPropelRouteCollection
>       options:
>         model:                BlogArticle
>         module:               article
>         with_wildcard_routes: true
>
>独自のルートを作成しモデルクラスの名前の代わりに独自の名前を引数としてタスクに渡すこともできます:
>
>     > php symfony propel:generate-admin backend BlogArticle --module=article

図14-5 - `backend`アプリケーション内部の`article`モジュールの`list`ビュー

![backendアプリケーション内部のarticleモジュールのlistビュー](http://github.com/masakielastic/symfonybook-ja/raw/master/images/F1405.png "backendアプリケーション内部のarticleモジュールのlistビュー")

図14-6 - バックエンドの`article`モジュールの`edit`ビュー

![バックエンドのarticleモジュールのeditビュー](http://github.com/masakielastic/symfonybook-ja/raw/master/images/F1406.png "バックエンドのarticleモジュールのeditビュー")

>**TIP**
>期待どおりの外見でなければ(スタイルシートと画像がない)、`plugin:publish-assets`タスクを実行してプロジェクトにアセットをインストールする必要があります:
>
>     $ php symfony plugin:publish-assets

### 生成コードを見る

`apps/backend/modules/article/`ディレクトリ内の記事のadministrationのコードは初期化だけ行われたので空に見えます。このモジュールの生成コードを吟味するための最良の方法はブラウザーを利用してこれと情報のやりとりをすることと`cache/`フォルダーの内容を確認することです。リスト14-4はキャッシュのなかで見つかる生成アクションとテンプレートのリストを表示します。

リスト14-4 - 生成されたadministrationの要素(`cache/backend/ENV/modules/article/`)

    // actions/actions.class.phpのアクション
    index            // テーブルのレコードのリストを表示する
    filter           // リストで使われるフィルターを更新する
    new              // 新しいレコードを作成するフォームを表示する
    create           // 新しいレコードを作成する
    edit             // レコードのフィールドを修正するフォームを表示する
    update           // 既存のレコードを更新する
    delete           // レコードを削除する
    batch            // 選択されたレコードのリストでアクションを実行する

    // templates/のなか
    _assets.php
    _filters.php
    _filters_field.php
    _flashes.php
    _form.php
    _form_actions.php
    _form_field.php
    _form_fieldset.php
    _form_footer.php
    _form_header.php
    _list.php
    _list_actions.php
    _list_batch_actions.php
    _list_field_boolean.php
    _list_footer.php
    _list_header.php
    _list_td_actions.php
    _list_td_batch_actions.php
    _list_td_stacked.php
    _list_td_tabular.php
    _list_th_stacked.php
    _list_th_tabular.php
    _pagination.php
    editSuccess.php
    indexSuccess.php
    newSuccess.php

これは生成されたadministrationモジュールがおもに3つのビュー、`list`、`new`と`edit`で構成されることを示します。コードを見てみると、モジュール性が非常に高く、読みやすく拡張性のあるものであることがわかります。

### generator.yml設定ファイルを導入する

生成されたadministrationモジュールはYAMLフォーマットの`generator.yml`設定ファイルで見つかるパラメーターに依存します。新しく生成されたadministrationモジュールのデフォルト設定を見るには、リスト14-5で再現されている、`backend/modules/article/config/`ディレクトリに設置された`generator.yml`ファイルを開いてください。

リスト14-5 - ジェネレーターのデフォルト設定(`backend/modules/article/config/generator.yml`)

    [yml]
    generator:
      class: sfPropelGenerator
      param:
        model_class:           BlogArticle
        theme:                 admin
        non_verbose_templates: true
        with_show:             false
        singular:              ~
        plural:                ~
        route_prefix:          article
        with_propel_route:     1

        config:
          actions: ~
          list:    ~
          filter:  ~
          form:    ~
          edit:    ~
          new:     ~

このコンフィギュレーションは基本的なadministrationを生成するのに十分です。カスタマイズした内容は`config`キーの下に追加されます。リスト14-6は`generator.yml`をカスタマイズした典型例を示しています。

リスト14-6 - 典型的なジェネレーターの全設定

    [yml]
    generator:
      class: sfPropelGenerator
      param:
        model_class:           BlogArticle
        theme:                 admin
        non_verbose_templates: true
        with_show:             false
        singular:              ~
        plural:                ~
        route_prefix:          article
        with_propel_route:     1

        config:
          actions:
            _new: { label: "Create a new article", credentials: editor }

          fields:
            author_id:    { label: Article author }
            published_on: { credentials: editor }

          list:
            title:          Articles
            display:        [title, author_id, category_id]
            fields:
              published_on: { date_format: dd/MM/yy }
            layout:         stacked
            params:         |
              %%is_published%%<strong>%%=title%%</strong><br /><em>by %%author%%
              in %%category%% (%%published_on%%)</em><p>%%content_summary%%</p>
            max_per_page:   2
            sort:           [title, asc]

          filter:
            display: [title, category_id, author_id, is_published]

          form:
            display:
              "Post":       [title, category_id, content]
              "Workflow":   [author_id, is_published, created_on]
            fields:
              published_at: { help: "Date of publication" }
              title:        { attributes: { style: "width: 350px" } }

          new:
            title:         New article

          edit:
            title:          Editing article "%%title%%"

このコンフィギュレーションでは、6つのセクションがあります。これらのうち4つはビューを表し(`list`、`filter`、`new`、と`edit`)とこれらのうちの2つは"バーチャル"(`fields`と`form`)で設定目的のためのみ存在します。

つぎのセクションでこの設定ファイルで利用可能なすべてのパラメーターの詳細内容を説明します。

ジェネレーターの設定
------------------

ジェネレーターの設定ファイルはとても強力で、生成されたadministrationを多くの方法で変更できます。しかしこの機能には代償があります: 全体の構文の記述が読んで学ぶには長いので、文章で説明したら、この章がこの本でもっとも長くなってしまいます。ですのでsymfonyのWebサイトはこの構文を学ぶための助けになる追加リソースを提示します: administrationジェネレーターのチートシートが図14-7で再現されてます。[http://www.symfony-project.org/uploads/assets/sfAdminGeneratorRefCard.pdf](http://www.symfony-project.org/uploads/assets/sfAdminGeneratorRefCard.pdf)から保存し、この章のつぎの例を読むときにこれを頭のなかにとどめておいてください。

このセクションの例は、`BlogComment`クラスの定義に基づいて、administrationの`comment`モジュールと同様に、administrationの`article`モジュールを調整します。`propel:generate-admin`タスクを実行して後者を作成します:

    > php symfony propel:generate-admin backend BlogComment --module=comment

図14-7 - administrationジェネレーターのチートシート

![administrationジェネレーターのチートシート](http://github.com/masakielastic/symfonybook-ja/raw/master/images/F1407.png "administrationジェネレーターのチートシート")

### フィールド

デフォルトでは、`list`ビューのカラムは`schema.yml`で定義されるカラムです。`new`と`edit`ビューのフィールドはモデルに関連づけされたフォームで定義します(`BlogArticleForm`)。`generator.yml`によって、どのフィールドが表示され、どれが非表示であるかを選び、オブジェクトモデルで直接対応するものがなくても独自フィールドを追加できます。

#### フィールドの設定

administrationジェネレーターは`schema.yml`ファイルのカラムごとに1つのフィールドを作ります。`fields`キーの下で、それぞれのフィールドの表示方法やフォーマット方法などを修正できます。たとえば、リスト14-7で示されるフィールドの設定は`title`フィールド用のカスタムラベルクラスと入力タイプ、そして`content`フィールド用のラベルとツールチップを定義します。つぎのセクションでそれぞれのパラメーターが動作する方法を詳細に説明します。

リスト14-7 - カラムに対してカスタムラベルを設定する

    [yml]
    config:
      fields:
        title:
          label: Article Title
          attributes: { class: foo }
        content: { label: Body, help: Fill in the article body }

リスト14-8でお手本が示されているように、すべてのビューに対するこのデフォルトの定義に加えて、任意のビュー(`list`、`filter`、`form`、`new`と`edit`)のためのフィールドの設定をオーバーライドできます。

リスト14-8 - ビュー単位でグローバルな設定ビューをオーバーライドする

    [yml]
    config:
      fields:
        title:     { label: Article Title }
        content:   { label: Body }

      list:
        fields:
          title:   { label: Title }

      form:
        fields:
          content: { label: Body of the article }

これは一般的な原則です: `fields`キーの下のモジュール全体に対して設定される設定項目はビュー固有の領域でオーバーライドできます。オーバーライドのルールはつぎのとおりです:

  * `new`と`edit`は`form`を継承し`form`は`fields`を継承する
  * `list`は`fields`を継承する
  * `filter`は`fields`を継承する

#### display設定にフィールドを追加する

`fields`セクションで定義したフィールドはそれぞれのビューに対して表示、隠す、順番に並べるなど、さまざまな方法で分類できます。`display`キーはこの目的のために使われます。たとえば、`comment`モジュールのフィールドを順番に並べるには、リスト14-9のコードを使います。

リスト14-9 - 表示フィールドを選択する(`modules/comment/config/generator.yml`)

    [yml]
    config:
      fields:
        article_id: { label: Article }
        created_at: { label: Published on }
        content:    { label: Body }

      list:
        display:    [id, article_id, content]

      form:
        display:
          NONE:     [article_id]
          Editable: [author, content, created_at]


図14-8のように`list`は3つのカラムを表示し、図14-9のように、`new`と`edit`フォームは2つのグループに集められた4つのフィールドを表示します。 

図14-8 - `comment`モジュールの`list`ビュー内部のカスタムカラム設定

![commentモジュールのlistビュー内部のカスタムカラム設定](http://github.com/masakielastic/symfonybook-ja/raw/master/images/F1408.png "commentモジュールのlistビュー内部のカスタムカラム設定")

図14-9 - `comment`モジュールの`edit`ビュー内部でフィールドを分類する

![commentモジュールのeditビュー内でフィールドを分類する](http://github.com/masakielastic/symfonybook-ja/raw/master/images/F1409.png "commentモジュールのeditビュー内でフィールドを分類する")

`display`設定を2つの方法で利用できます:

  * `list`ビューに対して: 表示するカラムと表示される順序を選ぶために単純な配列にフィールドを置きます。
  * `form`、`new`、と`edit`ビューに対して: グループの名前をキーとするフィールドをグループ分けするのに連想配列、もしくは名前のないグループに対して`NONE`を使います。値はカラムの名前で並べ替えられた配列のままです。フォームクラスに参照されるすべての必須フィールドの一覧を表示することには注意を払ってください。さもないと予期せぬバリデーションエラーに遭遇するかもしれません。

#### カスタムフィールド

当然のことながら、`generator.yml`で設定されたフィールドはスキーマで定義された実際のカラムに対応している必要はありません。関連クラスがカスタムゲッターを提供する場合、これを`list`ビューのためのフィールドとして使うことができます; ゲッターかつ/もしくはセッターが存在する場合、このクラスは`edit`ビューでも利用できます。たとえば、リスト14-10のように、`BlogArticle`モデルを`getNbComments()`メソッドで拡張できます。

リスト14-10 - モデルにカスタムゲッターを追加する(`lib/model/BlogArticle.class.php`)

    [php]
    public function getNbComments()
    {
      return $this->countBlogComments();
    }

リスト14-11のように、`nb_comments`は生成モジュールのフィールドとして利用できます(ゲッターはcamelCaseバージョンのフィールド名を使います)。

リスト14-11 - カスタムゲッターはadministrationモジュール用のカラムを提供する(`backend/modules/article/config/generator.yml`)

    [yml]
    config:
      list:
        display: [id, title, nb_comments, created_at]

`article`モジュールの`list`ビューの結果は図14-10で示されます。

図14-10 - `article`モジュールの`list`ビュー内のカスタムフィールド

![articleモジュールのlistビュー内部のカスタムフィールド](http://github.com/masakielastic/symfonybook-ja/raw/master/images/F1410.png "articleモジュールのlistビュー内部のカスタムフィールド")

カスタムフィールドは生のデータ以上のものを表示するためにHTMLのコードも返すことができます。たとえば、リスト14-12で示されるような`BlogComment`クラスを`getArticleLink()`メソッドで拡張できます。

リスト14-12 - HTMLを返すカスタムゲッターを追加する(`lib/model/BlogComment.class.php`)

    [php]
    public function getArticleLink()
    {
      return link_to($this->getBlogArticle()->getTitle(), 'article_edit', $this->getBlogArticle());
    }

リスト14-11と同じ構文でこの新しいゲッターを`comment/list`ビュー内のカスタムフィールドとして使うことができます。リスト14-13の例と、図14-11の結果をご覧ください。ゲッター(記事のハイパーリンク)によって出力されるHTMLのコードは記事の主キーの代わりに2番目のカラムに現れます。

リスト14-13 - HTMLの結果を返すカスタムゲッターは追加カラムとしても使える(`modules/comment/config/generator.yml`)

    [yml]
    config:
      list:
        display: [id, article_link, content]

図14-11 - `comment`モジュールの`list`ビュー内部のカスタムフィールド

![commentモジュールのlistビュー内部のカスタムフィールド](http://github.com/masakielastic/symfonybook-ja/raw/master/images/F1411.png "commentモジュールのlistビュー内部のカスタムフィールド")

#### 部分テンプレートフィールド

モデルに設置されたコードはプレゼンテーションから独立していなければなりません。前出の`getArticleLink()`メソッドの例はレイヤー分離の原則を順守していません。ビューコードのなかにはモデルレイヤーに現れるものがあるからです。正しい方法で、同じ結果を実現するには、カスタムフィールドに対応するHTMLを出力するコードは部分テンプレートで対応するほうがベターです。幸いにして、administrationジェネレーターによってアンダースコアのプレフィックスを持つフィールド名を宣言できます。この場合、リスト14-13の`generator.yml`ファイルはリスト14-14のように修正されます。

リスト14-14 - プレフィックスの`_`を使うことで部分テンプレートを追加カラムとして使うことができる、

    [yml]
    config:
      list:
        display: [id, _article_link, created_at]

これを機能させるには、リスト14-15で示されるように、`modules/comment/tempaltes/`ディレクトリのなかで`_article_link.php`部分テンプレートを作らなければなりません。

リスト14-15 - `list`ビュー用の部分テンプレートの例(`modules/comment/templates/_article_link.php`)

    [php]
    <?php echo link_to($form->getObject()->getBlogArticle()->getTitle(), '@article_edit', $form->getObject()->getBlogArticle()) ?>

部分テンプレートフィールドの部分テンプレートテンプレートはクラスによって命名された変数(この例では`$comment`)を通して現在のオブジェクトにアクセスできることに注意してください。たとえば、`UserGroup`という名前のクラスのためにビルドされたモジュールに対して、部分テンプレートは`$user_group`変数を通して現在のオブジェクトにアクセスできるようになります。

レイヤー分離の原則が順守されることを除いて、結果は図14-11と同じです。レイヤー分離の原則を順守することに慣れたら、アプリケーションはより維持しやすくなります。

部分テンプレートフィールドのパラメーターをカスタマイズする必要がある場合、`field`キーの下で通常のフィールドと同じ事を行います。アンダースコア(`_`)をキーの先頭に含めないでください。リスト14-16をご覧ください。

リスト14-16 - 部分テンプレートフィールドのプロパティは`fields`キーの下でカスタマイズできる

    [yml]
    config:
      fields:
        article_link: { label: Article }

部分テンプレートにロジックが混在するようになったら、おそらくはこれをコンポーネントで置き換えたいと思うでしょう。リスト14-17のように、プレフィックスの`_`を`~`に変更することで、コンポーネントフィールドを定義できます。

リスト14-17 - コンポーネントは追加カラムとして利用できる。プレフィックスの`~`を使う

    [yml]
    config:
      list:
        display: [id, ~article_link, created_at]

生成テンプレートにおいて、現在のモジュールの`articleLink`コンポーネントを呼び出すことでこれは生成されます。

>**NOTE**
>カスタムと部分テンプレートフィールドは`list`、`new`、`edit`と`filter`ビューのなかで使うことができます。複数のビューに対して同じ部分テンプレートを使う場合、内容(`list`、`new`、`edit`、もしくは`filter`)は変数`$type`に保存されます。

### ビューのカスタマイゼーション

`new`、`edit`と`list`ビューの外見を変更するために、テンプレートを変更したいと思うことがあります。しかしこれらは自動的に生成されるので、これを行うのはあまりよいアイディアではありません。代わりに`generator.yml`設定ファイルを使うべきです。モジュール性を損なうことなくほとんどすべての必要なことを行うことができるからです。

#### ビューのタイトルを変更する

フィールドのカスタムセットに加えて、`list`、`new`、`edit`ページはカスタムページタイトルを持ちます。たとえば、`article`ビューのタイトルをカスタマイズしたい場合、リスト14-18のように行います。`edit`ビューの結果は図14-12に描かれています

リスト14-18 - それぞれのビューに対してカスタムタイトルを設定する(`backend/modules/article/config/generator.yml`)

    [yml]
    config:
      list:
        title: List of Articles

      new:
        title: New Article

      edit:
        title: Edit Article %%title%% (%%id%%)

図14-12 - `article`モジュールの`edit`ビュー内部のカスタムタイトル

![articleモジュールのeditビュー内部のカスタムタイトル](http://github.com/masakielastic/symfonybook-ja/raw/master/images/F1412.png "articleモジュールのeditビュー内部のカスタムタイトル")

デフォルトのタイトルはクラスの名前を使うので、モデルが明白なクラスの名前を使うのであれば、これらのクラス名で十分であることはよくあります。

>**TIP**
>`generator.yml`の文字列の値において、フィールドの値は`%%`で囲まれたフィールドの名前を通してアクセスできます。

#### ツールチップを追加する

`list`、`new`、`edit`、と`filter`ビュー内部において、表示されるフィールドを記述するための助けになるツールチップを追加できます。たとえば、ツールチップを`comment`モジュールの`edit`ビューの`article_id`フィールドに追加するには、リスト14-19のように、`fields`の定義に`help`プロパティを追加します。結果は図14-13に示されています。

リスト14-19 - `edit`ビューでツールチップを設定する(`modules/comment/config/generator.yml`)

    [yml]
    config:
      edit:
        fields:
          article_id: { help: The current comment relates to this article }

図14-13 - `comment`モジュールの`edit`ビュー内部のツールチップ

![commentモジュールのeditビュー内部のツールチップ](http://github.com/masakielastic/symfonybook-ja/raw/master/images/F1413.png "commentモジュールのeditビュー内部のツールチップ")

`list`ビューにおいて、ツールチップはカラムのヘッダーに表示されます; `new`、`edit`、`filter`ビューにおいてこれらはfieldタグの下で現れます。

#### 日付の書式を修正する

リスト14-20でお手本が示されているように、`date_format`オプションを使うことで同時にカスタム書式で日付を表示できます。

リスト14-20 `list`ビューのなかで日付の書式設定をする

    [yml]
    config:
      list:
        fields:
          created_at: { label: Published, date_format: dd/MM }

このパラメーターは以前の章で説明された`format_date()`ヘルパーと同じフォーマットパラメーターを受けとります。

>**SIDEBAR**
>administrationテンプレートは国際化の準備ができています
>
>adminで生成されたモジュールはインターフェイスの文字列(デフォルトのアクションの名前、ページ分割のヘルプメッセージ、・・・)とカスタム文字列(タイトル、カラムのラベル、ヘルプメッセージ、エラーメッセージ、・・・)で構成されます。
>
>インターフェイスの文字列の多言語翻訳機能はsymfonyに搭載されています。しかし独自のものを追加するか`sf_admin`カタログ(`apps/frontend/i18n/sf_admin.XX.xml`、`XX`は言語のISOコード)用の`i18n`ディレクトリでカスタムXLIFFファイルを作成することで既存のものをオーバーライドできます
>
>生成されたテンプレートで見つかるすべてのカスタム文字列も自動的に国際化されます(すなわち、`__()`ヘルパーーへの呼び出しと一緒に)。このことは前の章で説明したように、`apps/frontend/i18n/`ディレクトリでXLIFFファイルにフレーズの翻訳を追加することで、生成されたadministrationを簡単に翻訳できることを意味します。
>
>`i18n_catalogue`パラメーターを指定することでカスタム文字列用に使われるデフォルトのカタログを変更できます:
>
>     [yml]
>     generator:
>       class: sfPropelGenerator
>       param:
>         i18n_catalogue: admin

### listビュー固有のカスタマイゼーション

`list`ビューは、表形式、もしくはすべての詳細な内容が一行に積み重ねられた状態で、レコードの詳細を表示できます。これはフィルター、ページ分割とソート機能も含みます。これらの機能は設定によって変更できます。詳細はつぎのセクションで説明します。

#### レイアウトを変更する

デフォルトでは、`list`ビューと`edit`ビュー間のハイパーリンクは主キーのカラムによって生成されます。図14-11に戻ると、コメントリストの`id`カラムがそれぞれのコメントの主キーを表示するだけでなく、ユーザーが`edit`ビューにアクセスすることを許可するハイパーリンクを提供します。

別のカラムに現れるレコードの詳細内容へのハイパーリンクが望ましいのであれば、`display`キーの等号(`=`)をプレフィックスとしてカラムの名前に追加します。リスト14-21は`list`コメントの表示フィールドから`id`を除去して代わりに`content`フィールドにハイパーリンクを置く方法を示しています。図14-14のスクリーンショットを確認してください。

リスト14-21 - `edit`ビューのためのハイパーリンクを`list`ビューに移動させる(`modules/comment/config/gererator.yml`)

    [yml]
    config:
      list:
        display:    [article_link, =content]

図14-14 - `comment`モジュールの`list`ビューのなかで、リンクを別のカラム上の`edit`ビューに移動させる

![commentモジュールのlistビュー内で、リンクを別のカラム上のeditビューに移動させる](http://github.com/masakielastic/symfonybook-ja/raw/master/images/F1414.png "commentモジュールのlistビュー内で、リンクを別のカラム上のeditビューに移動させる")

デフォルトでは、以前示されたように、`list`ビューはフィールドがカラムとして表示される`tabular`レイアウトを使います。しかし`stacked`レイアウトを使ってフィールドをテーブルの全長を詳しく記述する単独の文字列に連結することもできます。`stacked`レイアウトを選ぶ場合、`params`キーにリストのそれぞれの行の値を定義するパターンを組み込まなければなりません。たとえば、リスト14-22は`comment`モジュールの`list`ビューに対して`stacked`レイヤーを定義します。結果は図14-15です。

リスト14-22 - `list`ビューで`stacked`レイアウトを使う(`modules/comment/cofig/generator.yml`)

    [yml]
    config:
      list:
        layout:  stacked
        params:  |
          %%=content%%<br />
          (sent by %%author%% on %%created_at%% about %%article_link%%)
        display:  [created_at, author, content]

図14-15 - `comment`モジュールの`list`ビュー内部のstackedレイアウト

![commentモジュールのlistビュー内部のstackedレイアウト](http://github.com/masakielastic/symfonybook-ja/raw/master/images/F1415.png "commentモジュールのlistビュー内のstackedレイアウト")

`tabular`レイアウトは`display`キーの下でフィールドの配列を必要としますが、`stacked`レイアウトはそれぞれのレコードに対して生成されたHTMLコードのために`params`キーを使うことに留意してください。しかしながら、インタラクティブなソートに対して利用できるカラムヘッダーを決定するために`display`配列が`stacked`レイアウトでも使われます

#### 結果をフィルタリングする

`list`ビューにおいて、フィルターのインタラクションのセットを追加できます。これらのフィルターによって、ユーザーは結果をより少なく表示して速く望むものに到達できます。フィールド名の配列で、`filter`キーの下にフィルターを設定します。たとえば、図14-16のフィルターボックスと同じようなものを表示するには、リスト14-23のように、`article_id`、`author`フィールド、と`created_at`フィールド上のフィルターをコメントの`list`ビューに追加します。これを機能させるには、`BlogArticle`クラスに`__toString()`メソッドを追加する必要があります(たとえば、記事の`title`を返す)。

リスト14-23 - `list`ビューでフィルターを設定する(`modules/comment/config/generator.yml`)

    [yml]
    config:
      list:
        layout:  stacked
        params:  |
          %%=content%% <br />
          (sent by %%author%% on %%created_at%% about %%article_link%%)
        display:  [created_at, author, content]

      filter:
        display: [article_id, author, created_at]

図14-16 - `comment`モジュールの`list`ビュー内部のフィルター

![commentモジュールのlistビュー内部のフィルター](http://github.com/masakielastic/symfonybook-ja/raw/master/images/F1416.png "commentモジュールのlistビュー内部のフィルター")

symfonyに表示されるフィルターはスキーマで定義されるカラムの型に依存し、フィルターフォームクラスでカスタマイズできます:

  * テキストカラム(たとえば`comment`モジュール内の`author`フィールド)に対して、フィルターはテキストベースの検索を可能にするテキスト入力です(ワイルドカードが自動的に追加).
  * 外部キー(たとえば`comment`モジュール内部の`article_id`フィールドのリスト)に対して、フィルターは関連するテーブルのレコードのドロップダウンリストです。デフォルトでは、ドロップダウンリストのオプションは関連するクラスの`__toString()`メソッドによって返されるものです。
  * 日付のカラム(たとえば`comment`カラム内の`created_at`フィールド)に対して、フィルターはリッチな日付タグのペアで、時間の間隔を選択できるようにします。
  * ブール型のカラムに対して、フィルターは`true`と`false`と`true or false`オプションを持つドロップダウンリストです。最後の値はフィルターを再初期化します。

`new`と`edit`ビューがフォームクラスに結びつけられているように、モデルに関連するデフォルトのフィルターフォームクラスです(たとえば`BlogArticle`モデルに対して`BlogArticleFormFilter`)。フィルターフォームに対してカスタムクラスを定義することで、フォームフレームワークの力を活用しすべての利用可能なフィルターウィジェットを使うことでフィルターフィールドをカスタマイズできます。リスト14-24で示されるように、`filter`エントリーの下で`class`を定義することで簡単に実現できます。

リスト14-24 - フィルタリングに使われるフォームクラスをカスタマイズする

    [yml]
    config:
      filter:
        class: BackendArticleFormFilter

>**TIP**
>フィルターを一緒に無効にするには、フィルター用の`class`に`false`を指定するだけです。

フィルターを修正するために部分テンプレートフィルターも使うことができます。それぞれの部分テンプレートはフォームをレンダリングするさいに`form`とHTMLの`attributes`を受けとります。リスト14-25は部分テンプレート以外のデフォルトのふるまいを模倣する実装の例を示しています。

リスト14-25 - 部分テンプレートフィルターを使う

    [php]
    // templates/_state.phpで、部分テンプレートを定義する
    <?php echo $form[$name]->render($attributes->getRawValue()) ?>

    // config/generator.ymlで部分テンプレートフィルターのリストを追加する
    config:
      filter: [date, _state]

#### リストをソートする

`list`ビューにおいて、図14-18で示されるように、テーブルのヘッダーはリストを再び順番に並べるために使われるハイパーリンクです。これらのヘッダーは`tabular`レイアウトと`stacked`レイアウトの両方で表示されます。これらのリンクをクリックするとリストの順番を再配置する`sort`パラメーターでページがリロードされます。

図14-18 - `list`ビューのテーブルヘッダーはソートをコントロールする

![listビューのテーブルヘッダーはソートはコントロールする](http://github.com/masakielastic/symfonybook-ja/raw/master/images/F1418.png "listビューのテーブルヘッダーはソートをコントロールする")

あるひとつのカラムによってソートされたリストを直接指定する構文を再利用できます:

    [php]
    <?php echo link_to('日付ごとのコメントのリスト', '@comments?sort=created_at&sort_type=desc' ) ?>

`generator.yml`ファイル内部の`list`ビューに対して直接デフォルトの`sort`の順番を定義することも可能です。構文はリスト14-26で示された例に従います。

リスト14-26 - `list`ビューのなかでデフォルトの`sort`フィールドを設定する

    [yml]
    config:
      list:
        sort:   created_at
        # ソートの順序を指定するための、代替構文
        sort:   [created_at, desc]

>**NOTE**
>実際のカラムに対応するフィールドだけが、ソートのコントロール機能に変換されます。カスタムもしくは部分テンプレートフィールドには対応していません。

#### ページ分割をカスタマイズする

生成されたadministrationは大きなテーブルさえも効率的に処理します。`list`ビューがデフォルトでページ分割を使うからです。テーブル内の実際の列の数がページごとの列の最大数を越えるとき、ページ分割のコントロール機能がリストの底に現れます。たとえば、図14-19はページごとに表示される5つのコメントの制限ではなく、テーブル内で6つのテストのコメントを持つコメントのリストを表示しますが、ページごとに表示されるコメント数は最大5まで制限されています。ページ分割はよいパフォーマンスとユーザービリティを保証します。データベースから表示される列のみが効率的に検索され、administrationモジュールによって何百万の列をかかえるテーブルでさえも管理できるからです。

図14-19 - ページ分割のコントロール機能は長いリスト上に現れる

![ページ分割のコントロール機能は長いリスト上に現れる](http://github.com/masakielastic/symfonybook-ja/raw/master/images/F1419.png "ページ分割のコントロール機能は長いリスト上に現れる")

`max_per_page`パラメーターによってそれぞれのページが表示されるレコードの数をカスタマイズできます:

    [yml]
    config:
      list:
        max_per_page:   5

#### ページ配信を加速するためにJoinを使う

デフォルトでは、administrationジェネレーターはレコードのリストを検索する`doSelect()`を使います。しかし、リストで関連オブジェクトを使う場合、リストを表示するために必要なデータベースクエリの数が急に増えることがあります。たとえば、コメントのリストで記事の名前を表示したい場合、関連する`BlogArticle`オブジェクトを検索するためにリスト内のそれぞれの投稿に対する追加クエリが必要です。ですので、クエリの数を最適化するために、`doSelectJoinXXX()`メソッドを使うページャーを強制したい場合があるかもしれません。これは`peer_method`パラメーターで指定できます。

    [yml]
    config:
      list:
        peer_method: doSelectJoinBlogArticle

18章ではJoinの概念についてより広範囲に説明します。

### newとeditビュー固有のカスタマイズ

`new`もしくは`edit`ビューにおいて、ユーザーは新しいレコードもしくは渡されたレコードに対してそれぞれのカラムの値を修正できます。デフォルトでは、adminジェネレーターで使われるフォームはモデル:`BlogArticle`モデルに対して`BlogArticleForm`に関連するフォームです。リスト14-27で示されるように`form`エントリーの下で`class`を定義することで使うクラスをカスタマイズできます。

リスト14-27 - `new`と`edit`ビューに使われるフォームクラスをカスタマイズする

    [yml]
    config:
      form:
        class: BackendBlogArticleForm

カスタムフォームクラスを利用することでadminジェネレーター用のウィジェットとバリデーターすべてをカスタマイズできます。frontendアプリケーションに対してデフォルトのフォームクラスが使われ特別にカスタマイズされます。

リスト14-28で示されるように`generator.yml`設定ファイルでラベル、ヘルプメッセージ、とフォームのレイアウトを直接カスタマイズすることもできます。

リスト14-28 - フォーム表示をカスタマイズする

    [yml]
    config:
      form:
        display:
          NONE:     [article_id]
          Editable: [author, content, created_at]
        fields:
          content:  { label: body, help: "The content can be in the Markdown format" }

#### 部分テンプレートフィールドを扱う

部分テンプレートフィールドは`list`ビューのように`new`と`edit`ビューで扱えます。

### 外部キーを扱う

スキーマがテーブルのリレーションを定義する場合、生成されたadministrationモジュールはこれを活用して、より自動化されたコントロール方法を提供するので、リレーションの管理作業が大いに簡略化されます。

#### 一対多のリレーション

1対多のテーブルのリレーションはadministrationジェネレーターによって考慮されます。以前の図14-1で記述されたように、`blog_comment`テーブルは`article_id`フィールドを通して`blog_article`テーブルとの関係を持ちます。`BlogComment`クラスのモジュールをadministrationジェネレーターによって初期化する場合、`edit`ビューは`blog_article`テーブル内の利用可能なレコードのIDを示す`article_id`をドロップダウンリストとして表示します(説明図は図14-9を再度確認)。

加えて、`BlogArticle`オブジェクトで`__toString()`メソッドを定義する場合、ドロップダウンオプションのテキストは主キーの代わりにこのメソッドを使います。

`article`モジュール(多対一のリレーション)内部の記事に関連するコメントのリストを表示する場合、部分テンプレートフィールドの方法によってモジュールを少しカスタマイズする必要があります。

#### 多対多のリレーション

symfonyは図14-20で示されるように多対多のリレーションも考慮します。

図14-20 - 多対多のリレーション

![多対多のリレーション](http://github.com/masakielastic/symfonybook-ja/raw/master/images/F1420.png "多対多のリレーション")

リレーションをレンダリングするために使われるウィジェットをカスタマイズすることで、フィールドのレンダリングを調整できます(図14-21で説明):

図14-21 - 多対多のリレーションに対して利用できるコントロール機能

![多対多のリレーションに対して利用できるコントロール機能](http://github.com/masakielastic/symfonybook-ja/raw/master/images/F1421.png "多対多のリレーションに対して利用できるコントロール機能")

### インタラクションを追加する

administrationモジュールはユーザーが通常のCRUDオペレーションを実行できるようにしますが、ビューに対して独自のインタラクションを追加するもしくは可能なインタラクションを制限することもできます。たとえば、リスト14-31で示されているインタラクションを定義することで`article`モジュール上のすべてのCRUDアクションにデフォルトでアクセスできるようになります。

リスト14-31 - それぞれのビューに対してインタラクションを定義する(`backend/modules/article/config/generator.yml`)

    [yml]
    config:
      list:
        title:          List of Articles
        object_actions:
          _edit:         ~
          _delete:       ~
        batch_actions:
          _delete:       ~
        actions:
          _new:          ~

      edit:
        title:          Body of article %%title%%
        actions:
          _delete:       ~
          _list:         ~
          _save:         ~
          _save_and_add: ~

`list`ビューにおいて、アクションの設定が3つ存在します: すべてのオブジェクトに対して利用可能なアクション(`object_actions`)、オブジェクトの選択に対して利用可能なアクション(`batch_actions`)、ページ全体に対して利用可能なアクション(`actions`)です。リスト14-31で定義されたリストのインタラクションは図14-22のようにレンダリングします。それぞれの行はレコードを編集するためのボタンとレコードを削除するためのボタン、それらに加えて、レコードの選択を削除するためにそれぞれの行の上に1つのチェックボックスを表示します。リストの一番下で、1つのボタンによって新しいレコードを作ることができます。

図14-22 - `list`ビュー内部のインタラクション

![listビュー内部のインタラクション](http://github.com/masakielastic/symfonybook-ja/raw/master/images/F1422.png "listビュー内部のインタラクション")

`new`と`edit`ビューにおいて、一度に編集されるレコードは1つだけであり、(`actions`のもとで)定義するアクションのセットは1つだけです。リスト14-31で定義された`edit`インタラクションは図14-23のようにレンダリングされます。`save`アクションと`save_and_add`アクションは現在の編集をレコードに保存します。これらのアクションの違いは、`save`アクションは保存したあとで現在のレコード上に`edit`ビューを表示するのに対して、`save_adn_add`アクションは別のレコードを追加するために`new`ビューを表示することです。`save_and_add`アクションは続けざまに多くのレコードを追加するときに非常に便利なショートカットです。`delete`アクションの位置に関しては、これはほかのボタンから分離されているので、ユーザーが誤ってクリックすることはありません。

アンダースコア(`_`)で始まるインタラクション名はsymfonyに対してこれらのアクションに対応するデフォルトのアイコンとアクションを使うように伝えます。administrationジェネレーターは`_edit`、`_delete`、`_new`、`_list`、`_save`、`_save_and_add`、と`_create`を理解します。

図14-23 - `edit`ビュー内部のインタラクション

![editビュー内部のインタラクション](http://github.com/masakielastic/symfonybook-ja/raw/master/images/F1423.png "editビュー内部のインタラクション")

しかしカスタムインタラクションを追加することもできます。この場合リスト14-32で示されるように、アンダースコアで始まらない名前、と現在のモジュールのなかのターゲットのアクションを指定しなければなりません。

リスト14-32 - カスタムインタラクションを定義する

    [yml]
    list:
      title:          List of Articles
      object_actions:
        _edit:        -
        _delete:      -
        addcomment:   { label: Add a comment, action: addComment }

図14-24で示されるように、リストにおけるそれぞれのアクションは`Add a comment`リンクを表示します。これをクリックすることで現在のモジュール内で`addComment`アクションを呼び出すことが行われます。現在のオブジェクトの主キーはリクエストパラメーターに自動的に追加されます。

図14-24 - `list`ビュー内部のカスタムインタラクション

![listビュー内部ののカスタムインタラクション](http://github.com/masakielastic/symfonybook-ja/raw/master/images/F1424.png "listビュー内部のカスタムインタラクション")

`addComment`アクションはリスト14-33のように実装できます。

リスト14-33 - カスタムインタラクションアクション(`actions/actions.class.php`)

    [php]
    public function executeAddComment(sfWebRequest $request)
    {
      $comment = new BlogComment();
      $comment->setArticleId($request->getParameter('id'));
      $comment->save();

      $this->redirect('comments_edit', $comment);
    }

バッチスクリプトは`sf_admin_batch_selection`リクエストパラメーターのなかで選択されたレコードの主キーの配列を受けとります。

アクションについて最後の一言です: あるカテゴリのためにアクションを完全に抑制したい場合、リスト14-34で示されるように、空のリストを使います。

リスト14-34 - `list`ビューのすべてのアクションを除去する

    [yml]
    config:
      list:
        title:   List of Articles
        actions: {}

### フォームのバリデーション

バリデーションは`new`と`edit`ビューで使われるフォームを自動的に考慮します。対応するフォームクラスを編集することでこのフォームをカスタマイズできます。

### クレデンシャルを利用してユーザーのアクションを制限する

特定のadministrationモジュールに対して、利用可能なフィールドとインタラクションはログインユーザーのクレデンシャルによって変化します(symfonyのセキュリティ機能の説明に関しては6章を参照)。

適切なクレデンシャルを持つユーザーのみに対して表示されるようにするためにジェネレーター内部のフィールドは`credentials`パラメーターを考慮に入れます。これは`list`エントリーに対して機能します。加えて、ジェネレーターはクレデンシャルにしたがってインタラクションも隠すことができます。リスト14-37はこれらの機能のお手本を示しています。

リスト14-37 - クレデンシャルを使う(`generator.yml`)

    [yml]
    config:
      # adminクレデンシャルを持つユーザーに対してのみidカラムは表示される
      list:
        title:          List of Articles
        display:        [id, =title, content, nb_comments]
        fields:
          id:           { credentials: [admin] }

      # addcommentインタラクションはadminクレデンシャルを持つユーザーに制限される
      actions:
        addcomment: { credentials: [admin] }

生成モジュールのプレゼンテーションを修正する
------------------------------------------

独自のスタイルシートを適用するだけでなく、デフォルトのテンプレートで上書きすることで、生成されたモジュールのプレゼンテーションが既存のグラフィカルな表にマッチするように、そのモジュールを修正できます。

### カスタムスタイルシートを使う

生成されたHTMLコードは構造化された内容を持つので、プレゼンテーションであなたが望むことを大いに行うことができます。

リスト14-38で示されるように、`css`パラメーターを生成されたジェネレーターの設定に追加することで、administrationモジュールに対して、デフォルトの代わりにカスタムCSSを定義できます。

リスト14-38 - デフォルトの代わりにカスタムスタイルシートを使う

    [yml]
    class: sfPropelGenerator
    param:
      model_class:           BlogArticle
      theme:                 admin
      non_verbose_templates: true
      with_show:             false
      singular:              ~
      plural:                ~
      route_prefix:          article
      with_propel_route:     1
      css:                   mystylesheet

もう一つの方法として、ビュー単位でスタイルを上書きするためにモジュールの`view.yml`によって提供されるメカニズムも利用できます。

### カスタムヘッダーとフッターを生成する

`list`、`new`、`edit`ビューはヘッダーとフッター部分テンプレートを系統的に含むことができます。administrationモジュールの`templates/`ディレクトリのなかにはそのような部分テンプレートは存在しませんが、部分テンプレートを自動的にインクルードするには以下の名前の1つを追加する必要があります:

    _list_header.php
    _list_footer.php
    _form_header.php
    _form_footer.php

たとえば、カスタムヘッダーを`article/edit`ビューに追加したい場合、リスト14-39のように`_form_header.php`という名前のファイルを作ります。これは追加の設定なしで動作します。

リスト14-39 - `edit`ヘッダー部分テンプレートの例(`modules/article/templates/_form_header.php`)

    [php]
    <?php if ($article->getNbComments() > 0): ?>
      <h2>この記事には<?php echo $article->getNbComments() ?>のコメントがあります</h2>
    <?php endif; ?>

`edit`部分テンプレートはクラスによって命名された変数を通していつでも現在のオブジェクトにアクセス可能で、`list`部分テンプレートは`$pager`変数を通していつでも現在のページャーにアクセスできることに留意してください。

>**SIDEBAR**
>administrationアクションをカスタムパラメーターで呼び出す
>
>administrationモジュールアクションは`link_to()`ヘルパーの`query_string`引数を使ってカスタムパラメーターを受けとることができます。たとえば、記事に対するコメントへのリンクを持つ`previous_edit_header`部分テンプレートを拡張するには、つぎのように書きます:


    [php]
    <?php if ($article->getNbComments() > 0): ?>
      <h2>この記事には<?php echo link_to($article->getNbComments().'のコメント', 'comment/list', array('query_string' => 'filter=filter&filters%5Barticle_id%5D='.$article->getId())) ?>があります</h2>
    <?php endif; ?>


>**SIDEBAR**
>
>このクエリの文字列パラメーターはより読みやすいエンコードされたバージョンです。
>
>     [php]
>     'filter=filter&filters[article_id]='.$article->getId()
>
>これは`$article`に関連するコメントだけを表示するためにコメントをフィルタリングします。`query_string`引数を使うことで、ソートの順番かつ/またはカスタムlistビューを表示するフィルターを指定することもできます。これはカスタムインタラクションに対しても便利です。

### テーマをカスタマイズする

カスタム要件を満たすために、モジュールの`templates/`フォルダーのなかでオーバーライド可能でフレームワークから継承された別の部分テンプレートが存在します。

ジェネレーターのテンプレートは個別に上書きできる小さな部分に分割可能で、アクションは一つずつ変更できます。

しかしながら、同じ方法でいくつかのモジュールに対してこれらを上書きしたいのであれば、おそらくは再利用可能なテーマを作るべきです。テーマ(theme)はテンプレートとアクションのサブセットで、`generator.yml`の始めでテーマの値に指定された場合、administrationモジュールのなかで使えます。デフォルトのテーマと一緒に、symfonyは`$sf_symfony_lib_dir/plugins/sfPropelPlugin/data/generator/sfPropelModule/admin/`で定義されたファイルを使います。

テーマのファイルは`data/generator/sfPropelModule/[theme_name]/`ディレクトリ内部の、プロジェクトのツリー構造に設置しなければならず、デフォルトのテーマからオーバーライドしたいファイルをコピーすることで新しいテーマを使い始めることができます(`$sf_symfony_lib_dir/plugins/sfPropelPlugin/data/generator/sfPropelModule/admin/`ディレクトリに設置されます):

    // 部分テンプレート、[theme_name]/template/templates/
    _assets.php
    _filters.php
    _filters_field.php
    _flashes.php
    _form.php
    _form_actions.php
    _form_field.php
    _form_fieldset.php
    _form_footer.php
    _form_header.php
    _list.php
    _list_actions.php
    _list_batch_actions.php
    _list_field_boolean.php
    _list_footer.php
    _list_header.php
    _list_td_actions.php
    _list_td_batch_actions.php
    _list_td_stacked.php
    _list_td_tabular.php
    _list_th_stacked.php
    _list_th_tabular.php
    _pagination.php
    editSuccess.php
    indexSuccess.php
    newSuccess.php

    // アクション、[theme_name]/parts
    actionsConfiguration.php
    batchAction.php
    configuration.php
    createAction.php
    deleteAction.php
    editAction.php
    fieldsConfiguration.php
    filterAction.php
    filtersAction.php
    filtersConfiguration.php
    indexAction.php
    newAction.php
    paginationAction.php
    paginationConfiguration.php
    processFormAction.php
    sortingAction.php
    sortingConfiguration.php
    updateAction.php

テンプレートファイルは実際にはテンプレートのテンプレート(templates of templates)であることに注意してください。すなわち、PHPファイルはジェネレーターの設定に基づくテンプレートを生成する特別なユーティリティによって解析されます(これはコンパイレーションフェーズ(compilation phase)と呼ばれます)。生成テンプレートが実際にブラウジングしている間に実行されるPHPコードを含まなければならないので、テンプレートのテンプレートは最初のパスの間にPHPコードを実行しないようにするために代替構文を使います。リスト14-40はデフォルトのテンプレートのテンプレートの抜粋です。

リスト14-40 - テンプレートのテンプレートの構文

    [php]
    <h1>[?php echo <?php echo $this->getI18NString('edit.title') ?> ?]</h1>

    [?php include_partial('<?php echo $this->getModuleName() ?>/flashes') ?]

このリストにおいて、(コンパイル時に)`<?`によって導入されたPHPコードは即座に実行され、`[?`によって導入されたものは実行時のみに実行されますが、テンプレートエンジンは最後には`[?`タグを`<?`タグに変換するのでテンプレートの結果はつぎのようになります:

    [php]
    <h1><?php echo __('List of all Articles') ?></h1>

    <?php include_partial('article/flashes') ?>

テンプレートのテンプレートは扱いにくいので、独自のテーマを作りたい場合、もっともお勧めすることは`admin`テーマから始め、少しずつ修正し、こまめにテストすることです。

>**TIP**
>ジェネレーターのテーマをプラグインのパッケージにすることができるので、再利用性が高くなり、複数のアプリケーションにまたがってデプロイすることが簡単になります。詳細な情報は17章を参照してください。

-

>**SIDEBAR**
>独自のジェネレーターを開発する
>
>administrationジェネレーターはsymfonyの内部コンポーネントのセットを使います。内部コンポーネントはキャッシュ内部に生成されたアクションとテンプレート、テーマの使用、テンプレートのテンプレートの解析を自動化します。
>
>このことはsymfonyが既存のジェネレーターとは似ているもしくは完全に異なる独自のジェネレーターを作るためのすべてのツールを提供することを意味します。モジュールの生成は`sfGeneratorManager`クラスの`generate()`メソッドで管理されます。たとえば、administrationを生成するために、symfonyは内部でつぎのコードを呼び出します:
>
>     [php]
>     $manager = new sfGeneratorManager();
>     $data = $manager->generate('sfPropelGenerator', $parameters);
>
>独自のジェネレーターを開発したい場合、`sfGeneratorManeger`クラスと`sfGenerator`クラスのAPIドキュメントを見て、`sfModelGenerator`クラスと`sfPropelGenerator`クラスを例としてください。

まとめ
----

バックエンドのアプリケーションを自動的に生成したい場合、基本はよく定義されたスキーマとオブジェクトモデルです。administrationのカスタマイズによって生成されたモジュールのカスタマイズの大半は設定を通して行われます。

`generator.yml`ファイルは生成されたバックエンドのプログラミングにおいて中心的な役割を果たします。このファイルによって内容、機能、`list`、`new`と`edit`ビューの見た目を完全にカスタマイズできます。またPHPコードを一行も書かずに、フィールドラベル、ツールチップ、フィルター、ソートの順番、ページのサイズ、入力タイプ、外部のリレーション、カスタムインタラクション、とクレデンシャルをYAMLで直接管理できます。

administrationジェネレーターが必要な機能をネイティブにサポートしない場合、アクションをオーバーライドする部分テンプレートフィールドと機能は完全な拡張性を提供します。それに加えて、テーマのメカニズムのおかげで、administrationジェネレーターのメカニズムへの適合方法を再利用できます。
