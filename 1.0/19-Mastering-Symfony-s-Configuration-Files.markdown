第19章 - symfonyの設定ファイルをマスターする
============================================

現在あなたはsymfonyをとてもよく理解しています。すでにコアの設計を理解し、新しい隠し機能を見つけるためにコードを徹底的に調べる準備ができています。しかし、独自の要件に適用するためにsymfonyのクラスを拡張するまえに、設定ファイルをもっとよく見ることが必要です。すでに多くの機能はsymfonyに組み込まれ、設定を少し変更すれば有効になります。このことはクラスをオーバーライドしなくてもsymfonyコアのふるまいを調整できることを意味します。この章では設定ファイルとこれらの強力な機能を詳しく説明します。

symfonyの設定
-------------

`myapp/config/settings.yml`ファイルはおもに`myapp`アプリケーション用のsymfonyの設定を含みます。前の章でこのファイルから多くの設定の機能をすでに見てきましたが、再度これらを見ることにします。

5章で説明したように、このファイルは環境に依存します。すなわちそれぞれの設定が環境ごとに異なる値をとります。このファイルで定義されたそれぞれのパラメーターは`sfConfig`クラスを通してPHPクラスの内部からアクセスできることを覚えておいてください。パラメーター名は設定名にプレフィックスの`sf_`をつけたものです。たとえば、`cache`パラメーターの値を取得したい場合、必要なことは`sfConfig::get('sf_cache')`を呼び出すだけです。

### デフォルトのモジュールとアクション

ルーティングルールが`module`パラメーターもしくは`action`パラメーターを定義していない場合、代わりに`settings.yml`ファイルからの値が使われます; 

  * `default_module`: デフォルトの`module`リクエストパラメーター。デフォルトは`default`モジュール。
  * `default_action`: デフォルトの`action`リクエストパラメーター。デフォルトは`index`アクション。

symfonyは特殊な状況に対してデフォルトページを用意します。ルーティングエラーの場合、symfonyは`default`モジュールのアクションを実行します。このアクションは`$sf_symfony_data_dir_modules/default/`ディレクトリに保存されています。`settings.yml`ファイルはエラーごとに実行するアクションを定義します:

  * `error_404_module`と`error_404_action`: ユーザーによって入力されたURLがどのrouteにもマッチしないもしくは`sfError404Exception`が起動するときに呼び出されるアクションです。デフォルト値は`default/error404`です。
  * `login_module`と`login_action`: `security.yml`ファイルのなかで`secure`と定義されたページに認証されていないユーザーがアクセスしようとするときに呼び出されるアクション(詳細は6章を参照)。デフォルト値は`default/login`です。
  * `secure_module`と`secure_action`: ユーザーがアクションから求められるクレデンシャルを持たないときに呼び出されるアクションです。デフォルト値は`default/secure`です。
  * `module_disabled_module`と`module_disabled_action`: ユーザーが`module.yml`ファイルのなかで無効と宣言されたモジュールをリクエストしたときに呼び出されるアクション。デフォルト値は`default/disabled`です。
  * `unavailable_module`と`unavailable_action`: 無効なアプリケーションからユーザーがページをリクエストしたときに呼び出されるアクション。デフォルト値は`default/unavailable`です。アプリケーションを無効にするには、`settings.yml`ファイルのなかで`available`パラメーターを`off`に設定してください。

アプリケーションを運用サーバーにデプロイするまえにこれらのアクションはカスタマイズすべきです。`default`モジュールのテンプレートではページにsymfonyのロゴが含まれるからです。これらのページの1つのスクリーンショット、404エラーのページは図19-1をご覧ください。

図19-1 - デフォルトの404エラーページ

![デフォルトの404エラーページ](/images/book/F1901.jpg "デフォルトの404エラーページ")

以下の2つの方法でデフォルトのページをオーバーライドできます:

  * アプリケーションの`modules/`ディレクトリ内で独自の`default`モジュールを作成し、`settings.yml`ファイル(`index`、`error404`、`secure`、`disabled`、と`unavailable`)と関連するすべてのテンプレート(`indexSuccess.php`、`error404Success.php`、`loginSuccess.php`、`secureSuccess.php`、`disabledSuccess.php`と`unavailableSuccess.php`)内で定義されたすべてのアクションをオーバーライドできます。
  * アプリケーションのページを使うために`settings.yml`ファイルのデフォルトのモジュールとアクションの設定を変更できます。

ほかの2つのページはsymfonyの見た目を有しており、運用サーバーにデプロイするまえにこれらのページをカスタマイズすることも必要です。これらのページは`default`モジュールには存在しません。これはsymfonyが適切に動作しないときに呼び出されるからです。代わりに、これらのデフォルトページは`$sf_symfony_data_dir/web/errors/`ディレクトリのなかで見つかります:

  * `error500.php`: 運用環境で内部のサーバーエラーが起きるときに呼び出されるページ。ほかの環境(`SF_DEBUG`が`true`に設定されている)においてエラーが起きるとき、symfonyはすべての実行スタックと明快なエラーメッセージを表示します(詳細は16章を参照)。
  * `unavailable.php`: キャッシュがクリアされている間(すなわち、symfony の`clear-cache`タスクを呼び出したときからこのタスクの実行が終了するまでの間)にユーザーがページをリクエストしたときに呼び出されるページ。とても大きなキャッシュを持つシステム上において、キャッシュをクリアする処理は数秒かかる可能性があります。symfonyは部分的にクリアされたキャッシュでリクエストを実行できないので、処理が終わるまえに受理されたリクエストはこのページにリダイレクトされます。

これらのページをカスタマイズするには、アプリケーションの`web/errors/`ディレクトリのなかで`error500.php`ページと`unavailable.php`ページを作ります。symfonyは固有のページの代わりにこれらを使うようになります。

>**NOTE**
>必要なときにリクエストを`unavailable.php`ページにリダイレクトするには、アプリケーションの`settings.yml`のなかで`check_lock`設定を`on`にする必要があります。デフォルトではこの設定は無効です。この設定によってすべてのリクエストに対してわずかですがオーバーヘッドが追加されるからです。

### オプション機能の有効

`settings.yml`ファイルのパラメーターのなかには有効もしくは無効にできるsymfonyのオプション機能をコントロールするものがあります。使わない機能を無効にすることでパフォーマンスが少し押し上げられるので、アプリケーションをデプロイするまえにテーブル19-1に示されている設定の一覧を見直してください。

テーブル 19-1 - `settings.yml`ファイルを通したオプション機能の一式

パラメーター              | 説明  | デフォルト値
----------------------- | ----- | --------------
`use_database`          | データベースマネージャを有効にする。データベースを使わない場合は`off`に切り替える。 | `on`
`use_security`          | セキュリティ機能を有効にする(`secure`アクションとクレデンシャル; 6章を参照)。デフォルトのセキュリティフィルター(`sfBasicSecurityFilter`)は`on`の場合のみ有効です。 | `on`
`use_flash`             | flashパラメーター機能を有効にする(6章を参照)。アクション内で`set_flash()`メソッドを決して使わない場合は`off`に設定する。flashフィルター(`sfFlashFilter`)は`on`になっているときのみ有効。 | `on`
`i18n`                  | インターフェイスの翻訳機能を有効にする(13章を参照)。他言語アプリケーションのために`on`に設定する。 | `off`
`logging_enabled`       | symfonyのイベントのロギング機能を有効にする。`logging.yml`設定を無視してsymfonyのロギング機能を完全に`off`に切り替えたいときは`off`に設定する。 | `on`
`escaping_strategy`     | 出力エスケーピング機能の方針を有効にして定義する(7章を参照)。テンプレート内で`$sf_data`コンテナを使わない場合に`off`に設定する。|`bc`
`cache`                 | テンプレートキャッシュを有効にする(12章を参照)。モジュールの1つが`cache.yml`ファイルを含む場合に`on`に設定する。キャッシュフィルター(`sfCacheFilter`)が`on`になっている場合にのみ有効 | 開発環境では`off`、運用環境では`on`
`web_debug`             | 簡単なデバッグのためにWebデバッグツールバーを有効にする(16章を参照)。ツールバーをすべてのページに表示するために`on`に設定する。Webデバッグフィルター(`sfWebDebugFilter`)が`on`のときのみ有効。|`on`は開発環境、`off`は運用環境
`check_symfony_version` | すべてのリクエストに対してsymfonyのバージョンチェックを有効にする。symfonyをアップグレードした後に自動的にキャッシュをクリアするために`on`に設定する。アップグレードした後につねにキャッシュをクリアする場合は`off`のままにしておく。|`off`
`check_lock`            | `clear-cache`タスクと`disable`タスクで起動するアプリケーションのロックシステムを有効にする(前のセクションを参照)。無効されたアプリケーションへのリクエストを`$sf_symfony_data_dir/web/errors/unavailable.php`ページにリダイレクトするにはこのパラメーターを`on`に設定する。 | `off`
`compressed`            | PHPのレスポンス圧縮機能を有効にする。PHP圧縮ハンドラー経由で出力するHTMLを圧縮するには`on`にする。 | `off`
`use_process_cache`     | PHPアクセレータに基づいてsymfonyの最適化を有効にする。PHPアクセレータ(たとえば、APC、XCache、eAcceleratorなど)がインストールされたとき、symfonyはリクエスト間でオブジェクトと設定をメモリに保存する機能を利用する。開発環境、もしくはPHPアクセレータ最適化が必要ではない時に`off`に設定する。アクセレータがインストールされていなくても、`on`のままにしておいてもパフォーマンスに害をなさないことを留意する | `on`

### 機能のコンフィギュレーション

symfonyは組み込み機能、たとえば、バリデーション、キャッシュ、サードパーティのモジュールなどのふるまいを変更するにはいくつかの`settings.yml`ファイルのパラメーターを使います。

#### 出力エスケーピングの設定

出力エスケーピングの設定は変数がテンプレートにアクセスする方法をコントロールします(7章を参照)。`settings.yml`ファイルはこの機能のために2つの設定を含みます:

  * `escaping_strategy`設定は`bc`、`both`、`on`、もしくは`off`の値をとることができます。
  * `escaping_method`設定は`ESC_RAW`、`ESC_ENTITIES`、`ESC_JS`、もしくは`ESC_JS_ENTITIES`の値に設定できます。

#### ルーティングの設定

ルーティングの2つの設定(9章を参照)は`settings.yml`ファイルに保存されます:

  * `suffix`パラメーターは生成されたURLのためにデフォルトのサフィックスを設定します。デフォルト値はピリオド(`.`)で、どの接尾辞にも一致しません。たとえば、すべての生成されたURLが静的なページに見えるように`.html`に設定してください。
  * `no_script_name`パラメーターは生成されたURLのフロントコントローラー名を有効にします。さまざまなディレクトリでフロントコントローラーを保存し、デフォルトのURL書き換えルールを変更しないかぎり、`no_script_name`設定はプロジェクトの単独のアプリケーションでのみ`on`にできます。通常はあなたのメインアプリケーションの運用環境のみ`on`で他は`off`です。

#### フォームバリデーションの設定

フォームバリデーションの設定は`Validation`ヘルパーによるエラーメッセージが表示される方法をコントロールします(10章を参照)。これらのエラーは`<div>`に含まれ、`id`属性を形成するためにこれらのエラーは`validation_error_class`設定と`validation_error_id_prefix`設定を`class`属性として使います。デフォルト値は`form_error`と`error_for_`なので、`foobar`という名前の入力に対して`form_error()`ヘルパーへの呼び出しによる属性出力は`class="form_error" id="error_for_foobar"`になります。

2つの設定はそれぞれのエラーメッセージ: `validation_error_prefix`と`validation_error_suffix`の前後に来る文字を決定します。一度にすべてのエラーメッセージをカスタマイズするためにこれらの設定を変更できます。

#### キャッシュの設定

キャッシュ設定の大部分は`cache.yml`ファイルのなかで定義されますが。`settings.yml`ファイルのなかの以下の2つの設定は異なります。`cache`はテンプレートキャッシュメカニズムを有効にし、`etag`はサーバーサイド上のEtagハンドリングを有効にします(15章を参照)。

#### ロギングの設定

2つのロギングの設定(16章を参照)は`settings.yml`ファイルに保存されます:

  * `error_reporting`設定はどのイベントがPHPのログに記録されるのかを指定します。デフォルトでは、運用環境では`341`に設定され(記録されるイベントは`E_PARSE`、`E_COMPILE_ERROR`、`E_ERROR`、`E_CORE_ERROR`、と`E_USER_ERROR`)、開発環境では`4095`に設定されます(`E_ALL`と`E_STRICT`)。
  * `web_debug`設定はWebデバッグツールバーを有効にします。開発とテスト環境では`on`に設定します。

#### アセットへのパス

`settings.yml`ファイルはアセットへのパスも保存します。symfonyに搭載されたアセット以外の別のバージョンのアセットを使いたい場合、これらのパス設定を変更できます:

  * `rich_text_js_dir`に保存されるリッチなテキストエディタのJavaScriptファイル(デフォルトで`js/tiny_mce`)
  * `prototype_web_dir`に保存されるPrototypeライブラリ(デフォルトで`/sf/prototype`)
  * `admin_web_dir`に保存されadministrationジェネレーターが必要なファイル
  * `web_debug_web_dir`に保存されWebデバッグツールバーが必要なファイル
  * `calendar_web_dir`に保存されJavaScriptのカレンダーが必要なファイル

#### デフォルトのヘルパー

デフォルトのヘルパーは、すべてのテンプレートに対してロードされ、`standard_helpers`設定で宣言されます(7章を参照)。デフォルトでは、これらは`Partial`、`Cache`、`Form`ヘルパーグループです。アプリケーションのすべてのテンプレート内部でヘルパーグループを利用する場合、ヘルパーグループの名前を`standard_helpers`設定に追加すればそれぞれのテンプレート上で`use_helper()`ヘルパーを用いてヘルパーグループを宣言する煩わしい手続きを行わずにすみます。

#### 有効なモジュール

プラグインもしくはsymfonyコアから有効にされるモジュールは`enabled_modules`パラメーターで宣言されます。プラグインがモジュールを搭載する場合、`enabled_modules`パラメーターで宣言されないかぎりユーザーはこのモジュールをリクエストできません。`default`モジュールはsymfonyのデフォルトページ(congratulations、page not foundページなど)を提供し、デフォルトで唯一有効なモジュールです。

#### 文字集合

レスポンスの文字集合はアプリケーション全体の設定です。フレームワークの多くのコンポーネントで使われるからです(テンプレート、出力エスケーパ、ヘルパーなど)。定義される`charset`設定のデフォルト値は`utf-8`(推奨)です。

#### そのほかの設定

`settings.yml`ファイルはコアのふるまいのためにsymfonyが内部で利用するいくつかのパラメーターを含みます。リスト19-1は設定ファイルに現れるパラメーターの一覧です。

リスト19-1 - そのほかのコンフィギュレーション設定(`myapp/config/settings.yml`)

    # symfonyのコアクラスのコメントをcore_compile.ymlファイルで定義されたものとして除去する
    strip_comments:         on
    # クラスがリクエストされまだロードされていないときに呼び出される関数は
    # 呼び出し可能な配列を必要とする。フレームワークブリッジによって使われる
    autoloading_functions:  ~
    # 秒単位の、セッションのタイムアウト
    timeout:                1800
    # 例外の起動前のアクションの前のフォワードの最大回数
    max_forwards:           5
    # グローバル定数
    path_info_array:        SERVER
    path_info_key:          PATH_INFO
    url_format:             PATH

>**SIDEBAR**
>アプリケーションの設定を追加する
>
>`settings.yml`ファイルはアプリケーションに対してsymfonyの設定を定義します。5章で説明したように新しいパラメーターを追加したい場合、最適の場所は`myapp/config/app.yml`ファイルです。このファイルは環境にも依存しており、このファイルが定義する設定の値は設定名にプレフィックスの`app_`を付け加えることで`sfConfig`クラスを通して利用できます。
>
>
>     all:
>       creditcards:
>         fake:             off    # app_creditcards_fake
>         visa:             on     # app_creditcards_visa
>         americanexpress:  on     # app_creditcards_americanexpress
>
>
>プロジェクトの設定ディレクトリのなかで`app.yml`ファイルを書くこともできますが、これはカスタムプロジェクト設定を定義する方法を提供します。設定カスケードはこのファイルにも適用するので、アプリケーションの`app.yml`ファイルで定義された設定はプロジェクトレベルで定義された設定をオーバーライドします。

オートロード機能を拡張する
-------------------------

オートロード機能は2章で手短に説明しましたが、これによってコードが特定のディレクトリに設置していればクラスをrequireせずにすみます。このことは、必要なときだけ、適切な時点に必要なクラスだけをロードする作業をsymfonyに任せられることを意味します。

`autoload.yml`ファイルはオートロードされたクラスが保存されるパスの一覧を示します。この設定ファイルが最初に処理されたとき、symfonyはこのファイルに参照されたすべてのディレクトリを解析します。`.php`の拡張子を持つファイルがこれらのディレクトリの1つのなかで見つかるたびに、このファイル内で見つかるファイルパスとクラス名がオートロードクラスの内部リストに追加されます。このリストはキャッシュ、`config/config_autoload.yml`ファイルに保存されます。それから、実行時に、クラスが使われたとき、symfonyはクラスのパスをこのリストのなかで探し`.php`ファイルを自動的にインクルードします。

オートロード機能はクラスかつ/またはインターフェイスを含むすべての`.php`ファイルに対して機能します。

デフォルトでは、つぎのプロジェクトディレクトリに保存されたクラスはオートロード機能からの恩恵を受けます:

  * `myproject/lib/`
  * `myproject/lib/model`
  * `myproject/apps/myapp/lib/`
  * `myproject/apps/myapp/modules/mymodule/lib`

`autolaod.yml`ファイルはアプリケーションのデフォルトの設定ディレクトリのなかには存在しません。symfonyの設定を修正したい場合、たとえばファイル構造のどこかに保存されたクラスをオートロードするには、空の`autoload.yml`ファイルを作り、`$sf_symfony_data_dir/config/autoload.yml`ファイルもしくは独自ファイルの設定をオーバーライドします。

`autoload.yml`ファイルは`autoload:`キーで始まり、symfonyがクラスを探す場所のリストを記載しなければなりません。それぞれの場所はラベルを必要とします: これによってsymfonyのエントリーをオーバーライドできます。それぞれの位置に対して、`name`(`config_autload.yml.php`でコメントとして表示される)と絶対パス(`path`)を記入してください。それから、検索が再帰的(`recursive`)であるように定義すると、symfonyはすべてのサブディレクトリで`.php`ファイルを探します。また望むサブディレクトリを除外(`exclude`)します。リスト19-2はデフォルトで使われる場所とファイルの構文を示しています。

リスト19-2 - オートロードのデフォルト設定(`$sf_symfony_data_dir/config/autoload.yml`ファイル)

    autoload:

      # symfonyコア
      symfony:
        name:           symfony
        path:           %SF_SYMFONY_LIB_DIR%
        recursive:      on
        exclude:        [vendor]

      propel:
        name:           propel
        path:           %SF_SYMFONY_LIB_DIR%/vendor/propel
        recursive:      on

      creole:
        name:           creole
        path:           %SF_SYMFONY_LIB_DIR%/vendor/creole
        recursive:      on

      propel_addon:
        name:           propel addon
        files:
          Propel:       %SF_SYMFONY_LIB_DIR%/addon/propel/sfPropelAutoload.php

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
        path:           %SF_MODEL_LIB_DIR%
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

ルールのパスにワイルドカードを含めることは可能で、`constants.php`ファイルからファイルパスのパラメーターが使えます(つぎのセクションを参照)。設定ファイルのなかでこれらのパラメーターを使う場合、これらのパラメーターは大文字で始めと終わりを`%`で挟まなければなりません。

独自の`autoload.yml`ファイルを編集すれば新しい位置がsymfonyのオートロード機能に追加されますが、このメカニズムを拡張してsymfonyのハンドラーに独自のオートロードハンドラーを追加したいことがあります。これは`settings.yml`ファイルのなかの`autoloading_function`パラメーターを通して実現できます。つぎのように、このパラメーターは値として呼び出し可能なクラスの配列を必要とします:

    .settings:
      autoloading_functions:
        - [myToolkit, autoload]

symfonyは新しいクラスに遭遇するとき、最初に独自のオートロードシステムが試されます(そして`autoload.yml`で定義された位置が使われます)。クラスの定義が見つからない場合、クラスが見つかるまで、`settings.yml`ファイルからほかのオートロード機能を試します。望む数だけオートロードメカニズムを追加できます。たとえば、ほかのフレームワークコンポーネントへのブリッジを提供するためなどです(17章を参照)。

カスタムファイル構造
----------------------

symfonyフレームワークは何か(コアクラスからテンプレート、プラグイン、設定、など)を探すためにパスを使うたびに、実際のパスの代わりにパス変数を使います。これらの変数を変更することで、symfonyプロジェクトのディレクトリ構造を完全に変更して、顧客のファイル構造の要件に適合させることができます。

>**CAUTION**
>symfonyプロジェクトのディレクトリ構造をカスタマイズするのは可能ですが、かならずしもよいアイディアではありません。symfonyのようなフレームワークの強みの一つは。規約が尊重されることでWeb開発者が慣習を尊重して開発されたプロジェクトを見て安心できることです。独自のディレクトリ構造を利用することを決定するまえにかならずこの問題を考えてください。

### 基本的なファイル構造

パス変数は、アプリケーションが起動するときに含まれる、`$sf_symfony_data_dir/config/constants.php`ファイルのなかで定義されます。これらの変数は`sfConfig`オブジェクトに保存されるのでオーバーライドするのは簡単です。リスト19-3はこれらが参照するパス変数とディレクトリのリストを示しています。

リスト19-3 - デフォルトのファイル構造の変数(`$sf_symfony_data_dir/config/constants.php`)

    sf_root_dir           # myproject/
                          #   apps/
    sf_app_dir            #     myapp/
    sf_app_config_dir     #       config/
    sf_app_i18n_dir       #       i18n/
    sf_app_lib_dir        #       lib/
    sf_app_module_dir     #       modules/
    sf_app_template_dir   #       templates/
    sf_bin_dir            #   batch/
                          #   cache/
    sf_base_cache_dir     #     myapp/
    sf_cache_dir          #       prod/
    sf_template_cache_dir #         templates/
    sf_i18n_cache_dir     #         i18n/
    sf_config_cache_dir   #         config/
    sf_test_cache_dir     #         test/
    sf_module_cache_dir   #         modules/
    sf_config_dir         #   config/
    sf_data_dir           #   data/
    sf_doc_dir            #   doc/
    sf_lib_dir            #   lib/
    sf_model_lib_dir      #     model/
    sf_log_dir            #   log/
    sf_test_dir           #   test/
    sf_plugins_dir        #   plugins/
    sf_web_dir            #   web/
    sf_upload_dir         #     uploads/

重要なディレクトリへのすべてのパスは`_dir`で終わるパラメーターによって決定されます。あとで必要なときにパスを変更できるように、本当の(相対もしくは絶対)ファイルパスの代わりにパス変数をつねに使用してください。たとえば、ファイルをアプリケーションの`uploads/`ディレクトリに移動させたいとき、パスとして`SF_ROOT_DIR.'/web/uploads/'`の代わりに`sfConfig::get('sf_upload_dir')`を使います。

ルーティングシステムがモジュールの名前(`$module_name`)を定義するとき、モジュールのディレクトリ構造は実行時に定義されます。リスト19-4で示されるように、`constants.php`ファイルのなかで定義されたパス名にしたがって、ディレクトリは自動的に作成されます。

リスト19-4 - モジュールのファイル構造のデフォルトの変数

    sf_app_module_dir                 # modules/
    module_name                       #  mymodule/
    sf_app_module_action_dir_name     #    actions/
    sf_app_module_template_dir_name   #    templates/
    sf_app_module_lib_dir_name        #    lib/
    sf_app_module_view_dir_name       #    views/
    sf_app_module_validate_dir_name   #    validate/
    sf_app_module_config_dir_name     #    config/
    sf_app_module_i18n_dir_name       #    i18n/

ですので、たとえば、現在のモジュールの`validate/`ディレクトリへのパスは実行時に作成されます:

    [php]
    sfConfig::get('sf_app_module_dir').'/'.$module_name.'/'.sfConfig::get('sf_app_module_validate_dir_name')

### ファイル構造をカスタマイズする

アプリケーションを開発するさいに、顧客がすでにディレクトリ構造を定義しており、symfonyのロジックに適合させるために構造を変更する意志のない場合、おそらくデフォルトのプロジェクトファイル構造を修正する必要があります。`sfConfig`オブジェクトで`sf_XXX_dir`変数と`sf_XXX_dir_name`変数をオーバーライドすることで、デフォルトとはまったく異なるディレクトリ構造でsymfonyを動かすことができます。これを行う最適の場所はアプリケーションの`config.php`ファイルです。

>**CAUTION**
>`sfConfig`オブジェクトで`sf_XXX_dir`変数と`sf_XXX_dir_name`変数をオーバーライドするには、プロジェクトではなくアプリケーションの`config.php`を使います。プロジェクトの`config/config.php`ファイルは、`sfConfig`クラスがまだ存在せず`config/constants.php`ファイルがロードされていない、リクエスト期間の初期の段階でロードされます。

たとえば、すべてのアプリケーションにテンプレートのレイアウト用に1つの共通ディレクトリを共有させたい場合、`sf_template_dir`設定をオーバーライドするには以下の行を`myapp/config/config.php`ファイルに追加します:

    [php]
    sfConfig::set('sf_app_template_dir', sfConfig::get('sf_root_dir').DIRECTORY_SEPARATOR.'templates');

アプリケーションの`config.php`ファイルは空ではないので、このファイルでファイル構造の定義を格納する必要がある場合、ファイルの終わりの部分でこの作業を行うように注意してください。

### Web公開のrootディレクトリの修正

`constants.php`ファイルに組み込まれたすべてのパスはプロジェクトのrootディレクトリに依存します。このディレクトリパスはフロントコントローラー(`SF_ROOT_DIR`)によって定義されます。通常のrootディレクトリは`web/`ディレクトリの上のレベルですが、異なる構造を利用できます。メインのディレクトリ構造が2つのディレクトリから構成される場合を考えてみます。リスト19-5で示されるように、1つのディレクトリは公開領域で、もう1つのディレクトリは非公開領域に存在します。プロジェクトを共用ホスティングサービス上でホストするときにこのコンフィギュレーションを選ぶことはよくあります。

リスト19-5 - 共用サーバーのためのカスタムディレクトリ構造の例

    symfony/    # 非公開領域
      apps/
      batch/
      cache/
      ...
    www/        # 公開領域
      images/
      css/
      js/
      index.php

この場合、rootディレクトリは`symfony/`ディレクトリです。ですので、アプリケーションを動かすには`index.php`フロントコントローラーは`SF_ROOT_DIR`をつぎのように定義する必要があるだけです:

    [php]
    define('SF_ROOT_DIR', dirname(__FILE__).'/../symfony');

加えて、公開領域は通常の`web/`ディレクトリの代わりに`www/`ディレクトリで、つぎのように、アプリケーションの`config.php`ファイルのなかで2つのファイルパスをオーバーライドしなければなりません:

    [php]
    sfConfig::add(array(
      'sf_web_dir'      => SF_ROOT_DIR.'/../www',
      'sf_upload_dir'   => SF_ROOT_DIR.'/../www/'.sfConfig::get('sf_upload_dir_name'),
    ));

### symfonyのライブラリにリンクする

リスト19-6で見ることができるように、symfonyのファイルへのパスはプロジェクトの`config.php`ファイルで定義されます。

リスト19-6 - symfonyライブラリへのパス(`myproject/config/config.php`)

    [php]
    <?php

    // symfonyのディレクトリ
    $sf_symfony_lib_dir  = '/path/to/symfony/lib';
    $sf_symfony_data_dir = '/path/to/symfony/data';

`symfony init-project`タスクをコマンドラインから呼び出すとき、これらのパスは初期化されプロジェクトを作成するために使われるsymfonyの設置ディレクトリを参照します。これらはコマンドラインとMVCアーキテクチャの両方から利用されます。

このことはフレームワークファイルへのパスを変更することでsymfonyの別の設置ディレクトリに切り替えることが可能であることを意味します。

これらのパスは絶対パスですが、`dirname(__FILE__)`を利用することで、プロジェクト構造内部のファイルを参照しプロジェクトを設置するために選ばれたディレクトリの独立性を保つことができます。たとえば、つぎのコードのように、多くのプロジェクトがsymfonyの`lib/`ディレクトリをプロジェクトの`lib/symfony/`ディレクトリのシンボリックリンクとして設定し、`data/`ディレクトリに対しても同じような設定を行います:

    myproject/
      lib/
        symfony/ => /path/to/symfony/lib
      data/
        symfony/ => /path/to/symfony/data

この場合、プロジェクトの`config.php`ファイルはつぎのようにsymfonyのディレクトリを定義する必要があるだけです:

    [php]
    $sf_symfony_lib_dir  = dirname(__FILE__).'/../lib/symfony';
    $sf_symfony_data_dir = dirname(__FILE__).'/../data/symfony';

プロジェクトの`lib/vendor/`ディレクトリ内でsymfonyのファイルを`svn:externals`プロパティとして格納すること選んだ場合にも同じ原則があてはまります:

    myproject/
      lib/
        vendor/
          svn:externals symfony http://svn.symfony-project.com/branches/1.0

この場合、`config.php`ファイルはつぎのようになります:

    [php]
    $sf_symfony_lib_dir  = dirname(__FILE__).'/../lib/vendor/symfony/lib';
    $sf_symfony_data_dir = dirname(__FILE__).'/../lib/vendor/symfony/data';

>**TIP**
>アプリケーションを稼働させているサーバーが異なる場合symfonyのライブラリへのパスが異なることがあります。これを有効にする1つの方法は(`rsync_exclude.txt`に追加することで)プロジェクトの`config.php`ファイルを同期化の対象から除外することです。ほかの方法は`config.php`ファイルの開発バージョンと運用バージョンで同じパスを保つことですが、シンボリックリンクを指し示すこれらのパスがサーバーによって変わる可能性があります。

コンフィギュレーションハンドラーを理解する
----------------------

それぞれの設定ファイルはハンドラーを持ちます。コンフィギュレーションハンドラー(configuration handler)の仕事は設定カスケードを管理することと、実行時に設定ファイルを最適化して実行可能なPHPコードに変換することです。

### デフォルトのコンフィギュレーションハンドラー

デフォルトのハンドラー設定は`$sf_symfony_data_dir/config/config_handlers.yml`ファイルに保存されます。このファイルはファイルパスにしたがってハンドラーを設定ファイルにリンクします。リスト19-7はこのファイルの内容を抜粋したものです。

リスト19-7 - `$sf_symfony_data_dir/config/config_handlers.yml`ファイルの抜粋

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

`sfDefineEnvironmentConfigHandler`クラスによって処理される設定ファイルの設定は`sfConfig`クラス経由でコードのなかで直接利用できるようになり、`param`キーはプレフィックスの値を含みます。

設定ファイルを処理するために使われるハンドラーの追加もしくは修正ができます。たとえば、YAMLファイルの代わりに`INI`ファイルもしくは`XML`ファイルを使うためです。

>**NOTE**
>`config_handlers.yml`ファイル用のコンフィギュレーションハンドラーは`sfRootConfigHandler`クラスで、あきらかに変更できません。

設定の解析方法を修正する必要がある場合、アプリケーションの`cofig/`フォルダー内に空の`config_handlers.yml`ファイルを作り、`class`キーの行を書いたクラスでオーバーライドします。

### 独自ハンドラーを追加する

設定ファイルを処理するハンドラーを利用することで2つの大きな利点がもたらされます:

  * 設定ファイルはPHPの実行コードに変換され、このコードはキャッシュに保存されます。このことは、運用環境において設定は1回だけ解析されるのでパフォーマンスが最適化されていることを意味します。
  * 設定ファイルは異なるレベル(プロジェクトとアプリケーション)で定義することが可能で、最後のパラメーターの値はカスケードから由来します。プロジェクトレベルでパラメーターを定義し、アプリケーション単位でこれらをオーバーライドできます。

独自のコンフィギュレーションハンドラーを書きたい場合、`$sf_symfony_lib_dir/config/`ディレクトリのなかでsymfonyによって使われる構造の例にしたがってください。

アプリケーションが`myMapAPI`クラスを含む場合を考えてみましょう。`myMapAPI`クラスは地図を配信するサードパーティのサービスのためのインターフェイスを提供します。リスト19-8で示されるように、このクラスはURLとユーザー名で初期化することが必要です。

リスト19-8 - `myMapAPI`クラスの初期化の例

    [php]
    $mapApi = new myMapAPI();
    $mapApi->setUrl($url);
    $mapApi->setUser($user);

アプリケーションの`config/`ディレクトリ内に設置された、`map.yml`という名前のカスタム設定ファイルでこれら2つのパラメーターを保存するとよいでしょう。この設定ファイルはつぎのような内容を含むことがあります:

    api:
      url:  map.api.example.com
      user: foobar

これらの設定をリスト19-8と同等なコードに変換するために、コンフィギュレーションハンドラーを作成しなければなりません。それぞれのコンフィギュレーションハンドラーは`sfConfigHandler`クラスを拡張し`execute()`メソッドを提供しなければなりません。`execute()`メソッドはパラメーターとして設定ファイルへのファイルパスの配列が必要で、キャッシュファイルに書き込まれるデータを返さなければなりません。YAMLファイル用のハンドラーは`sfYamlConfigHandler`クラスを拡張します。このクラスはYAMLパーサーのために追加のファシリティを提供します。`map.yml`ファイルに対する典型的なコンフィギュレーションハンドラーはリスト19-9で示されるように書けます。

リスト19-9 - カスタムコンフィギュレーションハンドラー(`myapp/lib/myMapConfigHandler.class.php`)

    [php]
    <?php

    class myMapConfigHandler extends sfYamlConfigHandler
    {
      public function execute($configFiles)
      {
        $this->initialize();

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
>クラス(`class`)はオートロードする(上記の例)かファイルパスが`param`キーの元の`file`パラメーターで指定されたファイルのなかで定義しなければなりません。

アプリケーション内部で`map.yml`ファイルに基づき`myMapConfigHandler`ハンドラーによって生成されたコードが必要な場合、つぎの行を呼び出してください:

    [php]
    include(sfConfigCache::getInstance()->checkConfig(sfConfig::get('sf_app_config_dir_name').'/map.yml'));

`checkConfig()`メソッドを呼び出すとき、`map.yml.php`ファイルがキャッシュにまだ存在しないもしくは`map.yml`ファイルがキャッシュよりも新しい場合、symfonyは設定ディレクトリのなかの既存の`map.yml`ファイルを探し`config_handlers.yml`ファイルで指定されたハンドラーを利用してこれらのファイルを処理します。

>**TIP**
>YAMLの設定ファイル内部で環境を扱いたい場合、ハンドラーは`sfYamlConfigHandler`クラスの代わりに`sfDefineEnvironmentConfigHandler`クラスを拡張できます。設定を読みとるために`parseYaml()`メソッドを呼び出した後に、`mergeEnvironment()`メソッドを呼び出します。`$config = $this->mergeEnvironment($this->parseYamls ($configFiles));`を呼び出すことですべての内容を1行で実現できます。

-

>**SIDEBAR**
>既存のコンフィギュレーションハンドラーを利用する
>
>ユーザーが`sfConfig`クラス経由でコードから値を読みとることができるようにする必要があるだけなら、`sfDefineEnvironmentConfigHandler`コンフィギュレーションハンドラークラスを利用できます。たとえば、`url`と`user`パラメーターをそれぞれ`sfConfig::get('map_url')`と`sfConfig::get('map_user')`として利用できるようにするには、ハンドラーをつぎのように定義します:
>
>     config/map.yml:
>       class: sfDefineEnvironmentConfigHandler
>       param:
>         prefix: map_
>
>ほかのハンドラーによってすでに使われているプレフィックスを選ばないように気をつけてください。 既存のプレフィックスは`sf_`、`app`、と`mod_`です。

PHPの設定をコントロールする
---------------------------

アジャイル開発のルールとベストプラクティスと互換性のあるPHPの環境を保つために、symfonyは`php.ini`設定ファイルのいくつかの設定を確認してオーバーライドします。これが`php.yml`ファイルの目的です。リスト19-10はデフォルトの`php.yml`ファイルを示します。

リスト19-10 - symfonyのためのPHPのデフォルト設定(`$sf_symfony_data_dir/config/php.yml`)

    set:
      magic_quotes_runtime:        off
      log_errors:                  on
      arg_separator.output:        |
        &amp;

    check:
      zend.ze1_compatibility_mode: off

    warn:
      magic_quotes_gpc:            off
      register_globals:            off
      session.auto_start:          off

このファイルの主な目的はPHPの設定がアプリケーションと互換性があることを確認するためです。運用サーバーと同様に可能なかぎり、開発サーバーの設定を確認するためにも非常に便利です。プロジェクトの最初に運用サーバーの設定を検査し、PHPの設定をプロジェクトの`php.yml`ファイルに報告するのはそういうわけです。これによってプロジェクトを運用のプラットフォームにデプロイしてもいかなる互換性の問題に遭遇しないという信頼感を持って開発とテストを行うことができます。

`set`ヘッダーのもとで定義された変数は修正されます(サーバーの`php.ini`ファイルでどのように定義されていても)。`warn`カテゴリのもとで定義された変数はすぐに修正できませんが、これが適切に設定されていなくてもsymfonyは動きます。これらの設定をオンにすることはわるい習慣として見なされているので、この場合symfonyは警告を記録します。`check`カテゴリのもとで定義された変数は同じようにすぐに修正できませんが、symfonyを動かすためにこれらの変数は特定の値を持たなければなりません。ですので、この場合、`php.ini`ファイルが正しくなければ、例外が起こります。

symfonyのプロジェクトでエラーを追跡できるようにデフォルトの`php.yml`ファイルは`log_errors`ディレクティブを`on`に設定します。セキュリティの欠陥を防ぐために`register_globals`ディレクティブを`off`に設定することも推奨します。

これらの設定をsymfonyに適用したくない、もしくは`magic_quotes_gpc`ディレクティブと`register_globals`ディレクティブを`on`にして警告なしでプロジェクトを動かしたい場合、 アプリケーションの`config/`ディレクトリ内で`php.yml`ファイルを作り、変更したいディレクティブの値をオーバーライドしてください。

プロジェクトがPHPの追加の拡張機能を必要な場合、`extensions`カテゴリのもとで指定できます。つぎのように、このカテゴリは拡張機能の名前の配列を必要とします:

    extensions: [gd, mysql, mbstring]

まとめ
----

設定ファイル(configuration file)はsymfonyフレームワークの動作方法を大いに変更します。symfonyはコア機能とファイルの読み込みでさえも設定に依存するので、標準の専用ホストよりも多くの環境に適用できます。このすばらしい設定の柔軟性はsymfonyの主要な強さの1つです。設定ファイルのなかで学ぶべきたくさんの規約を見た初心者を怖がらせることがあるにせよ、このことによってsymfony製のアプリケーションは膨大な数のプラットフォーム、環境に対して、互換性があります。ひとたびsymfonyの設定を習得したら、あなたのアプリケーションを動かすことを拒むサーバーは存在しないでしょう。
