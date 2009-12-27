第6章 - コントローラーレイヤーの内側
================================

symfonyにおいて、コントローラーレイヤーはビジネスロジックとプレゼンテーションを結びつけるコードを含み、異なる目的で利用するためにいくつかのコンポーネントに分割されます:

  * フロントコントローラー(front controller)はアプリケーションへの唯一のエントリーポイント(entry point)です。設定をロードし、実行するアクションを決定します。
  * アクション(action)はアプリケーションのロジックを含みます。リクエストの整合性をチェックし、プレゼンテーションレイヤーが必要なデータを準備します。
  * リクエスト、レスポンス、セッションオブジェクトはリクエストパラメーター、レスポンスヘッダー、永続的なユーザーのデータにアクセスできます。これらはコントローラーレイヤー内部でよく使われます。
  * フィルター(filter)は、アクションの前あとで、すべてのリクエストに対して実行されるコードの一部です。たとえば、セキュリティとバリデーション(検証)フィルターはWebアプリケーションで共通に使われます。独自のフィルターを作成することでフレームワークを拡張できます。

この章では、これらすべてのコンポーネントを説明しますが、数の多さに怖がらないでください。基本的なページに対して必要なことはアクションクラスのなかで数行書くことだけです。ほかのコントローラーコンポーネントは特定の状況のみに使われます。

フロントコントローラー
--------------------

すべてのWebリクエストは1つのフロントコントローラー(front controller)によって処理されます。フロントコントローラーは特定の環境におけるアプリケーション全体への唯一のエントリーポイント(entry point - 入り口)です。

フロントコントローラーはリクエストを受けとるとき、ユーザーが入力した(もしくはクリックした)URLを用いてアクションとモジュールの名前をマッチさせるルーティングシステムを使います。たとえば、つぎのリクエストURLは`index.php`スクリプト(フロントコントローラー)の呼び出しを行い`mymodule`モジュールの`myAction`アクションの呼び出しとして理解されます:

    http://localhost/index.php/mymodule/myAction

symfonyの内部にご興味がなければ、フロントコントローラーについて知る必要のあることはこれだけです。これはsymfonyのMVCアーキテクチャの不可欠なコンポーネントですが、変更が必要になることはほとんどありません。フロントコントローラーの内部構造を本当に理解したいと思わなければ、つぎのセクションに飛ぶことができます。

### フロントコントローラーの仕事の詳細

フロントコントローラーはリクエストのディスパッチ(dispatch - 発送)を行いますが、このことは単に実行するアクションを決定することよりも少し多くのことが行われていることを意味します。実際、つぎのような、すべてのアクションに共通なコードを実行します:

  1. コア定数を定義する
  2. symfonyのライブラリを見つける
  3. symfonyのコアクラスをロードして初期化する
  4. 設定をロードする
  5. 実行するアクションとリクエストパラメーターを決定するためにリクエストURLをデコードする
  6. アクションが存在しない場合、404エラーのアクションにリダイレクトする
  7. フィルターを有効にする(たとえば、リクエストが認証を必要とする場合)
  8. フィルターを実行する(ファーストパス部分)*
  9. アクションを実行しビューをレンダリングする
  10. フィルターを実行する(セカンドパス部分)*
  11. レスポンスを出力する

(訳注)* は、図6-3 および リスト6-30 参照

### デフォルトのフロントコントローラー

デフォルトのフロントコントローラーは、`index.php`という名前でプロジェクトの`web/`ディレクトリに設置されています。これは、リスト6-1で示されるような、シンプルなPHPファイルです。

リスト6-1 - 運用環境用のデフォルトのフロントコントローラー

    [php]
    <?php

    define('SF_ROOT_DIR',    realpath(dirname(__FILE__).'/..'));
    define('SF_APP',         'myapp');
    define('SF_ENVIRONMENT', 'prod');
    define('SF_DEBUG',       false);

    require_once(SF_ROOT_DIR.DIRECTORY_SEPARATOR.'apps'.DIRECTORY_SEPARATOR.SF_APP.DIRECTORY_SEPARATOR.'config'.DIRECTORY_SEPARATOR.'config.php');

    sfContext::getInstance()->getController()->dispatch();

定数の定義は前のセクションで説明した最初のステップに対応します。それからフロントコントローラーはステップ2～4を処理するアプリケーションの`config.php`を含みます。`sfController`オブジェクトの`dispatch()`メソッドの呼び出し(symfonyのMVCアーキテクチャのコアコントローラーオブジェクト)はステップ5～7を処理するリクエストをディスパッチします。最後のステップはこの章の後ろの方で説明するフィルターチェーン(filter chain)によって処理されます。

### 環境を切り替えるためにほかのフロントコントローラーを呼び出す

1つの環境ごとに1つのフロントコントローラーが存在します。当然のことながら、これは環境を定義するフロントコントローラーの存在そのものです。環境は`SF_ENVIRONMENT`定数によって定義されます。

ブラウザーでアプリケーションを見ながら環境を変更するには、ほかのフロントコントローラーを選びます。`symfony init-app`タスクで新しいアプリケーション(アプリケーションの名前は`myapp`)を作成するときに利用できるデフォルトのフロントコントローラーは運用環境のための`index.php`と開発環境のための`myapp_dev.php`です。URLがフロントコントローラーのスクリプト名を含まないとき、`mod_rewrite`のデフォルト設定は`index.php`を使います。両方のURLは運用環境で同じページ(`mymodule/index`)を表示します:

    http://localhost/index.php/mymodule/index
    http://localhost/mymodule/index

そしてこのURLは開発環境で同じページを表示します:

    http://localhost/myapp_dev.php/mymodule/index

新しい環境を作成することは新しいフロントコントローラーを作成することと同じぐらい簡単です。たとえば、運用環境に移行するまえに顧客がアプリケーションをテストできるようにステージング環境(staging environment)が必要になることがあります。このステージング環境を作成するには、`web/myapp_dev.php`を`web/myapp_staging.php`にコピーして、`SF_ENVIRONMENT`定数の値を`staging`に変更するだけです。すべての設定ファイルにおいて、リスト6-2で示されるように、この環境に対して特定の値を設定するために新しい`staging:`セクションを追加できます。

リスト6-2 - ステージング環境のための特別な設定を持つ`app.yml`のサンプル

    staging:
      mail:
        webmaster:    dummy@mysite.com
        contact:      dummy@mysite.com
    all:
      mail:
        webmaster:    webmaster@mysite.com
        contact:      contact@mysite.com

この新しい環境でアプリケーションがどのように反応するのかを見たいのであれば、関連するフロントコントローラーを呼び出します:

    http://localhost/myapp_staging.php/mymodule/index

### バッチファイル

symfonyのすべてのクラスと機能にアクセスするスクリプトをコマンドラインで(もしくはcronテーブルを通して)実行したい場合を考えます。たとえば、Eメールジョブのバッチを起動させるため、もしくは、プロセスを集約する計算を通して定期的にモデルを更新するためなどです。そのようなスクリプトに対しては、最初のフロントコントローラーのように、フロントコントローラーの始めの部分などに同じ行を含めることが必要です。リスト6-3はバッチスクリプトの始めの部分を示しています。

リスト6-3 - バッチスクリプトのサンプル

    [php]
    <?php

    define('SF_ROOT_DIR',    realpath(dirname(__FILE__).'/..'));
    define('SF_APP',         'myapp');
    define('SF_ENVIRONMENT', 'prod');
    define('SF_DEBUG',       false);

    require_once(SF_ROOT_DIR.DIRECTORY_SEPARATOR.'apps'.DIRECTORY_SEPARATOR.SF_APP.DIRECTORY_SEPARATOR.'config'.DIRECTORY_SEPARATOR.'config.php');

    // ここにコードを追加する

唯一見つからない行は、`sfController`オブジェクトの`dispatch()`メソッドの呼び出しであることがわかります。この呼び出しは、バッチ処理ではなく、Webサーバーのみが使用できます。アプリケーションと環境を定義することで特定の設定にアクセスできます。アプリケーションの`config.php`をインクルードすることでコンテキストとオートロードが初期化されます。

>**TIP**
>symfonyのCLIは`init-batch`タスクを提供します。このタスクはリスト6-3の`batch/`ディレクトリのものと似たようなスケルトンを自動的に作ります。アプリケーション名、環境名、バッチ名を引数として渡すだけです。

アクション
----------

アクションはすべてのアプリケーションのロジックを格納するので、アプリケーションのなか心的な役割を果たします。これらはモデルを使用し、ビューのための変数を定義します。symfonyのアプリケーションでWebリクエストを作成するとき、URLはアクションとリクエストパラメーターを定義します。

### アクションクラス

アクションは`sfActions`クラスを継承する`moduleNameActions`クラスの`executeActionName`メソッドで、モジュールによって分類されます。モジュールのアクションクラスは、モジュールの`actions/`ディレクトリの、`actions.class.php`ファイルに保存されます。

リスト6-4は全体の`mymodule`モジュールに対して`index`アクションだけを持つ`actions.class.php`ファイルの例を示しています。

リスト6-4 - アクションクラスのサンプル(`apps/myapp/modules/mymodule/actions/actions.class.php`)

    [php]
    class mymoduleActions extends sfActions
    {
      public function executeIndex()
      {

      }
    }

>**CAUTION**
>PHPはメソッドの名前が大文字か小文字かを区別しませんがsymfonyは区別します。アクションメソッドは小文字の`execute`で始まり、つぎに最初が大文字のアクション名そのものが続くことを忘れないでください。

アクションをリクエストするには、パラメーターとしてモジュール名とアクション名を使用してフロントコントローラーのスクリプトを呼び出す必要があります。デフォルトでは、この作業は`module_name`/`action_name`の組をスクリプトに追加することで行われます。このことはリスト6-4で定義されたアクションがつぎのURLで呼び出されることを意味します:

    http://localhost/index.php/mymodule/index

リスト6-5で示されるように、より多くのアクションを追加することはより多くの`execute`メソッドを`sfActions`オブジェクトに追加することを意味します。

リスト6-5 - 2つのアクションを持つアクションクラス(`myapp/modules/mymodule/actions/actions.class.php`)

    [php]
    class mymoduleActions extends sfActions
    {
      public function executeIndex()
      {
        ...
      }

      public function executeList()
      {
        ...
      }
    }

アクションクラスのサイズが大きくなりすぎたら、幾分かリファクタリングを行いコードをモデルレイヤーに移動させることがおそらく必要です。アクションは短く保たれ(数行以内)、通常、すべてのビジネスロジックはモデル内部に存在すべきです。

それでも、1つのモジュール内部にたくさんのアクションが存在するのであれば、そのモジュールを2つに分割することが大切です。

>**SIDEBAR**
>symfonyのコーディング規約
>
>この本で示されたコードの例において、開き波かっこと閉じ波かっこ(`{`と`}`)がそれぞれ一行を占めることにおそらくお気づきでしょう。この規約によってコードはより読みやすくなります。
>
>symfonyのほかのコーディング規約では、インデントはつねに2つの空白文字で行われます: タブは使いません。これはテキストエディタによってタブは異なる空白文字の値を持つためであり、タブと空白のインデントが混在するコードを読むことは不可能だからです。
>
>symfonyコアと生成されたPHPファイルは通常の`?>`の閉じタグで終わりません。これは本当に必要がないからと、このタグの後ろに空白がある場合、出力の問題が作られる可能性があるからです。
>
>そして本当に注意を払っているのであれば、symfonyでは決して空白文字では終わりません。今回の場合、理由はつまらないことです: Fabien(筆者の一人)のテキストエディタでは空白で終わる行が不細工に見えるからです！

### アクションクラスの代替構文

アクションの代替構文は、1つのアクションごとに1つのファイルで、個別のファイルのアクションをディスパッチするために利用できます。この場合、それぞれのアクションクラスは(`sfActions`の代わりに)`sfAction`を継承し、`actionNameAction`と名づけられます。実際のアクションメソッドは単に`execute`と名づけられます。ファイル名はクラス名と同じです。このことはリスト6-5の同等の内容はリスト6-6と6-7で示される2つのファイルのように書くことができることを意味します。

リスト6-6 - 単独のアクションファイル(`myapp/modules/mymodules/action/indexAction.class.php`)

    [php]
    class indexAction extends sfAction
    {
      public function execute()
      {
        ...
      }
    }

リスト6-7 - 単独のアクションファイル(`myapp/modules/mymodules/actions/listAction.class.php`)

    [php]
    class listAction extends sfAction
    {
      public function execute()
      {
        ...
      }
    }

### アクションの情報をとり出す

アクションのクラスはコントローラー関連の情報とsymfonyのコアオブジェクトにアクセスする方法を提供します。リスト6-8はこれらの方法の実際の例を示しています。

Listing6-8 - `sfActions`の共通メソッド

    [php]
    class mymoduleActions extends sfActions
    {
      public function executeIndex()
      {
        // リクエストパラメーターをとり出す
        $password    = $this->getRequestParameter('password');

        // コントローラー情報をとり出す
        $moduleName  = $this->getModuleName();
        $actionName  = $this->getActionName();

        // フレームワークのコアオブジェクトをとり出す
        $request     = $this->getRequest();
        $userSession = $this->getUser();
        $response    = $this->getResponse();
        $controller  = $this->getController();
        $context     = $this->getContext();

        // 情報をテンプレートに渡すためにアクション変数を設定する
        $this->setVar('foo', 'bar');
        $this->foo = 'bar';            // 短いバージョン

      }
    }

>**SIDEBAR**
>Context Singleton
>
>フロントコントローラー内部で、`sfContext::getInstance()`への呼び出しをすでに見ました。アクションにおいて、`getContext()`メソッドは同じSingletonを返します。これは任意のリクエストに関連するすべてのsymfonyのコアオブジェクトへの参照を保存し、各オブジェクトに対してアクセサーを提供する非常に便利なオブジェクトです:
>
>  * `sfController`: コントローラーオブジェクト(`->getController()`)
>  * `sfRequest:` リクエストオブジェクト(`->getRequest()`)
>  * `sfResponse`: レスポンスオブジェクト(`->getResponse()`)
>  * `sfUser`: ユーザーセッションオブジェクト(`->getUser()`)
>  * `sfDatabaseConnection`: データベース接続(`->getDatabaseConnection()`)
>  * `sfLogger`: loggerオブジェクト(`->getLogger()`)
>  * `sfI18N`: 国際化オブジェクト(`->getI18N()`)
>
>`sfContext::getInstance()`: Singletonをコードの任意の部分から呼び出すことができます。

### アクションの終了方法

アクションの実行の結果においてさまざまなふるまいが可能です。アクションメソッドによって返された値はビューをレンダリングする方法を決定します。`sfView`クラスの定数はアクションの結果を表示するために使われるテンプレートを指定するために使われます。

呼び出すデフォルトのビューが存在する場合(もっとも共通の事例)、アクションはつぎのように終わります:

    [php]
    return sfView::SUCCESS;

symfonyは`actionNameSuccess.php`という名前のテンプレートを探します。これはアクションのデフォルトのふるまいとして定義されているので、アクションのメソッドにおいて`return`ステートメントを省略する場合、symfonyは`actionNameSuccess.php`テンプレートも探します。空のアクションはそのふるまいも引き起こします。成功したアクションの終了方法に関してはリスト6-9の例をご覧ください。

リスト6-9 - `indexSuccess.php`と`listSuccess.php`テンプレートを呼び出すアクション

    [php]
    public function executeIndex()
    {
      return sfView::SUCCESS;
    }

    public function executeList()
    {
    }

呼び出すエラービューが存在する場合、アクションはつぎのように終わります:

    [php]
    return sfView::ERROR;

symfonyは`actionNameError.php`という名前のテンプレートを探します。

カスタムビューを呼び出すには、つぎのように終わらせます:

    [php]
    return 'MyResult';

symfonyは`actionNameMyResult.php`によって呼び出されたテンプレートを探します。

呼び出すビューが存在しない場合、たとえばバッチプロセス内で実行されたアクションの場合、アクションはつぎのように終わります:

    [php]
    return sfView::NONE;

この場合、テンプレートは実行されません。このことはビューレイヤーを完全に回避して、アクションから直接HTMLコードを出力できることを意味します。リスト6-10で示されるように、この事例のためにsymfonyは特別な`renderText()`メソッドを提供します。これは11章で検討されるAjaxのインタラクションなどのアクションの非常に高い反応性を必要とするときに役立ちます。

リスト6-10 - レスポンスをechoして`sfView::NONE`を返すことでビューを回避する

    [php]
    public function executeIndex()
    {
      echo "<html><body>Hello, World!</body></html>";

      return sfView::NONE;
    }

    // つぎのものと同等
    public function executeIndex()
    {
      return $this->renderText("<html><body>Hello, World!</body></html>");
    }

いくつかの場合において、定義されたヘッダー以外のヘッダー(特に`X-JSON`ヘッダー)をともなう空のレスポンスを送信する必要があります。リスト6-11で示されているように、`sfResponse`オブジェクト経由でヘッダーを定義して、つぎの章で検討しますが、`sfView::HEADER_ONLY`定数を返します。

リスト6-11 - ビューのレンダリングを回避してヘッダーのみを送信する

    [php]
    public function executeRefresh()
    {
      $output = '<"title","My basic letter"],["name","Mr Brown">';
      $this->getResponse()->setHttpHeader("X-JSON", '('.$output.')');

      return sfView::HEADER_ONLY;
    }

アクションを特定のテンプレートによってレンダリングしなければならない場合、`return`ステートメントを無視して、代わりに`setTemplate()`メソッドを使います。

    [php]
    $this->setTemplate('myCustomTemplate');

### 別のアクションにスキップする

いくつかの場合において、アクションの実行が新しいアクションの実行をリクエストすることで終わります。たとえば、POSTリクエストにおいてフォーム投稿を処理するアクションは、通常の場合、データベースを更新したあとで別のアクションにリダイレクトします。ほかの例はアクションのエイリアスです: `index`アクションはしばし一覧を表示する方法であり、実際には`list`アクションへの転送です。

アクションのクラスは別のアクションを実行するために2つのメソッドを提供します:

  * アクションが別のアクションにフォワードする場合:

        [php]
        $this->forward('otherModule', 'index');

  * アクションがWebリダイレクトの結果になる場合:

        [php]
        $this->redirect('otherModule/index');
        $this->redirect('http://www.google.com/');

>**NOTE**
>アクションにおいてフォワードもしくはリダイレクトの後に設置されたコードは決して実行されません。これらの呼び出しは`return`ステートメントと同等のものとみなすことができます。アクションの実行を停止させるためにこれらは`sfStopException`を投げます; この例外はsymfonyによって後に捕捉され、単に無視されます。

時にリダイレクトもしくはフォワードを選ぶことがやりにくいことがあります。最良の解決法を選ぶには、フォワードはアプリケーションの内部に存在しユーザーには見えないことを覚えておいてください。ユーザーに関しては、表示されるURLはリクエストされたものと同じです。対照的に、リダイレクトはユーザーのブラウザーへのメッセージで、それからの新しいリクエストとURLの最後の結果の変更を含みます。

アクションが`method="post"`メソッドで投稿されたフォームから呼び出された場合、つねにリダイレクトを行うべきです。主な利点はユーザーが結果のページをリフレッシュするさいにフォームが再投稿されないことです; 加えて、フォームを表示して、ユーザーがPOSTリクエストを再投稿したいかどうかを確認するアラートを表示しないことで前のページに戻るボタンが期待どおりに動きます。

とてもよく使われる特殊なフォワードが1つあります。`forward404()`メソッドは"見つからないページ"用のアクションにフォワードします。アクションの実行に必要なパラメーターがリクエストのなかに存在しないときに、このメソッドはよく呼び出されます(誤って入力されたURLを検出します)。リスト6-12は`id`パラメーターを必要とする`show`アクションの例を示しています。

リスト6-12 - `forward404()`メソッドの使いかた

    [php]
    public function executeShow()
    {
      $article = ArticlePeer::retrieveByPK($this->getRequestParameter('id'));
      if (!$article)
      {
        $this->forward404();
      }
    }

>**TIP**
>404エラー用のアクションとテンプレートは`$sf_symfony_data_dir/modules/default/`ディレクトリで見つかります。このページをカスタマイズする方法は新しい`default`モジュールをアプリケーションに追加し、symfony内部の既存のものをオーバーライドし、`error404`アクションと`error404Success`テンプレート内部で定義することです。代わりの方法として、既存のアクションを使うために`settings.yml`ファイルでerror_404_moduleとerror_404_actionの定数を設定することです。

経験則によれば、大体の場合、リスト6-12のように、アクションは何かをテストした後にリダイレクトもしくはフォワードを行います。`sfActions`クラスにはいくつかのメソッド、`forwardIf()`、`forwardUndless()`、`forward404If()`、`forward404Unless()`、`redirectIf()`と`redirectUnless()`メソッドが存在するのはそういうわけです。リスト6-13で示されるように、これらのメソッドは、テストして(`xxxIf()`メソッドに対して)trueもしくは(`xxxUnless()メソッド`に対して)falseを返す場合、実行条件を表す1つの追加パラメーターをとります。

リスト6-13 - `forward404If()`メソッドの使いかた

    [php]
    // このアクションはリスト6-12で示されているものと同等である
    public function executeShow()
    {
      $article = ArticlePeer::retrieveByPK($this->getRequestParameter('id'));
      $this->forward404If(!$article);
    }

    // これも同じ
    public function executeShow()
    {
      $article = ArticlePeer::retrieveByPK($this->getRequestParameter('id'));
      $this->forward404Unless($article);
    }

これらのメソッドを使うことでコードを短く保つことができるだけでなく、読みやすくなります。

>**TIP**
>アクションが`forward404()`もしくはその仲間のメソッドを呼び出すとき、symfonyは404エラーのレスポンスを管理する`sfError404Exception`を投げます。コントローラーにアクセスしたくないどこかの場所から404エラーのメッセージを表示することが必要な場合、同じような例外が投げられることを意味します。

### モジュールのいくつかのアクションに対してコードを繰り返す

アクションの名前を`executeActionName()`(`sfActions`クラスの場合)もしくは`execute()`メソッド(`sfAction`クラスの場合)と命名する規約によってsymfonyがアクションのメソッドを探すことが保証されます。この規約によってメソッド名が`execute`で始まらないかぎり、アクションと見なされたくない独自のほかのメソッドを追加できるようになります。

実際にアクションを実行するまえにそれぞれのアクション内でいくつかのステートメントを繰り返す必要があるときのために別の便利な規約があります。これらのステートメントをアクションのクラスの `preExecute()`メソッドに展開できます。アクションが実行されるたびにそのあとでステートメントを繰り返す方法を推測できるでしょう: これらを`postExecute()`メソッドにラップすることです。これらのメソッドの構文はリスト6-14で示されています。

リスト6-14 - アクションのクラスのなかで`preExecute`、`postExecute`とカスタムメソッドを使う

    [php]
    class mymoduleActions extends sfActions
    {
      public function preExecute()
      {
        // ここに挿入されたコードはそれぞれのアクション呼び出しの始めに実行される
        ...
      }

      public function executeIndex()
      {
        ...
      }

      public function executeList()
      {
        ...
        $this->myCustomMethod();  // アクションのクラスのメソッドがアクセス可能
      }

      public function postExecute()
      {
        // ここに挿入されたコードはそれぞれのアクションの呼び出しの終わりに実行される
        ...
      }

      protected function myCustomMethod()
      {
        // "execute"で始まらないかぎり、独自のメソッドも追加できる
        // この場合、protectedもしくはprivateとしてこれらを宣言するほうがベターである
        ...
      }
    }

リクエストにアクセスする
------------------------

名前を利用してリクエストパラメーターの値をとり出すために使われる、`getRequestParameter('myparam')`メソッドに慣れました。当然のことながら、このメソッドはリクエストのパラメーターホルダーである`getRequest()->getParameter('myparam')`に対するプロキシです。アクションクラスはsymfonyの`sfWebRequest`という名前のリクエストオブジェクトと、`getRequest()`メソッド経由でそのオブジェクトのすべてのメソッドにアクセスできます。テーブル6-1はもっとも便利な`sfWebRequest`メソッドの一覧です。

テーブル6-1 - `sfWebRequest`オブジェクトのメソッド

名前                             | 機能                                |サンプルの出力
-------------------------------- | ------------------------------------|------------------
**リクエスト情報**               |                                    |
`getMethod()`                    | リクエストメソッド                  |`sfRequest::GET`もしくは`sfRequest::POST`定数を返す
`getMethodName()`                | リクエストメソッド名                |`'POST'` 
`getHttpHeader('Server')`        | 任意のHTTPヘッダーの値            |`'Apache/2.0.59 (Unix) DAV/2 PHP/5.1.6'` 
`getCookie('foo')`               | 名前つきCookieの値                |`'bar'` 
`isXmlHttpRequest()*`            | Ajaxリクエストであるか？            |`true` 
`isSecure()`                     | SSLリクエストであるか？             |`true` 
**リクエストパラメーター**         |                                    |
`hasParameter('foo')`            | リクエスト内にパラメーターが存在するか？|`true` 
`getParameter('foo')`            | 名前つきパラメーターの値            |`'bar'` 
`getParameterHolder()->getAll()` | すべてのリクエストパラメーターの配列  |
URI関連の情報                    |                                    |
`getUri()`                       | フルURI                             |`'http://localhost/myapp_dev.php/mymodule/myaction'`
`getPathInfo()`                  | パス情報                      |`'/mymodule/myaction'` 
`getReferer()**`                 | リファラ                            |`'http://localhost/myapp_dev.php/'` 
`getHost()`                      | ホスト名                     |`'localhost'` 
`getScriptName()`                | フロント・コントローラーのパス名    |`'myapp_dev.php'` 
**クライアントのブラウザー情報**   |                                    |
`getLanguages()`                 | 受信した言語の配列                  |`Array( [0] => fr [1] => fr_FR [2] => en_US [3] => en )` 
`getCharsets()`                  | 受信した文字集合の配列            |`Array( [0] => ISO-8859-1 [1] => UTF-8 [2] => * )` 
`getAcceptableContentTypes()`    | 受信したContent-Typeの配列      | `Array( [0] => text/xml [1] => text/html`

 * *Prototype、Mootools、とjQueryと連携する*

 ** *たまにプロキシによってブロックされます*

リスト6-15で示されるように、`sfActions`クラスはリクエストメソッドにより速くアクセスするプロキシメソッドをいくつか提供します。

リスト6-15 - アクションから`sfRequest`オブジェクトメソッドにアクセスする

    [php]
    class mymoduleActions extends sfActions
    {
      public function executeIndex()
      {
        $hasFoo = $this->getRequest()->hasParameter('foo');
        $hasFoo = $this->hasRequestParameter('foo');  // 短いバージョン
        $foo    = $this->getRequest()->getParameter('foo');
        $foo    = $this->getRequestParameter('foo');  // 短いバージョン
      }
    }

ユーザーがファイルを添付するmultipartのリクエストのために、リスト6-16で示されるように、`sfWebRequest`オブジェクトはこれらのファイルにアクセスして移動させる方法を提供します。

リスト6-16 - `sfWebRequest`オブジェクトは添付ファイルの処理方法を理解している

    [php]
    class mymoduleActions extends sfActions
    {
      public function executeUpload()
      {
        if ($this->getRequest()->hasFiles())
        {
          foreach ($this->getRequest()->getFileNames() as $uploadedFile)
          {
            $fileName  = $this->getRequest()->getFileName($uploadedFile);
            $fileSize  = $this->getRequest()->getFileSize($uploadedFile);
            $fileType  = $this->getRequest()->getFileType($uploadedFile);
            $fileError = $this->getRequest()->hasFileError($uploadedFile);
            $uploadDir = sfConfig::get('sf_upload_dir');
            $this->getRequest()->moveFile($uploadedFile, $uploadDir.'/'.$fileName);
          }
        }
      }
    }

サーバーが`$_SERVER`もしくは`$_ENV`変数をサポートするかどうかもしくはデフォルト値もしくはサーバーの互換問題に悩む必要はありません。`sfWebRequest`メソッドはすべてを代行してくれます。その上、それらの名前は明確なので、リクエストから情報を入手する方法を調べるためにPHPのドキュメントを調べる必要はもはやありません。

>**NOTE**
>上記のコードはあたかもファイルがアップロードされたように`$fileName`を使います。これは悪意のあるファイル名を持ったファイルを送信することで悪用される小さな機会が存在するので、つねにターゲットファイル名を標準化するか生成すべきです。


ユーザーセッション
------------------

symfonyはユーザーセッションの管理を自動化しユーザーのためのリクエスト間のデータを一貫したものに保ちます。symfonyはPHPの組み込みセッションのハンドリングメカニズムを使用してそれらをより柔軟に設定可能で使いやすいものにするために強化します。

### ユーザーセッションにアクセスする

現在のユーザーのためのセッションオブジェクトは`sfUser`クラスのインスタンスである`getUser()`メソッドを持つアクション内でアクセスできます。このクラスはユーザー属性の保存を可能にするパラメーターホルダーを含みます。リスト6-17で示されるように、このデータはユーザーセッションの終了までほかのリクエストで利用可能です。ユーザー属性はさまざまなタイプのデータ(文字列、配列と連想配列)を保存できます。これらはユーザーが認証されていないとしても個別のユーザーごとに設定できます。

リスト6-17 - `sfUser`オブジェクトは複数のリクエストにまたがって存在するカスタムユーザー属性を持つことができる

    [php]
    class mymoduleActions extends sfActions
    {
      public function executeFirstPage()
      {
        $nickname = $this->getRequestParameter('nickname');

        // データをユーザーセッションに保存する
        $this->getUser()->setAttribute('nickname', $nickname);
      }

      public function executeSecondPage()
      {
        // デフォルト値を指定してユーザーセッションからデータをとり出す
        $nickname = $this->getUser()->getAttribute('nickname', 'Anonymous Coward');
      }
    }

>**CAUTION**
>推奨されませんが、オブジェクトをユーザーのセッションに保存できます。これが可能な理由は、セッションオブジェクトがリクエストの間にシリアライズされて、ファイルに保存されるからです。セッションがデシリアライズされたとき、保存されたオブジェクトのクラスはすでにロードされていなければなりませんが、つねにあてはまりません。加えて、Propelオブジェクトを保存する場合、"膠着状態(stalled)"のオブジェクトが存在する可能性があります。

symfonyの多くのゲッターのように、`getAttribute()`メソッドは、属性が定義されていないときに使われるデフォルト値を指定する、2番目の引数を受けとります。ユーザーに対して属性が定義されているかどうかを確認するには、`hasAttribute()`メソッドを使います。属性は`getAttributeHolder()`メソッドによってアクセス可能なパラメーターホルダーに保存されます。リスト6-18で示すように、これによって通常のパラメーターホルダーのメソッドでユーザー属性を簡単に一掃できるようになります。

リスト6-18 - ユーザーのセッションからデータを削除する

    [php]
    class mymoduleActions extends sfActions
    {
      public function executeRemoveNickname()
      {
        $this->getUser()->getAttributeHolder()->remove('nickname');
      }

      public function executeCleanup()
      {
        $this->getUser()->getAttributeHolder()->clear();
      }
    }

リスト6-19で示されるように、ユーザーのセッションの属性は、現在の`sfUser`オブジェクトを保存する`$sf_user`変数を通して、テンプレートのなかでもデフォルトで利用できます。

リスト6-19 - テンプレートはユーザーのセッションの属性にもアクセスできる

    [php]
    <p>
      Hello, <?php echo $sf_user->getAttribute('nickname') ?>
    </p>

>**NOTE**
>現在のリクエストの持続時間に関する情報を保存する必要がある場合、たとえば、アクションの呼び出しのチェーンを通して情報を渡すため、`getAttribute()`メソッドと`setAttriubte()`メソッドを持つ`sfRequest`クラスを使うことが望ましいかもしれません。`sfUser`オブジェクトの属性だけがリクエスト間において一貫しています。

### flash属性

ユーザーの属性に関して繰り返し起きる問題は属性が不要になったときにユーザーセッションを消去することです。たとえば、フォームを通してデータを更新したあとに確認メッセージを表示したい場合を考えます。フォームを扱うアクションがリダイレクトを行う場合、このアクションからの情報をリダイレクトするアクションに渡す唯一の方法はユーザーのセッションに情報を保存することです。しかしいったん確認メッセージが表示されるら、属性をクリアする必要があります; そうでなければ、期限が切れるまでセッションが保持されることになります。

flash属性は開発者が定義して忘れることができる短命の属性です。これはすぐつぎのリクエストのあとで消えるので将来のユーザーセッションはクリーンな状態に保たれます。アクションのなかでは、つぎのようにflash属性を定義します:

    [php]
    $this->setFlash('attrib', $value);

テンプレートはレンダリングされ、別のアクションにリクエストを行うユーザーに配信されます。この2番目のアクションにおいて、つぎのようなflash属性の値を取得します

    [php]
    $value = $this->getFlash('attrib');

それから忘れてください。この2番目のページを配信したあとで、flash属性の`attrib`は消去されます。この2番目のアクションの期間にこの属性を求めない場合でも、どのみちflash属性はセッションから消えます。

テンプレートからflash属性にアクセスする必要がある場合、`$sf_flash`オブジェクトを使います:

    [php]
    <?php if ($sf_flash->has('attrib')): ?>
      <?php echo $sf_flash->get('attrib') ?>
    <?php endif; ?>

もしくはつぎのように書くこともできます:

    [php]
    <?php echo $sf_flash->get('attrib') ?>

flash属性はすぐつぎのリクエストに情報を渡すための正当な手段です。

### セッションの管理

symfonyのセッションのハンドリング機能によって完全にクライアントとサーバーのセッションIDの保存は開発者に対して覆い隠されます。しかしながら、セッション管理のメカニズムのデフォルトのふるまいを修正することも可能です。このセクションはおもに上級ユーザー向けです。

クライアントサイドにおいて、セッションはCookieによって処理されます。symfonyセッションのCookieの名前は`symfony`ですが、リスト6-20で示されるように、`factories.yml`設定ファイルを編集することでこの名前を変更できます。

リスト6-20 - セッションのCookie名を変更する(`apps/myapp/config/factories.yml`)

    all:
      storage:
        class: sfSessionStorage
        param:
          session_name: my_cookie_name

>**TIP**
>`auto_start`パラメーターが`true`として`factories.yml`で設定されている場合のみ(PHPの`session_start()`関数で)セッションは始まります(デフォルトの場合)。ユーザーセッションを手動で始めたい場合、このストレージファクトリの設定を無効にします。

symfonyのセッションハンドリングはPHPセッションに基づいています。Cookieの代わりにURLパラメーターによって扱われるセッションのクライアントサイドの管理が必要な場合、`php.ini`の`use_trans_sid`ディレクティブを変更する必要があります。しかしながら、これは推奨されていないことをご了承ください。

    session.use_trans_sid = 1

デフォルトでは、サーバーサイド上において、symfonyはユーザーのセッションをファイルに保存します。リスト6-21で示されるように、`factories.yml`の`class`パラメーターの値を変更することでこれらのセッションをデータベースに保存できます

リスト6-21 - サーバーのセッションの保存設定(`apps/myapp/config/factories.yml`)

    all:
      storage:
        class: sfMySQLSessionStorage
        param:
          db_table: SESSION_TABLE_NAME      # セッションを保存するテーブルの名前
          database: DATABASE_CONNECTION     # 使うデータベース接続の名前

利用可能なセッションのストレージクラスは`sfMySQLSessionStorage`、`sfPostgreSQLStorage`と`sfPDOSessionStorage`です; 後者のほうが望ましいです。オプションの`database`設定は使うデータベースの接続を定義します; symfonyはこの接続のための接続設定(ホスト名、データベース名、ユーザー名、パスワード)を決定するために`databases.yml`(8章を参照)を使います。

セッションの期限切れは`sf_timeout`秒後に自動的に起こります。この定数の値はデフォルトで30秒で、リスト6-22で示されるように、それぞれの環境の`settings.yml`設定ファイルで修正できます。

リスト6-22 - セッションの期限を変更する(`apps/myapp/config/settings.yml`)

    default:
      .settings:
        timeout:     1800           # 秒単位のセッションの期限

アクションのセキュリティ
-------------------------

アクションを実行する機能は特定の権限を持つユーザーに制限されます。この目的のためにsymfonyが提供するツールによってセキュアなアプリケーションを作ることができます。このアプリケーションでは、ユーザーはいくつかの機能もしくはアプリケーションの一部にアクセスするまえに認証される必要があります。アプリケーションをセキュアにするには2つのステップが必要です: それぞれのアクションに対してセキュリティ要件を宣言し、これらのセキュアなアクションにアクセスできるようにユーザーがログインして権限を持つことです。

### アクセスの制限

実行されるまえに、すべてのアクションは特別なフィルターに通されます。このフィルターは現在のユーザーがリクエストしたアクションにアクセスする権限を持つかどうかをチェックします。symfonyにおいて、権限は2つの部分で構成されます:

  * セキュアなアクションはユーザーを認証することを必要とします。
  * クレデンシャル(credential)はセキュリティをグループ単位で編成できるようにする名前つきのセキュリティ権限です。

アクションへのアクセスを制限する方法はモジュールの`config/`ディレクトリのなかにYAML形式の`security.yml`設定ファイルを作り編集することで簡単に行われます。このファイルにおいて、ユーザーがそれぞれのアクションもしくは`all`アクションに対して満たさなければならないセキュリティ要件を指定できます。リスト6-23はサンプルの`security.yml`を示しています。

リスト6-23 - アクセス制限の設定(`apps/myapp/modules/mymodule/config/security.yml`)

    read:
      is_secure:   off       # すべてのユーザーはreadアクションをリクエストできる

    update:
      is_secure:   on        # update アクションは認証されたユーザーに対してのみ

    delete:
      is_secure:   on        # adminクレデンシャルを持つ
      credentials: admin     # 認証されたユーザーのみ

    all:
      is_secure:  off        # ともかくoffはデフォルト値

デフォルトではアクションはセキュアではありませんので、`security.yml`もしくはアクションに関する記述が存在しない場合、アクションは誰でもアクセスできます。`security.yml`が存在する場合、symfonyはリクエストされたアクションの名前を探し、存在する場合、セキュリティの要件を満たしているかチェックします。ユーザーが制限されたアクションにアクセスしようとしたときに起きることはユーザーのクレデンシャル次第です:

  * ユーザーが認証され、適切なクレデンシャルを持つ場合、アクションは実行されます。
  * ユーザーが識別されなかった場合、ユーザーはデフォルトのログインアクションにリダイレクトされます。
  * ユーザーが識別されているが、適切なクレデンシャルを持たない場合、図6-1で示されるように、ユーザーはデフォルトのセキュアなアクションにリダイレクトされます。

デフォルトのログインとセキュアなページはとてもシンプルなので、おそらくカスタマイズしたくなるでしょう。リスト6-24で示されるように、アプリケーションの`settings.yml`でプロパティの値を変更することで、不十分な権限の場合に呼び出されるアクションを設定できます。

図6-1 - デフォルトのセキュアなアクションページ

![デフォルトのセキュアなアクションページ](http://github.com/masakielastic/symfonybook-ja/raw/master/images/F0601.jpg "デフォルトのセキュアなアクションページ")

リスト6-24 - デフォルトのセキュリティアクションを定義する(`apps/myapp/config/settings.yml`)

    all:
      .actions:
        login_module:           default
        login_action:           login

        secure_module:          default
        secure_action:          secure

### アクセス権を付与する

制限されたアクションにアクセスするには、ユーザーは認証されるかつ/もしくは特定のクレデンシャルを持つことが必要です。`sfUser`オブジェクトのメソッドを呼び出すことでユーザーの権限を拡張できます。ユーザーの認証ステータスは`setAuthenticated()`メソッドによって設定され`isAuthenticated()`によってチェックされます。リスト6-25はユーザー認証のシンプルな例を示します。

リスト6-25 - ユーザーの認証ステータスを設定する

    [php]
    class myAccountActions extends sfActions
    {
      public function executeLogin()
      {
        if ($this->getRequestParameter('login') == 'foobar')
        {
          $this->getUser()->setAuthenticated(true);
        }
      }

      public function executeLogout()
      {
        if ($this->getUser()->isAuthenticated())
        {
          $this->getUser()->setAuthenticated(false);
        }
      }
    }

クレデンシャルをチェック、追加、削除、クリアできるので、クレデンシャルの扱いは少し複雑です。リスト6-26は`sfUser`クラスのクレデンシャルメソッドを説明しています。

リスト6-26 - アクションのなかでユーザーのクレデンシャルを処理する

    [php]
    class myAccountActions extends sfActions
    {
      public function executeDoThingsWithCredentials()
      {
        $user = $this->getUser();

        // 1つもしくは複数のクレデンシャルを追加する
        $user->addCredential('foo');
        $user->addCredentials('foo', 'bar');

        // ユーザーがクレデンシャルを持つかどうかを確認する
        echo $user->hasCredential('foo');                      =>   true

        // ユーザーが1つのクレデンシャルを持つのか確認する
        echo $user->hasCredential(array('foo', 'bar'));        =>   true

        // ユーザーが両方のクレデンシャルを持つのか確認する
        echo $user->hasCredential(array('foo', 'bar'), false); =>   true

        // 1つのクレデンシャルを削除する
        $user->removeCredential('foo');
        echo $user->hasCredential('foo');                      =>   false

        // すべてのクレデンシャルをクリアする(ログアウト処理で便利)
        $user->clearCredentials();
        echo $user->hasCredential('bar');                      =>   false
      }
    }

ユーザーが`'foo'`クレデンシャルを持つ場合、そのクレデンシャルを必要とする`security.yml`に対してそのユーザーはアクセスできるようになります。リスト6-27で示されるように、クレデンシャルはテンプレート内の認証された内容を表示するためだけにも使用できます。

リスト6-27 - テンプレートのなかでユーザーのクレデンシャルを処理する

    [php]
    <ul>
      <li><?php echo link_to('section1', 'content/section1') ?></li>
      <li><?php echo link_to('section2', 'content/section2') ?></li>
      <?php if ($sf_user->hasCredential('section3')): ?>
      <li><?php echo link_to('section3', 'content/section3') ?></li>
      <?php endif; ?>
    </ul>

認証ステータスに関しては、ログイン処理の間に、クレデンシャルはしばしばユーザーに付与されます。中央管理方式でユーザーのセキュリティステータスを設定するために、しばし`sfUser`オブジェクトにログインとログアウトのメソッドが追加されて拡張される理由はそういうことです。

>**TIP**
>symfonyのプラグインのなかで、`sfGuardPlugin`[(http://www.symfony-project.org/plugins/sfGuardPlugin)](http://www.symfony-project.org/plugins/sfGuardPlugin)はログインとログアウトを簡単にするセッションクラスを拡張します。詳細な情報は17章を参照してください。

### 複雑なクレデンシャル

security.ymlファイルのなかで使われるYAMLファイルは、AND型とOR型の関係を利用することで、クレデンシャルの組み合わせを持つユーザーへのアクセスを制限できます。このような組み合わせによって、複雑なワークフローとユーザーの権限管理システム、たとえばCMS(Content Management System)を開発できます。CMSにおいて`admin`クレデンシャルを持つユーザーのみがバックオフィスにアクセス可能で、記事の作成は`editor`クレデンシャルを持つユーザーだけが、記事の公開は`publisher`クレデンシャルを持つユーザーだけが可能です。リスト6-28はこの例を示しています。

リスト6-28 - クレデンシャルの組み合わせ構文

    editArticle:
      credentials: [ admin, editor ]              # admin AND editor

    publishArticle:
      credentials: [ admin, publisher ]           # admin AND publisher

    userManagement:
      credentials: [[ admin, superuser ]]         # admin OR superuser

新しいレベルの角かっこを追加するたびにロジックのANDとORをお互いに交換できます。つぎのように、とても複雑なスクレデンシャルの組み合わせを作ることができます:

    credentials: [[root, [supplier, [owner, quasiowner]], accounts]]
                 # root OR (supplier AND (owner OR quasiowner)) OR accounts

バリデーションとエラー処理のメソッド
------------------------------------

アクションの入力バリデーションは、たいていはリクエストパラメーターですが、繰り返しが多い単調なタスクです。symfonyは、アクションのクラスのメソッドを利用する、組み込みのリクエストのバリデーションシステムを提供します。

具体例から始めましょう。ユーザーが`myAction`にリクエストするとき、symfonyはつねに最初に`validateMyAction`という名前のメソッドを探します。見つかる場合、symfonyはそのメソッドを実行します。このバリデーションメソッドの戻り値はつぎに実行するメソッドを決定します: `true`を返す場合、`executeMyAction()`が実行されます; そうでなければ、`handleErrorMyAction()`が実行されます。そして、後者の場合、`handlerErrorMyAction()`は存在しないので、symfonyは一般的な`handleError()`メソッドを探します。両方とも存在しない場合、`myActionError.php`テンプレートをレンダリングする`sfView::ERROR`を単に返すだけです。図6-2はこのプロセスを描写しています

図6-2 - バリデーションのプロセス

![バリデーションのプロセス](http://github.com/masakielastic/symfonybook-ja/raw/master/images/F0602.png "バリデーションのプロセス")

ですのでバリデーションの秘訣はアクションメソッドのために命名規約を尊重することです:

  * `validateActionName`はバリデーションメソッドで、`true`もしくは`false`を返します。`ActionName`アクションがリクエストされたときに最初に探されるメソッドです。存在しない場合、アクションメソッドは直接実行されます。
  * `handleErrorActionName`はバリデーションメソッドが失敗したときに呼び出されるメソッドです。存在しない場合、`Error`テンプレートが表示されます。
  * `executeActionName`はアクションメソッドです。すべてのアクションに対して存在しなければなりません。

リスト6-29はバリデーションメソッドによるアクションクラスの例を示します。この例において、バリデーションが成功するか失敗するかにかかわらず、`myActionSuccess.php`テンプレートは実行されますが、同じパラメーターでは実行されません。

リスト6-29 - バリデーションメソッドのサンプル

    [php]
    class mymoduleActions extends sfActions
    {
      public function validateMyAction()
      {
        return ($this->getRequestParameter('id') > 0);
      }

      public function handleErrorMyAction()
      {
        $this->message = "無効なパラメーター";

        return sfView::SUCCESS;
      }

      public function executeMyAction()
      {
        $this->message = "パラメーターは正しい";
      }
    }

`validate()`メソッドに望むコードを追加できます。これらのメソッドが`true`もしくは`false`のどちらかを返すものであることを確認してください。これは`sfActions`クラスのメソッドなので`sfRequest`と`sfUser`オブジェクトにも同じようにアクセスできます。これは入力とコンテキストのバリデーションのために非常に役立ちます。

フォームのバリデーションを実装するためにこのメカニズムを利用できますが(つまり、処理するまえにユーザーが入力した値をコントロールする)、これは繰り返されるタイプのタスクなので、10章で説明されるように、このためにsymfonyは自動化ツールを提供します。

フィルター
--------

セキュリティ処理はすべてのリクエストがアクションを実行するまえに通過しなければならない1つのフィルター(filter)として理解できます。フィルターのなかで実行されるいくつかのテストにしたがって、リクエストの処理は、たとえば実行されたアクションを変更することで修正されます(セキュリティフィルターの場合、リクエストされたアクションの代わりにdefault/secure)。symfonyはこのアイディアをフィルタークラスに発展させます。アクションの実行前、もしくはレスポンスのレンダリングのまえに実行されるフィルタークラスの数を指定し、すべてのリクエストに対してこれを行います。フィルターをコードにまとめる方法としてみなすことができます。これは`preExecute()`と`postExecute()`と似ていますが、より高いレベルです(モジュール全体の代わりにアプリケーション全体)。

### フィルターチェーン

symfonyは実際にはリクエストの処理をフィルターチェーン(filter chain)と見なします。リクエストがフレームワークによって受信されたとき、最初のフィルター(つねに`sfRenderingFilter`)が実行されます。ある時点で、チェーンのなかのつぎのフィルターを呼び出し、同じように続きます。最後のフィルター(つねに`sfExecutionFilter`)が実行されるとき、以前のフィルターを終了させることが可能で、フィルターのレンダリングなどに戻ります。図6-3は、連続したダイアグラムで、模造の小さなフィルターチェーン(本物はもっと多くのフィルターを含む)を使ってこのアイディアを説明しています。

図6-3 - フィルターチェーンのサンプル

![図6-3 - フィルターチェーンのサンプル](http://github.com/masakielastic/symfonybook-ja/raw/master/images/F0603.png "図6-3 - フィルターチェーンのサンプル")

このプロセスはフィルタークラスの構造が正しいことを証明します。これらすべては`sfFilter`クラスを継承し、`$filterChain`オブジェクトをパラメーターとして必要とする`execute()`メソッドを含みます。このメソッドのどこかで、フィルターは`$filterChain->execute()`を呼び出すことでチェーンのつぎのフィルターに移動します。リスト6-30を例としてご覧ください。基本的には、フィルターは2つの部分に分割されます:

  * `$filterChain->execute()`の呼び出し前のコードはアクションが実行されるまえに実行されます。
  * `$filterChain->execute()`の呼び出し後のコードはアクションが実行された後とレンダリングのまえに実行されます。

リスト6-30 - フィルタークラスの構造

    [php]
    class myFilter extends sfFilter
    {
      public function execute ($filterChain)
      {
        // この部分のコードは、アクションが実行されるまえに実行される(訳注：ファーストパス部分)
        ...

        // チェーンでつぎのフィルターを実行する
        $filterChain->execute();

        // この部分のコードは、アクションが実行された後、レンダリングが実行されるまえに実行される(訳注：セカンドパス部分)
        ...
      }
    }

リスト6-31で示されるように、デフォルトのフィルターチェーンは`filters.yml`という名前のアプリケーション設定ファイル内で定義されます。このファイルはすべてのリクエストに対して実行される予定のフィルターの一覧を表示します。

リスト6-31 - デフォルトのフィルターチェーン(`myapp/config/filters.yml`)

    rendering: ~
    web_debug: ~
    security:  ~

    # 一般的に、ここに独自のフィルターを挿入する

    cache:     ~
    common:    ~
    flash:     ~
    execution: ~

これらの宣言はパラメーターを持ちません(チルダ記号、`~`、はYAMLにおいて"null"を意味します)。なぜなら、これらはsymfonyのコアで定義されたパラメーターを継承するからです。コアにおいて、symfonyはそれぞれのフィルターに対して`class`と`param`設定を定義します。たとえば、リスト6-31は`rendering`フィルターに対するデフォルトのパラメーターを示しています。

リスト6-32 - フィルターをレンダリングするためのデフォルトのパラメーター(`$sf_symfony_data_dir/config/filters.yml`)

    rendering:
      class: sfRenderingFilter   # フィルタークラス
      param:                     # フィルターパラメーター
        type: rendering

空の値(`~`)をアプリケーションの`filters.yml`に残すことで、コア内部で定義されたデフォルト設定をフィルターに適用するようにsymfonyに伝えます。

さまざまな方法でフィルターチェーンをカスタマイズできます:

  * `enabled: off`パラメーターを追加することでチェーンからいくつかのフィルターを無効にできます。たとえば、Webデバッグフィルターを無効にするには、つぎのように書きます:

        web_debug:
          enabled: off

  * フィルターを無効にするには`filters.yml`からエントリーを削除しないでください； この場合、symfonyは例外を投じます。
  * カスタムフィルターを追加するには独自の宣言をチェーン(通常は`security`フィルターの後)のどこかに追加してください(つぎのセクションで検討)。`rendering`フィルターは最初のエントリーでなければならないこと、`execution`フィルターはフィルターチェーンの最後のエントリーでなければならないことに注意してください。
  * デフォルトのフィルターのデフォルトのクラスとパラメーターをオーバーライドします(特にセキュリティシステムを修正して、独自のセキュリティフィルターを使うため)。

>**TIP**
> `enabled: off`パラメーターは独自のフィルターを無効にするために十分に機能しますが、`settings.yml`ファイルを通して、`web_debug`、`use_security`、`cache`、`use_flash`設定の値を修正することでデフォルトのフィルターを無効にできます。これはそれぞれのデフォルトのフィルターがこれらの設定の値をテストする`condition`パラメーターを持つからです。

### 独自フィルターを開発する

フィルターを開発する方法はとてもシンプルです。オートロード機能を利用するために、リスト6-30で示されるような定義を作り、プロジェクトの`lib/`フォルダーの1つに設置します。

アクションは別のアクションにフォワードするかリダイレクトすることが可能で、結果としてフィルターのフルチェーンを再起動するので、独自フィルターの実行をリクエストの最初のアクション呼び出しに制限したいことがあります。この目的のために`sfFilter`クラスの`isFirstCall()`メソッドはブール値を返します。この呼び出しの意味があるのはアクションが実行される前だけです。

これらの概念は実例でよりあきらかになります。リスト6-33は、ログインアクションによって作られたことを前提とした、固有の`MyWebSite`Cookieを持つユーザーを自動ログインするために使われるフィルターを示します。ログインフォームに提供された"remember me"の機能を実装することは初歩的ですが実用的な方法です。

リスト6-33 - フィルタークラスのサンプル(`apps/myapp/lib/rememberFilter.class.php`)

    [php]
    class rememberFilter extends sfFilter
    {
      public function execute($filterChain)
      {
        // このフィルターを1回だけ実行する
        if ($this->isFirstCall())
        {
          // フィルターはリクエストとユーザーのオブジェクトに直接アクセスできない。
          // これらを手に入れるためにcontextオブジェクトを使う必要がある
          $request = $this->getContext()->getRequest();
          $user    = $this->getContext()->getUser();

          if ($request->getCookie('MyWebSite'))
          {
            // ログイン
            $user->setAuthenticated(true);
          }
        }

        // つぎのフィルターを実行する
        $filterChain->execute();
      }
    }

いくつかの場合において、フィルターチェーンを実行する代わりに、フィルターの最後で特定のアクションにフォワードすることが必要になります。`sfFilter`は`forward()`メソッドを持ちませんが`sfContoroller`が代行するので、つぎのコードを呼び出すことで簡単に実現できます:

    [php]
    return $this->getContext()->getController()->forward('mymodule', 'myAction');

>**NOTE**
>`sfFilter`クラスは`initialize()`メソッドを保有し、このメソッドはフィルターオブジェクトが作られたときに実行されます。独自の方法でフィルターパラメーター(次で説明されるように`filters.yml`ファイルで定義される)を処理する必要がある場合、カスタムフィルター内でそのメソッドをオーバーライドできます。

### フィルターの有効化とパラメーター

フィルターファイルを有効にするには作成するだけでは不十分です。フィルターをフィルターチェーンに追加する必要があります。そのためには、リスト6-34で示されるように、アプリケーションもしくはモジュールの`config/`ディレクトリに設置される、`filteres.yml`のなかでフィルタークラスを宣言しなければなりません。

リスト6-34 - フィルターのサンプルを有効にするファイル(`apps/myapp/config/filters.yml`)

    rendering: ~
    web_debug: ~
    security:  ~

    remember:                 # フィルターは独自の名前が必要
      class: rememberFilter
      param:
        cookie_name: MyWebSite
        condition:   %APP_ENABLE_REMEMBER_ME%

    cache:     ~
    common:    ~
    flash:     ~
    execution: ~

有効にされたとき、フィルターはそれぞれのリクエストに対して実行されます。フィルターの設定ファイルは`param`キーのもとで1つもしくは複数のパラメーターの定義を含みます。フィルタークラスは`getParameter()`メソッドを用いてこれらのパラメーターの値を得る機能を持ちます。リスト6-35ではフィルターパラメーターの値を得る方法を示しています。

リスト6-35 - パラメーターの値を取得する(`apps/myapp/lib/rememberFilter.class.php`)

    [php]
    class rememberFilter extends sfFilter
    {
      public function execute($filterChain)
      {
          ...
          if ($request->getCookie($this->getParameter('cookie_name')))
          ...
      }
    }

`condition`パラメーターはフィルターは実行されなければならないかを確かめるためにフィルターチェーンによってテストされます。リスト6-34のように、フィルターの宣言はアプリケーションの設定に依存する可能性があります。rememberフィルターはアプリケーションの`app.yml`がつぎの内容を示す場合のみ実行されます:

    all:
      enable_remember_me: on

### サンプルのフィルター

フィルター機能はすべてのアクションに対してコードを繰り返すために便利です。たとえば、外部の分析システムを利用する場合、おそらくは外部のトラッカースクリプトを呼び出すコードスニペットをすべてのページに設置することが必要です。このコードをグローバルレイアウトに設置できますが、すべてのアプリケーションに対して有効になってしまいます。代わりの方法として、リスト6-36で示されるように、コードをフィルターのなかで設置することが可能でモジュール単位で有効にできます。

リスト6-36 - Google Analyticsのフィルター

    [php]
    class sfGoogleAnalyticsFilter extends sfFilter
    {
      public function execute($filterChain)
      {
        // アクションの前は何もしない
        $filterChain->execute();

        // トラッカーコードでレスポンスを飾り付ける
        $googleCode = '
    <script src="http://www.google-analytics.com/urchin.js"  type="text/javascript">
    </script>
    <script type="text/javascript">
      _uacct="UA-'.$this->getParameter('google_id').'";urchinTracker();
    </script>';
        $response = $this->getContext()->getResponse();
        $response->setContent(str_ireplace('</body>', $googleCode.'</body>',$response->getContent()));
       }
    }

フィルターはトラッカーをHTMLではないレスポンスに追加しないので、フィルターは完全なものではないことに注意してください。

別の例は、リクエストをSSLに切り替えるフィルターです。リスト6-37で示されるように、このフィルターはコミュニケーションを安全にするために、まだSSLの切り替えが行われていない場合に切り替えを行います。

リスト6-37 - セキュアなコミュニケーションフィルター

    [php]
    class sfSecureFilter extends sfFilter
    {
      public function execute($filterChain)
      {
        $context = $this->getContext();
        $request = $context->getRequest();

        if (!$request->isSecure())
        {
          $secure_url = str_replace('http', 'https', $request->getUri());

          return $context->getController()->redirect($secure_url);
          // フィルターチェーンを継続しない
        }
        else
        {
          // リクエストはすでにセキュアなので、続けることができる
          $filterChain->execute();
        }
      }
    }

フィルターはアプリケーションの機能をグローバルに拡張できるので、フィルターはプラグインで広く使われます。プラグインに関してさらに学習するには17章を参照してください。より多くのフィルターの例に関して公式サイトのwiki([http://trac.symfony-project.org/wiki](http://trac.symfony-project.org/wiki))をご覧ください。

モジュールの設定
----------------

モジュールのふるまいのいくつかは設定に依存します。修正するには、モジュールの`config/`ディレクトリに`module.yml`ファイルを作り、環境ごとに(もしくはすべての環境のための`all: `ヘッダー)のための設定を定義しなければなりません。リスト6-38は`mymodule`モジュールのための`module.yml`の例を示します。

リスト6-38 - モジュールの設定(`apps/myapp/modules/mymodule/config/module.yml`)

    all:                 # すべての環境用
      enabled:     true
      is_internal: false
      view_class:  sfPHP

`enabled`パラメーターによってモジュールのすべてのアクションを無効にできます。すべてのアクションは`module_disabled_module/module_disabled_action`アクションにリダイレクトされます(`settings.yml`で定義)。

`is_internal`パラメーターによってモジュールのすべてのアクションの実行を内部呼び出しに制限できます。この機能は、たとえば、Eメールのメッセージを、外部からではなく、内部から送るために、別のアクションから呼び出さなければならないメールアクションに対して便利です。

`view_name`パラメーターはビュークラスを定義します。このパラメーターは`sfView`から継承しなければなりません。この値をオーバーライドすることでSmartyといったほかのテンプレートシステムを用いてほかのビューシステムを利用できるようになります。

まとめ
----

symfonyにおいて、コントロールレイヤーは2つの部分(フロントコントローラーとアクション)に分割されます。フロントコントローラー(front controller)は任意の環境、ページロジックを含むアクションのためのアプリケーションへの唯一のエントリーポイント(entry point - 入り口)です。アクション(action)はページロジックを含みます。アクションは`sfView`定数の1つを返すことでビューが実行される方法を決める機能を持ちます。アクションの内部では、リクエストオブジェクト(`sfRequest`)と現在のユーザーセッションのオブジェクト(`sfUser`)を含む、文脈の異なる要素を操作することができます。

セッションオブジェクト、アクションオブジェクトとセキュリティの設定の力を一体化することで、アクセス制限機能とクレデンシャル(credential)を持つ完全なセキュリティシステムが提供されます。特別な`validate()`と`handleError()`メソッドによってリクエストのバリデーション(validation)をアクションのなかで扱うことができます。そして`preExecute()`と`postExecute()`メソッドがモジュール内部のコードの再利用のために役立つ場合、フィルター(filter)は、コントローラーのコードをリクエストごとに実行することで、すべてのアプリケーションに対して同じ再利用性を公認します。
