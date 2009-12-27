第14章 - ジェネレーター
=====================

多くのアプリケーションはデータベースに保存されたデータに基づいており、それにアクセスするためのインターフェイスを提供します。symfonyはPropelのオブジェクトに基づいたデータ操作機能を提供するモジュール作成の反復的なタスクを自動化します。オブジェクトモデルが適切に定義したのであれば、symfonyはサイト全体の管理機能(administration)も自動的に生成します。この章では、symfonyに搭載された2つのジェネレーター: scaffoldingジェネレーターとadministrationジェネレーターをお教えします。後者は完全な構文による特別な設定ファイルに依存するので、この章の多くはadministrationジェネレーターのさまざまな可能性について説明します。

モデルに基づいたコード生成
--------------------------

Webアプリケーションにおいて、データアクセスのオペレーションはつぎのように分類できます:

  * レコードの作成
  * レコードの検索
  * レコードの更新(とカラムの修正)
  * レコードの削除

これらのオペレーションは共通なので、頭文字を取った専用の略語であるCRUD(Create、Retrieval、Update、Deletion)が存在します。多くのページはこれらの1つに還もとできます。たとえば、フォーラムのアプリケーションにおいて、最新投稿のリストは検索オペレーション(retrieve)で、投稿への返答は作成オペレーション(create)に対応します。

任意のテーブルに対するCRUDオペレーションを実装する基本的なアクションとテンプレートはWebアプリケーション内部で繰り返し作られます。symfonyにおいて、開発もしくはバックエンドインターフェイスの初期の部分を加速するために、モデルレイヤーはCRUDオペレーションを生成できるようにするための十分な情報を持ちます。

モデルに基づいたすべてのコード生成タスクはモジュール全体を作成します。これはつぎの形式のsymfonyコマンドラインへの呼び出しによって行われます:

    > symfony <TASK_NAME> <APP_NAME> <MODULE_NAME> <CLASS_NAME>

コード生成タスクは`propel-init-crud`、`propel-generate-crud`と`propel-init-admin`です。

### scaffoldingとadministration

アプリケーションの開発期間において、2つの異なる目的のためにコード生成機能を利用できます:

  * scaffolding(訳注：日本語で足場を意味する)は任意のテーブル上でCRUDを実行するために必要とされる基本構造(アクションとテンプレート)です。開発のためのガイドラインとして提供することが目的なので、コードの量は必要最小限です。これはロジックとプレゼンテーションにマッチするように適合しなければならない起点です。多くの場合scaffoldingは開発フェーズで使われます。データベースへのWebアクセスを提供する、プロトタイプを開発する、テーブルに基づいてモジュールを初期にブートストラップするためなどです。
  * administrationは、バックエンドの管理専用の、データ操作のための洗練されたインターフェイスです。administrationのコードを手作業で修正することは想定されていないので、administrationはscaffoldingとは異なります。これらをカスタマイズする、拡張する、設定もしくは継承を通して集約できます。これらのプレゼンテーションは重要であり、これらはソーティングやパジネーション、フィルタリングといった追加機能を利用します。administrationは完成品のソフトウェアとして作られ顧客に渡されます。

symfonyのコマンドラインはscaffoldingを指定するためにcrudという単語を使い、administrationのためにadminという単語を使います。

### コードを初期化するもしくは生成する

symfonyはコードを生成する方法を2つ提供します。継承(`init`)による方法とコード生成(`generate`)による方法です。

モジュールを初期化すると、symfonyから継承した空のクラスが作成され、アクションとテンプレートのPHPコードは修正されないように覆い隠されます。データ構造が完成していない場合、もしくはレコードを操作するデータベースへの素早いインターフェイスが必要な場合に便利です。実行時に実行されるコードはアプリケーションではなくキャッシュに設置されます。この種の生成用のコマンドラインタスクは`propel-init-`で始まります。

初期化されたアクションのコードは空です。たとえば、初期化された`article`モジュールはつぎのようなアクションを持ちます:

    [php]
    class articleActions extends autoarticleActions
    {
    }

一方で、修正可能なアクションとテンプレートのコードも生成できます。それゆえモジュールの結果はsymfonyのクラスから独立しているので、設定ファイルを使用して変更できません。この種類の生成用のコマンドラインタスクは`propel-generate-`で始まります。

scaffoldingは開発のための土台として提供するためにビルドされるので、scaffoldingを生成することがベストであることがよくあります。一方で、administrationは設定の変更を通して更新することが容易なので、データモデルを変更した場合でも有効です。administrationが初期化のみ行われる理由はそういうわけです。

### データモデルの例

この章全体を通して、リストはシンプルな例に基づいたsymfonyのジェネレーターの機能を示します。これによって8章を思い出すでしょう。これは2つの`Article`クラスと`Comment`クラスを含む、Webログのアプリケーションのよく知られた例です。図14-1で描かれているように、リスト14-1はスキーマを示します。

リスト14-1 - blogアプリケーションの`schema.yml`の例

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

図14-1 データモデルの例

![データモデルの例](/images/book/F1401.png "データモデルの例")

コード生成を可能にするためにスキーマ作成の期間で従わなければならない特別なルールは存在しません。symfonyはスキーマをそのまま使用し、scaffoldingもしくはadministrationを生成するためにスキーマの属性を解釈します。

>**TIP**
>この章を最大限に活用するには、例の内容を実際に行う必要があります。リストで説明されたすべてのステップを眺めるのであれば、symfonyが何を生成し、生成されたコードで何が行われるのかということをより理解できるようになります。以まえに説明されたようなデータ構造を作成したり、`blog_article`と`blog_comment`テーブルをともなうデータベースを作成したり、サンプルデータをこのデータベースに投入したくなるでしょう。

scaffolding
-----------

scaffoldingは初期のアプリケーション開発において大いに役立ちます。1つのコマンドで、symfonyは任意のテーブルの記述に基づいてモジュール全体を作成します。

### scaffoldingを生成する

`Article`モデルクラスに基づいた`article`モジュール用のscaffoldingを生成するには、つぎのコマンドを入力します:

    > symfony propel-generate-crud myapp article Article

symfonyは`schema.yml`のなかの`Article`クラスの定義を読み込み、`myapp/modules/article/`ディレクトリのなかでそれをもとにテンプレートとアクションのセットを作ります。

生成モジュールはビューを3つ含みます。`list`ビューはデフォルトのビューで、図14-2で再現された[http://localhost/myapp_dev.php/article](http://localhost/myapp_dev.php/article)をブラウジングするとき、`blog_article`テーブルの列を表示します。

図14-2 - `article`モジュールの`list`ビュー

![articleモジュールのlistビュー](/images/book/F1402.png "articleモジュールのlistビュー")

`show`ビューを表示する記事の識別子をクリックしてください。図14-3のように1つのページで一列の詳細な情報が表示されます。

図14-3 - `article`モジュールの`show`ビュー

![articleモジュールのshowビュー](/images/book/F1403.png "articleモジュールのshowビュー")

edit linkをクリックすると記述内容を編集する、もしくは`list`ビュー内部のcreate linkをクリックして新しい記事を作ると、図14-4で再現された`edit`ビューが表示されます。

このモジュールを利用することで、新しい記事を作成する、既存の記事を修正するもしくは削除することができます。生成されたコードはさらなる開発のためのよい土台です。リスト14-2は新しいモジュールの生成されたアクションとテンプレートの一覧を示しています。

図14-4 - `article`モジュールの`edit`ビュー

![articleモジュールのeditビュー](/images/book/F1404.png "articleモジュールのeditビュー")

リスト14-2 - 生成されたCRUD要素(`myapp/modules/article/`)

    // actions/actions.class.phpにおいて
    index           // つぎのlistアクションにフォワードする
    list            // テーブルのすべてのレコードのリストを表示する
    show            // 1つのレコードのすべてのカラムのリストを表示する
    edit            // 1つのレコードのカラムを修正するために一つのフォームを表示する
    update           // editアクションフォームによって呼び出されたアクション
    delete           // レコードを削除する
    create           // 新しいレコードを作成する

    // テンプレートにおいて/
    editSuccess.php  // レコード編集フォーム(editビュー)
    listSuccess.php  // すべてのレコードのリスト(listビュー)
    showSuccess.php  // 1つのレコードの詳細 (showビュー)

これらのアクションとテンプレートのロジックのすべてを再生成して説明するよりも、はるかにシンプルで明快です。リスト14-3は生成されたアクションのクラス内部で生成されたものの一部を示しています。

リスト14-3 - 生成された`Action`クラス(`myapp/modules/article/actions/actions.class.php`)

    [php]
    class articleActions extends sfActions
    {
      public function executeIndex()
      {
        return $this->forward('article', 'list');
      }

      public function executeList()
      {
        $this->articles = ArticlePeer::doSelect(new Criteria());
      }

      public function executeShow()
      {
        $this->article = ArticlePeer::retrieveByPk($this->getRequestParameter('id'));
        $this->forward404Unless($this->article);
      }
      ...

あなたの要件に満たすように生成されたコードを修正し、情報のやりとりをしたいすべてのテーブルに対して、CRUDの生成を繰り返せば、基本的な作業用アプリケーションが手に入ります。scaffoldingを生成することで開発が大いにはかどります; 汚い仕事はsymfonyに任せ、インターフェイスと仕様に集中しましょう。

### scaffoldingを初期化する

データベースのデータにアクセスできることを確認する必要があるとき、scafoldingの初期化がもっとも役立ちます。すべてが立派に動作していることを確認したらビルドして削除することを素早く行うことができます。

`Article`クラス名のレコードを処理する`article`モジュールを作成するPropel のscaffoldingを初期化するには、つぎのコマンドを入力します:

    > symfony propel-init-crud myapp article Article

デフォルトのアクションを使用して`list`ビューにアクセスできます:

    http://localhost/myapp_dev.php/article

ページの結果は生成されたscaffoldingのためのものとまったく同じです。これらをデータベースへのシンプルなWebインターフェイスとして使うことができます。

`article`モジュール内部で新しく作成された`actions.class.php`を確認すると、これが空であることがわかります: すべての内容は自動生成されたクラスから継承されたものです。テンプレートにも同じことがあてはまります: `templates/`ディレクトリ内部にはテンプレートファイルは存在しません。初期化されたアクションとテンプレートの背後にあるコードは生成されたscaffooldingに関して同じですが、これらはアプリケーションのキャッシュ(`myproject/cache/myapp/pord/module/autoArticle/`)内部だけに存在します。

アプリケーションの開発期間において、インターフェイスにかかわらず、開発者はデータと情報のやりとりを行うscaffoldingを初期化します。コードをカスタマイズすることは想定されていません; 初期化されたscaffoldingはデータを管理するphpMyadminのシンプルな代替物とみなすことができます。

administration
--------------

symfonyは、アプリケーションのバックエンドのために、`schema.yml`ファイルからのモデルクラス定義に基づいて、高度なモジュールを作成できます。生成されたadministrationモジュールだけを用いてサイト全体のadministrationを作成できます。このセクションの例において`backend`アプリケーションに追加されたadministraionモジュールを説明します。プロジェクトが`backend`アプリケーションを持たない場合、`init-app`タスクを呼び出してスケルトンを作成してください:

    > symfony init-app backend

administrationモジュールは`generator.yml`という名前の特別な設定ファイルを通してモデルを解釈します。すべての生成されたコンポーネントとモジュールの外見を拡張するために`generator.yml`ファイルを変更できます。このようなモジュールは通常のモジュールメカニズムからの恩恵を受けます(レイアウト、バリデーション、ルーティング、カスタム設定、オートロードなど)。独自機能を生成されたadministrationに統合するために、生成されたアクションもしくはテンプレートをオーバーライドすることもできますが、`generator.yml`はもっとも共通の要件を考慮して、PHPのコードの使いかたを限定的なものにします。

### administrationモジュールを初期化する

symfonyによって、モジュールを基本単位としてadministrationをビルドします。`propel-init-admin`タスクを用いることで、モジュールはPropelのオブジェクトに基づいて生成されます。`propel-init-admin`タスクはscaffoldingを初期化するために使われる構文と似た構文を使います:

    > symfony propel-init-admin backend article Article

この呼び出しだけで`backend`アプリケーションのなかに`Article`クラスの定義に基づいた`article`モジュールが作成されるので、つぎのURLからアクセスできます:

    http://localhost/backend.php/article

図14-5、図14-6で描かれている生成されたモジュールの外見は商業用のアプリケーションとしてそのまま利用できるほど十分に洗練されています。

図14-5 - `backend`アプリケーション内部の`article`モジュールの`list`ビュー

![backendアプリケーション内部のarticleモジュールのlistビュー](/images/book/F1405.png "backendアプリケーション内部のarticleモジュールのlistビュー")

図14-6 - バックエンドで`article`モジュールの`edit`ビュー

![バックエンドでarticleモジュールのeditビュー](/images/book/F1406.png "バックエンドでarticleモジュールのeditビュー")

administrationのインターフェイスとscaffoldingのインターフェイスの違いは現時点は顕著には見えないかもしれませんが、administrationの柔軟な設定によってPHPのコードを使わずに多くの追加機能を持つ基本的なレイアウトを強化できます。

>**NOTE**
>administrationモジュールだけを初期化することができます(生成はできません)。

### 生成されたコードを見る

`apps/backend/modules/article/`ディレクトリ内のadministrationの`Article`モジュールのコードは初期化だけ行われたので空です。このモジュールの生成されたコードを吟味するための最良の方法はブラウザーを利用してこれと情報のやりとりをすることと`cache/`フォルダーの内容を確認することです。リスト14-4はキャッシュで見つかる生成されたアクションとテンプレートのリストを表示します。

リスト14-4 - 生成されたadministrationの要素(`cache/backend/ENV/modules/article/`)

    // actions/actions.class.phpにおいて
    create           // editにフォワードする
    delete 	          //レコードを削除する
    edit             // レコードのフィールドを修正するためにフォームを表示する
                     // フォーム投稿を扱う
    index            // listに進む
    list             // テーブルのすべてのレコードのリストを表示する
    save             // editに進む

    // templates/において
    _edit_actions.php
    _edit_footer.php
    _edit_form.php
    _edit_header.php
    _edit_messages.php
    _filters.php
    _list.php
    _list_actions.php
    _list_footer.php
    _list_header.php
    _list_messages.php
    _list_td_actions.php
    _list_td_stacked.php
    _list_td_tabular.php
    _list_th_stacked.php
    _list_th_tabular.php
    editSuccess.php
    listSuccess.php

これは生成されたadministrationモジュールがおもに2つのビュー、`edit`ビューと`list`ビューで構成されることを示します。コードを見てみると、モジュール性が非常に高く、読みやすく拡張性のあるものであることがわかります。

### generator.yml設定ファイルを導入する

(administrationで生成されたモジュールは`show`アクションを持たないこということは別にして)scaffoldingとadministrationの主要な違いはadministrationがYAML形式の`generator.yml`設定ファイルで見つかるパラメーターに依存することです。新しく生成されたadministrationモジュールのデフォルト設定を見るには、リスト14-5で再現されている、`backend/modules/article/config/generator.yml`ディレクトリに設置された`generator.yml`ファイルを開いてください。

リスト14-5 - ジェネレーターのデフォルト設定(`backend/modules/article/config/generator.yml`)

    generator:
      class:              sfPropelAdminGenerator
      param:
        model_class:      Article
        theme:            default

この設定は基本的なadministrationを生成するために十分なものです。カスタマイズされた内容は`theme`の行の後の`param`キーの下に追加されます(`generator.yml`ファイルの底に追加されたすべての行は少なくとも適切にインデントされた4つの空白スペースで始めなければならないことを意味します)。リスト14-6はカスタマイズされた典型的な`generator.yml`を示しています。

リスト14-6 - 典型的なジェネレーターの完全な設定

    generator:
      class:              sfPropelAdminGenerator
      param:
        model_class:      Article
        theme:            default

        fields:
          author_id:      { name: Article author }

        list:
          title:          List of all articles
          display:        [title, author_id, category_id]
          fields:
            published_on: { params: date_format='dd/MM/yy' }
          layout:         stacked
          params:         |
            %%is_published%%<strong>%%=title%%</strong><br /><em>by %%author%%
            in %%category%% (%%published_on%%)</em><p>%%content_summary%%</p>
          filters:        [title, category_id, author_id, is_published]
          max_per_page:   2

        edit:
          title:          Editing article "%%title%%"
          display:
            "Post":       [title, category_id, content]
            "Workflow":   [author_id, is_published, created_on]
          fields:
            category_id:  { params: disabled=true }
            is_published: { type: plain}
            created_on:   { type: plain, params: date_format='dd/MM/yy' }
            author_id:    { params: size=5 include_custom=>> Choose an author << }
            published_on: { credentials:  }
            content:      { params: rich=true tinymce_options=height:150 }

つぎのセクションでこの設定ファイルで利用可能なすべてのパラメーターの詳細内容を説明します。

ジェネレーターの設定
------------------

ジェネレーターの設定ファイルはとても強力で、生成されたadministrationを多くの方法で変更できます。しかしこの機能には代償があります: 全体の構文の記述が読んで学ぶには長いので、文章で説明したら、この章がこの本でもっとも長くなってしまいます。ですのでsymfonyのWebサイトはこの構文を学ぶための助けになる追加リソースを提示します: administrationジェネレーターのチートシートが図14-7で再現されてます。[http://www.symfony-project.org/uploads/assets/sfAdminGeneratorRefCard.pdf](http://www.symfony-project.org/uploads/assets/sfAdminGeneratorRefCard.pdf)から保存し、この章のつぎの例を読むときにこれを頭のなかにとどめておいてください。

このセクションの例は、`Comment`クラス定義に基づいて、administrationの`comment`モジュールと同様に、administrationの`article`モジュールを調整します。`propel-init-admin`タスクで後者を作成してください:

    > symfony propel-init-admin backend comment Comment

図14-7 - administrationジェネレーターのチートシート

![administrationジェネレーターチートのシート](/images/book/F1407.png "administrationジェネレーターチートのシート")

### フィールド

デフォルトでは、`list`ビューのカラムと`edit`ビューのフィールドは`schema.yml`で定義されたカラムです。`generator.yml`によって、どのフィールドが表示され、隠されるかを選ぶことが可能で、これらがオブジェクトモデルに直接対応していなくても、独自のフィールドを追加できます。

#### フィールドの設定

administrationジェネレーターは`schema.yml`ファイルのカラムごとに`field`を作ります。`fields`キーの下で、それぞれのフィールドの表示方法やフォーマット方法などを修正できます。たとえば、リスト14-7で示されるフィールドの設定は`title`フィールド用のカスタムラベルクラスと入力タイプ、そして`content`フィールド用のラベルとツールチップを定義します。つぎのセクションでそれぞれのパラメーターが動作する方法を詳細に説明します。

リスト14-7 - カラムに対してカスタムラベルを設定する

    generator:
      class:              sfPropelAdminGenerator
      param:
        model_class:      Article
        theme:            default

        fields:
          title:          { name: Article Title, type: textarea_tag, params: class=foo }
          content:        { name: Body, help: Fill in the article body }

リスト14-8でお手本が示されているように、すべてのビューに対するこのデフォルトの定義に加えて、任意のビュー(`list`と`edit`)のためのフィールドの設定をオーバーライドできます。

リスト14-8 - ビュー単位でグローバルな設定ビューをオーバーライドする

    generator:
      class:              sfPropelAdminGenerator
      param:
        model_class:      Article
        theme:            default

        fields:
          title:          { name: Article Title }
          content:        { name: Body }

        list:
          fields:
            title:        { name: Title }

        edit:
          fields:
            content:      { name: Body of the article }

一般的な原則があります: `fields`キーの下のモジュール全体に対して設定される任意の設定も後に続くビュー固有の領域(`list`と`edit`)でオーバーライドできます。

#### フィールドをdisplay設定に追加する

`fields`セクションで定義したフィールドはそれぞれのビューに対して表示、隠す、順番に並べるなど、さまざまな方法で分類できます。`display`キーはこの目的のために使われます。たとえば、`comment`モジュールのフィールドを順番に並べるには、リスト14-9のコードを使います。

リスト14-9 - 表示フィールドを選択する(`modules/comment/config/generator.yml`)

    generator:
      class:              sfPropelAdminGenerator
      param:
        model_class:      Comment
        theme:            default

        fields:
          article_id:     { name: Article }
          created_at:     { name: Published on }
          content:        { name: Body }

        list:
          display:        [id, article_id, content]

        edit:
          display:
            NONE:         [article_id]
            Editable:     [author, content, created_at]

図14-8のように`list`は3つのカラムを表示し、図14-9のように、`edit`フォームは2つのグループに集められた4つのフィールドを表示します。 

図14-8 - `comment`モジュールの`list`ビュー内部のカスタムカラム設定

![commentモジュールのlistビュー内部のカスタムカラム設定](/images/book/F1408.png "commentモジュールのlistビュー内部のカスタムカラム設定")

図14-9 - `comment`モジュールの`edit`ビュー内部でフィールドを分類する

![commentモジュールのeditビュー内でフィールドを分類する](/images/book/F1409.png "commentモジュールのeditビュー内でフィールドを分類する")

`display`設定を2つの方法で利用できます:

  * 表示するカラムとそれらが表示される順番を選ぶために、以前の`list`ビューのように、フィールドをシンプルな配列に記入します。
  * フィールドを分類するために、グループ名をキーとして持つ連想配列、もしくは名なしのグループに対しては`NONE`を使います。値は順番に並べられたカラム名の配列です。

>**TIP**
>デフォルトでは、主キーのカラムはどちらのビューでも決して現れません。

#### カスタムフィールド

当然のことながら、`generator.yml`で設定されたフィールドはスキーマで定義された実際のカラムに対応している必要はありません。関連するクラスがカスタムゲッターを提供する場合、これを`list`ビューのためのフィールドとして使うことができます; ゲッターかつ/もしくはセッターが存在する場合、このクラスは`edit`ビューでも利用できます。たとえば、リスト14-10のように、Articleモデルを`getNbComments()`メソッドで拡張できます。

リスト14-10 - カスタムゲッターをモデルに追加する(`lib/model/Article.class.php`)

    [php]
    public function getNbComments()
    {
      return $this->countComments();
    }

リスト14-11のように、`nb_comments`は生成されたモジュールのなかのフィールドとして利用できます(ゲッターはcamelCaseバージョンのフィールド名を使います)。

リスト14-11 - カスタムゲッターはadministrationモジュールに対して追加カラムを提供する(`backend/modules/article/config/generator.yml`)

    generator:
      class:              sfPropelAdminGenerator
      param:
        model_class:      Article
        theme:            default

        list:
          display:        [id, title, nb_comments, created_at]

`article`モジュールの`list`ビューの結果は図14-10で示されます。

図14-10 - `article`モジュールの`list`ビュー内のカスタムフィールド

![articleモジュールのlistビュー内部のカスタムフィールド](/images/book/F1410.png "articleモジュールのlistビュー内部のカスタムフィールド")

カスタムフィールドは生のデータ以上のものを表示するためにHTMLのコードも返します。たとえば、リスト14-12で示されるような`Comment`クラスを`getArticleLink()`メソッドで拡張できます。

リスト14-12 - HTMLを返すカスタムゲッターを追加する(`lib/model/Comment.class.php`)

    [php]
    public function getArticleLink()
    {
      return link_to($this->getArticle()->getTitle(), 'article/edit?id='.$this->getArticleId());
    }

リスト14-11と同じ構文でこの新しいゲッターを`comment/list`ビュー内のカスタムフィールドとして使うことができます。リスト14-13の例と、図14-11の結果をご覧ください。ゲッター(記事のハイパーリンク)によって出力されるHTMLのコードは記事の主キーの代わりに2番目のカラムに現れます。

リスト14-13 - HTML結果を返すカスタムゲッターは追加カラムとして使うこともできる(`modules/comment/config/generator.yml`)

    generator:
      class:              sfPropelAdminGenerator
      param:
        model_class:      Comment
        theme:            default

        list:
          display:        [id, article_link, content]

図14-11 - `comment`モジュールの`list`ビュー内部のカスタムフィールド

![commentモジュールのlistビュー内部のカスタムフィールド](/images/book/F1411.png "commentモジュールのlistビュー内部のカスタムフィールド")

#### 部分テンプレートフィールド

モデルに設置されたコードはプレゼンテーションから独立していなければなりません。初期の`getArticleLink()`メソッドの例はレイヤー分離の原則を順守していません。ビューのコードのなかにはモデルレイヤーに現れるものがあるからです。正しい方法で、同じ結果を実現するには、部分テンプレート内部のカスタムフィールドに対してHTMLを出力するコードを設置したほうがベターです。幸いして、administrationジェネレーターによってアンダースコアのプレフィックスを持つフィールド名を宣言できます。この場合、リスト14-13の`generator.yml`ファイルはリスト14-14のように修正されます。

リスト14-14 - `_`のプレフィックスをつけることで部分テンプレートを追加カラムとして使うことができる、

        list:
          display:        [id, _article_link, created_at]

これを機能させるには、リスト14-15で示されるように、`_article_link.php`部分テンプレートを`modules/comment/tempaltes/`ディレクトリのなかに作らなければなりません。

リスト14-15 - `list`ビューのための部分テンプレートの例(`modules/comment/templates/_article_link.php`)

    <?php echo link_to($comment->getArticle()->getTitle(), 'article/edit?id='.$comment->getArticleId()) ?>

部分テンプレートフィールドの部分テンプレートテンプレートはクラスによって命名された変数(この例では`$comment`)を通して現在のオブジェクトにアクセスできることに注意してください。たとえば、`UserGroup`という名前のクラスのためにビルドされたモジュールに対して、部分テンプレートは`$user_group`変数を通して現在のオブジェクトにアクセスできるようになります。

レイヤー分離の原則が順守されることを除いて、結果は図14-11と同じです。レイヤー分離の原則を順守することに慣れたら、アプリケーションはより維持しやすくなります。

部分テンプレートフィールドのパラメーターをカスタマイズする必要がある場合、`field`キーの下で通常のフィールドと同じ事を行います。アンダースコア(`_`)をキーの先頭に含めないでください。リスト14-16をご覧ください。

リスト14-16 - 部分テンプレートフィールドのプロパティは`fields`キーの下でカスタマイズできる

          fields:
            article_link:   { name: Article }

部分テンプレートがロジックで混雑するようになったら、おそらくはこれをコンポーネントで置き換えたいと思うでしょう。リスト14-17のように、`_`のプレフィックスを`~`に変更することで、コンポーネントフィールドを定義できます。

リスト14-17 - コンポーネントは追加のカラムとして利用できる。`~`のプレフィックスを使う

        ...
        list:
          display:        [id, ~article_link, created_at]

生成されたテンプレートにおいて、現在のモジュールの`articleLink`コンポーネントを呼び出すことでこれは生成されます。

>**NOTE**
>カスタムと部分テンプレートのフィールドは`list`ビュー、`edit`ビューのなかと、フィルターに対して利用できます。いくつかのビューに対して同じ部分テンプレートを使う場合、コンテキスト(`'list'`、`'edit'`、もしくは `'filter'`)が`$type`変数に保存されます。

### ビューのカスタマイゼーション

`edit`ビューと`list`ビューの外見を変更するために、テンプレートを変えたいと思うことがあります。しかしこれらは自動的に生成されるので、これを行うのはあまりよいアイディアではありません。代わりに`generator.yml`設定ファイルを使用すべきです。モジュール性を損なうことなくほとんどすべての必要なことを行うことができるからです。

#### ビューのタイトルを変更する

フィールドのカスタムセットに加えて、`list`ページと`edit`ページはカスタムページタイトルを持ちます。たとえば、`article`ビューのタイトルをカスタマイズしたい場合、リスト14-18のように行います。`edit`ビューの結果は図14-12に描かれています

リスト14-18 - それぞれのビューに対してカスタムタイトルを設定する(`backend/modules/article/config/generator.yml`)

        list:
          title:          List of Articles
          ...

        edit:
          title:          Body of article %%title%%
          display:        [content]

図14-12 - `article`モジュールの`edit`ビュー内部のカスタムタイトル

![articleモジュールのeditビュー内部のカスタムタイトル](/images/book/F1412.png "articleモジュールのeditビュー内部のカスタムタイトル")

デフォルトのタイトルはクラスの名前を使うので、モデルが明白なクラスの名前を使うのであれば、これらのクラス名で十分であることはよくあります。

>**TIP**
>`generator.yml`の文字列の値において、フィールドの値は`%%`で囲まれたフィールドの名前を通してアクセスできます。

#### ツールチップを追加する

`list`と`edit`ビュー内部において、表示されるフィールドを記述するための助けになるツールチップを追加できます。たとえば、ツールチップを`comment`モジュールの`edit`ビューの`article_id`フィールドに追加するには、リスト14-19のように、`help`プロパティを`fields`定義のなかに追加します。結果は図14-13に示されています。

リスト14-19 - `edit`ビューでツールチップを設定する(`modules/comment/config/generator.yml`)

        edit:
          fields:
            ...
            article_id:   { help: The current comment relates to this article }

図14-13 - `comment`モジュールの`edit`ビュー内部のツールチップ

![commentモジュールのeditビュー内部のツールチップ](/images/book/F1413.png "commentモジュールのeditビュー内部のツールチップ")

`list`ビューにおいて、ツールチップはカラムヘッダーに表示されます; `edit`ビュー内において、これらはinputの下に現れます。

#### 日付の書式を修正する

リスト14-20でお手本が示されているように、`date_format`パラメーターを使い始めると同時にカスタム書式で日付を表示できます。

リスト14-20 `list`ビューのなかで日付の書式設定をする

        list:
          fields:
            created_at:         { name: Published, params: date_format='dd/MM' }

このパラメーターは以前の章で説明された`format_date()`ヘルパーと同じフォーマットパラメーターをとります。

>**SIDEBAR**
>administrationテンプレートは国際化の準備ができています
>
>生成されたテンプレートのなかで見つかるすべてのテキストは自動的に国際化されます(たとえば、`__()`ヘルパーへの呼び出しに囲まれます)。このことは、以前の章で説明されたように、`apps/myapp/i18n/`ディレクトリ内において、語句の翻訳をXLIFFファイルに追加することで、生成されたadministrationを簡単に翻訳できることを意味します。

### listビュー固有のカスタマイズ

`list`ビューは、表形式、もしくはすべての詳細な内容が一行に積み重ねられた状態で、レコードの詳細を表示できます。これはフィルター、パジネーションとソート機能も含みます。これらの機能は設定によって変更できます。詳細はつぎのセクションで説明します。

#### レイアウトを変更する

デフォルトでは、`list`ビューと`edit`ビュー間のハイパーリンクは主キーのカラムによって生じています。図14-11に戻ると、コメントリストの`id`カラムがそれぞれのコメントの主キーを表示するだけでなく、ユーザーが`edit`ビューにアクセスすることを許可するハイパーリンクを提供します。

別のカラム上に現れるレコードの詳細へのハイパーリンクが望ましいのであれば、`display`キーの等号(`=`)をプレフィックスとしてカラムの名前に追加します。リスト14-21は`list`コメントの表示されるフィールドから`id`を除去して代わりに`content`フィールド上にハイパーリンクを置く方法を示しています。図14-14のスクリーンショットを確認してください。

リスト14-21 - `edit`ビューのためのハイパーリンクを`list`ビューに移動させる(`modules/comment/config/gererator.yml`)

        list:
          display:    [article_link, =content]

図14-14 - `comment`モジュールの`list`ビューのなかで、リンクを別のカラム上の`edit`ビューに移動させる

![commentモジュールのlistビュー内で、リンクを別のカラム上のeditビューに移動させる](/images/book/F1414.png "commentモジュールのlistビュー内で、リンクを別のカラム上のeditビューに移動させる")

デフォルトでは、以前示されたように、`list`ビューは、フィールドがカラムとして表示される`tabular`レイアウトを使います。しかし`stacked`レイアウトを使用してフィールドをテーブルの全長を詳しく記述する単独の文字列に連結することもできます。`stacked`レイアウトを選ぶ場合、`params`キーにリストのそれぞれの行の値を定義するパターンを組み込まなければなりません。たとえば、リスト14-22は`comment`モジュールの`list`ビューに対して`stacked`レイヤーを定義します。結果は図14-15です。

リスト14-22 - `list`ビューで`stacked`レイアウトを使う(`modules/comment/cofig/generator.yml`)

        list:
          layout:  stacked
          params:  |
            %%=content%% <br />
            (sent by %%author%% on %%created_at%% about %%article_link%%)
          display:  [created_at, author, content]

図14-15 - `comment`モジュールの`list`ビュー内部のstackedレイアウト

![commentモジュールのlistビュー内のstackedレイアウト](/images/book/F1415.png "commentモジュールのlistビュー内のstackedレイアウト")

`tabular`レイアウトは`display`キーの下でフィールドの配列を必要としますが、`stacked`レイアウトはそれぞれのレコードに対して生成されたHTMLコードのために`params`キーを使うことに留意してください。しかしながら、インタラクティブなソートに対して利用できるカラムヘッダーを決定するために`display`配列が`stacked`レイアウトでも使われます

#### 結果をフィルタリングする

`list`ビューにおいて、フィルターのインタラクションのセットを追加できます。これらのフィルターによって、ユーザーは結果をより少なく表示して速く望むものに到達できます。フィールド名の配列で、`filters`キーの下にフィルターを設定します。たとえば、図14-16のフィルターボックスと同じようなものを表示するには、リスト14-23のように、`article_id`、`author`フィールド、と`created_at`フィールド上のフィルターをコメントの`list`ビューに追加します。これを機能させるには、`__toString()`メソッドを`Article`クラス(たとえば、記事の`title`を返す)に追加する必要があります。

リスト14-23 - `list`ビューでフィルターを設定する(`modules/comment/config/generator.yml`)

        list:
          filters: [article_id, author, created_at]
          layout:  stacked
          params:  |
            %%=content%% <br />
            (sent by %%author%% on %%created_at%% about %%article_link%%)
          display:  [created_at, author, content]

図14-16 - `comment`モジュールの`list`ビュー内部のフィルター

![commentモジュールのlistビュー内部のフィルター](/images/book/F1416.png "commentモジュールのlistビュー内部のフィルター")

symfonyによって表示されるフィルターはカラム型に依存します:

  * テキストカラム(たとえば`comment`モジュール内の`author`フィールド)に対して、フィルターはワイルドカード(`*`)によるテキストベースの検索を可能にするテキスト入力です。
  * 外部キー(たとえば`comment`モジュール内部の`article_id`フィールドのリスト)に対して、フィルターは関連するテーブルのレコードのドロップダウンリストです。通常の`object_select_tag()`に関しては、ドロップダウンリストのオプションは関連するクラスの`__toString()`メソッドによって返されるものです。
  * 日付のカラム(たとえば`comment`カラム内の`created_at`フィールド)に対して、フィルターはリッチな日付タグのペアで(カレンダーウィジェットによって入力されたテキストフィールド)、時間の間隔を選択できるようにします。
  * ブール型のカラムに対して、フィルターは`true`と`false`と`true or false`オプションを持つドロップダウンリストです。最後の値はフィルターを再初期化します。

リストのなかで部分テンプレートフィールドを使うことと同じように、symfonyが独自に処理できないフィルターを作成するために部分テンプレートフィルターも利用できます。たとえば、2つの値(`open`と`closed`)だけ含む`state`フィールドが存在するが、何らかの理由で、テーブルのリレーションを使う代わりにこれらの値をフィールドに直接保存する理由がある場合を想像してください。このフィールドのシンプルなフィールド(の`string`型)はテキストベースの検索ですが、あなたが欲しいと思うものはおそらく値のドロップダウンリストです。これを部分テンプレートフィルターで実現することは簡単です。リスト14-24の実装例をご覧ください。

リスト14-24 - 部分テンプレートフィルターを使う

    [php]
    // templates/_state.phpにおいて、部分テンプレートを定義する
    <?php echo select_tag('filters[state]', options_for_select(array(
      '' => '',
      'open' => 'open',
      'closed' => 'closed',
    ), isset($filters['state']) ? $filters['state'] : '')) ?>

    // config/generator.ymlにおいて部分テンプレートフィルターをフィルターリストに追加する
        list:
          filters:        [date, _state]

部分テンプレートが`$filters`の値にアクセス可能で、フィルターの現在の値を取得するために便利であることを覚えておいてください。

空の値を探すためにとても便利な最後の1つのオプションがあります。著者を持たないコメントを表示するコメントのリストをフィルタリングしたい場合を想像してください。問題はauthorのフィルターを空のままにしておく場合、これが無視されることです。解決方法はリスト14-25のように、`filter_is_empty`フィールド設定を`true`に設定と、フィルターは追加のチェックボックスを表示するので、図14-17で図示されているように、空の値を探すことができます。

リスト14-25 - `list`ビュー内部の`author`フィールドの上に空の値のフィルタリングを追加する

        list:
          fields:
            author:   { filter_is_empty: true }
          filters:    [article_id, author, created_at]

図14-17 - 空の`author`の値をフィルタリングすることを可能にする

![空のauthorの値をフィルタリングすることを可能にする](/images/book/F1417.png "空のauthorの値をフィルタリングすることを可能にする")

#### リストをソートする

`list`ビューにおいて、図14-18で示されるように、テーブルのヘッダーはリストを再び順番に並べるために使われるハイパーリンクです。これらのヘッダーは`tabular`レイアウトと`stacked`レイアウトの両方で表示されます。これらのリンクをクリックするとリストの順番を再配置する`sort`パラメーターでページがリロードされます。

図14-18 - `list`ビューのテーブルヘッダーはソートをコントロールする

![listビューのテーブルヘッダーはソートはコントロールする](/images/book/F1418.png "listビューのテーブルヘッダーはソートをコントロールする")

カラムにしたがって直接ソートされるリストを指し示す構文を再利用できます:

    [php]
    <?php echo link_to('日付順のコメントリスト', 'comment/list?sort=created_at&type=desc' ) ?>

`generator.yml`ファイル内部の`list`ビューに対して直接デフォルトの`sort`の順番を定義することも可能です。構文はリスト14-26で示された例に従います。

リスト14-26 - `list`ビューのなかでデフォルトの`sort`フィールドを設定する

        list:
          sort:   created_at
          # ソートの順番を指定するための、代替構文
          sort:   [created_at, desc]

>**NOTE**
>実際のカラムに対応するフィールドだけが、カスタムもしくは部分テンプレートフィールドではなく、ソートのコントロール機能に変換されます。

#### パジネーションをカスタマイズする

生成されたadministrationは大きなテーブルさえも効率的に処理します。`list`ビューがデフォルトでパジネーションを使うからです。テーブル内の実際の列の数がページごとの列の最大数を越えるとき、パジネーションのコントロール機能がリストの底に現れます。たとえば、図14-19はページごとに表示される5つのコメントの制限ではなく、テーブル内で6つのテストのコメントを持つコメントのリストを表示しますが、ページごとに表示されるコメント数は最大5まで制限されています。パジネーションはよいパフォーマンスとユーザービリティを保証します。データベースから表示される列のみを効率的に検索し、administrationモジュールによって何百万の列をかかえるテーブルでさえも管理できるからです。

図14-19 - パジネーションのコントロール機能は長いリスト上に現れる

![パジネーションのコントロール機能は長いリスト上に現れる](/images/book/F1419.png " パジネーションのコントロール機能は長いリスト上に現れる")

`max_per_page`パラメーターによってそれぞれのページが表示されるレコードの数をカスタマイズできます:

        list:
          max_per_page:   5

#### ページ配信を加速するためにJoinを使う

デフォルトでは、administrationジェネレーターはレコードのリストを検索する`doSelect()`を使います。しかし、リストで関連するオブジェクトを使う場合、リストを表示するために必要なデータベースのクエリの回数が急に増えることがあります。たとえば、コメントのリストで記事の名前を表示したい場合、関連する`Article`オブジェクトを検索するために追加クエリがリスト内のそれぞれの投稿に対して必要です。ですので、クエリの数を最適化するために、`doSelectJoinXXX()`メソッドを使うページャーを強制したい場合があるかもしれません。これは`peer_method`パラメーターで指定できます。

        list:
          peer_method:   doSelectJoinArticle

18章ではJoinの概念についてより広範囲に説明します。

### editビュー固有のカスタマイズ

`edit`ビューにおいて、ユーザーは任意のレコードのためにそれぞれのカラムの値を修正できます。symfonyはカラムのデータ型にしたがって表示する入力のタイプを決定します。symfonyは`object_*_tag()`ヘルパーを生成し、そのヘルパーに編集するオブジェクトとプロパティを渡します。たとえば、ユーザーが`title`フィールドを編集できることを記事の`edit`ビューの設定が保証する場合です:

        edit:
          display: [title, ...]

このカラムはスキーマで`varchar`型として定義されているので、`edit`ページはタイトルを編集するための通常のテキスト入力タグを表示します。

    [php]
    <?php echo object_input_tag($article, 'getTitle') ?>

#### 入力タイプを変更する

デフォルトの型からフィールドへの変換ルールはつぎのとおりです:

  * `integer`、`float`、`char`、`varchar(size)`として定義されたカラムは`edit`ビュー内で`object_input_tag()`として現れます。
  * `longvarchar`として定義されたカラムは`object_textarea_tag()`として現れます。
  * 外部キーのカラムは`object_select_tag()`として現れます。
  * `boolean`として定義されたカラムは`object_checkbox_tag()`として現れます。
  * `timestamp`もしくは`date`として定義されたカラムは`object_input_date_tag()`として現れます。

任意のフィールドに対してカスタム入力タイプを指定するためにこれらのルールをオーバーライドしたい場合を考えます。その範囲で、`fields`定義内の`type`パラメーターを特定のフォームヘルパー名に設定します。生成された`object_*_tag()`オプションに関しては、`params`パラメーターで変更可能です。リスト14-27の例をご覧ください。

リスト14-27 - `edit`ビューに対してカスタム入力タイプとパラメーターを設定する

    generator:
      class:              sfPropelAdminGenerator
      param:
        model_class:      Comment
        theme:            default

        edit:
          fields:
                          ## 入力をドロップし、プレーンテキストを表示するだけ
            id:           { type: plain }
                          ## 入力は編集可能ではない
            author:       { params: disabled=true }
                          ## 入力はテキストエリアT(object_textarea_tag)
            content:      { type: textarea_tag, params: rich=true css=user.css tinymce_options=width:330 }
                          ## 入力は選択 (object_select_tag)
            article_id:   { params: include_custom=Choose an article }
             ...

`param`パラメーターは生成された`object_*_tag()`にオプションとして渡されます。たとえば、先の`article_id`に対する`params`の定義はつぎのようなテンプレートのなかで生成されます:

    [php]
    <?php echo object_select_tag($comment, 'getArticleId', 'related_class=Article', 'include_custom=Choose an article') ?>

このことは通常フォームヘルパーで利用できるすべてのオプションは`edit`ビューでカスタマイズできることを意味します。

#### 部分テンプレートフィールドを扱う

部分テンプレートフィールドは`list`ビューのように`edit`ビューで使用できます。違いは、アクションにおいて、部分テンプレートフィールドによって送られたリクエストパラメーターの値にしたがって、カラムの更新を手動で扱わなければならないことです。symfonyは(実際のカラムに対応する)通常のフィールドを処理する方法を理解していますが、部分テンプレートフィールド内にインクルードした入力を処理する方法を推測することはできません。

たとえば、利用可能なフィールドが`id`、`nickname`、と`password`である`User`クラスのためのadministrationモジュールを想像してください。サイトの管理者はリクエスト上のユーザーのパスワードを変更できなければなりませんが、`edit`ビューはセキュリティの理由からパスワードフィールドの値を表示してはなりません。代わりに、値を変更するためにサイトの管理者が入力できる空のパスワード入力を表示します。このような`edit`ビューのためのジェネレーターの設定はリスト14-28と同じです。

リスト14-28 - 部分テンプレートフィールドを`edit`ビューに含める

        edit:
          display:        [id, nickname, _newpassword]
          fields:
            newpassword:  { name: Password, help: Enter a password to change it, leave the field blank to keep the current one }

`templates/_newpassword.php`部分テンプレートはつぎのようなコードを含みます:

    [php]
    <?php echo input_password_tag('newpassword', '') ?>

この部分テンプレートは、オブジェクトフォームヘルパーではなく、シンプルなフォームヘルパーを使うことに注意してください。フォームの入力を投入するために現在の`User`オブジェクトからパスワードの値を検索するとユーザーのパスワードが公開されてしまう可能性があるので、望ましくないからです。

では、アクションのなかでオブジェクトを更新するためにこのコントロール機能から値を使うには、アクションのなかで`updateUserFromrequest()`メソッドを拡張する必要があります。これを行うには、リスト14-29のように、部分テンプレートフィールドの入力に対してカスタムふるまいを持つアクションのファイルのなかに同じ名前を持つメソッドを作ります。

リスト14-29 - アクションの部分テンプレートフィールドを処理する(`modules/user/actions/actions.class.php`)

    [php]
    class userActions extends autouserActions
    {
      protected function updateUserFromRequest()
      {
        // 部分テンプレートフィールドの入力を扱う
        $password = $this->getRequestParameter('newpassword');

        if ($password)
        {
          $this->user->setPassword($password);
        }

        // symfonyに別のフィールドを扱う
        parent::updateUserFromRequest();
      }
    }

>**NOTE**
>実際の世界において、通常の場合、間違った入力を避けるために`user/edit`ビューは2つの`password`フィールドを含みます。実際には、10章で見た通り、これはバリデーターを通して行われます。administraionによって生成されたモジュールは通常のモジュールと同じようにこのメカニズムから恩恵を受けます。

### 外部キーを扱う

スキーマがテーブルのリレーションを定義する場合、生成されたadministrationモジュールはこれを活用して、より自動化されたコントロール方法を提供するので、リレーションの管理作業が大いに簡素化されます。

#### 一対多のリレーション

1対多のテーブルのリレーションはadministrationジェネレーターによって考慮されます。以前の図14-1で記述されたように、`blog_comment`テーブルは`article_id`フィールドを通して`blog_article`テーブルとの関係を持ちます。`Comment`クラスのモジュールをadministrationジェネレーターによって初期化する場合、`comment/edit`アクションは`blog_article`テーブル内の利用可能なレコードのIDを示す`article_id`をドロップダウンリストとして表示します(説明図は図14-9を再度確認)。

加えて、`Article`オブジェクトで`__toString()`メソッドを定義する場合、ドロップダウンのオプションのテキストは主キーの代わりにこのメソッドを使います。

`article`モジュール(多対一のリレーション)内部の記事に関連するコメントのリストを表示する場合、部分テンプレートフィールドの方法によってモジュールを少しカスタマイズする必要があります。

#### 多対多のリレーション

symfonyはテーブルの多対多のリレーションも考慮しますが、これらをスキーマで定義できないので、いくつかのパラメーターを`generator.yml`ファイルに追加する必要があります。

多対多のリレーションの実装は中間のテーブルを必要とします。たとえば、`blog_article`と`blog_author`の間に多対多のリレーションがある場合(記事は複数の著者によって書かれ、あきらかに、著者は複数の記事を書くことができる)、もしくは図14-20と同じように、データベースはつねに`blog_article_author`と呼ばれるテーブルで終わります。

図14-20 - 多対多のリレーションを実装するためにthrough_classを利用する

![多対多のリレーションを実装するためにthrough_classを利用する](/images/book/F1420.png "多対多のリレーションを実装するために"through class"を利用する")

モデルは`ArticleAuthor`と呼ばれるクラスを持ち、administrationジェネレーターが必要とする唯一のものです。しかし、これをフィールドの`through_class`パラメーターとして渡さなければなりません。

たとえば、`Article`クラスに基づいて生成されたモジュールにおいて、リスト14-30のように`generator.yml`を書く場合、`Author`クラスによる新しい多対多のリレーションを作るためにフィールドを追加できます。

リスト14-30 - `through_class`パラメーターで多対多のリレーションを扱う

        edit:
          fields:
            article_author: { type: admin_double_list, params: through_class=ArticleAuthor }

フィールドは既存のオブジェクト間のリンクを処理するので、通常のドロップダウンリストは十分ではありません。そのためには特別な入力タイプを使わなければなりません。symfonyは2つのリストの関連するメンバーを助けする3つのウィジェットを提供します(図14-21に描かれています):

  * `admin_double_list`は2つの拡張された選択のコントロール機能のセットで、最初のリスト(利用可能な要素)から2番目のリスト(選択された要素)に要素を切り替えるボタンがあります。
  * `admin_select_list`は多くの要素を選択できる拡張された選択のコントロール機能です。
  * `admin_check_list`はチェックボックスタグのリストです。

図14-21 - 多対多のリレーションに対して利用できるコントロール機能

![多対多のリレーションに対して利用できるコントロール機能](/images/book/F1421.png "多対多のリレーションに対して利用できるコントロール機能")

### インタラクションを追加する

administrationモジュールはユーザーが通常のCRUDオペレーションを実行できるようにしますが、ビューに対して独自のインタラクションを追加するもしくは可能なインタラクションを制限することもできます。たとえば、リスト14-31で示されているインタラクションを定義することで`article`モジュール上のデフォルトのすべてのCRUDアクションにアクセスできるようになります。

リスト14-31 - それぞれのビューに対してインタラクションを定義する(`backend/modules/article/config/generator.yml`)

        list:
          title:          List of Articles
          object_actions:
            _edit:         ~
            _delete:       ~
          actions:
            _create:       ~

        edit:
          title:          Body of article %%title%%
          actions:
            _list:         ~
            _save:         ~
            _save_and_add: ~
            _delete:       ~

`list`ビューにおいて、2つのアクションの設定が存在します: すべてのオブジェクトに対して利用可能なアクションのリストと全体のページに対して利用可能なアクションのリストです。リスト14-31で定義された`list`インタラクションは図14-22のようにレンダリングされます。それぞれの行はレコードを編集するボタンと削除するボタンを表示します。リストの一番下で、ボタンによって新しいレコードを作ることができます。

図14-22 - `list`ビュー内部のインタラクション

![listビュー内部のインタラクション](/images/book/F1422.png "listビュー内部のインタラクション")

`edit`ビューにおいて、一度に編集されるレコードは1つだけであり、定義するアクションのセットは1つだけです。リスト14-31で定義された`edit`インタラクションは図14-23のようにレンダリングされます。`save`アクションと`save_and_add`アクションは現在の編集をレコードに保存します。これらのアクションの違いは、`save`アクションは保存したあとで現在のレコード上に`edit`ビューを表示するのに対して、`save_adn_add`アクションは別のレコードを追加するために空の`empty`ビューを表示することです。`save_and_add`アクションは続けざまに多くのレコードを追加するときに非常に便利なショートカットです。`delete`アクションの位置に関しては、これはほかのボタンから分離されているので、ユーザーが誤ってクリックすることはありません。

アンダースコア(`_`)で始まるインタラクション名はsymfonyに対してこれらのアクションに対応するデフォルトのアイコンとアクションを使うように伝えます。administrationジェネレーターは`_edit`、`_delete`、`_list`、`_save`、`_save_and_add`、と`_create`を理解します。

図14-23 - `edit`ビュー内部のインタラクション

![editビュー内部のインタラクション](/images/book/F1423.png "editビュー内部のインタラクション")

しかし、リスト14-32のように、アンダースコアでは始まらない名前を指定しなければならない場合において、カスタムインタラクションも追加できます。

リスト14-32 - カスタムインタラクションを定義する

        list:
          title:          List of Articles
          object_actions:
            _edit:        -
            _delete:      -
            addcomment:   { name: Add a comment, action: addComment, icon: backend/addcomment.png }

図14-24で示されるように、リストにおけるそれぞれのアクションは`web/images/addcomment.png`を表示します。これをクリックすることで現在のモジュール内で`addComment`アクションを呼び出すことが行われます。現在のオブジェクトの主キーはリクエストパラメーターに自動的に追加されます。

図14-24 - `list`ビュー内部のカスタムインタラクション

![listビュー内部ののカスタムインタラクション](/images/book/F1424.png "listビュー内部のカスタムインタラクション")

`addComment`アクションはリスト14-33のように実装できます。

リスト14-33 - カスタムインタラクションアクション(`actions/actions.class.php`)

    [php]
    public function executeAddComment()
    {
      $comment = new Comment();
      $comment->setArticleId($this->getRequestParameter('id'));
      $comment->save();

      $this->redirect('comment/edit?id='.$comment->getId());
    }

アクションについて最後の一言です: あるカテゴリのためにアクションを完全に抑制したい場合、リスト14-34で示されるように、空のリストを使います。

リスト14-34 - `list`ビューのすべてのアクションを除去する

        list:
          title:          List of Articles
          actions:        {}

### フォームのバリデーション

あなたのプロジェクトの`cache/`ディレクトリのなかで生成された`_edit_form.php`テンプレートを見る場合、`form`フィールドが特別な命名規約を使っていることがわかります。生成された`edit`ビューにおいて、入力名はアンダースコアの構文のモデルクラスの名前と角かっこで囲まれたフィールド名を連結することで作られます。

たとえば、`Article`クラスのための`edit`ビューが`title`フィールドを持つ場合、テンプレートはリスト14-35のようになりフィールドは`article[title]`として識別されます。

リスト14-35 - 生成された入力名の構文

    // generator.yml
    generator:
      class:              sfPropelAdminGenerator
      param:
        model_class:      Article
        theme:            default
        edit:
          display: [title]

    // _edit_form.php テンプレートの結果
    <?php echo object_input_tag($article, 'getTitle', array('control_name' => 'article[title]')) ?>

    // 結果のHTML
    <input type="text" name="article[title]" id="article[title]" value="My Title" />

これは内部のフォームの扱い処理の間において多くの利点があります。しかしながら、10章で説明されたように、バリデーション設定が少し扱いにくくなるので、`fields`定義において、角かっこの`[]`を波かっこの`{}`に変更しなければなりません。しかし、バリデーターに対してフィールド名をパラメーターとして使うとき、生成されたHTMLのコードで現れるものとしてその名前を使うべきです(すなわち波かっこであるが、引用符のあいだで)。生成されたフォームのための特別なバリデーターの構文の詳細についてはリスト14-36を参照してください。

リスト14-36 - administrationで生成されたフォームのためのバリデーターファイルの構文

    ## フィールドリストのなかの角かっこを波かっこで置き換える
    fields:
      article{title}:
        required:
          msg: You must provide a title
        ## バリデーターパラメーターに対して、引用符で囲んだオリジナルのフィールド名を使う
        sfCompareValidator:
          check:        "user[newpassword]"
          compare_error: The password confirmation does not match the password.

### クレデンシャルを利用してユーザーのアクションを制限する

特定のadministrationモジュールに対して、利用可能なフィールドとインタラクションはログインしたユーザーのクレデンシャルによって変化します(symfonyのセキュリティ機能の説明に関しては6章を参照)。

適切なクレデンシャルを持つユーザーのみに対して表示されるようにするためにジェネレーター内部のフィールドは`credentials`パラメーターを考慮に入れます。これは`list`ビューと`edit`ビューに対して機能します。加えて、ジェネレーターはクレデンシャルにしたがってインタラクションも隠すことができます。リスト14-37はこれらの機能のお手本を示しています。

リスト14-37 - クレデンシャルを使う(`generator.yml`)

    ## idカラムはadminクレデンシャルを持つユーザーに対してのみ表示される
        list:
          title:          List of Articles
          layout:         tabular
          display:        [id, =title, content, nb_comments]
          fields:
            id:           { credentials: [admin] }

    ## addcommentインタラクションはadminクレデンシャルを持ったユーザーに制限される
        list:
          title:          List of Articles
          object_actions:
            _edit:        -
            _delete:      -
            addcomment:   { credentials: [admin], name: Add a comment, action: addComment, icon: backend/addcomment.png }

生成されたモジュールのプレゼンテーションを修正する
--------------------------------------------------

独自のスタイルシートを適用するだけでなく、デフォルトのテンプレートでオーバーライドすることで、生成されたモジュールのプレゼンテーションが既存のグラフィカルな表にマッチするように、そのモジュールを修正できます。

### カスタムスタイルシートを使う

生成されたHTMLコードは構造化された内容を持つので、プレゼンテーションであなたが望むことを大いに行うことができます。

リスト14-38で示されるように、`css`パラメーターを生成されたジェネレーターの設定に追加することで、administrationモジュールに対して、デフォルトの代わりにカスタムCSSを定義できます。

リスト14-38 - デフォルトの代わりにカスタムスタイルシートを使う

    generator:
      class:              sfPropelAdminGenerator
      param:
        model_class:      Comment
        theme:            default
        css:              mystylesheet

もう一つの方法として、ビュー単位でスタイルをオーバーライドするためにモジュールの`view.yml`によって提供されるメカニズムも利用できます。

### カスタムヘッダーとフッターを生成する

`list`ビューと`edit`ビューはヘッダーとフッター部分テンプレートを系統的に含むことができます。administrationモジュールの`templates/`ディレクトリのなかにはそのような部分テンプレートは存在しませんが、部分テンプレートを自動的にインクルードするには以下の名前の1つを追加する必要があります:

    _list_header.php
    _list_footer.php
    _edit_header.php
    _edit_footer.php

たとえば、カスタムヘッダーを`article/edit`ビューに追加したい場合、リスト14-39のように`_edit_header.php`という名前のファイルを作ります。これは追加の設定なしで動作します。

リスト14-39 - `edit`ヘッダー部分テンプレートの例(`modules/articles/template/_edit_header.php`)

    [php]
    <?php if ($article->getNbComments() > 0): ?>
      <h2>この記事には<?php echo $article->getNbComments() ?>のコメントがあります</h2>
    <?php endif; ?>

`edit`部分テンプレートはモジュールとして同じ名前を持つ変数を通してつねに現在のオブジェクトにアクセスできるので、`$pager`変数を通して現在のページャーにアクセスできることに注意してください。

>**SIDEBAR**
>administrationアクションをカスタムパラメーターで呼び出す
>
>administrationモジュールアクションは`link_to()`ヘルパーの`query_string`引数を使用してカスタムパラメーターを受けとることができます。たとえば、記事に対するコメントへのリンクを持つ`previous_edit_header`部分テンプレートを拡張するには、つぎのように書きます:
>
>     [php]
>     This article has <?php echo link_to($article->getNbComments().' comments', 'comment/list', array('query_string' => 'filter=filter&filters%5Barticle_id%5D='.$article->getId())) ?> comments.
>
>このクエリの文字列パラメーターはより読みやすいエンコードされたバージョンです。
>
>     [php]
>     'filter=filter&filters[article_id]='.$article->getId()
>
>これは`$article`に関連するコメントだけを表示するためにコメントをフィルタリングします。`query_string`引数を使うことで、ソートの順番かつ/またはカスタムlistビューを表示するフィルターを指定することもできます。これはカスタムインタラクションに対しても便利です。

### テーマをカスタマイズする

カスタム要件を満たすために、モジュールの`templates/`フォルダーのなかでオーバーライドできるフレームワークから継承された別の部分テンプレートが存在します。

ジェネレーターのテンプレートは個別にオーバーライドできる小さな部分に分割可能で、アクションは1つずつ変更できます。

しかしながら、同じ方法でいくつかのモジュールに対してこれらをオーバーライドしたいのであれば、おそらくは再利用可能なテーマを作るべきです。テーマ(theme)はテンプレートとアクションの完全なセットで、`generator.yml`の始めでテーマの値に指定された場合、administrationモジュールのなかで使用できます。デフォルトのテーマと一緒に、symfonyは`$sf_symfony_data_dir/generator/sfPropelAdmin/default/`で定義されたファイルを使います。

テーマのファイルは`data/generator/sfPropelAdmin/[theme_name]/template/`ディレクトリ内部の、プロジェクトのツリー構造に設置しなければならず、デフォルトのテーマからファイルをコピーすることで新しいテーマを使い始めることができます(`$sf_symfony_data_dir/generator/sfPropelAdmin/default/template/`ディレクトリに設置されます)。この方法では、テーマのために必要なすべてのファイルはカスタムテーマのなかに存在します:

    // [theme_name]/template/templates/内の、部分テンプレート
    _edit_actions.php
    _edit_footer.php
    _edit_form.php
    _edit_header.php
    _edit_messages.php
    _filters.php
    _list.php
    _list_actions.php
    _list_footer.php
    _list_header.php
    _list_messages.php
    _list_td_actions.php
    _list_td_stacked.php
    _list_td_tabular.php
    _list_th_stacked.php
    _list_th_tabular.php

    // [theme_name]/template/actions/actions.class.php内のアクション
    processFilters()     // リクエストフィルターを処理する
    addFiltersCriteria() // フィルターをCriteriaオブジェクトに追加する
    processSort()
    addSortCriteria()

テンプレートファイルは実際にはテンプレートのテンプレート(templates of templates)であることに注意してください。すなわち、PHPのファイルはジェネレーターの設定に基づいたテンプレートを生成する特別なユーティリティによって解析されます(これはコンパイレーションフェーズ(compilation phase)と呼ばれます)。生成されたテンプレートが実際にブラウジングしている間に実行されるPHPのコードを含まなければならないので、テンプレートのテンプレートは最初のパスの間にPHPのコードを実行しないようにするために代替構文を使います。リスト14-40はデフォルトのテンプレートのテンプレートの抜粋です。

リスト14-40 - テンプレートのテンプレートの構文

    [php]
    <?php foreach ($this->getPrimaryKey() as $pk): ?>
    [?php echo object_input_hidden_tag($<?php echo $this->getSingularName() ?>,'get<?php echo $pk->getPhpName() ?>') ?]
    <?php endforeach; ?>

このリストにおいて、(コンパイル時に)`<?`によって導入されたPHPのコードは即座に実行され、`[?`によって導入されたものは実行時のみに実行されますが、テンプレートエンジンは最後には`[?`タグを`<?`タグに変換するのでテンプレートの結果はつぎのようになります:

    [php]
    <?php echo object_input_hidden_tag($article, 'getId') ?>

テンプレートのテンプレートは扱いにくいので、独自のテーマを作りたい場合、もっともお勧めすることはデフォルトのテーマから始め、少しずつ修正し、頻繁にテストすることです。

>**TIP**
>ジェネレーターのテーマをプラグインのパッケージにすることができるので、再利用性が高くなり、複数のアプリケーションにまたがってデプロイすることが簡単になります。詳細な情報は17章を参照してください。

-

>**SIDEBAR**
>独自のジェネレーターを開発する
>
>scaffoldingとadministrationジェネレーターの両方はsymfonyの内部コンポーネントのセットを使います。内部コンポーネントはキャッシュ内部に生成されたアクションとテンプレート、テーマの使用、テンプレートのテンプレートの解析を自動化します。
>
>このことはsymfonyが既存のジェネレーターとは似ているもしくは完全に異なる独自のジェネレーターを作成するためのすべてのツールを提供することを意味します。モジュールの生成は`sfGeneratorManager`クラスの`generate()`メソッドで管理されます。たとえば、administrationを生成するために、symfonyは内部でつぎのコードを呼び出します:
>
>     [php]
>     $generator_manager = new sfGeneratorManager();
>     $data = $generator_manager->generate('sfPropelAdminGenerator', $parameters);
>
>独自のジェネレーターを開発したい場合、`sfGeneratorManeger`クラスと`sfGenerator`クラスのAPIドキュメントを見て、`sfAdminGenerator`クラスと`sfCRUDGenerator`クラスを例としてください。

まとめ
----

モジュールをブートストラップするもしくはバックエンドのアプリケーションを自動的に生成するには、基本は洗練されたスキーマとオブジェクトモデルです。scaffoldingのPHPのコードは修正可能ですが、administrationによって生成されたモジュールの多くは設定を通して修正されます。

`generator.yml`ファイルは生成されたバックエンドのプログラミングにおいて中心的な役割を果たします。このファイルによって内容、機能、`list`ビューと`edit`ビューの見た目を完全にカスタマイズできます。またPHPのコードを一行も書かずに、フィールドラベル、ツールチップ、フィルター、ソートの順番、ページのサイズ、入力タイプ、外部のリレーション、カスタムインタラクション、とクレデンシャルをYAMLで直接管理できます。

administrationジェネレーターが必要な機能をネイティブにサポートしない場合、アクションをオーバーライドする部分テンプレートフィールドと機能は完全な拡張性を提供します。それに加えて、テーマのメカニズムのおかげで、administrationジェネレーターのメカニズムへの適合方法を再利用できます。
