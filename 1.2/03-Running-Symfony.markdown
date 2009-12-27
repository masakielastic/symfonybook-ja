第3章 - symfonyを動かす
=======================

前回の章で、symfonyがPHPで書かれたファイルの集まりであることを学びました。symfonyはこれらのファイルを使うので、symfonyをインストールすることは、これらのファイルを手に入れてプロジェクトのために利用できるようになることを意味します。

symfonyは少なくともPHP 5.2が必要です。PHP 5がインストールされているか確認するにはコマンドラインを開きつぎのコマンドを入力します:

    > php -v

    PHP 5.2.5 (cli) (built: Nov 20 2007 16:55:40) 
    Copyright (c) 1997-2007 The PHP Group
    Zend Engine v2.2.0, Copyright (c) 1998-2007 Zend Technologies

バージョンが5.2かそれ以降であるならば、この章で説明されているように、あなたはインストールする準備ができています。

サンドボックスをインストールする
-------------------------------

symfonyが何をできるのか知りたいのであれば、すぐにインストールしたいことでしょう。その場合、サンドボックスが必要です。

サンドボックスはシンプルなファイルのアーカイブです。必要なすべてのライブラリ(symfony、lime、PrototypeとScriptaculous、Doctrine、PropelそしてPhing)とデフォルトのアプリケーションと基本設定を含む空のsymfonyプロジェクトが収納されています。サーバーを設定したりパッケージを追加しなくても、そのまま動きます。

インストールするには、[http://www.symfony-project.org/get/sf_sandbox_1_2.tgz](http://www.symfony-project.org/get/sf_sandbox_1_2.tgz) からsandboxアーカイブをダウンロードします。サーバー用に設定したrootディレクトリ(通常は`web/`もしくは`www/`)のもとで解凍してください。説明の一貫性を保つために、この章では、ダウンロードしたアーカイブを`sf_sandbox/`ディレクトリに解凍したことを前提とします。

>**CAUTION**
>Web公開ディレクトリのrootにすべてのファイルを設置するのはローカルホストで独自のテストを行う分にはよいですが、運用サーバーではわるい習慣です。アプリケーションの内部がエンドユーザーに見られてしまいます。

symfonyのCLI(Command Line Interface - コマンドラインインターフェイス)を実行してインストールしたsymfonyをテストします。新しい`sf_sandbox/`ディレクトリに移動し、つぎのコマンドを入力します:

    > php symfony -V

サンドボックスのバージョンが表示されます:

    symfony version 1.2.0 (/path/to/the/symfony/lib/dir/used/by/the/sandbox)

ブラウザーでつぎのURLをリクエストしてサンドボックスを閲覧できるか確認してください:

    http://localhost/sf_sandbox/web/frontend_dev.php/

図3-1のような初期(congratulations)ページが表示され、これはインストール作業が終了したことを意味します。そうではない場合、必要な設定変更を行うよう指示するエラーメッセージが表示されます。この章の後のほうの"トラブルシューティング"のセクションを参照することもできます。

図3-1 - サンドボックスの初期ページ

![サンドボックスの初期ページ](http://github.com/masakielastic/symfonybook-ja/raw/master/images/F0301.jpg "サンドボックスの初期ページ")

サンドボックスの目的はローカルコンピュータでsymfonyを練習することであり、Webに公開予定の複雑なアプリケーションを開発するためではありません。しかしながら、サンドボックスに搭載したsymfonyのバージョンは十分な機能を持ちPEARからインストールしたものと同等です。

サンドボックスをアンインストールするには、 `web/`フォルダーから`sf_sandbox/`ディレクトリを削除するだけです。

symfonyのライブラリをインストールする
-------------------------------------

アプリケーションを開発するとき、おそらくsymfonyを2回インストールする必要があります: 1回目は開発環境のため、(まだsymfonyがホストにインストールされていないのであれば)2回目はホストサーバーのためです。それぞれのサーバーのために、1つのみもしくは複数のアプリケーションを開発するとしても、symfonyのファイルを一ヶ所に保ち重複を避けたいと思うことでしょう。

symfonyフレームワークは早く進化するので、最初にインストールした数日後に新しいバージョンがリリースされる可能性があります。フレームワークのアップグレード作業は重要な問題と考える必要があります。これがすべてのsymfonyプロジェクトをまたがってsymfonyのライブラリの1つのインスタンスを共有すべきである別の理由です。

実際のアプリケーション開発のためにライブラリをインストールする方法は2つあります:

  * PEARによるインストールは多くの人にお勧めです。共有とアップグレード作業を簡単に行うことが可能で、インストール方法は単刀直入です。
  * Subversion(SVN)によるインストール方法は最新のパッチを利用する、独自の機能を追加する、かつ/もしくはsymfonyコア開発に貢献をする、などのPHP上級者が利用することだけを意図しています。

symfonyはいくつかのパッケージを統合しています:

  * limeはユニットテストのユーティリティです。
  * PropelはORMです。オブジェクトの永続性とクエリサービスを提供します。
  * Phingはモデルクラスを生成するためにPropelで使われるビルドシステムです。

limeはsymfonyの開発チームによって開発されました。Propel、Phingはほかの開発チームからもたらされ、LGPL(GNU Lesser Public General License)の下でリリースされています。これらのパッケージはsymfonyに搭載されています。

>**TIP**
>symfonyフレームワークはMITライセンスのもとで供与されます。搭載されたサードパーティのライブラリに関するすべての著作権情報は `COPYRIGHT`ファイルで見つかり関連するライセンス情報は`licenses/`ディレクトリに保存されています。

### symfonyのPEARパッケージをインストールする

symfonyのPEARパッケージにはsymfonyライブラリとすべての依存ライブラリが入ります。`symfony`コマンドを含むCLIを拡張するスクリプトも含まれます。

インストールするための最初のステップはつぎのコマンドを入力してPEARにsymfonyのチャンネルを追加することです:

    > pear channel-discover pear.symfony-project.com

チャンネルで利用可能なライブラリを見るにはつぎのコマンドを入力します:

    > pear remote-list -c symfony

これでsymfonyの最新の安定バージョンをインストールする準備ができました。つぎのコマンドを入力してください:

    > pear install symfony/symfony

    downloading symfony-1.2.0.tgz ...
    Starting to download symfony-1.1.0.tgz (1,283,270 bytes)
    .................................................................
    .................................................................
    .............done: 1,283,270 bytes
    install ok: channel://pear.symfony-project.com/symfony-1.1.0

これでお終いです。symfonyのファイルとCLIがインストールされました。バージョン番号を問い合わせる新しい`symfony`コマンドを呼び出してインストールが成功したか確認してください:

    > symfony -V

    symfony version 1.2.0 (/path/to/the/pear/symfony/lib/dir)

symfonyのライブラリはつぎのディレクトリにインストールされています:

  * `$php_dir/symfony/`には主要なライブラリが入ります。
  * `$data_dir/symfony/`にはsymfonyのdefaultモジュールによって使われるWebアセット(web assets)が入ります。
  * `$doc_dir/symfony/`にはドキュメントが入ります。
  * `$test_dir/symfony/`にはsymfonyコアのユニットテストと機能テストが入ります。

`_dir`変数はPEARの設定の一部です。これらの変数を見るには、つぎのコマンドを入力します:

    > pear config-show

### SVNリポジトリからsymfonyをチェックアウトする

運用サーバー、もしくはPEARを選択しないとき、checkoutコマンドでリクエストすることでsymfonyのSubversionリポジトリからsymfonyライブラリの最新バージョンを直接ダウンロードできます:

    > mkdir /path/to/symfony
    > cd /path/to/symfony
    > svn checkout http://svn.symfony-project.com/tags/RELEASE_1_2_0/ .

>**TIP**
>1.2 branch (1.2.x)の最新の安定版のバグ修正リリースに関しては 
>([http://www.symfony-project.org/installation/1_2](http://www.symfony-project.org/installation/1_2))を参照してください。

`symfony`コマンドは、PEARでインストールした場合のみ利用可能で、`/path/to/symfony/data/bin/symfony`スクリプトの呼び出しです。ですので、SVNでインストールした場合、つぎのコマンドが`symfony -V`と同等です:

    > php /path/to/symfony/data/bin/symfony -V

    symfony version 1.2.0 (/path/to/the/svn/symfony/lib/dir)

SVNでインストールする方法を選ぶ場合、おそらくすでにsymfonyプロジェクトがあるでしょう。symfonyのファイルを使うそのプロジェクトのために、プロジェクトの`config/ProjectConfiguration.class.php`ファイルで定義されているパスをつぎのように変更する必要があります:

    [php]
    <?php

    require_once '/path/to/symfony/lib/autoload/sfCoreAutoload.class.php';
    sfCoreAutoload::register();

    class ProjectConfiguration extends sfProjectConfiguration
    {
      // ...
    }

19章ではsymfonyプロジェクトと設置ディレクトリ(シンボリックリンクと相対パスを含む)をリンクする別の方法を提示します。

>**TIP**
>代わりに、PEARパッケージをダウンロードできます。1.2系の最新のリリースに関しては  
>([http://www.symfony-project.org/installation/1_2](http://www.symfony-project.org/installation/1_2))
>を参照してください。チェックアウトと同じ結果になります。

アプリケーションをセットアップする
----------------------------------

2章で学んだように、symfonyはプロジェクトに関連するアプリケーションを集約します。1つのプロジェクトのすべてのアプリケーションは同じデータベースを共有します。アプリケーションをセットアップするには、まずプロジェクトをセットアップしなければなりません。

### プロジェクトを作成する

symfonyプロジェクトはあらかじめ定義されたディレクトリ構造に従います。symfonyコマンドラインは適切なツリー構造とアクセス権限を収納するプロジェクトのスケルトンを初期化することで、新しいプロジェクトの作成を自動化します。プロジェクトを作成するには、プロジェクト用の新しいディレクトリを作りsymfonyコマンドを実行します。

PEARでインストールした場合、以下のコマンドを入力します:

    > mkdir ~/myproject
    > cd ~/myproject
    > symfony generate:project myproject

SVNでインストールした場合、つぎのコマンドでプロジェクトを作成します:

    > mkdir ~/myproject
    > cd ~/myproject
    > php /path/to/symfony/data/bin/symfony generate:project myproject

symfonyはつぎのようなディレクトリ構造を作ります:

    apps/
    cache/
    config/
    data/
    doc/
    lib/
    log/
    plugins/
    test/
    web/

>**TIP**
>`generate:project`タスクはプロジェクトのrootディレクトリに`symfony`スクリプトを追加します。このPHPスクリプトはPEARでインストールした`symfony`コマンドと同じことを行うので、もしネイティブなコマンドラインがサポートされない場合(SVNでインストールした場合)`symfony`の代わりに`php symfony`を呼び出します。

### アプリケーションを作る

プロジェクトを閲覧する準備がまだできていません。少なくとも1つのアプリケーションが必要だからです。アプリケーションを初期化するには、`symfony generate:app`コマンドに引数としてアプリケーションの名前を渡します:

    > php symfony generate:app frontend

このコマンドによってプロジェクトのrootの`apps/`フォルダーのもとで`frontend/`ディレクトリが作られます。これらはアプリケーションのデフォルト設定とディレクトリの一式を持ちWebサイトのファイルをホストする準備ができています:

    apps/
      frontend/
        config/
        i18n/
        lib/
        modules/
        templates/

それぞれのデフォルト環境に対応するPHPファイルのなかにはプロジェクトの`web`ディレクトリのもとで作られるものもあります:

    web/
      index.php
      frontend_dev.php

`index.php`は新しいアプリケーションの運用環境用のフロントコントローラーです。プロジェクトのアプリケーションを作りましたので、symfonyが`frontend.php`の代わりに`index.php`という名前のファイルを作りました(もし`backend`という名前の新しいアプリケーションを作った場合、新しい運用環境用のフロントコントローラーは`backend.php`という名前になります)。開発環境でアプリケーションを動かすには、フロントコントローラーである`frontend_dev.php`を呼び出します。ここで留意すべきは、セキュリティ上の理由により、デフォルトでは開発環境のコントローラーを使えるのはlocalhostのみです。5章でこれらの環境の詳細を学びます。

`symfony`コマンドはいつもかならずプロジェクトのrootディレクトリ(先の例では`myproject/`)から呼び出さなければなりません。なぜならこのコマンドによって実行される全てのタスクはプロジェクト固有のものだからです。

Webサーバーを設定する
----------------------

`web/`ディレクトリのスクリプトはアプリケーションへのエントリーポイント(入り口)です。インターネットからアクセスできるようにするには、Webサーバーを設定しなければなりません。プロフェッショナルなホスティング会社と同じように、あなたは開発サーバーのApacheを設定する権限を持ち、バーチャルホストをセットアップできるでしょう。共用サーバーではおそらく`.htaccess`ファイルへのアクセス権があるだけです。

### バーチャルホストをセットアップする

リスト3-1はApacheの設定例で、`httpd.conf`ファイルに新しいバーチャルホストを追加します(訳注：Apache2.2以降では`httpd-vhosts.conf`などのバーチャルホスト専用のファイルが用意されています)。

リスト3-1 `apache/conf/httpd.conf`の設定サンプル

    <VirtualHost *:80>
      ServerName myapp.example.com
      DocumentRoot "/home/steve/myproject/web"
      DirectoryIndex index.php
      Alias /sf /$sf_symfony_data_dir/web/sf
      <Directory "/$sf_symfony_data_dir/web/sf">
        AllowOverride All
        Allow from All
      </Directory>
      <Directory "/home/steve/myproject/web">
        AllowOverride All
        Allow from All
      </Directory>
    </VirtualHost>

リスト3-1の設定では、`$sf_symfony_data_dir`プレースホルダーは実際のパスに置き換えなければなりません。たとえば、Unix系でPEAR版のパッケージをインストールした場合、つぎのようなディレクティブを記入します:

    Alias /sf /usr/local/lib/php/data/symfony/web/sf

>**NOTE**
>`web/sf/`ディレクトリのエイリアスは必須ではありません。この設定によってApacheはWebデバッグツールバー、adminジェネレーター、デフォルトページ、Ajaxサポート用の画像、スタイルシート、JavaScriptを見つけられるようになります。このエイリアスの代わりの方法はシンボリックリンクを作るか`/path/to/symfony/data/web/sf/`ディレクトリを`myporject/web/sf/`ディレクトリにコピーすることです。

-

>**TIP**
>PEARを通してsymfonyをインストールして
>symfonyの共有データが見つからない場合、PEARのconfigコマンドで表示される`data_dir`を見てください:
>
>    pear config-show

Apacheを再起動すれば作業は終了です。つぎのURLから新たに作られたアプリケーションを標準的なブラウザーで見ることができます:

    http://localhost/frontend_dev.php/

最初の方で示した図3-1に似た初期ページが見えます。

>**SIDEBAR**
>URLの書き換え
>
>symfonyは"スマートURL"を表示するためにURLの書き換え機能(rewriting)を利用します。これによって検索エンジンには意味のあるロケーションを表示し、技術的なすべてのデータをユーザーから隠します。このルーティング(routing)と呼ばれる機能は9章で詳しく学びます。
>
>Apacheが`mod_rewrite`モジュールでコンパイルされていない場合、DSO(Dynamic Shared Object - 動的共有オブジェクト)の`mod_rewrite`がインストールされておりつぎの行が`http.conf`のなかに存在することを確認してください。
>
>     AddModule mod_rewrite.c
>     LoadModule rewrite_module modules/mod_rewrite.so
>
>Internet Information Services(IIS)の場合、`isapi/rewrite`がインストールされている状態で稼働していることが必須です。詳細なIISのインストールガイドに関してはsymfony公式サイトのcookbookを確認してください。

### 共用ホストのサーバーを設定する

共用サーバーでアプリケーションをセットアップするには少し巧妙なやりかたが必要です。通常のホストでは利用者はディレクトリのレイアウトを変更できないからです。

>**CAUTION**
>共用サーバーではテストと開発を直接行うことはよい習慣ではありません。1つの理由はテストと開発が終了していなくても、アプリケーションを見ることができるので、内部情報が流出し、大きなセキュリティの欠陥を公開するからです。ほかの理由はデバッグツールでアプリケーションを効率よく見るには共用ホストのパフォーマンスが不十分だからです。ですので最初から共用ホストにインストールして開発を始めるべきではなく、ローカルでアプリケーションを開発して開発が終了してから共用ホストにデプロイすべきです。16章でデプロイの技術とツールについて詳しく説明をします。

Webフォルダーは`web/`の代わりに`www/`と名づけ、`httpd.conf`にアクセスすることは不可能でWebフォルダーの`.htaccess`ファイルのみにアクセス可能である共用ホストを想像してください。

symfonyプロジェクトにおいて、ディレクトリへのすべてのパスを設定することは可能です。19章で詳しく説明しますが、しばらくの間は`web`ディレクトリを`www`にリネームします。リスト3-2で示されるように設定を変更すればアプリケーションはこの設定を考慮するようになります。つぎの行を`config/ProjectConfiguration.class.php`ファイルの末尾に追加します。

リスト3-2 - ディレクトリ構造のデフォルト設定を変更する(`config/ProjectConfiguration.class.php`)

    [php]
    class ProjectConfiguration extends sfProjectConfiguration
    {
       public function setup()
       {
         $this->setWebDir($this->getRootDir().'/www');
       }
    }

デフォルトではプロジェクトのWeb公開ディレクトリのrootには`.htaccess`ファイルが入ります。このファイルの内容はリスト3-3で示されています。あなたの共用ホストの要件を満たすように修正してください。

リスト3-3 - `myproject/www/.htaccess`のデフォルト設定

    Options +FollowSymLinks +ExecCGI

    <IfModule mod_rewrite.c>
      RewriteEngine On

      # もしno_script_nameを機能させると問題が発生するなら
      # つぎの行のコメントを解除する
      #RewriteBase /

      # .something拡張子を持つファイルをすべてスキップする
      #RewriteCond %{REQUEST_URI} \..+$
      #RewriteCond %{REQUEST_URI} !\.html$
      #RewriteRule .* - [L]

      # .htmlバージョンがここにあるか確認する(キャッシュ)
      RewriteRule ^$ index.html [QSA]
      RewriteRule ^([^.]+)$ $1.html [QSA]
      RewriteCond %{REQUEST_FILENAME} !-f

      # いいえ、Webのフロントコントローラーにリダイレクトする
      RewriteRule ^(.*)$ index.php [QSA,L]
    </IfModule>

アプリケーションをブラウザーで見る準備ができました。つぎのURLをリクエストして初期ページを確認してください:

    http://www.example.com/frontend_dev.php/

>**SIDEBAR**
>そのほかのサーバーの設定
>
>symfonyはそのほかのサーバーの設定に対して互換性があります。たとえば、バーチャルホストの代わりにエイリアスを使ってsymfonyのアプリケーションにアクセスできます。IISサーバーでもsymfonyのアプリケーションを動かすことができます。設定の数と同じぐらいたくさんのテクニックがあり、それらすべてを説明することはこの本の目的ではありません。
>
>特定のサーバー設定のための手引きを見つけるには、段階的なチュートリアルを多く掲載するsymfony公式サイトのwiki([http://trac.symfony-project.org](http://trac.symfony-project.org))を参照してください。

トラブルシューティング
----------------------

インストール作業の最中に問題に遭遇したら、シェルかブラウザーに表示されるエラーもしくは例外を最大限活用してください。得られる情報はしばしば一目瞭然で、問題に関するWeb上の詳細なリソースへのリンクを含むこともあります。

### 典型的な問題

symfonyを動かすことに関してまだ問題を抱えている場合、つぎの項目を確認してください:

  * インストールされているPHPのなかにはPHP 4とPHP 5のコマンドが付属しているものがあります。この場合、コマンドラインはおそらく`php`の代わりに`php5`です。`symfony`コマンドの代わりに、`php5 symfony`を呼び出してみてください。`SetEnv PHP_VER 5`を`.htaccess`の設定に追加する、`web/`ディレクトリのスクリプトを`.php`から`.php5`にリネームすることなども必要かもしれません。PHP 4のコマンドラインでsymfonyにアクセスしようとするとつぎのようなエラーが表示されます:

        Parse error, unexpected ',', expecting '(' in .../symfony.php on line 19.

  * `php.ini`で定義されるメモリの制限は少なくとも`32M`にしなければなりません。この問題に関する通常の症例はsymfonyをPEARもしくはコマンドライン経由でインストールしているときにエラーメッセージが表示されることです。

        Allowed memory size of 8388608 bytes exhausted

  * `php.ini`の`zend.ze1_compatibility_mode`ディレクティブは`off`にしなければなりません。そうではない場合、Webスクリプトの1つをブラウザーで見ようとすると、"implict cloning"のエラーが表示されます:

        Strict Standards: Implicit cloning object of class 'sfTimer'because of 'zend.ze1_compatibility_mode'

  * プロジェクトの`log/`と`cache/`ディレクトリはWebサーバーが書き込みできるようにしなければなりません。これらのディレクトリのパーミッションを持たないsymfonyアプリケーションをブラウザーで見ようとすると例外が表示されて止まります:

        sfCacheException [message] Unable to write cache file"/usr/myproject/cache/frontend/prod/config/config_config_handlers.yml.php"

  * システムの環境変数PATHには`php`コマンドへのパス(訳注:通常のUnix系であれば/usr/bin)が含まれなければならず、(PEARを利用する場合)`php.ini`のincludeディレクティブにはPEARへのパスが含まれなければなりません。

  * (たとえばWAMPパッケージを利用する場合など)サーバーのファイルシステムには複数の`php.ini`が存在することがあります(訳注:たとえばXAMPPであればCLI用にxampp/php、Apache用にxampp/apache2/binにそれぞれ1つずつ)。アプリケーションで使われる`php.ini`の正確な位置を知るには、`phpinfo()`関数を使います。

>**NOTE**
>強制ではありませんが、パフォーマンス上の理由から`php.ini`の`magic_quotes_gpc`と`register_globals`ディレクティブを`off`にしておくことを強くお勧めします。

### symfonyのリソース

あなたの問題がほかの人がすでに遭遇した問題なのか、またさまざまなサイトで書かれた解決方法を見つけるにはつぎの場所を通して調べることができます:

  * symfonyのインストール方法に関するフォーラム([http://www.symfony-project.org/forum/](http://www.symfony-project.org/forum/)) はプラットフォーム、環境、設定、ホストなどのすべてインストールの質問を扱っています。
  * ユーザーのメーリングリストのアーカイブ ([http://groups.google.fr/group/symfony-users](http://groups.google.fr/group/symfony-users)) も検索できます。あなた自身のものと似たような体験談が見つかるかもしれません。
  * 公式サイトのwiki ([http://trac.symfony-project.org/#Installingsymfony](http://trac.symfony-project.org/#Installingsymfony)) にはインストール方法についてsymfonyのユーザーから寄稿された段階的なチュートリアルが掲載されています。

回答が見つからない場合、symfonyのコミュニティに質問してください。もっとも活動的なコミュニティのメンバーからのフィードバックを得るにはフォーラム、メーリングリスト、`#symfony` IRCチャンネルを利用できます。

ソースコードのバージョン管理
---------------------------

アプリケーションをセットアップした時点で、バージョン管理を始めることをお勧めします。バージョン管理によってコードのすべての修正を追跡し、以前のリリースにアクセスすることが可能になります。また円滑なパッチ作業を行い、そしてチームの作業が効率的になります。symfonyはネイティブでCVSをサポートしますが、Subversion([http://subversion.tigris.org/](http://subversion.tigris.org/))がお勧めです。つぎの例はSubversionのコマンドを示しており、すでにSubversionサーバーをインストールしてプロジェクトのために新しいリポジトリを作ることを前提とします。Windowsユーザーの場合、お勧めのSubversionクライアントはTortoiseSVN([http://tortoisesvn.tigris.org/](http://tortoisesvn.tigris.org/))です。バージョン管理とここで言及されたコマンドに関して詳しい情報はSubversionのドキュメント(訳注：[「Subversionによるバージョン管理」](http://svnbook.red-bean.com/index.html)を参照))

つぎの例は`$SVNREP_DIR`が環境変数として定義されていることを前提とします。定義されていない場合、`$SVNREP_DIR`の位置をリポジトリの実際の位置に置き換える必要があります。

`myproject`プロジェクトのために新しいリポジトリを作ります:

    > svnadmin create $SVNREP_DIR/myproject

リポジトリの基本構造(レイアウト)は`trunk`、`tags`と`branches`ディレクトリで構成されつぎの少々長いコマンドで作ります:

    > svn mkdir -m "layout creation" file:///$SVNREP_DIR/myproject/trunk file:///$SVNREP_DIR/myproject/tags file:///$SVNREP_DIR/myproject/branches

これは最初のリビジョンになります。つぎにcacheとlogのなかの一時ファイル以外のプロジェクトのファイルをインポートする必要があります:

    > cd ~/myproject
    > rm -rf cache/*
    > rm -rf log/*
    > svn import -m "initial import" . file:///$SVNREP_DIR/myproject/trunk

つぎのコマンドを入力してコミットされたファイルを確認します:

    > svn ls file:///$SVNREP_DIR/myproject/trunk/

よさそうです。SVNリポジトリはすべてのプロジェクトファイルの参照バージョン(と履歴)を保存します。このことは実際の`~/myproject/`ディレクトリのファイルはリポジトリを参照する必要があるということを意味します。そのために、最初に`myproject/`ディレクトリをリネームし、新しいディレクトリのもとでリポジトリをチェックアウトします。うまくいったら削除します:

    > cd ~
    > mv myproject myproject.origin
    > svn co file:///$SVNREP_DIR/myproject/trunk myproject
    > ls myproject

これでお終いです。`~/myproject/`に設置されたファイルで作業を行い、修正をリポジトリにコミットできます。不要な`myproject.origin/`ディレクトリを削除することをお忘れなく。

まだ必要なセットアップ作業が残っています。あなたの作業(カレント)ディレクトリをリポジトリにコミットした場合、プロジェクトの`cache`や`log`ディレクトリに置かれたいくつかのファイルを意図せずにコピーしてしまう可能性があります。このプロジェクトのためにSVNが無視するリストを指定する必要があります。また再び`cache/`と`log/`ディレクトリに777のパーミッションを設定する必要があります:

    > cd ~/myproject
    > chmod 777 cache
    > chmod 777 log
    > svn propedit svn:ignore log
    > svn propedit svn:ignore cache

SVNで設定されたデフォルトのテキストエディタが起動します。もし起動しない場合、つぎのコマンドを入力してSubversionが好みのエディタを使うように設定してください:

    > export SVN_EDITOR=<name of editor>
    > svn propedit svn:ignore log
    > svn propedit svn:ignore cache

logとcacheのなかのすべてのファイルを無視するようにアスタリスクを追加します:

    *

保存したら作業は終了です。

まとめ
----

ローカルサーバーでsymfonyを試して遊ぶためのベストな方法はサンドボックスです。サンドボックスにはあらかじめ設定されたsymfonyの環境が含まれるからです。

実際に開発するには、もしくは運用サーバーにおいて、PEARによるインストールもしくはSVNのチェックアウトする方法を選びます。これらの作業を通してsymfonyライブラリをインストールし、プロジェクトとアプリケーションを初期化することも必要です。 アプリケーションのセットアップの最終段階はサーバーの設定で、多くの方法があります。symfonyはApacheのバーチャルホストで完璧に動作し、これが推奨の解決方法です。

インストールの最中に何か問題に遭遇しましたら、symfonyの公式サイトで多くのチュートリアルとよく聞かれる質問への回答を調べます。必要であれば、symfonyのコミュニティであなたの問題を質問すればすぐに回答を得られるでしょう。

いったんプロジェクトが初期化されたら、バージョン管理を始めることはよい習慣です。

これでsymfonyを使う準備ができたので、基本的なWebアプリケーションを開発する方法を見る段階にあります。
