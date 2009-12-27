第19章 - symfonyの設定ファイルをマスターする
============================================

現在あなたはsymfonyをとてもよく理解しています。すでにコアの設計を理解し、新しい隠し機能を見つけるためにコードを徹底的に調べる準備ができています。しかし、独自要件に適用するためにsymfonyのクラスを拡張するまえに、設定ファイルをもっとよく見ることが必要です。すでに多くの機能はsymfonyに組み込まれ、設定を少し変更すれば有効になります。このことはクラスをオーバーライドしなくてもsymfonyコアのふるまいを調整できることを意味します。この章では設定ファイルとこれらの強力な機能を詳しく説明します。

symfonyの設定
-------------

`frontend/config/settings.yml`ファイルはおもに`frontend`アプリケーション用のsymfonyの設定を含みます。前の章でこのファイルから多くの設定の機能をすでに見てきましたが、再度これらを見ることにします。

5章で説明したように、このファイルは環境に依存します。すなわちそれぞれの設定が環境ごとに異なる値をとります。このファイルで定義されたそれぞれのパラメーターは`sfConfig`クラスを通してPHPクラスの内部からアクセスできることを覚えておいてください。パラメーター名は設定名にプレフィックスの`sf_`をつけたものです。たとえば、`cache`パラメーターの値を取得したい場合、必要なことは`sfConfig::get('sf_cache')`を呼び出すだけです。

### デフォルトのモジュールとアクション

symfonyは特殊な状況に対してデフォルトページを用意します。ルーティングエラーの場合、symfonyは`default`モジュールのアクションを実行します。このモジュールは`$sf_symfony_lib_dir/controller/default/`ディレクトリに保存されています。`settings.yml`ファイルはエラーごとに実行されるアクションを定義します:

  * `error_404_module`と`error_404_action`: ユーザーによって入力されたURLがどのrouteにもマッチしないもしくは`sfError404Exception`が起動するときに呼び出されるアクションです。デフォルト値は`default/error404`です。
  * `login_module`と`login_action`: `security.yml`ファイルのなかで`secure`と定義されたページに認証されていないユーザーがアクセスしようとするときに呼び出されるアクション(詳細は6章を参照)。デフォルト値は`default/login`です。
  * `secure_module`と`secure_action`: ユーザーがアクションから求められるクレデンシャルを持たないときに呼び出されるアクションです。デフォルト値は`default/secure`です。
  * `module_disabled_module`と`module_disabled_action`: ユーザーが`module.yml`ファイルのなかで無効と宣言されたモジュールをリクエストしたときに呼び出されるアクション。デフォルト値は`default/disabled`です。

運用サーバーにデプロイするまえにこれらのアクションはアプリケーションをカスタマイズすべきです。`default`モジュールのテンプレートではページにsymfonyのロゴが含まれるからです。これらのページの1つのスクリーンショット、404エラーのページは図19-1をご覧ください。

図19-1 - デフォルトの404エラーページ

![デフォルトの404エラーページ](http://github.com/masakielastic/symfonybook-ja/raw/master/images/F1901.jpg "デフォルトの404エラーページ")

以下の2つの方法でデフォルトのページをオーバーライドできます:

  * アプリケーションの`modules/`ディレクトリ内で独自のデフォルトモジュールを作成し、`settings.yml`ファイルで定義されたすべてのアクション(`index`、`error404`、`login`、`secure`、`disabled`)と関連するテンプレート(`indexSuccess.php`、`error404Success.php`、`loginSuccess.php`、 `secureSuccess.php`、`disabledSuccess.php`)をオーバーライドします。
  * アプリケーションのページを使うために`settings.yml`ファイルのデフォルトのモジュールとアクションの設定を変更できます。

ほかの2つのページはsymfonyの見た目を有しており、運用サーバーにデプロイするまえにこれらのページはカスタマイズすることも必要です。これらのページは`default`モジュールには存在しません。これはsymfonyが適切に動作しないときに呼び出されるからです。代わりに、これらのデフォルトページは`$sf_symfony_lib_dir/exception/data/`ディレクトリで見つかります:

  * `error500.php`: 運用環境で内部のサーバーエラーが起きるときに呼び出されるページ。ほかの環境(`debug`が`true`に設定されている)においてエラーが起きるとき、symfonyはすべての実行スタックと明快なエラーメッセージを表示します(詳細は16章を参照)。
  * `unavailable.php`: アプリケーションが(`disable`タスクで)無効にされている間にユーザーがページをリクエストしたときに呼び出されるページ。このページはキャッシュがクリアされている間も呼び出されます(すなわち、`php symfony cache:clear`タスクの呼び出しとこのタスクの終了時の間)。とても大きなキャッシュを持つシステム上では、キャッシュのクリア処理は数秒かかる可能性があります。symfonyは部分的にクリアしたキャッシュでリクエストを実行できないので、処理が終わるまえに受理されたリクエストはこのページにリダイレクトされます。

これらのページをカスタマイズするには、プロジェクトもしくはアプリケーションの`web/errors/`ディレクトリのなかで`error500.php`ページと`unavailable.php`ページを作ります。symfonyは固有のページの代わりにこれらを使うようになります。

>**NOTE**
>必要なときにリクエストを`unavailable.php`ページにリダイレクトするには、アプリケーションの`settings.yml`のなかで`check_lock`設定を`on`にする必要があります。デフォルトではこの設定は無効です。この設定によってすべてのリクエストに対してわずかですがオーバーヘッドが追加されるからです。

### オプション機能の有効

`settings.yml`ファイルのパラメーターのなかには有効もしくは無効にできるsymfonyのオプション機能をコントロールするものがあります。使わない機能を無効にすることでパフォーマンスが少し押し上げられるので、アプリケーションをデプロイするまえにテーブル19-1に示されている設定の一覧を見直してください。

テーブル 19-1 - `settings.yml`ファイルを通したオプション機能の一式

パラメーター              | 説明        | デフォルト値
----------------------- | ----------- | -------------
`use_database`          | データベースマネージャを有効にする。データベースを使わない場合は`off`に切り替える。 | `on`
`i18n`                  | インターフェイスの翻訳機能を有効にする(13章を参照)。他言語アプリケーションのために`on`に設定する。 | `off`
`logging_enabled`       | symfonyのイベントのロギングを有効にする。symfonyのロギング機能を完全にオフにしたい場合はoffに設定する。 | `on`
`escaping_strategy`     | 出力エスケーピング機能を有効にする(7章を参照)。テンプレートに渡すデータをエスケープしたい場合は`on`に設定する。| `off`
`cache`                 | テンプレートキャッシュを有効にする(12章を参照)。モジュールの1つが`cache.yml`ファイルを含む場合に`on`に設定する。キャッシュフィルター(`sfCacheFilter`)が`on`になっている場合にのみ有効 | 開発環境では`off`、運用環境では`on`
`web_debug`             | 簡単なデバッグのためのWebデバッグツールバーを有効にする(16章を参照)。ツールバーをすべてのページに表示するには`on`に設定する。| 開発環境では`on`、運用環境では`off`
`check_symfony_version` | すべてのリクエストに対してsymfonyのバージョンチェックを有効にする。symfonyをアップグレードした後に自動的にキャッシュをクリアするために`on`に設定する。アップグレードした後につねにキャッシュをクリアする場合は`off`のままにしておく。|`off`
`check_lock`            | アプリケーションのロックシステムを有効にする。`cache:clear`と`project:disable`タスクによって起動する(以前のセクションを参照)。`$sf_symfony_lib_dir/exception/data/unavailable.php`のページにリダイレクトするように無効なアプリケーションにリクエストを行うために`on`に設定する。| `off`
`compressed`            | PHPのレスポンス圧縮機能を有効にする。PHP圧縮ハンドラー経由で出力するHTMLを圧縮するには`on`にする。 | `off`

### 機能のコンフィギュレーション

symfonyは組み込み機能、たとえば、バリデーション、キャッシュ、サードパーティのモジュールなどのふるまいを変更するにはいくつかの`settings.yml`ファイルのパラメーターを使います。

#### 出力エスケーピングの設定

出力エスケーピングの設定は変数がテンプレートにアクセスする方法をコントロールします(7章を参照)。`settings.yml`ファイルはこの機能のために2つの設定を含みます:

  * `escaping_strategy`設定は`on`もしくは`off`の値を取ることができます。
  * `escaping_method`設定は`ESC_RAW`、`ESC_SPECIALCHARS`、 `ESC_ENTITIES`、`ESC_JS`、もしくは`ESC_JS_NO_ENTITIES`の値に設定できます。

#### ルーティングの設定

ルーティングの設定(9章を参照)は`factories.yml`ファイルのなかの`routing`キーの下で定義されます。リスト19-1はルーティングのデフォルトコンフィギュレーションを示しています。

リスト19-1 - ルーティングコンフィギュレーションの設定(`frontend/config/factories.yml`)

    routing:
      class: sfPatternRouting
      param:
        load_configuration: true
        suffix:             .
        default_module:     default
        default_action:     index
        variable_prefixes:  [':']
        segment_separators: ['/', '.']
        variable_regex:     '[\w\d_]+'
        debug:              %SF_DEBUG%
        logging:            %SF_LOGGING_ENABLED%
        cache:
          class: sfFileCache
          param:
            automatic_cleaning_factor: 0
            cache_dir:                 %SF_CONFIG_CACHE_DIR%/routing
            lifetime:                  31556926
            prefix:                    %SF_APP_DIR%

  * `suffix`パラメーターは生成されたURLのためにデフォルトのサフィックスを設定します。デフォルト値はピリオド(`.`)で、どの接尾辞にも一致しません。たとえば、すべての生成されたURLが静的なページに見えるように`.html`に設定します。
  * ルーティングルールが`module`パラメーターもしくは`action`パラメーターを定義しないとき、代わりに`factories.yml`ファイルからの値が使われます:
    * `default_module`: デフォルトの`module`リクエストパラメーター。デフォルトは`default`モジュール。
    * `default_action`: デフォルトの`action`リクエストパラメーター。デフォルトは`index`アクション。
  * デフォルトでは、routeのパターンはコロンの(`:`)のプレフィックスによって名前つきのワイルドカードを識別します。PHPによりフレンドリな構文でルールを書きたいのであれば、ドル記号(`$`)を`variable_prefixes`配列に追加できます。この方法では、'/article/:year/:month/:day/:title'の代わりに'/article/$year/$month/$day/$title'のようなパターンを書けます。
  * パターンのルーティングは区切り文字の間の名前つきのワイルドカードを見分けます。デフォルトの区切り文字はスラッシュとドットですが、望むのであれば`segment_separators`パラメーターにより多くの区切り文字を追加できます。たとえば、ダッシュ(`-`)を追加したい場合、'/article/:year-:month-:day/:title'のようなパターンを書けます。
  * 運用モードにおいて、外部URLと内部URIの間の変換を加速するために、パターンのルーティングは独自のキャッシュを使います。デフォルトでは、このキャッシュはファイルシステムを利用しますが、クラスと設定を`cache`パラメーターで宣言すれば任意のキャッシュクラスを使用できます。利用可能なキャッシュストレージクラスのリストに関しては15章を参照してください。運用環境でルーティングキャッシュを無効にするには、`debug`パラメーターを`on`に設定します。

`sfPatternRouting`クラス専用の設定があります。アプリケーションのルーティング、独自もしくはsymfonyのルーティングファクトリ(`sfNoRouting`と`sfPathInfoRouting`)の1つのどちらかに対して別のクラスを利用できます。これらの2つのどちらかを用いることで、すべての外部URLは'module/action?key1=param1'のようになります。カスタマイズする必要はありませんが、速いです。違いは最初のものはPHPの`GET`を使用し、2番目は`PATH_INFO`を使います。おもにバックエンドのインターフェイスにこれらを使います。

ルーティングに関連する追加パラメーターが1つありますが、これは `settings.yml`ファイルに保存されます:

  * `no_script_name`設定は生成されたURLのなかでフロントコントローラーを有効にします。フロントコントローラーをさまざまなディレクトリに保存してデフォルトのURL書き換えルールを変更しないかぎり、`no_script_name`設定はプロジェクト内の単独のアプリケーションに対してのみonです。通常この設定は運用環境のメインアプリケーションに対してonでその他に対してoffです。

#### フォームバリデーションの設定

>**NOTE**
>このセクションで説明されている機能はsymfonyのバージョン1.1では廃止され`sfCompat10`プラグインを有効にしている場合のみに動作します。

フォームバリデーションの設定は`Validation`ヘルパーによるエラーメッセージが表示される方法をコントロールします(10章を参照)。これらのエラーは`<div>`に含まれ、`id`属性を形成するためにこれらのエラーは`validation_error_class`設定と`validation_error_id_prefix`設定を`class`属性として使います。デフォルト値は`form_error`と`error_for_`なので、`foobar`という名前の入力に対して`form_error()`ヘルパーへの呼び出しによる属性出力は`class="form_error" id="error_for_foobar"`になります。

2つの設定はそれぞれのエラーメッセージ: `validation_error_prefix`と`validation_error_suffix`の前後に来る文字を決定します。一度にすべてのエラーメッセージをカスタマイズするためにこれらの設定を変更できます。

#### キャッシュの設定

キャッシュ設定の大部分は`cache.yml`ファイルで定義されますが、`settings.yml`ファイルのなかの2つは異なります: `cache`はテンプレートキャッシュのメカニズムを有効にし、`etag`はサーバーサイド上のETagハンドリングを有効にします(15章を参照)。`factories.yml`ファイルのなかで2つのすべてのキャッシュシステム(ビューキャッシュ、ルーティングキャッシュと、国際化キャッシュ)に対してどのストレージを使うのかを指定することもできます。リスト19-2はビューのキャッシュファクトリのデフォルトコンフィギュレーションを示しています。

リスト19-2 - ビューのキャッシュコンフィギュレーション(`frontend/config/factories.yml`)

    view_cache:
      class: sfFileCache
      param:
        automatic_cleaning_factor: 0
        cache_dir:                 %SF_TEMPLATE_CACHE_DIR%
        lifetime:                  86400
        prefix:                    %SF_APP_DIR%/template

`class`の値は`sfFileCache`、`sfAPCCache`、`sfEAcceleratorCache`、`sfXCacheCache`、`sfMemcacheCache`、と `sfSQLiteCache`のどれかになります。`sfCache`を継承しキャッシュのキーの読みとりと削除をする、設定用の同じ一般的なメソッドを提供するのであれば、独自クラスも可能です。ファクトリのパラメーターは選んだクラスに依存しますが、定数が存在します:

  * `lifetime`はキャッシュが削除されるまでの秒数を定義します
  * `prefix`はすべてのキャッシュキーに追加されるプレフィックスです(環境によって異なるキャッシュを利用するためにプレフィックスの環境を使う)。2つのアプリケーションのあいだでキャッシュを共有させたい場合は同じプレフィックスを使います。


それぞれの特定のファクトリに関しては、キャッシュストレージの位置を定義しなければなりません。

 * `sfFileCache`に関しては、`cache_dir`パラメーターはキャッシュディレクトリへの絶対パスを探します
 * `sfAPCCache`、`sfEAcceleratorCache`、と`sfXCacheCache`は位置パラメーターをとりません。これらがAPC、EAcceleratorもしくはXCacheキャッシュシステムとコミュニケーションするためにPHPのネイティブ関数を使うからです
 * `sfMemcacheCache`に関しては、Memcachedサーバーのホスト名を`host`パラメーターに、もしくはホストの配列を`servers`パラメーターに入力します
 * `sfSQLiteCache`に関しては、SQLiteデータベースファイルへの絶対パスは `database`パラメーターに入力されます。

追加パラメーターに関しては、それぞれのキャッシュクラスのAPIドキュメントを確認してください。

ビューはキャッシュを使える唯一のコンポーネントではありません。`routing`ファクトリと`I18N`ファクトリの両方は、ビューキャッシュと同じように、キャッシュファクトリを設定できる`cache`パラメーターを提供します。たとえば、リスト19-1はデフォルトで加速戦術用にファイルキャッシュを使うルーティングを示していますが、望むものは何でも変更できます。

#### ロギングの設定

2つのロギングの設定(16章を参照)は`settings.yml`ファイルに保存されます:

  * `error_reporting`はPHPログに記録されるイベントを指定します。デフォルトでは、運用環境に対しては`E_PARSE | E_COMPILE_ERROR | E_ERROR | E_CORE_ERROR | E_USER_ERROR`に(ロギングされるイベントは`E_PARSE`、`E_COMPILE_ERROR`、`E_ERROR`、`E_CORE_ERROR`、と `E_USER_ERROR`)、開発環境では`E_ALL | E_STRICT`に設定されます。
  * `web_debug`設定はWebデバッグツールバーを有効にします。開発とテスト環境のみでは`on`に設定します。

#### アセットへのパス

`settings.yml`ファイルはアセットへのパスも保存します。symfonyに搭載されたアセット以外の別のバージョンのアセットを使いたい場合、これらのパス設定を変更できます:

  * `rich_text_js_dir`に保存されるリッチなテキストエディタのJavaScriptファイル(デフォルトは`js/tiny_mce`)
  * `prototype_web_dir`に保存されるPrototypeライブラリ(デフォルトは`/sf/prototype`)
  * `admin_web_dir`に保存されadministrationジェネレーターが必要なファイル
  * `web_debug_web_dir`に保存されWebデバッグツールバーが必要なファイル
  * `calendar_web_dir`に保存されJavaScriptのカレンダーが必要なファイル

#### デフォルトのヘルパー

デフォルトのヘルパーは、すべてのテンプレートに対してロードされ、`standard_helpers`設定で宣言されます(7章を参照)。デフォルトでは、これらは`Partial`、`Cache`、`Form`ヘルパーグループです。アプリケーションのすべてのテンプレート内部でヘルパーグループを利用する場合、`standard_helpers`設定にヘルパーグループの名前を追加すればそれぞれのテンプレート上で`use_helper()`ヘルパーを用いてヘルパーグループを宣言する煩わしい手続きを行わずにすみます。

#### 有効なモジュール

プラグインもしくはsymfonyコアから有効にされるモジュールは`enabled_modules`パラメーターで宣言されます。プラグインがモジュールを搭載する場合、`enabled_modules`パラメーターで宣言されないかぎりユーザーはこのモジュールをリクエストできません。`default`モジュールはsymfonyのデフォルトページ(congratulations、page not foundページなど)を提供し、デフォルトで唯一有効なモジュールです。

#### 文字集合

レスポンスの文字集合はアプリケーション全体の設定です。フレームワークの多くのコンポーネントで使われるからです(テンプレート、出力エスケーパ、ヘルパーなど)。定義される`charset`設定のデフォルト値は`utf-8`(推奨)です。

#### そのほかの設定

`settings.yml`ファイルはコアのふるまいのためにsymfonyが内部で利用するいくつかのパラメーターを含みます。リスト19-3は設定ファイルに現れるパラメーターの一覧です。

リスト19-3 - そのほかの設定(`frontend/config/settings.yml`)

    # symfonyのコアクラスのコメントをcore_compile.ymlファイルで定義されたものとして除去する
    strip_comments:         on
    # 例外の起動前のアクションの前のフォワードの最大回数
    max_forwards:           5
    # グローバル定数
    path_info_array:        SERVER
    path_info_key:          PATH_INFO
    url_format:             PATH

>**SIDEBAR**
>アプリケーションの設定を追加する
>
>`settings.yml`ファイルはアプリケーションに対してsymfonyの設定を定義します。5章で説明したように新しいパラメーターを追加したい場合、最適の場所は`frontend/config/app.yml`ファイルです。このファイルは環境にも依存しており、このファイルが定義する設定の値は`sfConfig`クラスとプレフィックスの`app_`を通して利用できます。
>
>
>     all:
>       creditcards:
>         fake:             off    # app_creditcards_fake
>         visa:             on     # app_creditcards_visa
>         americanexpress:  on     # app_creditcards_americanexpress
>
>
>プロジェクトの設定ディレクトリのなかで`app.yml`ファイルを書くこともでき、カスタムプロジェクト設定を定義できます。設定カスケードはこのファイルにも適用されるので、アプリケーションの`app.yml`ファイルで定義された設定はプロジェクトレベルで定義された設定をオーバーライドします。

オートロード機能を拡張する
-------------------------

オートロード機能は2章で手短に説明しましたが、これによってコードが特定のディレクトリに設置していればクラスをrequireせずにすみます。このことは、symfonyに、必要なときだけ、適切な時点に必要なクラスだけをロードする作業を任せられることを意味します。

`autoload.yml`ファイルはオートロードされたクラスが保存されるパスの一覧を示します。この設定ファイルが最初に処理されたとき、symfonyはこのファイルに参照されたすべてのディレクトリを解析します。`.php`の拡張子を持つファイルがこれらのディレクトリの1つのなかで見つかるたびに、このファイルのなかで見つかるファイルパスとクラス名がオートロードクラスの内部リストに追加されます。このリストはキャッシュ、`config/config_autoload.yml`ファイルに保存されます。それから、実行時に、クラスが使われたとき、このリストのなかでsymfonyはクラスのパスを探し`.php`ファイルを自動的にインクルードします。

オートロード機能はクラスかつ/またはインターフェイスを含むすべての`.php`ファイルに対して機能します。

デフォルトでは、つぎのプロジェクトディレクトリに保存されたクラスはオートロード機能からの恩恵を受けます:

  * `myproject/lib/`
  * `myproject/lib/model`
  * `myproject/apps/frontend/lib/`
  * `myproject/apps/frontend/modules/mymodule/lib`

`autolaod.yml`ファイルはアプリケーションのデフォルトの設定ディレクトリ内には存在しません。symfonyの設定を修正したい場合、たとえばファイル構造のどこかに保存されたクラスをオートロードするには、空の`autoload.yml`ファイルを作り、`$sf_symfony_lib_dir/config/autoload.yml`ファイルもしくは独自ファイルの設定をオーバーライドします。

`autoload.yml`ファイルは`autoload:`キーで始まり、symfonyがクラスを探す場所のリストを記載しなければなりません。それぞれの場所はラベルを必要とします: これによってsymfonyのエントリーをオーバーライドできます。それぞれの場所に対して、`name`(`config_autload.yml.php`でコメントとして表示される)と絶対パス(`path`)を記入してください。それから、検索が再帰的(`recursive`)であるように定義すると、symfonyはすべてのサブディレクトリで`.php`ファイルを探します。また望むサブディレクトリを除外(`exclude`)します。リスト19-2はデフォルトで使われる場所とファイルの構文を示しています。

リスト19-4 - オートロードのデフォルトコンフィギュレーション(`$sf_symfony_lib_dir/config/autoload.yml`)

    autoload:
      # プラグイン
      plugins_lib:
        name:           plugins lib
        path:           %SF_PLUGINS_DIR%/*/lib
        recursive:      on

      plugins_module_lib:
        name:           plugins module lib
        path:           %SF_PLUGINS_DIR%/*/modules/*/lib
        prefix:         2
        recursive:      on

      # プロジェクト
      project:
        name:           project
        path:           %SF_LIB_DIR%
        recursive:      on
        exclude:        [model, symfony]

      project_model:
        name:           project model
        path:           %SF_LIB_DIR%/model
        recursive:      on

      # アプリケーション
      application:
        name:           application
        path:           %SF_APP_LIB_DIR%
        recursive:      on

      modules:
        name:           module
        path:           %SF_APP_DIR%/modules/*/lib
        prefix:         1
        recursive:      on

ルールのパスにワイルドカードを含めることは可能で設定クラスのなかで定義されるファイルパスのパラメーターが使えます(つぎのセクションを参照)。設定ファイルのなかでこれらのパラメーターを使う場合、これらのパラメーターは大文字で始めと終わりを`%`で挟まなければなりません。

独自の`autoload.yml`ファイルを編集すれば 新しい位置がsymfonyのオートロード機能に追加されますが、このメカニズムを拡張してsymfonyのハンドラーに独自のオートロードハンドラーを追加したいことがあります。symfonyはクラスのオートロードを管理するために標準の`spl_autoload_register()`関数を使うので、アプリケーションの設定クラスに複数のコールバックを登録できます:

    [php]
    class frontendConfiguration extends sfApplicationConfiguration
    {
      public function initialize()
      {
        parent::initialize(); // symfonyのオートロード機能を最初にロードする

        // 独自のオートロードコールバックをここに挿入する
        spl_autoload_register(array('myToolkit', 'autoload'));
      }
    }

PHPのオートロードシステムは新しいクラスに遭遇するとき、最初にsymfonyのオートロードメソッドが試されます(そして`autoload.yml`ファイルのなかで定義された位置が使われます)。クラスの定義が見つからない場合、クラスが見つかるまで`spl_autoload_register()`で登録されたすべてのcallableが呼び出されます。たとえば、ほかのフレームワークコンポーネントへのブリッジを提供するために(17章参照)、望む数だけオートロードメカニズムを追加できます。

カスタムファイル構造
----------------------

symfonyフレームワークは何か(コアクラスからテンプレート、プラグイン、設定、など)を探すためにパスを使うたびに、実際のパスの代わりにパス変数を使います。これらの変数を変更することで、symfonyプロジェクトのディレクトリ構造を完全に変更して、顧客のファイル構造の要件に適合させることができます。

>**CAUTION**
>symfonyプロジェクトのディレクトリ構造をカスタマイズするのは可能ですが、かならずしもよいアイディアではありません。symfonyのようなフレームワークの強みの一つは、規約を尊重されることでWeb開発者が慣習を尊重して開発されたプロジェクトを見て安心できることです。独自のディレクトリ構造を利用することを決定するまえにかならずこの問題を考えてください。

### 基本的なファイル構造

パス変数は`sfProjectConfiguration`と`sfApplicationConfiguration`クラスのなかで定義され`sfConfig`オブジェクトに保存されます。リスト19-5はパス変数とこれらが参照するディレクトリの一覧を示しています。

リスト19-5 - `sfProjectConfiguration`と`sfApplicationConfiguration`のなかで定義された、デフォルトのファイル構造の変数

    sf_root_dir           # myproject/
    sf_apps_dir           #   apps/
    sf_app_dir            #     frontend/
    sf_app_config_dir     #       config/
    sf_app_i18n_dir       #       i18n/
    sf_app_lib_dir        #       lib/
    sf_app_module_dir     #       modules/
    sf_app_template_dir   #       templates/
    sf_cache_dir          #   cache/
    sf_app_base_cache_dir #     frontend/
    sf_app_cache_dir      #       prod/
    sf_template_cache_dir #         templates/
    sf_i18n_cache_dir     #         i18n/
    sf_config_cache_dir   #         config/
    sf_test_cache_dir     #         test/
    sf_module_cache_dir   #         modules/
    sf_config_dir         #   config/
    sf_data_dir           #   data/
    sf_doc_dir            #   doc/
    sf_lib_dir            #   lib/
    sf_log_dir            #   log/
    sf_test_dir           #   test/
    sf_plugins_dir        #   plugins/
    sf_web_dir            #   web/
    sf_upload_dir         #     uploads/

重要なディレクトリへのすべてのパスは`_dir`で終わるパラメーターによって決定されます。あとで必要なときにパスを変更できるように、本当の(相対もしくは絶対)ファイルパスの代わりにパス変数をつねに使用してください。たとえば、ファイルをアプリケーションの`uploads/`ディレクトリに移動させたいとき、パスに対して`sfConfig::get('sf_root_dir').'/web/uploads/'`の代わりに`sfConfig::get('sf_upload_dir')`を使います。

### ファイル構造をカスタマイズする

アプリケーションを開発するさいに、顧客がすでにディレクトリ構造を定義しており、symfonyのロジックに適合させるために構造を変更する意志のない場合、おそらくデフォルトのプロジェクトファイル構造を修正する必要があります。`sf_XXX_dir`変数を`sfConfig`でオーバーライドすることで、デフォルトとはまったく異なるディレクトリ構造でsymfonyを動かすことができます。これを行う最良の場所はプロジェクトのディレクトリに対してはアプリケーションの`ProjectConfiguration`クラス、もしくはアプリケーションのディレクトリに対しては`XXXConfiguration`クラスです。

たとえば、すべてのアプリケーションにテンプレートのレイアウト用の共通ディレクトリを共有させたい場合、`sf_app_template_dir`設定をオーバーライドするためにつぎの行を`ProjectConfiguration`クラスの`configure()`メソッドに追加します:

    [php]
    sfConfig::set('sf_app_template_dir', sfConfig::get('sf_root_dir').DIRECTORY_SEPARATOR.'templates');

>**NOTE**
>`sfConfig::set()`を呼び出してプロジェクトのディレクトリ構造を変更できる場合でも、すべての関連パスの変更が考慮されるのでプロジェクトとアプリケーションの設定クラスによって定義された専用メソッドを使うほうが優れています。たとえば、`setCacheDir()`メソッドはつぎの定数: `sf_cache_dir`、`sf_app_base_cache_dir`、`sf_app_cache_dir`、`sf_template_cache_dir`、`sf_i18n_cache_dir`、`sf_config_cache_dir`、`sf_test_cache_dir`、と`sf_module_cache_dir`を変更します。

### Web公開のrootディレクトリの修正

設定クラスに組み込まれたすべてのパスはプロジェクトのrootディレクトリに依存します。このディレクトリパスはプロジェクトのなかの`ProjectConfiguration`ファイルによって決定されます。通常のrootディレクトリは`web/`ディレクトリの上位にありますが、異なる構造を利用できます。メインのディレクトリ構造が2つのディレクトリから構成される場合を考えてみます。リスト19-7で示されるように、1つのディレクトリは公開領域で、もう1つのディレクトリは非公開領域に存在します。プロジェクトを共用ホスティングサービス上でホストするときにこのコンフィギュレーションを選ぶことはよくあります。

リスト19-7 - 共用サーバーのためのカスタムディレクトリ構造の例

    symfony/    # 非公開領域
      apps/
      config/
      ...
    www/        # 公開領域
      images/
      css/
      js/
      index.php

この場合、rootディレクトリは`symfony/`ディレクトリです。ですのでアプリケーションを動かすには`index.php`フロントコントローラーが`config/ProjectConfiguration.class.php`ファイルをインクルードすることがだけが必要です:

    [php]
    require_once(dirname(__FILE__).'/../symfony/config/ProjectConfiguration.class.php');

加えて、つぎのように、公開領域を通常の`web/`から`www/`に変更するには`setWebDir()`メソッドを使います:

    [php]
    class ProjectConfiguration extends sfProjectConfiguration
    {
      public function setup()
      {
        // ...

        $this->setWebDir($this->getRootDir().'/../www');
      }
    }

### symfonyのライブラリにリンクする

リスト19-8で見られるように、symfonyへのパスは、`config/`ディレクトリ内に設置された、`ProjectConfiguration`クラスのなかで定義されます。

リスト19-8 - symfonyライブラリへのパス(`myproject/config/ProjectConfiguration.class.php`)

    [php]
    <?php

    require_once '/path/to/symfony/lib/autoload/sfCoreAutoload.class.php';
    sfCoreAutoload::register();

    class ProjectConfiguration extends sfProjectConfiguration
    {
      public function setup()
      {
      }
    }

コマンドラインから`php symfony generate:project`タスクを呼び出すとき、パスは初期化され、プロジェクトをビルドするために使われるsymfonyの設置ディレクトリを参照します。これはコマンドラインとMVCアーキテクチャの両方から利用されます。

このことはsymfonyのファイルへのパスを変更することでsymfonyの別の設置ディレクトリに切り替え可能であることを意味します。

このパスは絶対パスでなければなりませんが、`dirname(__FILE__)`を利用することで、プロジェクト構造内部のファイルを参照してプロジェクトを設置するために選ばれたディレクトリの独立性を保つことができます。たとえば、つぎのコードのように、多くのプロジェクトがsymfonyの`lib/`ディレクトリをプロジェクトの`lib/vendor/symfony/`ディレクトリのシンボリックリンクとして設定します:

    myproject/
      lib/
        vendor/
          symfony/ => /path/to/symfony/

この場合、つぎのように`ProjectConfiguration`クラスはsymfonyのlibディレクトリを必要とするだけです:

    [php]
    <?php

    require_once dirname(__FILE__).'/../lib/vendor/symfony/lib/autoload/sfCoreAutoload.class.php';
    sfCoreAutoload::register();

    class ProjectConfiguration extends sfProjectConfiguration
    {
      public function setup()
      {
      }
    }

プロジェクトの`lib/vendor/`ディレクトリ内でsymfonyのファイルを`svn:externals`プロパティとして格納することを選んだ場合にも同じ原則があてはまります:

    myproject/
      lib/
        vendor/
          svn:externals symfony http://svn.symfony-project.com/branches/1.1

>**TIP**
>アプリケーションを稼働させているサーバーが異なる場合symfonyのライブラリへのパスが異なることがあります。これを有効にする1つの方法は(`rsync_exclude.txt`に追加することで)`ProjectConfiguration.class.php`ファイルを同期化の対象から除外することです。ほかの方法は開発と運用の両方の`ProjectConfiguration.class.php`ファイルで同じパスを保つことですが、シンボリックリンクを指し示すこれらのパスがサーバーによって変わる可能性があります。

コンフィギュレーションハンドラーを理解する
----------------------

それぞれの設定ファイルはハンドラーを持ちます。コンフィギュレーションハンドラー(configuration handler)の仕事は設定カスケードを管理することと、実行時に設定ファイルを最適化して実行可能なPHPコードに変換することです。

### デフォルトのコンフィギュレーションハンドラー

デフォルトのハンドラー設定は`$sf_symfony_lib_dir/config/config/config_handlers.yml`ファイルに保存されます。このファイルはファイルパスにしたがってハンドラーを設定ファイルにリンクします。リスト19-9はこのファイルの内容を抜粋したものです。

リスト19-9 - `$sf_symfony_lib_dir/config/config/config_handlers.yml`ファイルの抜粋

    config/settings.yml:
      class:    sfDefineEnvironmentConfigHandler
      param:
        prefix: sf_

    config/app.yml:
      class:    sfDefineEnvironmentConfigHandler
      param:
        prefix: app_

    config/filters.yml:
      class:    sfFilterConfigHandler

    modules/*/config/module.yml:
      class:    sfDefineEnvironmentConfigHandler
      param:
        prefix: mod_
        module: yes

それぞれの設定ファイルに対して(`config_handlers.yml`ファイルはワイルドカードを持つファイルパスでそれぞれのファイルを識別する)、ハンドラークラスは`class`キーの下で指定されます。

`sfDefineEnvironmentConfigHandler`クラスによって処理される設定ファイルの設定は`sfConfig`クラスを通してコードのなかで直接利用できるようになり、`param`キーはプレフィックスの値を含みます。

設定ファイルを処理するために使われるハンドラーの追加もしくは修正ができます。たとえば、YAMLファイルの代わりに`INI`ファイルもしくは`XML`ファイルを使うためです。

>**NOTE**
>`config_handlers.yml`ファイル用のコンフィギュレーションハンドラーは`sfRootConfigHandler`クラスで、あきらかに変更できません。

設定の解析方法を修正する必要がある場合、アプリケーションの`cofig/`フォルダー内に空の`config_handlers.yml`ファイルを作り、`class`キーの行を書いたクラスでオーバーライドします。

### 独自ハンドラーを追加する

設定ファイルを処理するハンドラーを利用することで2つの大きな利点がもたらされます:

  * 設定ファイルはPHPの実行コードに変換され、このコードはキャッシュに保存されます。このことは、運用環境において設定は1回だけ解析されるのでパフォーマンスが最適化されていることを意味します。
  * 設定ファイルは異なるレベル(プロジェクトとアプリケーション)で定義することが可能で、最後のパラメーターの値はカスケードから由来します。プロジェクトレベルでパラメーターを定義し、アプリケーション単位でこれらをオーバーライドできます。

独自のコンフィギュレーションハンドラーを書きたい場合、`$sf_symfony_lib_dir/config/`ディレクトリのなかでsymfonyによって使われる構造の例にしたがってください。

アプリケーションが`myMapAPI`クラスを含む場合を考えてみましょう。`myMapAPI`クラスは地図を配信するサードパーティのサービスのためのインターフェイスを提供します。リスト19-10で示されるように、このクラスはURLとユーザー名で初期化することが必要です。

リスト19-10 - `myMapAPI`クラスの初期化の例

    [php]
    $mapApi = new myMapAPI();
    $mapApi->setUrl($url);
    $mapApi->setUser($user);

アプリケーションの`config/`ディレクトリ内に設置された、`map.yml`という名前のカスタム設定ファイルでこれら2つのパラメーターを保存するとよいでしょう。この設定ファイルはつぎのような内容を含むことがあります:

    api:
      url:  map.api.example.com
      user: foobar

これらの設定をリスト19-8と同等なコードに変換するために、コンフィギュレーションハンドラーを作成しなければなりません。それぞれのコンフィギュレーションハンドラーは`sfConfigHandler`クラスを拡張し`execute()`メソッドを提供しなければなりません。`execute()`メソッドはパラメーターとして設定ファイルへのファイルパスの配列が必要で、キャッシュファイルに書き込まれるデータを返さなければなりません。YAMLファイル用のハンドラーは`sfYamlConfigHandler`クラスを拡張します。このクラスはYAMLパーサーのために追加のファシリティを提供します。`map.yml`ファイルに対する典型的なコンフィギュレーションハンドラーはリスト19-11で示されるように書けます。

リスト19-11 - カスタムコンフィギュレーションハンドラー(`frontend/lib/myMapConfigHandler.class.php`)

    [php]
    <?php

    class myMapConfigHandler extends sfYamlConfigHandler
    {
      public function execute($configFiles)
      {
        // yamlを解析する
        $config = $this->parseYamls($configFiles);

        $data  = "<?php\n";
        $data .= "\$mapApi = new myMapAPI();\n";

        if (isset($config['api']['url'])
        {
          $data .= sprintf("\$mapApi->setUrl('%s');\n", $config['api']['url']);
        }

        if (isset($config['api']['user'])
        {
          $data .= sprintf("\$mapApi->setUser('%s');\n", $config['api']['user']);
        }

        return $data;
      }
    }

symfonyが`execute()`メソッドに渡す`$configFiles`配列は`config/`フォルダーのなかで見つかるすべての`map.yml`ファイルへのパスを格納します。`parseYamls()`メソッドは設定カスケードを扱います。

この新しいハンドラーを`map.yml`ファイルと関連づけるには、つぎのような内容を持つ`config_handlers.yml`設定ファイルを作成しなければなりません:

    config/map.yml:
      class: myMapConfigHandler

>**NOTE**
>`class`はオートロードするか(上記の例)ファイルパスが`param`キーの下の`file`パラメーターで指定されたファイルのなかで定義しなければなりません。

ほかの多くのsymfony設定ファイルに関しては、PHPコードにコンフィギュレーションハンドラーを直接追加することもできます:

    sfContext::getInstance()->getConfigCache()->registerConfigHandler('config/map.yml', 'myMapConfigHandler', array());

アプリケーション内部で`map.yml`ファイルに基づき`myMapConfigHandler`ハンドラーによって生成されたコードが必要な場合、つぎの行を呼び出してください:

    [php]
    include(sfContext::getInstance()->getConfigCache()->checkConfig('config/map.yml'));

`checkConfig()`メソッドを呼び出すとき、`map.yml.php`ファイルがキャッシュにまだ存在しないもしくは`map.yml`ファイルがキャッシュよりも新しい場合、symfonyは設定ディレクトリ内で既存の`map.yml`ファイルを探し`config_handlers.yml`ファイルで指定されたハンドラーを利用してこれらのファイルを処理します。

>**TIP**
>YAMLの設定ファイル内部で環境を扱いたい場合、ハンドラーは`sfYamlConfigHandler`クラスの代わりに`sfDefineEnvironmentConfigHandler`クラスを拡張できます。設定を読みとるには、`parseYaml()`メソッドの代わりに`getConfiguration()`メソッド: `$config = $this->getConfiguration($configFiles)`を呼び出します。

-

>**SIDEBAR**
>既存のコンフィギュレーションハンドラーを利用する
>
>ユーザーが`sfConfig`クラス経由でコードから値を読みとることができるようにする必要があるだけなら、`sfDefineEnvironmentConfigHandler`コンフィギュレーションハンドラークラスを利用できます。たとえば、`url`と`user`パラメーターをそれぞれ`sfConfig::get('map_url')`と`sfConfig::get('map_user')`として使えるようにするには、ハンドラーをつぎのように定義します:
>
>     config/map.yml:
>       class: sfDefineEnvironmentConfigHandler
>       param:
>         prefix: map_
>
>すでにほかのハンドラーによって使われているプレフィックスを選ばないように気をつけてください。 既存のプレフィックスは`sf_`、`app`、と`mod_`です。

まとめ
----

設定ファイル(configuration file)はsymfonyフレームワークの動作方法を大いに変更します。symfonyはコア機能とファイルの読み込みでさえも設定に依存するので、標準の専用ホストよりも多くの環境に適用できます。このすばらしい設定の柔軟性はsymfonyの主要な強さの1つです。設定ファイルのなかで学ぶべきたくさんの規約を見た初心者を怖がらせることがあるにせよ、このことによってsymfony製のアプリケーションは膨大な数のプラットフォーム、環境に対して、互換性があります。ひとたびsymfonyの設定を習得したら、あなたのアプリケーションを動かすことを拒むサーバーは存在しないでしょう。
