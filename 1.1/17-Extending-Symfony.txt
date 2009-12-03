第17章 - symfonyを拡張する
==========================

結局のところ、symfonyのふるまいを変える必要があります。特定のクラスのふるまいを修正するもしくは独自のカスタム機能を追加する必要があるにせよ、変更作業は必然です。フレームワークが予想できない固有の要望を顧客が持つからです。実際のところ、この状況はとてもありふれているのでsymfnyは単純なクラスの継承を越えて、実行時に既存のクラスを拡張する方法を提供します。ファクトリ(factory)の設定を利用することで、symfonyのコアクラスを独自のものに置き換えできます。いったん拡張機能(エクステンション)を作れば、それをプラグインとして簡単にパッケージにできるので、別のアプリケーションもしくは別のsymfonyのユーザーが再利用できます。

イベント
--------

PHPの現在の制限のなかで、もっとも悩ましいことは複数のクラスを継承できるクラスを持ていないことです。別の制限は既存のクラスに新しいクラスを追加できないこともしくは既存のクラスをオーバーライドできないことです。これらの2つの制限を緩和してsymfonyフレームワークを本当に拡張できるものにするために、symfonyは*イベントシステム*を導入します。この機能はCocoaフレームワークのイベントマネージャにインスパイアされObserverデザインパターンに基づいています([http://en.wikipedia.org/wiki/Observer_pattern](http://en.wikipedia.org/wiki/Observer_pattern))。

### イベントを理解する

symfonyのクラスのなかにはさまざまな瞬間に"イベントを通知する"ものがあります。たとえば、ユーザーがcultureを変更するとき、ユーザーオブジェクトは`change_culture`イベントを通知します。このイベントはプロジェクト空間のなかで大声でつぎの内容を叫ぶことに似ています: "それをやっています。あなたがお望みのことは何でもしてください。"

イベントが起きるときに何か特別なことを実行することに決めることができます。たとえば、ユーザーが望むcultureを記録するために、`change_culture`イベントが通知されるたびにユーザーのcultureをデータベースのテーブルに保存できます。これを行うには、*イベントリスナーを登録する必要があります*。イベントリスナー(event listener)はイベントが起きるときに呼び出される関数を宣言することを伝える複雑なステートメントです。リスト17-1はリスナーを`change_culture`イベントに登録する方法を示してます。

リスト17-1 - イベントリスナーを登録する

    [php]
    $dispatcher->connect('user.change_culture', 'changeUserCulture');
    
    function changeUserCulture(sfEvent $event)
    {
      $user = $event->getSubject();
      $culture = $event['culture'];

      // ユーザーのcultureで何かを行う
    }

すべてのイベントとリスナーの登録は*Event Dispatcher*と呼ばれる特別なオブジェクトによって管理されます。このオブジェクトはSingletonの`sfContext`を利用することでsymfonyのどこからでもアクセス可能で、たいていのsymfonyのオブジェクトはそれにアクセスできる`getEventDispatcher()`メソッドを提供します。ディスパッチャの`connect()`メソッドを利用することで、イベントが起きるときに呼び出されるPHPのcallable(クラスのメソッドもしくは関数)を登録できます。`connect()`の最初の引数はイベントの識別子で、名前空間と名前で構成される文字列です。2番目の引数はPHPのcallableです。

いったん関数がEvent Dispatcherに登録されると、イベントが停止されるまで待機します。Event Dispatcherはすべてのイベントリスナーの記録を行い、イベントが通知されるときにどれが呼び出されるのかを知っています。これらのメソッドもしくは関数を呼び出すとき、ディスパッチャはこれらに`sfEvent`オブジェクトをパラメーターとして渡します。

イベントオブジェクトは通知されたイベントに関する情報を保存します。`getSubject()`メソッドのおかげでイベント通知機能(notifier)を読みとることが可能で、イベントパラメーターはイベントオブジェクトを配列として利用することでアクセスできます(たとえば、`user.change_culture`を通知するとき`sfUser`によって渡される`culture`パラメーターを読みとるために`$event['culture']`を利用できます)。

まとめると、イベントシステムによって、継承を利用せずに、既存のクラスに機能を追加するもしくは実行時にメソッドを修正できるようになります。

>**NOTE**: バージョン1.0において、symfonyはよく似ているが異なる構文を持つシステムを利用していました。Event Dispatcherのメソッドを呼び出す変わりに、イベントを登録して通知する`sfMixer`クラスのstaticなメソッドの呼び出しを見ることがあります。`sfMixer`の呼び出しは非推奨ですが、symfony 1.1でもまだ動作します。

### イベントを通知する

symfonyのクラスがイベントを通知するように、独自のクラスは実行時の拡張性を提供し特定の状況時にイベントを通知できます。 たとえば、アプリケーションがいくつかのサードパーティのWebサービスをリクエストしこれらのリクエストのRESTロジックをラップするために`sfRestRequest`クラスを書いた場合を考えてみましょう。このクラスが新たにリクエストするたびにイベントを通知することはよいアイデアといえます。これによって将来のロギング機能もしくはキャッシュ機能の追加が簡単になります。リスト17-2はイベントを通知するために既存の`fetch()`メソッドに追加するコードを示してます。

リスト17-2 - イベントを通知する

    [php]
    class sfRestRequest
    {
      protected $dispatcher = null;

      public function __construct(sfEventDispatcher $dispatcher)
      {
        $this->dispatcher = $dispatcher;
      }
      
      /**
       * 外部のWebサービスにクエリを行う
       */
      public function fetch($uri, $parameters = array())
      {
        // 取得プロセスの開始を通知する
        $this->dispatcher->notify(new sfEvent($this, 'rest_request.fetch_prepare', array(
          'uri'        => $uri,
          'parameters' => $parameters
        )));
        
        // リクエストを行い結果を変数$resultに保存する
        // ...
        
        // 取得プロセスの終了を通知する
        $this->dispatcher->notify(new sfEvent($this, 'rest_request.fetch_success', array(
          'uri'        => $uri,
          'parameters' => $parameters,
          'result'     => $result
        )));
        
        return $result;
      }
    }

Event Dispatcherの`notify()`メソッドは`sfEvent`オブジェクトをパラメーターとして必要とします; これはイベントリスナーに渡されるオブジェクトそのものです。このオブジェクトはつねに参照を通知機能とイベント識別子に運びます(これがイベントのインスタンスが`this`で初期化される理由)。オプションとして、これはパラメーターの連想配列を受けとります。これはリスナーに通知機能のロジックと情報をやりとりする方法を提供します。

>**TIP**
>イベントを通知するクラスだけがイベントシステムの方法によって拡張できます。将来クラスを拡張する必要があるかわからない場合、重要なメソッドに通知機能を追加することはつねによい考えです。

### リスナーがイベントを扱うまでイベントを通知する

`notify()`メソッドを利用することで、通知されたイベントに登録されたすべてのリスナーが実行されることを確認できます。しかしいくつかの場合において、リスナーがイベントを停止させて通知されないようにさせることが必要です。この場合、`notify()`メソッドの代わりに`notifyUntil()`メソッドを使うべきです。ディスパッチャは特定のリスナーが`true`を返すまですべてのリスナーを実行します。言い換えると、`notifyUntil()`メソッドはプロジェクトの空間で"やってます。誰かが気づいたら、誰かに伝えることを止めます。"ということを伝えるようなものです。リスト17-3はメソッドを追加するために`__call()`マジックメソッドを組み合わせるテクニックの使いかたを示しています。

リスト17-3 - リスナーがtrueを返すまでイベントを通知する

    [php]
    class sfRestRequest
    {
      // ...
      
      public function __call($method, $arguments)
      {
        $event = $this->dispatcher->notifyUntil(new sfEvent($this, 'rest_request.method_not_found', array(
          'method'    => $method, 
          'arguments' => $arguments
        )));
        if (!$event->isProcessed())
        {
          throw new sfException(sprintf('Call to undefined method %s::%s.', get_class($this), $method));
        }
        
        return $event->getReturnValue();
      }
    }

`rest_request.method_not_found`イベントに登録されたすべてのリスナーはリクエストされた`$method`をテストしそれを扱うことを決定する、もしくはつぎの呼び出し可能なイベントリスナーへの移動します。リスト17-4において、このトリックを利用してサードパーティのクラスが`sfRestRequest`クラスに`put()`メソッドと`delete()`メソッドを実行時に追加する方法を見ることができます。

リスト17-4 - "条件を満たすまで通知する"-タイプのイベントを扱う

    [php]
    class frontendConfiguration extends sfApplicationConfiguration
    {
      public function configure()
      {
        // ...

        // リスナーを登録する
        $this->dispatcher->connect('rest_request.method_not_found', array('sfRestRequestExtension', 'listenToMethodNotFound'));
      }
    }

    class sfRestRequestExtension
    {
      static public function listenToMethodNotFound(sfEvent $event)
      {
        switch ($event['method'])
        {
          case 'put':
            self::put($event->getSubject(), $event['arguments'])

            return true;
          case 'delete':
            self::delete($event->getSubject(), $event['arguments'])

            return true;
          default:
            return false;
        }
      }

      static protected function put($restRequest, $arguments)
      {
        // putリクエストを行い結果を変数$resultに保存する
        // ...
        
        $event->setReturnValue($result);
      }
      
      static protected function delete($restRequest, $arguments)
      {
        // deleteリクエストを行い結果を変数$resultに保存する
        // ...
        
        $event->setReturnValue($result);
      }
    }

実際には、`notifyUntil()`メソッドは、mixin(サードパーティのメソッドを既存のクラスに追加する)よりも、多重継承の機能をPHPに提供します。これで新しいメソッドを継承の方法では拡張できないオブジェクトに"注入"(inject)できます。そしてこれは実行時に行われます。これでsymfonyを利用するときPHPのオブジェクト指向の機能によって制限されません。

>**TIP**
>`notifyUntil`イベントをキャッチする最初のリスナーはそれ以降のイベント通知を防止するので、リスナーの実行順序が不安になるかもしれません。この順序はリスナーの登録順序に対応します。つまり最初に登録されたものが最初に実行されます。しかし実際には、登録順序が原因でほかのリスナーの実行を妨げることはほとんどありません。2つのリスナーが特定のイベントで衝突することに気がついたら、一方のクラスに新たなイベント通知を追加すればよいのです。たとえばメソッド実行のはじめとおわりなどです。そして既存のクラスに新しいメソッドを追加するためにイベントを使う場合、メソッドを追加する時点でほかのものが衝突しないようにメソッドの名前をつけてください。メソッドの名前のプレフィックスをリスナークラスの名前にすることはよい習慣です。

### メソッドの戻り値を変更する

リスナーがイベントによって渡された情報を利用するだけでなく、通知機能のオリジナルのロジックを変更するためにそれを修正する方法も想像できるでしょう。これを実現したければ、`notify()`メソッドよりもEvent Dispatcherの`filter()`メソッドを使うべきです。すべてのイベントリスナーは2つのパラメーター: イベントオブジェクト、とフィルタリングする値で呼び出されます。値を変更するかしないかにかかわらず、イベントリスナーは値を返さなければなりません。リスト17-5はWebサービスからレスポンスをフィルタリングしてそのレスポンス内で特別な文字をエスケープするために使われる`filter()`メソッドを表示します。

リスト17-5 - フィルターイベントを通知して扱う

    [php]
    class sfRestRequest
    {
      // ...
  
      /**
       * 外部のWebサービスにクエリを行う
       */
      public function fetch($uri, $parameters = array())
      {
        // リクエストを行い結果を$result変数に保存する
        // ...
        
        // 取得プロセスの終了を通知する
        return $this->dispatcher->filter(new sfEvent($this, 'rest_request.filter_result', array(
          'uri'        => $uri,
          'parameters' => $parameters,
        ), $result));
      }
    }

    // Webサービスのレスポンスにエスケーピング機能を追加する
    $dispatcher->connect('rest_request.filter_result', 'rest_htmlspecialchars');

    function rest_htmlspecialchars(sfEvent $event, $result)
    {
      return htmlspecialchars($result, ENT_QUOTES, 'UTF-8');
    }

### 組み込みのイベント

symfonyのクラスの多くは組み込みのイベントを持つので、クラス自身を変更することなくフレームワークを拡張できます。テーブル17-1はこれらのイベント、それぞれのタイプと引数の一覧を示しています。

テーブル17-1 - symfonyのイベント

| **イベントの名前空間** | **イベントの名前**     | **型**    | **通知者**          | **引数**               |
| ------------------- | ------------------ | ----------- | ---------------------- | --------------------------- |
| **application**     | log                | notify      | lot of classes         | priority                    |
|                     | throw_exception    | notifyUntil | sfException            | -                           |
| **command**         | log                | notify      | sfCommand* classes     | priority                    |
|                     | pre_command        | notifyUntil | sfTask                 | arguments, options          |
|                     | post_command       | notify      | sfTask                 | -                           |
|                     | filter_options     | filter      | sfTask                 | command_manager             |
| **configuration**   | method_not_found   | notifyUntil | sfProjectConfiguration | method, arguments           |
| **component**       | method_not_found   | notifyUntil | sfComponent            | method, arguments           |
| **context**         | load_factories     | notify      | sfContext              | -                           |
| **controller**      | change_action      | notify      | sfController           | module, action              |
|                     | method_not_found   | notifyUntil | sfController           | method, arguments           |
|                     | page_not_found     | notify      | sfController           | module, action              |
| **plugin**          | pre_install        | notify      | sfPluginManager        | channel, plugin, is_package |
|                     | post_install       | notify      | sfPluginManager        | channel, plugin             |
|                     | pre_uninstall      | notify      | sfPluginManager        | channel, plugin             |
|                     | post_uninstall     | notify      | sfPluginManager        | channel, plugin             |
| **request**         | filter_parameters  | filter      | sfWebRequest           | path_info                   |
|                     | method_not_found   | notifyUntil | sfRequest              | method, arguments           |
| **response**        | method_not_found   | notifyUntil | sfResponse             | method, arguments           |
|                     | filter_content     | filter      | sfResponse             | -                           |
| **routing**         | load_configuration | notify      | sfRouting              | -                           |
| **task**            | cache.clear        | notifyUntil | sfCacheClearTask       | app, type, env              |
| **template**        | filter_parameters  | filter      | sfViewParameterHolder  | -                           |
| **user**            | change_culture     | notify      | sfUser                 | culture                     |
|                     | method_not_found   | notifyUntil | sfUser                 | method, arguments           |
| **view**            | configure_format   | notify      | sfView                 | format, response, request   |
|                     | method_not_found   | notifyUntil | sfView                 | method, arguments           |
| **view.cache**      | filter_content     | filter      | sfViewCacheManager     | response, uri, new  

これらのイベントにすべてのリスナーを自由に登録できます。呼び出し可能なリスナーは、`notifyUntil`型のイベントに登録されたときブール値を返し、`filter`型のイベントに登録されたときはフィルタリングされた値を返すことを確認してください。

イベントの名前空間はクラスの役割にかならずしもマッチする必要はないことに注意してください。たとえば、すべてのsymfonyのクラスは何かをログファイル(とWebデバッグツールバー)に表示する必要があるときに`application.log`イベントを通知します:

    [php]
    $dispatcher->notify(new sfEvent($this, 'application.log', array($message)));

これがつじつまが合っているとき独自のクラスは同じことを行いsymfonyのイベントに通知も行います。

### リスナーを登録する場所は？

symfonyのリクエストの初期段階でイベントリスナーを登録することが必要です。実際には、イベントリスナーを登録する正しい場所はアプリケーションの設定クラスです。このクラスは`configure()`メソッドで使えるEvent Dispatcherへの参照を持ちます。リスト17-6は上記の例の`rest_request`イベントの1つにリスナーを登録する方法を示しています。

リスト17-6 - アプリケーション設定クラスにリスナーを登録する(`apps/frontend/config/ApplicationConfiguration.class.php`)

    [php]
    class frontendConfiguration extends sfApplicationConfiguration
    {
      public function configure()
      {
        $this->dispatcher->connect('rest_request.method_not_found', array('sfRestRequestExtension', 'listenToMethodNotFound'));
      }
    }

プラグイン(下記を参照)は独自のイベントリスナーを登録できます。これらのプラグインはプラグインの`config/config.php`スクリプトでこれを行います。これはアプリケーションの初期化の間に実行され`$this->dispatcher`を通してEvent Dispatcherにアクセスできます。

>**SIDEBAR**
>Propelのビヘイビアー
>
>以前8章で説明しましたが、Propelはイベントシステムを利用します。正直に言えば、これらはsymfony1.0のイベントシステムを利用しますが、問題ありません。これらはPropelが生成したオブジェクトの拡張を有効にするためのイベントの登録と扱うコードをパッケージにします。例を見てみましょう。
>
>Propelオブジェクトはすべてが`delete()`メソッドを持つデータベースのテーブルに対応します。`delete`メソッドはデータベースから関連するレコードを削除します。しかし、レコードを削除できない`Invoice`クラスに関して、データベース内部のレコードを維持できるように`delete()`メソッドを変更して、`is_deleted`属性の値をtrueに変更するとよいでしょう。通常のオブジェクトの読みとりメソッド(`doSelect()`、`retrieveByPk()`)は`is_deleted`が`false`であるとレコードをみなすだけです。本当にレコードを削除できる`forceDelete()`と呼ばれる別のメソッドを追加することも必要です。実際、これらすべての修正は`ParanoidBehavior`と呼ばれる新しいクラスにまとめられます。最後の`Invoice`クラスはPropelの`BaseInvoice`クラスを継承し、ミックスインされた`ParanoidBehaviorMixin`のメソッドを持ちます。
>
>ビヘイビアーはPropelオブジェクト上のミックスインです。実際に、symfonyにおいて"ビヘイビアー"という用語は複数の内容をカバーします: ミックスインはプラグインとしてパッケージになります。先ほど述べた`ParanoidBehavior`クラスは`sfPropelParanoidBehaivorPlugin`と呼ばれる実在のsymfonyプラグインに対応します。とこのプラグインのインストール方法と使いかたに関する詳細な内容はsymfony公式サイトのwiki([http://www.symfony-project.org/plugins/sfPropelParanoidBehaviorPlugin](http://www.symfony-project.org/plugins/sfPropelParanoidBehaviorPlugin))を参照してください。
>
>ビヘイビアーに関して最後の一言です: これらをサポートできるようにするには、生成されたPropelオブジェクトはかなりの数のイベント通知機能を収納しなければなりません。ビヘイビアーを使わない場合、これらが実行を遅くしてパフォーマンスに不利益をもたらすことがあります。イベントがデフォルトで有効になっていないのはそういうわけです。これを追加することでビヘイビアーのサポートを有効にするには、最初に`propel.ini`ファイルのなかの`propel.builder.addBehaviors`プロパティを`true`に設定してモデルをリビルドしなければなりません。

ファクトリ
----------

ファクトリ(factory)は特定のタスクのためのクラスの定義です。symfonyはコントローラーやセッション機能などのコア機能についてファクトリに依存します。たとえば、symfonyが新しいリクエストオブジェクトを作成する必要がある場合、symfonyはこの目的のために使うクラスの名前のためのファクトリの定義を検索します。リクエスト用のデフォルトのファクトリの定義は`sfWebRequest`クラスで、symfonyはリクエストを処理するためにこのクラスのオブジェクトを作成します。ファクトリの定義の利点はsymfonyのコア機能をとても簡単に変更できることです: ファクトリの定義を変更し、symfonyは自身の代わりにカスタムリクエストクラスを使います。

ファクトリの定義は`factories.yml`設定ファイルに保存されます。リスト17-7はデフォルトのファクトリの定義ファイルを示します。それぞれの定義はオートロードされるクラスの名前と(オプションとして)パラメーターの一式から構成されます。たとえば、セッションストレージのファクトリ(`storage:`キーの下で設定)は一貫したセッションを可能にするためにクライアントコンピュータ上で作成されたCookieに名前をつける`session_name`パラメーターを使います。

リスト17-7 - デフォルトのファクトリファイル(`frontend/config/factories.yml`)

    prod:
      logger:
        class:   sfNoLogger
        param:
          level:   err
          loggers: ~

    cli:
      controller:
        class: sfConsoleController
      request:
        class: sfConsoleRequest
      response:
        class: sfConsoleResponse

    test:
      storage:
        class: sfSessionTestStorage
        param:
          session_path: %SF_TEST_CACHE_DIR%/sessions

    #all:
    #  controller:
    #    class: sfFrontWebController
    #
    #  request:
    #    class: sfWebRequest
    #    param:
    #      formats:
    #        txt:  text/plain
    #        js:   [application/javascript, application/x-javascript, text/javascript]
    #        css:  text/css
    #        json: [application/json, application/x-json]
    #        xml:  [text/xml, application/xml, application/x-xml]
    #        rdf:  application/rdf+xml
    #        atom: application/atom+xml
    #
    #  response:
    #    class: sfWebResponse
    #    param:
    #      logging: %SF_LOGGING_ENABLED%
    #      charset: %SF_CHARSET%
    #
    #  user:
    #    class: myUser
    #    param:
    #      timeout:         1800
    #      logging:         %SF_LOGGING_ENABLED%
    #      use_flash:       true
    #      default_culture: %SF_DEFAULT_CULTURE%
    #
    #  storage:
    #    class: sfSessionStorage
    #    param:
    #      session_name: symfony
    #
    #  view_cache:
    #    class: sfFileCache
    #    param:
    #      automatic_cleaning_factor: 0
    #      cache_dir:                 %SF_TEMPLATE_CACHE_DIR%
    #      lifetime:                  86400
    #      prefix:                    %SF_APP_DIR%
    #
    #  i18n:
    #    class: sfI18N
    #    param:
    #      source:               XLIFF
    #      debug:                off
    #      untranslated_prefix:  "[T]"
    #      untranslated_suffix:  "[/T]"
    #      cache:
    #        class: sfFileCache
    #        param:
    #          automatic_cleaning_factor: 0
    #          cache_dir:                 %SF_I18N_CACHE_DIR%
    #          lifetime:                  86400
    #          prefix:                    %SF_APP_DIR%
    #
    #  routing:
    #    class: sfPatternRouting
    #    param:
    #      load_configuration: true
    #      suffix:             .
    #      default_module:     default
    #      default_action:     index
    #      variable_prefixes:  [':']
    #      segment_separators: ['/', '.']
    #      variable_regex:     '[\w\d_]+'
    #      debug:              %SF_DEBUG%
    #      logging:            %SF_LOGGING_ENABLED%
    #      cache:
    #        class: sfFileCache
    #        param:
    #          automatic_cleaning_factor: 0
    #          cache_dir:                 %SF_CONFIG_CACHE_DIR%/routing
    #          lifetime:                  31556926
    #          prefix:                    %SF_APP_DIR%
    #
    #  logger:
    #    class: sfAggregateLogger
    #    param:
    #      level: debug
    #      loggers:
    #        sf_web_debug:
    #          class: sfWebDebugLogger
    #          param:
    #            condition:      %SF_WEB_DEBUG%
    #            xdebug_logging: true
    #        sf_file_debug:
    #          class: sfFileLogger
    #          param:
    #            file: %SF_LOG_DIR%/%SF_APP%_%SF_ENVIRONMENT%.log


ファクトリを変更する最良の方法はデフォルトのファクトリから継承した新しいクラスを作り、新しいメソッドをそのクラスに追加することです。たとえば、ユーザーセッションのファクトリは`myUser`クラス(`frontend/lib/`ディレクトリに設置)に設定され、`sfUser`クラスから継承します。既存のファクトリを利用するには同じメカニズムを使います。リスト17-8はリクエストオブジェクトのための新しいファクトリの例を示しています。

リスト17-8 - ファクトリをオーバーライドする

    [php]
    //オートロードされるディレクトリ内でmyRequest.class.phpを作る
    // たとえばfrontend/lib/において
    <?php

    class myRequest extends sfRequest
    {
      // あなたのコードをここに
    }

    // factories.ymlでクラスをリクエストファクトリとして宣言する
    all:
      request:
        class: myRequest

ほかのフレームワークのコンポーネントと統合する
--------------------------------------------

サードパーティのクラスによって提供された機能が必要な場合、このクラスを`lib/`ディレクトリの1つにコピーしたくない場合、おそらくはsymfonyがファイルを探す通常の場所の外側にそのクラスをインストールすることになります。この場合、クラスを利用するには、オートロードを利用するsymfonyのsplオートロード統合機能を使わないかぎり、`requre`ステートメントを手動でコードに含めることになります。

symfonyは(まだ)すべてのためのツールを提供していません。PDFジェネレーター、Google MapsのAPI、PHPによるLucene検索エンジンの実装など、おそらくZend Frameworkからいくつかのライブラリが必要になります。PHPで直接イメージを操作する、Eメールを読むためにPOP3アカウントに接続する、コンソールのインターフェイスを設計することなどを行いたい場合、eZcomponentsからライブラリを選ぶことがあるかもしれません。幸いにして、正しい設定を定義すると、これらのライブラリからのコンポーネントはsymfonyで正常に動作します。

(PEAR経由でサードパーティのライブラリをインストールしないかぎり)最初に、アプリケーションの`app.yml`ファイルのなかでライブラリのrootディレクトリへのパスを宣言する必要があります:

    all:
      zend_lib_dir:   /usr/local/zend/library/
      ez_lib_dir:     /usr/local/ezcomponents/
      swift_lib_dir:  /usr/local/swiftmailer/

それから、symfonyがオートロードを失敗するとき、考慮するライブラリを指定することでPHPのオートロードシステムを拡張します。リスト17-9のように、オートロードクラスをアプリケーションの設定クラスに登録することでこれを行うことができます(詳細は19章を参照)。

リスト17-9 - サードパーティのコンポーネントのオートロードを有効にする(`apps/frontend/config/ApplicationConfiguration.class.php`)

    [php]
    class frontendConfiguration extends sfApplicationConfiguration
    {
      public function initialize()
      {
        parent::initialize(); // 最初にsymfonyのオートロード機能をロードする

        // Zend Frameworkを統合する
        if ($sf_zend_lib_dir = sfConfig::get('app_zend_lib_dir'))
        {
          set_include_path($sf_zend_lib_dir.PATH_SEPARATOR.get_include_path());
          require_once($sf_zend_lib_dir.'/Zend/Loader.php');
          spl_autoload_register(array('Zend_Loader', 'autoload'));
        }

        // eZ Componentsを統合する
        if ($sf_ez_lib_dir = sfConfig::get('app_ez_lib_dir'))
        {
          set_include_path($sf_ez_lib_dir.PATH_SEPARATOR.get_include_path());
          require_once($sf_ez_lib_dir.'/Base/base.php');
          spl_autoload_register(array('ezcBase', 'autoload'));
        }

        // Swift Mailerを統合する
        if ($sf_swift_lib_dir = sfConfig::get('app_swift_lib_dir'))
        {
          set_include_path($sf_swift_lib_dir.PATH_SEPARATOR.get_include_path());
          require_once($sf_swift_lib_dir.'/Swift/ClassLoader.php');
          spl_autoload_register(array('Swift_ClassLoader', 'load'));
        }
      }
    }

ロードされていないクラスの新しいオブジェクトを作るときに起きることは単純です:

  1. symfonyのオートロード機能は最初に`autoload.yml`ファイルで宣言されたパスのなかでクラスを探します。
  2. クラスパスが見つからなければ、`spl_autoload_register()`によって登録されたコールバックメソッドはそれらの1つが`true`を返すまで次から次へと呼び出されます。そして`Zend_Loader::autoload()`、`ezcBase::autoload()`、と`Swift_ClassLoader::load()`のうちの1つがクラスを見つけるまでこれらが呼び出されます。
  3. これらも`false`を返す場合、PHPはエラーを生成します。

このことはほかのフレームワークコンポーネントはオートロードメカニズムから恩恵を受け、独自環境よりもこれらを簡単に使えることを意味します。たとえば、Lucene検索エンジンと同等のものをPHPで実装するためにZend Frameworkの`Zend_Search`コンポーネントを使いたい場合、通常は`require`ステートメントが必要です: 

    [php]
    require_once 'Zend/Search/Lucene.php';
    $doc = new Zend_Search_Lucene_Document();
    $doc->addField(Zend_Search_Lucene_Field::Text('url', $docUrl));
    // ...

symfonyとsplのオートロード機能によって、上記のコードはよりシンプルになります。`require`ステートメントを省略できるのでincludeのパスとクラスの位置に悩まなくてすみます:

    [php]
    $doc = new Zend_Search_Lucene_Document(); // クラスがオートロードされた
    $doc->addField(Zend_Search_Lucene_Field::Text('url', $docUrl));
    // ...

プラグイン
----------

1つのsymfonyアプリケーションのために開発したコードピースの再利用がおそらく必要になります。このコードのピースを単独のクラスのパッケージにすることができるのであれば、問題ありません: クラスを別のアプリケーションの`lib/`フォルダーの1つに設置すればオートローダが残りを引き受けます。しかし、コードが複数のファイルに散在している場合、たとえば、administrationジェネレーター用の完全に新しいテーマ、もしくは好みの視覚効果を自動化するJavaScriptファイルとヘルパーの組み合わせなどの場合、ファイルをコピーするだけの方法は最良の解決方法ではありません。

プラグインはいくつかのファイルにまたがるコードをパッケージにする方法と、いくつかのプロジェクトをまたがってこのコードを再利用する方法を提供します。プラグインのなかで、クラス、フィルター、イベントリスナー、ヘルパー、設定、タスク、モジュール、スキーマ、モデルの拡張、フィクスチャ、Webアセットなどをパッケージにすることができます。プラグインをインストール、アップグレード、アンインストールする方法は簡単です。これらは`.tgz`アーカイブ、PEARパッケージ、もしくはコードリポジトリとして配布可能で、コードのリポジトリから簡単にチェックアウトできます。PEARパッケージとなったプラグインは依存関係の管理機能を利用し、アップグレードと自動検出が簡単です。symfonyのロードメカニズムはプラグインを考慮し、プラグインによって提供される機能はあたかもフレームワークの一部であるかのようにプロジェクトで利用できます。

ですので、プラグインは基本的にsymfonyプロジェクトのために拡張機能をパッケージにしたものです。プラグインによってアプリケーションを越えて独自コードを再利用できるだけでなく、別の投稿者によって開発されたものも再利用可能でsymfonyコアにサードパーティの拡張機能を追加できます。

### symfonyのプラグインを見つける

symfonyの公式サイトにはプラグイン専用のページが存在しており、つぎのURLからアクセスできます:

    http://www.symfony-project.org/plugins/

ここに掲載されている各プラグインはそれぞれのページでインストールについての説明やドキュメントが整備されています。

ここにあるプラグインはコミュニティから寄せられたものもあれば、symfonyのコア開発者が開発したものもあります。後者のものについては以下を参照してください。

  * `sfFeed2Plugin`: RSSとAtomフィードの操作を自動化する
  * `sfThumbnailPlugin`: たとえばアップロードされたイメージのためにサムネイルを作る。
  * `sfMediaLibraryPlugin`: メディアのアップロードと管理を可能にします。リッチテキスト内部でイメージの編集を可能にするリッチテキストエディタのための拡張機能を含む
  * `sfShoppingCartPlugin`: ショッピングカートの運用を可能にする
  * `sfPagerNavigationPlugin`: `sfPager`オブジェクトに基づいた古典的でAjaxによるページャーコントロールを提供する
  * `sfGuardPlugin`: 認証、承認とsymfonyの標準的なセキュリティ機能を上回る別のユーザーの管理機能を提供する
  * `sfPrototypePlugin`: prototypeとscript.aculo.usのJavaScriptファイルをスタンドアロンのライブラリとして提供する
  * `sfSuperCachePlugin`: Webのrootディレクトリ下にキャッシュを書き出すことで、Webサーバーの処理を可能なかぎり早めます
  * `sfOptimizerPlugin`: 運用環境において実行が速くなるようにアプリケーションのコードを最適化する(詳細はつぎの章を参照)
  * `sfErrorLoggerPlugin`: データベースにすべての404エラーと500エラーをログに記録しこれらのエラーを閲覧するためのadministrationモジュールを提供する
  * `sfSslRequirementPlugin`: アクションのためのSSL暗号化サポートを提供する

公式サイトはビヘイビアー(behavior)と呼ばれる、Propelオブジェクトを拡張するために設計されたプラグインも提示します。これらのなかで、つぎのものが見つかります:

  * `sfPropelParanoidBehaviorPlugin`: オブジェクトの削除を無効にして、`deleted_at`カラムの更新で置き換える
  * `sfPropelOptimisticLockBehaviorPlugin`: Propelオブジェクト用にオプティミスティックロック(楽観的ロック)を実装する

公式サイトの専用ページを定期的に確認すべきです。いつも新しいプラグインが追加され、これらはWebアプリケーションのプログラミングの多くの面にとても便利なショートカットをもたらしてくれます。

公式サイトの専用ページは別にして、プラグインを配布するほかの方法はダウンロードのためのプラグインアーカイブを提供することと、PEARチャンネルでプラグインをホストすること、もしくは公開のバージョンコントロールリポジトリに保存することです。

### プラグインをインストールする

プラグインのインストール作業はプラグインパッケージの作成方法によって異なります。つねに`README`ファイルかつ・もしくはプラグインのダウンロードのページ上のインストールの手引きを参照してください。

プラグインはプロジェクト単位でインストールされます。つぎのセクションで説明されるすべての方法ではすべてのプラグインのファイルを`myproject/plugins/pluginName/`ディレクトリに設置します。

### PEARプラグイン

公式サイトの専用ページの一覧に記載されているプラグインはPEARチャンネル: `plugins:symfony-project.org`を通して入手できます。プラグインをインストールするには、リスト17-10で示されるように、プラグインの名前と一緒に`plugin:install`タスクを使います。

リスト17-10 - 公式サイトのPEARチャンネルからプラグインをインストールする

    > cd myproject
    > php symfony plugin:install pluginName

代わりの方法として、プラグインをダウンロードしてディスクからインストールすることもできます。この場合、リスト17-11で示されるように、パッケージアーカイブへのパスを使います。

リスト17-11 - ダウンロードしたパッケージからプラグインをインストールする

    > cd myproject
    > php symfony plugin:install /home/path/to/downloads/pluginName.tgz

プラグインのなかには外部のPEARチャンネルでホストされるものがあります。リスト17-12で示されるように、`plugin:install`タスクでそれらをインストールしてチャンネルを登録してチャンネルの名前を記載することを忘れないでください。

リスト17-12 - PEARチャンネルからプラグインをインストールする

    > cd myproject
    > php symfony plugin:add-channel channel.symfony.pear.example.com
    > php symfony plugin:install --channel=channel.symfony.pear.example.com pluginName

これら3つのタイプのインストール方法はすべてPEARパッケージを使うので、"PEARプラグイン"という用語はsymfonyプラグインのPEARチャンネル、外部のPEARチャンネル、もしくはダウンロードしたPEARパッケージからインストールしたプラグインを区別なく説明するために使われます。

リスト17-13で示されているように、`plugin:install`タスクは多くのオプションをとります。

リスト17-13 - いくつかのオプションをつけてプラグインをインストールする

    > php symfony plugin:install --stability=beta pluginName
    > php symfony plugin:install --release=1.0.3 pluginName
    > php symfony plugin:install --install-deps pluginName

>**TIP**
>すべてのsymfonyタスクに関しては、`php symfony help plugin:install`を実行すれば`plugin:install`のオプションと引数の説明を見ることができます

#### アーカイブのプラグイン

プラグインのなかには単純にファイルのアーカイブとしてやってくるものがあります。それらをインストールするために、アーカイブをプロジェクトの`plugins/`ディレクトリに解凍してください。プラグインが`web/`サブディレクトリを含む場合、リスト17-14で示されるように、このディレクトリをコピーするかプロジェクトの`web/`ディレクトリにシンボリックリンクを作ることを忘れないでください。最後に、キャッシュをクリアすることを忘れないでください。

リスト17-14 - アーカイブからプラグインをインストールする

    > cd plugins
    > tar -zxpf myPlugin.tgz
    > cd ..
    > ln -sf plugins/myPlugin/web web/myPlugin
    > php symfony cc

#### バージョン管理システムのリポジトリからプラグインをインストールする

プラグインはときにバージョン管理システム用の独自のソースコードリポジトリを持つことがあります。`plugins/`ディレクトリのなかでチェックアウトするだけでこれらのプラグインをインストールできますが、プロジェクト自身がバージョン管理システムの管理下にある場合、この作業によって問題が引き起こされる可能性があります。

代わりの方法として、プラグインを外部依存のライブラリとして宣言することが可能で、すべてのプロジェクトのソースコードを更新するとプラグインのソースコードも更新されます。たとえば、Subversionは`svn:externals`プロパティで外部依存を保存します。ですので、リスト17-15で示されているように、このプロパティを編集してソースコードをあとで更新することでプラグインを追加できます。

リスト17-15 - ソースのバージョン管理リポジトリからプラグインをインストールする

    > cd myproject
    > svn propedit svn:externals plugins
      pluginName   http://svn.example.com/pluginName/trunk
    > svn up
    > php symfony cc

>**NOTE**
>プラグインが`web/`ディレクトリを含む場合、アーカイブのプラグインに関しては同じようにそのディレクトリへのシンボリックリンクを作らなければなりません。

#### プラグインモジュールを有効にする

プラグインのなかにはモジュール全体を含むものがあります。プラグインモジュールと古典的なモジュールの違いはプラグインモジュールが`myproject/frontend/modules/`ディレクトリに現れないことだけです(簡単にアップグレードできる状態を保つため)。リスト17-16で示されるように、`settings.yml`ファイルのなかでこれらを有効にしなければなりません。

リスト17-16 - プラグインモジュールを有効にする(`frontend/config/settings.yml`)

    all:
      .settings:
        enabled_modules:  [default, sfMyPluginModule]

これはプラグインモジュールを必要としないアプリケーションが誤ってそのプラグインを利用できるように設定する状況を避けるためです。その状況ではセキュリティの欠陥を公開してしまう可能性があります。`frontend`モジュールと`backend`モジュールを提供するプラグインを考えてください。`frontend`モジュールは`frontend`アプリケーション専用として、`backend`モジュールは`backend`アプリケーション専用として有効にする必要があります。プラグインモジュールがデフォルトで有効にされない理由はそういうわけです。

>**TIP**
>defaultモジュールはデフォルトで唯一有効なモジュールです。これは本当のプラグインモジュールではありません。フレームワークの`$sf_symfony_lib_dir/controller/default/`に所属するからです。これは初期ページと、404エラー用のデフォルトページとクレデンシャルが必要なエラーページを提供するモジュールです。symfonyのデフォルトページを使いたくない場合、このモジュールを`enabled_modules`設定から除外します。

#### インストールしたプラグインの一覧を表示する

プロジェクトの`plugins/`ディレクトリをざっと見るとプラグインがインストールされている場所がわかります。そして`plugin:list`タスクはより詳細な情報を示します: バージョン番号とインストールしたそれぞれのプラグインのチャンネル名です。

リスト17-17 - インストールされたプラグインの一覧

    > cd myproject
    > php symfony plugin:list

    Installed plugins:
    sfPrototypePlugin               1.0.0-stable # plugins.symfony-project.com (symfony)
    sfSuperCachePlugin              1.0.0-stable # plugins.symfony-project.com (symfony)
    sfThumbnail                     1.1.0-stable # plugins.symfony-project.com (symfony)

#### プラグインのアップグレードとアンインストール

PEARのプラグインをアンインストールするには、リスト17-18で示されるように、プロジェクトのrootディレクトリから`plugin:uninstall`タスクを呼び出します。プラグインの名前にプラグインをインストールしたチャンネル名をプレフィックスとして追加しなければなりません(このチャンネルを決めるために`plugin:list`タスクを使います)。

リスト17-18 - プラグインをアンインストールする

    > cd myproject
    > php symfony plugin:uninstall sfPrototypePlugin
    > php symfony cc

アーカイブからインストールしたプラグインもしくはSVNリポジトリからインストールしたプラグインをアンインストールするには、プロジェクトの`plugins/`と`web/`ディレクトリからプラグインのファイルを手動で削除してキャッシュをクリアします。

プラグインをアップグレードするには、`plugin:upgrade`タスク(PEARプラグインの場合)もしくは`svn update`を実行します(バージョン管理システムのリポジトリからプラグインを入手した場合)。アーカイブからインストールしたプラグインは簡単にアップグレードできません。

### プラグインの分析

プラグインはPHPで書かれています。アプリケーションの編成方法を理解しているのであれば、プラグインの構造を理解できます。

#### プラグインのファイル構造

プラグインのディレクトリはおおよそプロジェクトのディレクトリと同じように編成されています。必要な時にsymfonyによって自動的にロードされるようにするためにプラグインファイルは正しいディレクトリに存在しなければなりません。ファイル構造の記述に関してはリスト17-19をご覧ください。

リスト17-19 - プラグインのファイル構造

    pluginName/
      config/
        *schema.yml        // データスキーマ
        *schema.xml
        config.php         // 特定のプラグイン設定
      data/
        generator/
          sfPropelAdmin
            */             // administrationジェネレーターテーマ
              template/
              skeleton/
        fixtures/
          *.yml            // フィクスチャファイル
      lib/
        *.php              // クラス
        helper/
          *.php            // ヘルプ
        model/
          *.php            // モデルクラス
        task/
          *Task.class.php  // CLIタスク
      modules/
        */                 // モジュール
          actions/
            actions.class.php
          config/
            module.yml
            view.yml
            security.yml
          templates/
            *.php
          validate/
            *.yml
      web/
        *                  // アセット

#### プラグインの機能

プラグインは多くのものを含みます。コマンドラインでタスクを呼び出すときに実行中のアプリケーションはこれらの内容を自動的に考慮します。しかしプラグインを適切に機能させるには、いくつかの規約を遵守しなければなりません:

  * データベースのスキーマは`propel-`タスクによって検出されます。`propel-build-model`タスクを呼び出すと、プロジェクトモデルとすべてのプラグインモデルがリビルドされます。リスト17-20で示されるように、 プラグインスキーマはつねに`plugins.pluginName.lib.model`形式で`package`属性を持つことに注意してください。

リスト17-20 - スキーマ宣言の例(`myPlugin/config/schema.yml`)

    propel:
      _attributes:    { package: plugins.myPlugin.lib.model }
      my_plugin_foobar:
        _attributes:    { phpName: myPluginFoobar }
          id:
          name:           { type: varchar, size: 255, index: unique }
          ...

  * プラグインの設定はプラグインのブートストラップスクリプト(`config.php`)に含まれています。このファイルはアプリケーションとプロジェクト設定のあとで実行されるので、symfonyはその時点ですでに起動しています。たとえば、既存のクラスをイベントリスナーもしくはビヘイビアーで拡張するためです。
  * プラグインの`data/fixtures/`ディレクトリに設置されたフィクスチャファイルは`propel:load-data`タスクで処理されます。
  * プロジェクトの`lib/`フォルダーに設置されたクラスのようにカスタムクラスはオートロードされます。
  * テンプレートのなかで`use_helper()`ヘルパーを呼び出すときにヘルパーは自動的に発見されます。これらはプラグインの`lib/`ディレクトリの1つの`helper/`サブディレクトリに存在しなければなりません。
  * `myplugin/lib/model/`ディレクトリ内のモデルクラスは(`myplugin/lib/model/om/`ディレクトリと`myplugin/lib/model/map/`ディレクトリ)内部のPropelビルダによって生成されたモデルクラスを専門に扱います。もちろんこれらもオートロードされます。独自プロジェクトのディレクトリ内で生成されたプラグインのモデルクラスはオーバーライドできません。
  * プラグインをインストールすればタスクはsymfonyコマンドですぐに利用できます。プラグインは新しいタスクを追加もしくは既存のものをオーバーライドできます。タスク用にプラグインの名前を名前空間として使うことは最良の習慣です。プラグインに追加されたものを含めて、利用可能なタスクの一覧を見るには、`php symfony`を入力してください。
  * アプリケーションの`enabled_modules`設定で宣言すれば、モジュールは外部からアクセス可能な新しいアクションを提供します。
  * サーバーはWebアセット(イメージ、スクリプト、スタイルシートなど)を利用できます。コマンドライン経由でプラグインをインストールしたとき、システムが許可するのであればsymfonyはプロジェクトの`web/`ディレクトリにシンボリックリンクを作るもしくは`web/`ディレクトリの内容をプロジェクトのディレクトリにコピーします。プラグインがアーカイブもしくはバージョン管理ツールのリポジトリからインストールされた場合、手動で`web/`ディレクトリにコピーしなければなりません(プラグインに添付されている`README`に記載されています)。

>**TIP** **ルーティングルールをプラグインに登録する**
>
>プラグインはルーティングシステムに新しいルールを追加できますが、そのためにカスタム`routing.yml`設定ファイルを使用できません。これはルールが定義された順序が非常に重要で、symfonyのYAMLファイルのシンプルなカスケード設定システムはこのジョン所をごちゃごちゃにするからです。代わりにプラグインはイベントリスナーを`routing.load_configuration`イベントに登録してリスナーの先頭にルールを手動で追加する必要があります:
>
>     [php]
>     // plugins/myPlugin/config/config.phpのなか
>     $this->dispatcher->connect('routing.load_configuration', array('myPluginRouting', 'listenToRoutingLoadConfigurationEvent'));
>     
>     // plugins/myPlugin/lib/myPluginRouting.phpのなか
>     class myPluginRouting
>     {
>       static public function listenToRoutingLoadConfigurationEvent(sfEvent $event)
>       {
>         $routing = $event->getSubject();
>         // 既存のルーティングルールの上にプラグインのルールを追加する
>         $routing->prependRoute('my_route', '/my_plugin/:action', array('module' => 'myPluginAdministrationInterface'));
>       }
>     }
>

#### 手動によるプラグインのセットアップ

`plugin:install`タスクが独自に処理できない要素がいくつかあります。インストール作業の間にこれらを手動でセットアップする必要があります:

  * カスタムアプリケーション設定はプラグインのコードで使われますが(たとえば、`sfConfig::get('app_myplugin_foo')`を利用する)、デフォルト値をプラグインの`config/`ディレクトリに設置された`app.yml`ファイルに設定できません。デフォルト値を処理するには、`sfConfig::get()`メソッドの2番目の引数を使います。設定はまだアプリケーションレベルでオーバーライドできます(リスト17-26で例をご覧ください)。
  * カスタムルーティングルールはアプリケーションの`routing.yml`に手動で追加しなければなりません。
  * カスタムフィルターはアプリケーションの`filters.yml`に手動で追加しなければなりません。
  * カスタムファクトリはアプリケーションの`factories.yml`に手動で追加しなければなりません。

一般的に言えば、アプリケーションの設定ファイルの1つに帰結するようなすべての設定は手動で追加しなければなりません。このような手動のセットアップが必要なプラグインは`README`ファイルで詳細なインストール方法を説明しています。

#### アプリケーションのためにプラグインをカスタマイズする

プラグインをカスタマイズしたいときは、決して`plugins/`ディレクトリ内で見つかるコードを変更してはなりません。これを行うと、プラグインをアップグレードするときにすべての修正内容が失われてしまいます。必要なカスタマイズを行うために、プラグインはカスタム設定を提供し、オーバーライドをサポートします。

リスト17-21で示されるように、よく設計されたプラグインはアプリケーションの`app.yml`ファイルで変更できる設定を利用します。

リスト17-21 - アプリケーションの設定を利用するプラグインをカスタマイズする

    [php]
    // プラグインのコードの例
    $foo = sfConfig::get('app_my_plugin_foo', 'bar');

    // アプリケーションのapp.ymlで'foo'のデフォルト値('bar')を変更する
    all:
      my_plugin:
        foo:       barbar

モジュールの設定とデフォルト値はプラグインの`README`ファイルで詳しく説明されています。

独自のアプリケーション内部で同じ名前のモジュールを作成することでプラグインモジュールのデフォルトの内容を置き換えることができます。プラグイン要素の代わりにアプリケーション要素が使われているので、本当のオーバーライドではありません。プラグインの名前と同じ名前のテンプレートと設定ファイルを作ればプラグインモジュールは立派に機能します。

一方で、アクションをオーバーライドする機能を持つモジュールをプラグインに持たせたい場合、プラグインモジュール内の`actions.class.php`のメソッドがアプリケーションモジュールの`actions.class.php`によって継承できるように、`actions.class.php`は空でなければならずオートロードクラスから継承しなければなりません。お手本に関してはリスト17-22を参照してください。

リスト17-22 - プラグインのアクションをカスタマイズする

    [php]
    // myPlugin/modules/mymodule/lib/myPluginmymoduleActions.class.phpのなか
    class myPluginmymoduleActions extends sfActions
    {
      public function executeIndex()
      {
        // ここに何らかのコード
      }
    }

    // myPlugin/modules/mymodule/actions/actions.class.phpにて

    require_once dirname(__FILE__).'/../lib/myPluginmymoduleActions.class.php';

    class mymoduleActions extends myPluginmymoduleActions
    {
      // なし
    }

    // frontend/modules/mymodule/actions/actions.class.phpにて
    class mymoduleActions extends myPluginmymoduleActions
    {
      public function executeIndex()
      {
        // ここでプラグインのコードをオーバーライドする
      }
    }

>**SIDEBAR**
>**symfony 1.1の新しい機能**: プラグインのスキーマをカスタマイズする
>
>モデルをビルドするとき、symfonyは、つぎのルールにしたがって、プラグインのものを含めて、それぞれの既存のスキーマのためにカスタムYAMLファイルを探します:
>
>オリジナルのスキーマ名                  | カスタムスキーマ名
>-------------------------------------- | ------------------------------
>config/schema.yml                      | schema.custom.yml
>config/foobar_schema.yml               | foobar_schema.custom.yml
>plugins/myPlugin/config/schema.yml     | myPlugin_schema.custom.yml
>plugins/myPlugin/config/foo_schema.yml | myPlugin_foo_schema.custom.yml
>
>カスタムスキーマはアプリケーションとプラグインの`config/`ディレクトリを探すので、プラグインは別のプラグインのスキーマをオーバーライドをして、スキーマ単位で複数のカスタマイズが可能です。
>
>symfonyはそれぞれのテーブルの`phpName`に基づいて2つのスキーマをマージします。マージ処理によってテーブル、カラム、カラムの属性の追加もしくは修正できます。たとえば、つぎの一覧はカスタムスキーマがカラムをプラグインのスキーマで定義されたテーブルに追加する方法を示しています。
>
>     # オリジナルのスキーマ(plugins/myPlugin/config/schema.yml)
>     propel:
>       article:
>         _attributes:    { phpName: Article }
>         title:          varchar(50)
>         user_id:        { type: integer }
>         created_at:
>
>     # カスタムスキーマ(myPlugin_schema.custom.yml)
>     propel:
>       article:
>         _attributes:    { phpName: Article, package: foo.bar.lib.model }
>         stripped_title: varchar(50)
>
>     # スキーマの結果、内部でマージされモデルとSQLの生成用に内部で使われる
>     propel:
>       article:
>         _attributes:    { phpName: Article, package: foo.bar.lib.model }
>         title:          varchar(50)
>         user_id:        { type: integer }
>         created_at:
>         stripped_title: varchar(50)
>
>マージ処理はテーブルの`phpName`をキーとして使うので、スキーマのなかで同じ`phpName`を保つのであれば、データベース内のプラグインテーブルの名前を変更できます。

### プラグインの書き方

`plugin:install`タスクではPEARパッケージ形式のプラグインのみがインストールされます。このようなプラグインは公式サイトの専用ページ、PEARチャンネル経由もしくはダウンロードできる通常のファイルとして配布されていることを覚えておいてください。プラグインを編集したい場合は、単純なアーカイブよりもPEARパッケージとして公開したほうがベターでしょう。加えて、プラグインをPEARパッケージにすればアップグレード作業が簡単になり、依存関係の宣言が可能で、自動的にアセットを`web/`ディレクトリにデプロイできます。

#### ファイルのコンフィギュレーション

新しい機能を開発し、プラグインとしてパッケージにすることを考えてみましょう。最初の段階はファイルを論理的に編成して、symfonyのロードメカニズムが必要なときにこれらのファイルを見つけることができるようにしましょう。この目的のために、リスト17-19で示されているディレクトリ構造に従う必要があります。リスト17-23は`sfSamplePlugin`プラグインのためのファイル構造の例を示しています。

リスト17-23 - プラグインとしてパッケージにするファイルの一覧の例

    sfSamplePlugin/
      README
      LICENSE
      config/
        schema.yml
      data/
        fixtures/
          fixtures.yml
      lib/
        model/
          sfSampleFooBar.php
          sfSampleFooBarPeer.php
        task/
          sfSampleTask.class.php
        validator/
          sfSampleValidator.class.php
      modules/
        sfSampleModule/
          actions/
            actions.class.php
          config/
            security.yml
          lib/
            BasesfSampleModuleActions.class.php
          templates/
            indexSuccess.php
      web/
        css/
          sfSampleStyle.css
        images/
          sfSampleImage.png

編集に関して、プラグインのディレクトリの位置(リスト17-23の`sfSamplePlugin/`)は重要ではありません。これはディスク上の任意の場所に設置できます。

>**TIP**
>既存のプラグインを練習問題として考え、初めてプラグインを作るさいには、これらの名前の規約とファイルの構造を再現してみてください。

#### package.xmlファイルを作る

プラグイン編集のつぎの段階はプラグインディレクトリのrootで`package.xml`ファイルを追加することです。`package.xml`はPEARの構文に従います。リスト17-24の典型的なsymfonyプラグインの`package.xml`をご覧ください。

リスト17-24 - symfonyプラグイン用の`package.xml`

    [xml]
    <?xml version="1.0" encoding="UTF-8"?>
    <package packagerversion="1.4.6" version="2.0" xmlns="http://pear.php.net/dtd/package-2.0" xmlns:tasks="http://pear.php.net/dtd/tasks-1.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://pear.php.net/dtd/tasks-1.0 http://pear.php.net/dtd/tasks-1.0.xsd http://pear.php.net/dtd/package-2.0 http://pear.php.net/dtd/package-2.0.xsd">
     <name>sfSamplePlugin</name>
     <channel>plugins.symfony-project.org</channel>
     <summary>symfony sample plugin</summary>
     <description>Just a sample plugin to illustrate PEAR packaging</description>
     <lead>
      <name>Fabien POTENCIER</name>
      <user>fabpot</user>
      <email>fabien.potencier@symfony-project.com</email>
      <active>yes</active>
     </lead>
     <date>2006-01-18</date>
     <time>15:54:35</time>
     <version>
      <release>1.0.0</release>
      <api>1.0.0</api>
     </version>
     <stability>
      <release>stable</release>
      <api>stable</api>
     </stability>
     <license uri="http://www.symfony-project.org/license">MIT license</license>
     <notes>-</notes>
     <contents>
      <dir name="/">
       <file role="data" name="README" />
       <file role="data" name="LICENSE" />
       <dir name="config">
        <!-- model -->
        <file role="data" name="schema.yml" />
       </dir>
       <dir name="data">
        <dir name="fixtures">
         <!-- fixtures -->
         <file role="data" name="fixtures.yml" />
        </dir>
       </dir>
       <dir name="lib">
        <dir name="model">
         <!-- model classes -->
         <file role="data" name="sfSampleFooBar.php" />
         <file role="data" name="sfSampleFooBarPeer.php" />
        </dir>
        <dir name="task">
         <!-- tasks -->
         <file role="data" name="sfSampleTask.class.php" />
        </dir>
        <dir name="validator">
         <!-- validators -->
         <file role="data" name="sfSampleValidator.class.php" />
        </dir>
       </dir>
       <dir name="modules">
        <dir name="sfSampleModule">
         <file role="data" name="actions/actions.class.php" />
         <file role="data" name="config/security.yml" />
         <file role="data" name="lib/BasesfSampleModuleActions.class.php" />
         <file role="data" name="templates/indexSuccess.php" />
        </dir>
       </dir>
       <dir name="web">
        <dir name="css">
         <!-- stylesheets -->
         <file role="data" name="sfSampleStyle.css" />
        </dir>
        <dir name="images">
         <!-- images -->
         <file role="data" name="sfSampleImage.png" />
        </dir>
       </dir>
      </dir>
     </contents>
     <dependencies>
      <required>
       <php>
        <min>5.1.0</min>
       </php>
       <pearinstaller>
        <min>1.4.1</min>
       </pearinstaller>
       <package>
        <name>symfony</name>
        <channel>pear.symfony-project.com</channel>
        <min>1.1.0</min>
        <max>1.2.0</max>
        <exclude>1.2.0</exclude>
       </package>
      </required>
     </dependencies>
     <phprelease />
     <changelog />
    </package>

ここで注目すべき部分は`<contents>`タグと`<dependencies>`タグで、つぎに説明します。残りのタグに関しては、symfony固有のものではありませんので、`package.xml`フォーマットに関する詳細な内容はPEARオンラインマニュアル([http://pear.php.net/manual/](http://pear.php.net/manual/)) を参照してください。

#### 内容

`<contents>`タグはプラグインのファイル構造を記述しなければならない場所です。このタグはコピーするファイルとその場所をPEARに伝えます。`<dir>`タグと`<file>`タグでファイル構造を記述してください。すべての`<file>`タグは`role="data"`属性を持たなければなりません。リスト17-24の`<contents>`タグの部分はリスト17-23の正しいディレクトリ構造を記載しています。

>**NOTE**
>`<dir>`タグの使用は義務ではありません。`<file>`タグ内で相対パスを`name`の値として利用できるからです。`package.xml`ファイルを読みやすくするためにお勧めです。

#### プラグインの依存関係

任意のバージョンのPHP、PEAR、symfony、PEARパッケージ、もしくはほかのプラグインの一式で動くようにプラグインは設計されています。`<dependencies>`タグでこれらの依存関係を宣言すれば必要なパッケージがすでにインストールされていることを確認してそうでなければ例外を起動するようPEARに伝えることになります。

最小要件として、少なくとも開発環境に対応したPHP、PEARとsymfonyへの依存関係をつねに宣言します。何を追加すればよいのかわからなければ、PHP 5.1、PEAR 1.4とsymfony 1.0の要件を追加してください。

それぞれのプラグインに対してsymfonyの最大のバージョン番号を追加することも推奨されます。これによって上位バージョンのsymfonyでプラグインを使うときにエラーメッセージが表示され、プラグインを再リリースするまえにこのバージョンでプラグインが正しく動作するのかを確認することをプラグインの作者に義務づけます。無言でプラグインの動作が失敗するよりも警告を発してダウンロードとアップグレードするほうがベターです。

プラグインを依存関係のあるものとして指定すれば、ユーザーはプラグインとすべての依存関係を1つのコマンドでインストールできるようになります:

    > php symfony plugin:install --install-deps sfSamplePlugin

#### プラグインをビルドする

PEARコンポーネントはパッケージの`.tgz`アーカイブを作るコマンド(`pear package`)を持ちます。リスト17-25では、`package.xml`を含むディレクトリでこのコマンドを呼び出しています。

リスト17-25 - プラグインをPEARパッケージにする

    > cd sfSamplePlugin
    > pear package

    Package sfSamplePlugin-1.0.0.tgz done

いったんプラグインのパッケージがビルドされたら、リスト17-26で示されるように、あなたの環境にこれをインストールして動作を確認してください。

リスト17-26 - プラグインをインストールする

    > cp sfSamplePlugin-1.0.0.tgz /home/production/myproject/
    > cd /home/production/myproject/
    > php symfony plugin:install sfSamplePlugin-1.0.0.tgz

`<contents>`タグにある説明にしたがって、パッケージにされたファイルは最終的にプロジェクトの異なるディレクトリに設置されます。リスト17-27はインストールのあとで`sfSamplePlugin`のファイルが設置される場所を示しています。

リスト17-27 - プラグインファイルは`plugin/`ディレクトリと`web/`ディレクトリにインストールされる

    plugins/
      sfSamplePlugin/
        README
        LICENSE
        config/
          schema.yml
        data/
          fixtures/
            fixtures.yml
        lib/
          model/
            sfSampleFooBar.php
            sfSampleFooBarPeer.php
          task/
            sfSampleTask.class.php
          validator/
            sfSampleValidator.class.php
        modules/
          sfSampleModule/
            actions/
              actions.class.php
            config/
              security.yml
            lib/
              BasesfSampleModuleActions.class.php
            templates/
              indexSuccess.php
    web/
      sfSamplePlugin/               ## システム次第で、コピーもしくはシンボリックリンク
        css/
          sfSampleStyle.css
        images/
          sfSampleImage.png

このプラグインのふるまいをアプリケーションでテストしてください。きちんと動くのであれば、プラグインを複数のプロジェクトにまたがって配布するもしくはsymfonyコミュニティに寄付する準備ができています。

#### 公式サイトでプラグインを配布する

symfonyのプラグインは以下の手順にしたがって`symfony-project.org`のWebサイトで配布されるときにもっとも幅広い利用者を得ます。独自プラグインをつぎのような方法で配布できます:

  1. `README`ファイルにプラグインのインストール方法と使いかたが、`LICENSE`ファイルにはライセンスの詳細が記述されていることを確認する。`README`はMarkdownの構文 ([http://daringfireball.net/projects/markdown/syntax](http://daringfireball.net/projects/markdown/syntax)) で記述する。
  2. 公式サイトのアカウント (http://www.symfony-project.org/user/new) を作りプラグインのページ (http://www.symfony-project.org/plugins/new) を作る。
  3. `pear package`コマンドを呼び出してプラグイン用のPEARパッケージを作りテストする。PEARパッケージの名前は`sfSamplePlugin-1.0.0.tgz` (1.0.0はプラグインのバージョン)でなければならない。
  4. PEARパッケージをアップロードする (`sfSamplePlugin-1.0.0.tgz`)。
  5. アップロードしたプラグインは一覧ページ ([http://www.symfony-project.org/plugins/](http://www.symfony-project.org/plugins/)) に表示される。

この手続きを行えば、ユーザーはプロジェクトのディレクトリでつぎのコマンドを入力するだけでプラグインをインストールできるようになります:

    > php symfony plugin:install sfSamplePlugin

#### 命名規約

`plugin/`ディレクトリをきれいに保つために、すべてのプラグインの名前がcamelCaseであり`Plugin`のサフィックスで終わることを確認してください(たとえば、`shoppingCartPlugin`、`feedPlugin`)。プラグインに名前をつけるまえに、同じ名前のプラグインが存在しないことを確認してください。

>**NOTE**
>Propelに依存するプラグインの名前は`Propel`を含みます。たとえば、Propelのデータアクセスオブジェクトを利用する認証プラグインは`sfPropelAuth`という名前になります。

プラグインには使用条件と選んだライセンスを説明する`LICENSE`ファイルをつねに含めなければなりません。バージョンの履歴、プラグインの目的、効果、インストールと設定の手引きなどを含めることも推奨されます。

まとめ
----

symfonyのクラスはアプリケーションレベルで修正できる機能を提供する`sfMixer`フックを含みます。ミックスイン(mixin)のメカニズムはPHPの制約が禁止している実行時のクラスの多重継承とオーバーライドを可能にします。ですのでそのためにコアクラスを修正しなければならないとしても、またファクトリ(factory)の設定がそこに存在するとしてもsymfonyの機能を簡単に拡張できます。

すでに多くの拡張機能(エクステンション)が存在し、プラグインとしてパッケージが作成されています。symfonyのコマンドラインによってインストール、アップグレード、アンインストールするのが簡単です。プラグインをPEARパッケージを作成するのと同じぐらい簡単で、複数のアプリケーションをまたがって再利用できます。

symfony公式サイトのwikiには多くのプラグインが含まれ、あなた自身のプラグインも追加できます。これであなたは方法を理解したので、私たちsymfonyの開発者はあなたが多くの便利な拡張機能でsymfonyコアを強化して下さることを望んでおります！
