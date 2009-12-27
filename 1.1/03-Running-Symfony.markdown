第3章 - symfonyを動かす
=======================

前回の章で、symfonyがPHPで書かれたファイルの集まりであることを学びました。symfonyはこれらのファイルを使うので、symfonyをインストールすることは、これらのファイルを手に入れてプロジェクトのために利用できるようになることを意味します。

symfonyは少なくともPHP 5.1が必要です。PHP 5がインストールされているか確認をするにはコマンドラインを開きつぎのコマンドを入力します:

    > php -v

    PHP 5.2.5 (cli) (built: Nov 20 2007 16:55:40) 
    Copyright (c) 1997-2007 The PHP Group
    Zend Engine v2.2.0, Copyright (c) 1998-2007 Zend Technologies

バージョンが5.1かそれ以降であるならば、この章で説明されているように、あなたはインストールする準備ができています。

サンドボックスをインストールする
--------------------------------

symfonyが何をできるのか知りたいのであれば、すぐにインストールしたいことでしょう。その場合、サンドボックスが必要です。

サンドボックスはシンプルなファイルのアーカイブです。必要なライブラリ(symfony、lime、Creole、PropelそしてPhing)のすべてとデフォルトのアプリケーションと基本設定を含む空のsymfonyプロジェクトが含まれます。特別なサーバーの設定やパッケージを追加しなくても、そのまま動きます。

インストールするには、[http://www.symfony-project.org/get/sf_sandbox_1_1.tgz](http://www.symfony-project.org/get/sf_sandbox_1_1.tgz) からsandboxアーカイブをダウンロードしてください。サーバー用に設定したrootディレクトリ(通常は`web/`もしくは`www/`)の下に解凍してください。統一性を保つために、この章では、ダウンロードしたアーカイブを`sf_sandbox/`ディレクトリに解凍したことを前提とします。

>**CAUTION**
>Web公開ディレクトリのrootにすべてのファイルを設置するのはローカルホストで独自のテストを行う分にはいいですが、運用サーバーではわるい習慣です。あなたのアプリケーションの内部がエンドユーザーに見られてしまいます。

symfonyのCLI(Command Line Interface - コマンドラインインターフェイス)を実行してインストールしたsymfonyをテストしてください。新しい`sf_sandbox/`ディレクトリに移動し、つぎのコマンドを入力します:

    > php symfony -V

サンドボックスの番号が表示されます:

    symfony version 1.1.0 (/path/to/the/symfony/lib/dir/used/by/the/sandbox)

つぎのURLをリクエストしてブラウザーでサンドボックスを閲覧できるか確認してください:

    http://localhost/sf_sandbox/web/frontend_dev.php/

図3-1のような初期ページが表示され、これはインストール作業が終了したことを意味します。そうではない場合、必要な設定変更を行うよう指示するエラーメッセージが表示されます。この章の後のほうの"トラブルシューティング"のセクションを参照することもできます。

図3-1 - サンドボックスの初期ページ

![サンドボックスの初期ページ](http://github.com/masakielastic/symfonybook-ja/raw/master/images/F0301.jpg "サンドボックスの初期ページ")

サンドボックスの目的はローカルコンピュータ上でsymfonyを練習することであり、Webに公開する予定の複雑なアプリケーションを開発するためではありません。しかしながら、サンドボックスを搭載したsymfonyのバージョンは十分な機能を持ちPEARからインストールしたものと同等です。

サンドボックスをアンインストールするには、 `web/`フォルダーから`sf_sandbox/`ディレクトリを削除するだけです。

symfonyのライブラリをインストールする
-------------------------------------

アプリケーションを開発するとき、おそらくsymfonyを2回インストールする必要があります: 1回目は開発環境のため、(すでにホストにsymfonyがインストールされていないのであれば)2回目はホストサーバーのためです。それぞれのサーバーのために、1つのアプリケーションだけ、もしくはいくつかのアプリケーションを開発していようとも、1つの場所にsymfonyのファイルを保存することで重複を避けたいと思うでしょう。

symfonyフレームワークは早く進化するので、最初にインストールした数日後に新しいバージョンがリリースされる可能性があります。フレームワークのアップグレード作業は主要な問題と考える必要があります。これがすべてのsymfonyのプロジェクトをまたがってsymfonyのライブラリの1つのインスタンスを共有すべきである別の理由です。

実際のアプリケーション開発のためにライブラリをインストールする方法に関しては、代わりの方法が2つあります:

  * PEARによるインストールは多くの人にお勧めです。共有とアップグレード作業を簡単に行うことが可能で、インストール方法は単刀直入です。
  * Subversion(SVN)によるインストール方法は最新のパッチを利用する、独自の機能を追加する、かつ/もしくはsymfonyのプロジェクトに貢献をする、などのPHPの上級者が利用することだけを意図しています。

symfonyはいくつかのパッケージを統合しています:

  * limeはユニットテストのユーティリティです。
  * Creoleはデータベース抽象化エンジンです。PHP Data Objects(PDO)のようなもので、あなたのコードとデータベースのSQLコード間のインターフェイスを提供し、ほかのエンジンに切り替えることを可能にします。 
  * PropelはORMです。オブジェクトの永続性とクエリサービスを提供します。
  * Phingはモデルクラスを生成するためにPropelで使われるビルドシステムです。

limeはsymfonyのチームによって開発されました。Creole、Propel、Phingは別のチームからもたらされ、LGPL(GNU Lesser Public General License)の下でリリースされています。これらのパッケージはsymfonyに搭載されています。

>**TIP**
>symfonyフレームワークはMITライセンスのもとで供与されます。搭載されたサードパーティのライブラリに関するすべての著作権情報は `COPYRIGHT`ファイルで見つかり関連するライセンスは`licenses/`ディレクトリに保存されています。

### symfonyのPEARパッケージをインストールする

symfonyのPEARパッケージはsymfonyライブラリとすべての依存関係を含みます。`symfony`コマンドを含むCLIを拡張するスクリプトも含まれます。

インストールするための最初のステップはつぎのコマンドを入力してsymfonyのチャンネルをPEARに追加することです:

    > pear channel-discover pear.symfony-project.com

チャンネルで利用可能なライブラリを見るにはつぎのコマンドを入力します:

    > pear remote-list -c symfony

これでsymfonyの最新の安定バージョンをインストールする準備ができました。つぎのコマンドを入力してください:

    > pear install symfony/symfony

    downloading symfony-1.1.0.tgz ...
    Starting to download symfony-1.1.0.tgz (1,283,270 bytes)
    .................................................................
    .................................................................
    .............done: 1,283,270 bytes
    install ok: channel://pear.symfony-project.com/symfony-1.1.0

これでお終いです。symfonyのファイルとCLIがインストールされました。バージョン番号を問い合わせる新しい`symfony`コマンドを呼び出してインストールが成功したか確認してください:

    > symfony -V

    symfony version 1.1.0 (/path/to/the/pear/symfony/lib/dir)

symfonyのライブラリはつぎのディレクトリにインストールされています:

  * `$php_dir/symfony/` は主要なライブラリを含みます。
  * `$data_dir/symfony/` symfonyのデフォルトモジュールによって使われるWebアセット(web assets)を含みます。
  * `$doc_dir/symfony/` ドキュメントを含みます。
  * `$test_dir/symfony/` symfonyのコアのユニットテストと機能テストを含みます。

`_dir`変数はPEARの設定の一部です。これらの変数を見るために、つぎのコマンドを入力してください:

    > pear config-show

### SVNリポジトリからsymfonyをチェックアウトする

運用サーバー、もしくはPEARを選択しないとき、checkoutコマンドでリクエストすることでsymfonyのSubversionリポジトリから最新バージョンのsymfonyのライブラリを直接ダウンロードできます:

    > mkdir /path/to/symfony
    > cd /path/to/symfony
    > svn checkout http://svn.symfony-project.com/tags/RELEASE_1_1_0/ .

>**TIP**
>1.1 branch(1.1.x)の最新の安定版のバグ修正リリースに関しては 
>([http://www.symfony-project.org/installation/1_1](http://www.symfony-project.org/installation/1_1))を参照してください。

`symfony`コマンドは、PEARでインストールした場合のみ利用可能で、`/path/to/symfony/data/bin/symfony`スクリプトへの呼び出しです。ですので、SVNでインストールした場合、つぎのコマンドは`symfony -V`と同等です:

    > php /path/to/symfony/data/bin/symfony -V

    symfony version 1.1.0 (/path/to/the/svn/symfony/lib/dir)

SVNでインストールすることを選択する場合、おそらくあなたは既存のsymfonyのプロジェクトを持っています。symfonyのファイルを利用するこのプロジェクトのために、あなたのプロジェクトの`config/ProjectConfiguration.class.php`ファイルで定義されているパスをつぎのように変更する必要があります:

    [php]
    <?php

    require_once '/path/to/symfony/lib/autoload/sfCoreAutoload.class.php';
    sfCoreAutoload::register();

    class ProjectConfiguration extends sfProjectConfiguration
    {
      // ...
    }

19章ではプロジェクトとsymfonyの設置ディレクトリ(シンボリックリンクと相対パスを含む)をリンクする別の方法を提示します。

>**TIP**
>代わりに、PEARパッケージをダウンロードできます。1.1系の最新のリリースに関しては  
>([http://www.symfony-project.org/installation/1_1](http://www.symfony-project.org/installation/1_1))
>を参照してください。チェックアウトと同じ結果になります。

アプリケーションをセットアップする
----------------------------------

2章で学んだように、symfonyはプロジェクトに関連するアプリケーションを集結させます。1つのプロジェクトのすべてのアプリケーションは同じデータベースを共有します。アプリケーションをセットアップするには、まずプロジェクトをセットアップしなければなりません。

### プロジェクトを作成する

簡単なsymfonyのプロジェクトはあらかじめ定義されたディレクトリ構造に従います。symfonyコマンドラインは適切なツリー構造とアクセス権限を含む、プロジェクトのスケルトンを初期化することで、新しいプロジェクトの作成を自動化します。プロジェクトを作成するには、新しいディレクトリを作り、そのディレクトリをプロジェクトにするようにsymfonyに命令します。

PEARでインストールした場合、以下のコマンドを入力します:

    > mkdir ~/myproject
    > cd ~/myproject
    > symfony generate:project myproject

SVNでインストールした場合、つぎのコマンドでプロジェクトを作ります:

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
>この`generate:project`タスクはプロジェクトのrootディレクトリに`symfony`のスクリプトを追加します。このPHPスクリプトはPEARでインストールした`symfony`コマンドと同じことを行うので、もしネイティブなコマンドラインのサポートがない場合(SVNでインストールした場合)`symfony`の代わりに`php symfony`を呼び出すことができます。

### アプリケーションを作成する

プロジェクトを閲覧する準備がまだできていません。なぜなら少なくとも1つのアプリケーションが必要だからです。アプリケーションを初期化するには、`symfony generate:app`コマンドを使いアプリケーションの名前を引数として渡します:

    > php symfony generate:app frontend

このコマンドによってプロジェクトのrootの`apps/`フォルダーに`frontend/`ディレクトリが作られます。これらはデフォルトのアプリケーション設定とディレクトリの一式を持ちWebサイトのファイルをホストする準備ができています:

    apps/
      frontend/
        config/
        i18n/
        lib/
        modules/
        templates/

それぞれのデフォルト環境に対応するPHPファイルのなかにはプロジェクトの`web`ディレクトリに作られるものもあります:

    web/
      index.php
      frontend_dev.php

`index.php`は新しいアプリケーションの運用環境用のフロントコントローラーです。プロジェクトのアプリケーションを作成したので、symfonyが`frontend.php`の代わりに`index.php`という名前のファイルを作成しました。(もし`backend`という名前の新しいアプリケーションを作成した場合、新しい運用環境用のフロントコントローラーは`backend.php`という名前になります)。開発環境でアプリケーションを動かすには、フロントコントローラーである`frontend_dev.php`を呼び出します。ここで留意すべきは、開発環境のコントローラーはセキュリティ上の理由によりデフォルトではlocalhostだけが使用可能になっています。5章でこれらの環境の詳細を学びます。

`symfony`コマンドはいつもかならずプロジェクトのルートディレクトリ(先の例では`myproject/`)から呼ばれなければなりません、なぜならこのコマンドによって実行される全てのタスクはプロジェクト固有のものだからです。

Webサーバーを設定する
----------------------

`web/`ディレクトリのスクリプトはアプリケーションへのエントリーポイント(入り口)です。インターネットからアクセスできるようにするため、Webサーバーを設定しなければなりません。プロフェッショナルなホスティング会社と同じように、あなたは開発サーバーのApacheを設定する権限が持ち、バーチャルホストをセットアップできるでしょう。共用サーバーにおいてはおそらく`.htaccess`ファイルへのアクセス権があるだけです。

### バーチャルホストをセットアップする

リスト3-1はApacheの設定例で、新しいバーチャルホストが`httpd.conf`ファイルに追加されます(訳注：Apache2.2以降では`httpd-vhosts.conf`などのバーチャルホスト専用のファイルが用意されています)。

リスト3-1 Apacheの設定サンプル(`apache/conf/httpd.conf`)

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

リスト3-1の設定では、`$sf_symfony_data_dir`プレースホルダーは実際のパスに置き換えなければなりません。たとえば、Unix系でPEAR版のパッケージをインストールした場合、つぎのようなディレクティブを入力します:

    Alias /sf /usr/local/lib/php/data/symfony/web/sf

>**NOTE**
>`web/sf/`ディレクトリのエイリアスは必須ではありません。この設定によってApacheはWebデバッグツールバー、adminジェネレーター、デフォルトのsymfonyのページ、Ajaxサポートのための画像、スタイルシート、JavaScriptを見つけるこができるようになります。このエイリアスの代わりの方法はシンボリックリンクを作るか`/path/to/symfony/data/web/sf/`ディレクトリを`myporject/web/sf/`ディレクトリにコピーすることです。

-

>**TIP**
>PEARを通してsymfonyをインストールして
>symfonyの共有データが見つからない場合、PEARのconfigコマンドで表示される`data_dir`を見てください:
>
>    pear config-show

Apacheを再起動すれば作業は終わりです。つぎのURLに標準的なブラウザーを通して新しく作られたアプリケーションを呼び出して見ることができます:

    http://localhost/frontend_dev.php/

最初の方で示した図3-1に似た初期ページが見えます。

>**SIDEBAR**
>URLの書き換え
>
>symfonyは"スマートURL"を表示するためにURLの書き換え機能(rewriting)を利用します。これによって検索エンジンには意味のあるロケーションを表示し、技術的なデータのすべてをユーザーから隠します。このルーティング(routing)と呼ばれる機能は9章で詳しく学びます。
>
>Apacheのバージョンが`mod_rewrite`モジュールでコンパイルされていない場合、DSO(Dynamic Shared Object - 動的共有オブジェクト)である`mod_rewrite`がインストールされておりつぎの行が`http.conf`に存在することを確認してください
>
>    AddModule mod_rewrite.c
>    LoadModule rewrite_module modules/mod_rewrite.so
>
>Internet Information Services(IIS)の場合、`isapi/rewrite`がインストールされて稼働していることが必要です。詳細なIISのインストールガイドに関してはsymfonyのオンラインのcookbookを確認してください。

### 共用ホストのサーバーを設定する

共用サーバーでアプリケーションをセットアップするには少々巧妙な方法が必要です。通常ホストは利用者が変更できない固有のディレクトリのレイアウトを持つからです。

>**CAUTION**
>テストと開発を共用サーバーで直接行うことはよい習慣ではありません。1つの理由はテストと開発が終了していなくても、アプリケーションを見ることができるので、内部情報が流出し、大きなセキュリティの欠陥を公開するからです。ほかの理由はデバッグツールでアプリケーションを効率的に閲覧するには共用ホストのパフォーマンスが不十分だからです。ですので最初から共用ホストにインストールして開発を始めるべきではなく、ローカルでアプリケーションを開発して開発が終了してから共用ホストにデプロイすべきです。16章でデプロイの技術とツールについて詳しく説明をします。

Webフォルダーは`web/`の代わりに`www/`と名づけ、`httpd.conf`にアクセスすることは不可能でWebフォルダーの`.htaccess`ファイルのみにアクセス可能である共用ホストを想像してください。

symfonyのプロジェクトにおいて、ディレクトリへのすべてのパスを設定することは可能です。19章で詳しく説明しますが、しばらくの間は、`web`ディレクトリを`www`にリネームして、リスト3-2で示されるように設定を変更することでアプリケーションはこの設定を考慮するようになります。これらの行を`config/ProjectConfiguration.class.php`ファイルの最後に追加します。

リスト3-2 - デフォルトのディレクトリ構造の設定を変更する(`config/ProjectConfiguration.class.php`)

    [php]
    class ProjectConfiguration extends sfProjectConfiguration
    {
       public function setup()
       {
         $this->setWebDir($this->getRootDir().'/www');
       }
    }

プロジェクトのWeb公開ディレクトリのrootにはデフォルトで`.htaccess`ファイルが入ります。このファイルの内容はリスト3-3で示されています。あなたの共用ホストの要件を満たすように修正してください。

リスト3-3 - `.htaccess`のデフォルト設定(`myproject/www/.htaccess`)

    Options +FollowSymLinks +ExecCGI

    <IfModule mod_rewrite.c>
      RewriteEngine On

      # もしno_script_nameを機能させると問題が発生するなら
      # つぎの行のコメントを解除する
      #RewriteBase /

      # .somethingを持つファイルをすべてスキップする
      #RewriteCond %{REQUEST_URI} \..+$
      #RewriteCond %{REQUEST_URI} !\.html$
      #RewriteRule .* - [L]

      # .htmlバージョンがここにあるか(キャッシュ)確認する
      RewriteRule ^$ index.html [QSA]
      RewriteRule ^([^.]+)$ $1.html [QSA]
      RewriteCond %{REQUEST_FILENAME} !-f

      # いいえ、Webのフロントコントローラーにリダイレクトする
      RewriteRule ^(.*)$ index.php [QSA,L]
    </IfModule>

あなたのアプリケーションをブラウザーで見る準備ができました。つぎのURLをリクエストして初期ページを確認してください:

    http://www.example.com/frontend_dev.php/

>**SIDEBAR**
>そのほかのサーバーの設定
>
>symfonyはそのほかのサーバーの設定に対して互換性があります。たとえば、バーチャルホストの代わりにエイリアスを使用してsymfonyのアプリケーションにアクセスできます。IISサーバーでもsymfonyのアプリケーションを動かすことができます。設定の数と同じぐらいたくさんのテクニックがあり、それらすべてを説明することはこの本の目的ではありません。
>
>特定のサーバー設定のための手引きを見つけるには、段階的なチュートリアルを多く掲載するsymfony公式サイトの wiki([http://trac.symfony-project.org](http://trac.symfony-project.org))を参照してください。

トラブルシューティング
----------------------

インストール作業の最中に問題に遭遇したら、シェルかブラウザーに投じられたエラーもしくは例外を最大限活用してください。得られる情報はしばしば一目瞭然で、あなたの問題に関するWeb上の詳細なリソースへのリンクを含むこともあります。

### 典型的な問題

symfonyを動かすことに関してまだ問題を抱えている場合、つぎの項目を確認してください:

  * PHPインストレーションのなかにはPHP 4とPHP 5のコマンドが付属しているものがあります。この場合、コマンドラインはおそらく`php`の代わりに`php5`です。`symfony`コマンドの代わりに、`php5 symfony`を呼び出してみてください。`SetEnv PHP_VER 5`を`.htaccess`の設定に追加する、`web/`ディレクトリのスクリプトを`.php`から`.php5`にリネームすることなども必要かもしれません。PHP 4のコマンドラインでsymfonyにアクセスしようとするとつぎのようなエラーが投じられます:

        Parse error, unexpected ',', expecting '(' in .../symfony.php on line 19.

  * `php.ini`で定義されるメモリの制限は少なくとも`32M`にしなければなりません。この問題に関する通常の症状はsymfonyをPEARもしくはコマンドライン経由でインストールしているときにエラーメッセージが表示されることです。

        Allowed memory size of 8388608 bytes exhausted

  * `php.ini`の`zend.ze1_compatibility_mode`ディレクティブは`off`にしなければなりません。そうではない場合、Webスクリプトの1つをブラウザーで見ようとすると、"implict cloning"のエラーが生み出されます:

        Strict Standards: Implicit cloning object of class 'sfTimer'because of 'zend.ze1_compatibility_mode'

  * プロジェクトの`log/`と`cache/`ディレクトリはWebサーバーが書き込みできるようにしなければなりません。これらのディレクトリのパーミッションを持たないsymfonyアプリケーションをブラウザーで見ようとすると例外で終わります:

        sfCacheException [message] Unable to write cache file"/usr/myproject/cache/frontend/prod/config/config_config_handlers.yml.php"

  * システムのincludeパスは`php`コマンドへのパスを含まなければならず、(PEARを利用した場合)`php.ini`のincludeパスはPEARへのパスが入っていなければなりません。

  * (たとえばWAMPパッケージを利用する場合など)ときに、サーバーのファイルシステム上に複数の`php.ini`が存在します。アプリケーションによって利用される`php.ini`の正確な位置を知るために、`phpinfo()`を呼び出します。

>**NOTE**
>強制ではありませんが、パフォーマンス上の理由から`php.ini`の`magic_quotes_gpc`と`register_globals`ディレクティブを`off`にしておくことを強くお勧めします。

### symfonyのリソース

あなたの問題がほかの人がすでに遭遇した問題なのか、またさまざまな場所で解決方法を見つけることができないかつぎのような場所で調べることができます:

  * symfonyのインストール方法に関するフォーラム ([http://www.symfony-project.org/forum/](http://www.symfony-project.org/forum/)) はプラットフォーム、環境、設定、ホストなどのすべてインストールの質問を扱っています。
  * ユーザーのメーリングリストのアーカイブ ([http://groups.google.fr/group/symfony-users](http://groups.google.fr/group/symfony-users)) も検索できます。あなた自身のものと似たような体験談が見つかるかもしれません。
  * 公式サイトのwiki ([http://trac.symfony-project.org/#Installingsymfony](http://trac.symfony-project.org/#Installingsymfony)) はインストール方法についてsymfonyのユーザーから寄稿された段階的なチュートリアルを含みます。

回答が見つからない場合、symfonyのコミュニティに質問を投稿してください。もっとも活動的なコミュニティのメンバーからのフィードバックを得るにはフォーラム、メーリングリスト、`#symfony` IRCチャンネルに質問を投稿することもできます。

ソースコードのバージョン管理
----------------------------

アプリケーションをセットアップした時点で、バージョン管理(version control)を始めることをお勧めします。バージョン管理によってコードのすべての修正を追跡し、以前のリリースへアクセスすることが可能で、円滑なパッチ作業を行い、そしてチームの作業を効率的にできるようになります。symfonyはネイティブでCVSをサポートしますが、Subversion([http://subversion.tigris.org/](http://subversion.tigris.org/))がお勧めです。つぎの例はSubversionのコマンドを示しており、すでにSubversionサーバーをインストールしてプロジェクトのために新しいリポジトリを作りたいということを前提としています。Windowsユーザーの場合、お勧めのSubversionクライアントはTortoiseSVN([http://tortoisesvn.tigris.org/](http://tortoisesvn.tigris.org/))です。バージョン管理とここで使われたコマンドに関して詳しい情報はSubversionのドキュメント(訳注：[「Subversionによるバージョン管理」](http://svnbook.red-bean.com/index.html)を参照))

つぎの例は`$SVNREP_DIR`が環境変数として定義されていることを前提とします。定義されていない場合、`$SVNREP_DIR`の位置にリポジトリの実際の位置を置き換える必要があります。

`myproject`プロジェクトのために新しいリポジトリを作ります:

    > svnadmin create $SVNREP_DIR/myproject

リポジトリの基本構造(レイアウト)は`trunk`、`tags`と`branches`ディレクトリでつぎの少々長いコマンドで作ります:

    > svn mkdir -m "layout creation" file:///$SVNREP_DIR/myproject/trunk file:///$SVNREP_DIR/myproject/tags file:///$SVNREP_DIR/myproject/branches

これは最初のリビジョンになります。つぎにcacheとlogの一時ファイル以外のプロジェクトのファイルをインポートする必要があります:

    > cd ~/myproject
    > rm -rf cache/*
    > rm -rf log/*
    > svn import -m "initial import" . file:///$SVNREP_DIR/myproject/trunk

つぎのコマンドを入力してコミットされたファイルをチェックしてください:

    > svn ls file:///$SVNREP_DIR/myproject/trunk/

よさそうです。SVNリポジトリはすべてのプロジェクトファイルの参照バージョン(と履歴)を持ちます。このことは実際の`~/myproject/`ディレクトリのファイルはリポジトリを参照する必要があるということを意味します。そのためには、最初に`myproject/`ディレクトリをリネームし、すべてがうまくいったらすぐに削除しますが、新しいディレクトリでリポジトリのチェックアウトします:

    > cd ~
    > mv myproject myproject.origin
    > svn co file:///$SVNREP_DIR/myproject/trunk myproject
    > ls myproject

これでお終いです。`~/myproject/`に設置されたファイルにとり組み、あなたの修正をリポジトリにコミットできます。クリーンナップすることと、不要になった`myproject.origin/`ディレクトリを削除することを忘れないでください。

まだセットアップする必要があることが残っています。あなたのワーキング(カレント)ディレクトリをリポジトリにコミットした場合、プロジェクトの`cache`や`log`ディレクトリに設置された、望まないいくつかのファイルをコピーするかもしれません。このプロジェクトのためにSVNが無視するリストを指定する必要があります。`cache/`と`log/`ディレクトリにフルアクセスする設定を再度行う必要があります:

    > cd ~/myproject
    > chmod 777 cache
    > chmod 777 log
    > svn propedit svn:ignore log
    > svn propedit svn:ignore cache

SVNのために設定されたデフォルトのテキストエディタが起動します。もし起動しない場合、つぎのコマンドを入力してSubversionが好みのエディタを使うように設定してください:

    > export SVN_EDITOR=<name of editor>
    > svn propedit svn:ignore log
    > svn propedit svn:ignore cache

コミットを行うときにSVNが無視する`myproject/`のサブディレクトリからすべてのファイルを追加してください:

    *

保存して終了させてください。作業は終わりました。

まとめ
----

ローカルサーバーでsymfonyを試して遊ぶには、インストール方法のうちベストな選択肢は、間違いなくサンドボックスです。サンドボックスにはあらかじめ設定されたsymfonyの環境が含まれるからです。

実際の開発のために、もしくは運用サーバーにおいて、PEARによるインストールもしくはSVNのチェックアウトする方法を選択してください。これらの作業によってsymfonyのライブラリをインストールし、プロジェクトとアプリケーションを初期化することも必要です。 アプリケーションのセットアップの最後の段階はサーバーの設定で、多くの方法で行われます。symfonyはバーチャルホストで完璧に動作し、これが推奨の解決方法です。

インストールをしている間に何か問題がありましたら、symfonyのWebサイト上で多くのチュートリアルとよく聞かれる質問への回答を調べます。必要であれば、あなたの問題をsymfonyのコミュニティに投稿すれば、素早くて効果的な回答を得るでしょう。

いったんプロジェクトが初期化されたら、バージョン管理のプロセスを始めることはよい習慣です。

これでsymfonyを使う準備ができたので、基本的なWebアプリケーションを開発する方法を見る段階にあります。
