第9章 - リンクとルーティングシステム
====================================

リンクとURLはWebアプリケーションのフレームワークにおいて特別な扱いをする価値があります。アプリケーション唯一のエントリーポイント(フロントコントローラー)とヘルパーを利用することでURLの動作方法とそれらの表現方法を完全に分離できるようになります。この機能はルーティング(routing)と呼ばれます。ルーティングはアプリケーションをガジェットよりもユーザーフレンドリでセキュアにするための便利なツールです。この章ではsymfonyのアプリケーションでURLを扱うために知る必要があることをすべてお伝えします。

  * ルーティングシステムとは何か、またそれがどのように動作するのか
  * 対外的なURLのルーティングを有効にするためにテンプレート内でリンクヘルパーを使う方法
  * URLの表示方法を変更するためにルーティングルールを変更する方法

ルーティングのパフォーマンスと最後の仕上げを習得するためにいくつかのトリックを見ることにもなります。

ルーティングとは何か？
----------------------

ルーティング(routing)とはURLをよりユーザーフレンドリに書き換えるメカニズムです。しかし、なぜこれが重要なのかを理解するには、URLについて数分考えなければなりません。

### サーバーに対する命令としてのURL

URLはユーザーが望むアクションを成立させるためにブラウザーからの情報をサーバーに運びます。たとえば、つぎの例のように、伝統的なURLはリクエストを完了させるために必要なスクリプトへのファイルパスとパラメーターを含みます:

    http://www.example.com/web/controller/article.php?id=123456&format_code=6532

このURLはアプリケーションのアーキテクチャとデータベースに関する情報を運びます。通常、開発者はインターフェイス内のアプリケーションのインフラを隠します(たとえば、開発者は"QZ7.65"よりも"個人のプロファイルページ"のようなページタイトルを選びます)。アプリケーション内部への重要な手がかりをURLに露出することはこの努力に相反することで重大な欠陥を晒すことになります。

  * URLに表示される技術上のデータは潜在的なセキュリティの欠点を作ります。前の例において、悪意のあるユーザーが`id`パラメーターの値を変更したら何が起こるでしょうか？アプリケーションがデータベースへの直接のインターフェイスを提供することは何を意味するのでしょうか？もしくはユーザーが面白半分にほかのスクリプト名、たとえば`admin.php`を試したらどうなるでしょうか？一般的に、生のURLを使うとアプリケーションを簡単に不正利用するための方法を提供することになるので、これらの方法によってセキュリティの管理がほとんど不可能になります。
  * 不明瞭なURLを使うと、どこに表示されようとも鬱陶しく、周囲の内容の印象が希薄になります。そして今日において、URLはアドレバーだけに表示されません。検索結果と同様に、これらはユーザーがリンクの上にマウスをホーバーしたときにも表示されます。ユーザーが情報を探すとき、図9-1のようなややこしいURLよりも、ユーザーのために探しものに関して簡単に理解できる手がかりを与えたいと開発者は願うことでしょう。

図9-1 - URLは検索結果などの多くの場所で表示される

![URLは検索結果などの多くの場所で表示される](/images/book/F0901.png "URLは検索結果などの多くの場所で表示される")

  * URLの1つを変更しなければならない場合(たとえば、スクリプト名もしくはパラメーターの1つが修正される場合)、このURLへのすべてのリンクも同様に変更しなければなりません。コントローラー構造の修正作業は重量級で高くつくので、アジャイル開発において理想的ではありません。

symfonyがフロントコントローラーのパラダイムを利用しない場合、事態はもっと悪化する可能性があります; すなわち、つぎのように、アプリケーションが、多くのディレクトリのなかで、インターネットからアクセスできるスクリプトが多く含まれている場合です:

    http://www.example.com/web/gallery/album.php?name=my%20holidays
    http://www.example.com/web/weblog/public/post/list.php
    http://www.example.com/web/general/content/page.php?name=about%20us

この場合、開発者はURLの構造をファイル構造とマッチさせる必要があり、どちらかの構造を変更したときに、結果は悪夢のようなメンテナンス作業になります。

### インターフェイスの一部としてのURL

ルーティングの背後にあるアイディアはURLをインターフェイスの一部としてみなすことです。アプリケーションは情報をユーザーにもたらすためにURLを整形し、ユーザーはアプリケーションのリソースにアクセスするためにURLを利用します。

これはsymfonyのアプリケーションで実現可能です。エンドユーザーに表示されるURLはリクエストを実行するために必要なサーバーへの命令とは無関係だからです。代わりに、URLをリクエストされるリソースに関連づけして、自由に形式を整えることができます。たとえば、symfonyはつぎのURLを理解し、この章の最初のURLのように同じページを表示できます:

    http://www.example.com/articles/finance/2006/activity-breakdown.html

恩恵は計り知れません:

  * URLは実際に何かを意味するので、これはユーザーがリンクの背後にあるページのなかに望むものが含まれるかどうかを判断するための助けになります。リンクは返すリソースに関して追加の詳細内容を含めることができます。これはとりわけ検索エンジンの結果に対して役立ちます。加えて、URLは時にページタイトルの参照なしに表示されるので(URLをEメールのメッセージにコピーするときを考えてください)、この場合、そのURLはそれ自身が何らかの意味を持たなければなりません。ユーザーフレンドリなURLの例に関しては図9-2をご覧ください。

図9-2 - 刊行日のように、URLはページに関する追加情報を運ぶ

![刊行日のように、URLはページに関する追加情報を運ぶ](/images/book/F0902.png "刊行日のように、URLはページに関する追加情報を運ぶ")

  * 紙のドキュメントに書かれているURLは入力しやすく覚えやすいものにすべきです。あなたの名刺に会社のWebサイトが`http://www.example.com/controller/web/index.jsp?id=ERD4`と記載されていたら、多くの訪問者を得ることはないでしょう。
  * URLは、 直感的な方法でアクションを実行するもしくは情報を読みとるために、独自のコマンドラインになることができます。このような機能を提供するアプリケーションによってパワーユーザーは速く作業を行うことができるようになります。

        // 結果のリスト: 結果のリストを絞るために新しいタグを追加する
        http://del.icio.us/tag/symfony+ajax
        // ユーザーのプロファイルページ: 別のユーザープロファイルを取得するために名前を変更する
        http://www.askeet.com/user/francois

  * URLの整形方法とアクションの名前/パラメーターを、それぞれ個別に修正することで、変更できます。全体的に、アプリケーションをゴチャゴチャにすることなく、最初に開発をしてあとでURLの形式を整えることができます。
  * アプリケーションの内部を再編成するときでも、URLは外部の世界に対して同じ状態を保つことができます。動的なページをブックマークできるようになるので、URLの永続性は必須です。
  * 検索エンジンはWebサイトをインデックスに登録するとき、(`.php`、`.asp`などで終わる)動的なページを無視しがちです。検索エンジンが動的なページに遭遇したときでも、静的な内容をブラウジングしていると考えさせるためにURLの形式を整えることができます。結果としてインデックスに登録されるアプリケーションのページの内容がよくなります。
  * より安全です。承認されていないURLを閲覧しようとすると開発者が指定したページにリダイレクトされるので、ユーザーが試しにURLを入力してもWebのrootのファイル構造を閲覧することはできません。リクエストによって呼び出された実際のスクリプト名は、そのパラメーターと同様に、隠匿されています。

ユーザーと実際のスクリプト名とリクエストパラメーターに表示されるURL間の対応は設定を通して修正できるパターンに基づいた、ルーティングシステムによって実現されます。

>**NOTE**
>アセット(asset)はどうでしょうか？幸運にも、URLのアセット(画像、スタイルシート、とJavaScript)はブラウジングの間は大量に表示されないので、これらに対してルーティングを実際に設定する必要はありません。symfonyにおいて、すべてのアセットは`web/`ディレクトリに設置され、URLはファイルシステムの設置場所に一致します。しかしながら、アセットヘルパー内部で生成されたURLを利用することで(アクションによって処理された)動的なアセットを管理できます。たとえば、動的に生成された画像を表示するには、`image_tag('captcha/image?key='.$key)`ヘルパーを使います。

### どのように動作するのか

symfonyは外部のURLと内部のURIを切り離します。これら2つを対応させる作業はルーティングシステムによって行われます。わかりやすくするために、symfonyは通常のURLの構文とよく似た内部URIのための構文を使います。リスト9-1は例を示しています。

リスト9-1 - 外部URLと内部URI

    // 内部のURI構文
    <module>/<action>[?param1=value1][&param2=value2][&param3=value3]...

    // 内部URIの例で、エンドユーザーに決して表示されない
    article/permalink?year=2006&subject=finance&title=activity-breakdown

    // 外部URLの例で、エンドユーザーに表示される
    http://www.example.com/articles/finance/2006/activity-breakdown.html

ルーティングシステムを定義するには`routing.yml`という名前の特別な設定ファイルを使います。リスト9-2で示されているルールを考えます。このルールは`articles/*/*/*`のようなパターンを定義し、ワイルドカードとマッチする内容の一部を命名します(訳注：既存の設定ファイルで試す場合、default:よりも上側にしないと機能しません。また`php symfony cache:clear`でキャッシュをクリアする必要があります)。

リスト9-2 - ルーティングルールのサンプル

    article_by_title:
      url:    articles/:subject/:year/:title.html
      param:  { module: article, action: permalink }

symfonyのアプリケーションに送信されるすべてのリクエストは最初ルーティングシステムによって分析されます(単独のフロントコントローラーによって処理されるのでシンプルです)。ルーティングシステムはリクエストされたURLとマッチするルーティングルールで定義されたパターンを探します。マッチするパターンが見つかった場合、名前つきのワイルドカードはリクエストパラメーターになり`param:`キーで定義されたものと統合されます。どのように動くのかはリスト9-3をご覧ください。

リスト9-3 - ルーティングシステムはやってくるリクエストURLを解釈する

    // ユーザーがつぎの外部URLを入力する(クリックする)
    http://www.example.com/articles/finance/2006/activity-breakdown.html

    // フロントコントローラーはリクエストURLがarticle_by_titleルールにマッチするか見る
    // ルーティングシステムはつぎのリクエストパラメーターを作成する
      'module'  => 'article'
      'action'  => 'permalink'
      'subject' => 'finance'
      'year'    => '2006'
      'title'   => 'activity-breakdown'

>**TIP**
>外部URLの`.html`拡張子はシンプルな飾り付けでルーティングシステムは無視します。その唯一の利点は動的なページを静的なページに見えるようにすることです。この章の後のほうにある"ルーティングの設定"でこの拡張子を有効にする方法を見ることになります。

リクエストは`article`モジュールの`permalink`アクションに渡されます。アクションは表示する記事を決定するためにリクエストパラメーターで求められたすべての情報を持ちます。

しかしながらメカニズムはまったく逆のことも行わなければなりません。リンクに外部URLを表示するアプリケーションに対して、どのルールを適用するのか決定するために十分なデータを持つルーティングルールを提供しなければなりません。ルーティングを完全に無視する`<a>`タグで直接ハイパーリンクを書いてはなりません。代わりに、リスト9-4で示されるように特別なヘルパーを使います。

リスト9-4 - ルーティングシステムはテンプレート内部のURLの出力形式を整える

    [php]
    // url_for()ヘルパーは内部URIを外部URLに変換する
    <a href="<?php echo url_for('article/permalink?subject=finance&year=2006&title=activity-breakdown') ?>">ここをクリック</a>

    // ヘルパーはURIがarticle_by_titleルールにマッチすることを見る
    // ルーティングシステムはそれから外部URLを作成する
     => <a href="http://www.example.com/articles/finance/2006/activity-breakdown.html">ここをクリック</a>

    // link_to()ヘルパーは直接ハイパーリンクを出力し
    // PHPとHTMLを混合させることを回避する
    <?php echo link_to(
      'ここをクリック',
      'article/permalink?subject=finance&year=2006&title=activity-breakdown') ?>
    ) ?>

    // 内部では、link_to()はurl_for()への呼び出しを行うので結果はつぎのものと同じ
    => <a href="http://www.example.com/articles/finance/2006/activity-breakdown.html">ここをクリック</a>

ルーティングは2つの方法のメカニズムで、すべてのリンクの形式を整える`link_to()`ヘルパーを使う場合のみに機能します。

URLを書き換える
---------------

ルーティングシステムに深く関わるまえに、1つのことをあきらかにする必要があります。以前のセクションで示された例において、内部URIのフロントコントローラー(`index.php`もしくは`frontend_dev.php`)の説明をしていません。フロントコントローラーは、アプリケーションの要素ではありませんが、環境を決定します。ですのですべてのリンクは環境に依存しなければならず、フロントコントローラー名は内部URLに決して現れることはありません。

生成されたURLの例にはスクリプト名が存在しません。デフォルトの運用環境では生成されたURLはスクリプト名を含まないからです。`settings.yml`ファイルの`no_script_name`パラメーターは生成されたURL内でフロントコントローラー名の表示を正確にコントロールします。リスト9-5で示されるように、`off`に設定すれば、リンクヘルパーによるURLの出力はフロントコントローラー名をすべてのリンク内部に記載します。

リスト9-5 - フロントコントローラー名をURL内部に表示する(`apps/frontend/settings.yml`)

    prod:
      .settings
        no_script_name:  off

生成されたURLはつぎのように示されます:

    http://www.example.com/index.php/articles/finance/2006/activity-breakdown.html

運用環境を除いたすべての環境において、`no_script_name`パラメーターはデフォルトでは`off`に設定されます。たとえば、開発環境のアプリケーションをブラウザーで見るとき、フロントコントローラー名はつねにURLに表示されます。

    http://www.example.com/frontend_dev.php/articles/finance/2006/activity-breakdown.html

運用環境において、`no_script_name`パラメーターは`on`に設定されるので、URLはルーティング情報だけを示し、よりユーザーフレンドリです。技術的な情報は現れません。

    http://www.example.com/articles/finance/2006/activity-breakdown.html

しかしながら、アプリケーションはどのフロントコントローラーが呼び出されるのかどのようにして知るのでしょうか？これはURLの書き換えが行われる場所です。URLのなかに何も存在しないときに、Webサーバーが任意のスクリプトを呼び出すために設定できます。

Apacheにおいて、これはいったん`mod_rewrite`拡張機能を有効にすれば可能です。すべてのsymfonyのプロジェクトは`.htaccess`ファイルを備えており、このファイルは`web`ディレクトリのために`mod_rewrite`の設定をサーバーの設定に追加します。このファイルのデフォルトの内容はリスト9-6で示されています。

リスト9-6 - `myproject/web/.htaccess`ファイルのなかの、Apacheのためのデフォルトの書き換えルール

    <IfModule mod_rewrite.c>
      RewriteEngine On

      # .somethingを持ったすべてのファイルをスキップする
      RewriteCond %{REQUEST_URI} \..+$
      RewriteCond %{REQUEST_URI} !\.html$
      RewriteRule .* - [L]

      # .htmlバージョンがここ(キャッシュ)であるか確認する
      RewriteRule ^$ index.html [QSA]
      RewriteRule ^([^.]+)$ $1.html [QSA]
      RewriteCond %{REQUEST_FILENAME} !-f

      # いいえ、Webのフロントコントローラーにリダイレクトする
      RewriteRule ^(.*)$ index.php [QSA,L]
    </IfModule>

Webサーバーは受けとったURLの形式を検査します。URLがサフィックスを含まず、かつ利用可能なページのキャッシュバージョンが存在しない場合、リクエストは`index.php`に渡されます(12章でキャッシュを説明します)。

しかしながら、symfonyプロジェクトの`web/`ディレクトリはプロジェクトのすべてのアプリケーションと環境のあいだで共有されます。これは通常の場合`web`ディレクトリにあるフロントコントローラーが複数存在することを意味します。たとえば、`frontend`アプリケーションと`backend`アプリケーション、と`dev`環境と`prod`環境を持つプロジェクトは4つのフロントコントローラーのスクリプトを`web/`ディレクトリに含みます:

    index.php         // prodのfrontend
    frontend_dev.php  // devのfrontend
    backend.php       // prodのbackend
    backend_dev.php   // devのbackend

mod_rewriteの設定はデフォルトのスクリプトの名前だけを指定します。すべてのアプリケーションと環境に対して`no_script_name`を`on`に設定する場合、すべてのURLは`prod`環境の`frontend`アプリケーションへのリクエストとして解釈されます。これが任意のプロジェクトに対してURLの書き換えを利用できる1つの環境を持つアプリケーションを1つだけ持つことができる理由です。

>**TIP**
>スクリプトの名前を持たないアプリケーションを複数持つ方法が1つあります。サブディレクトリをwebのrootに作り、フロントコントローラーをこれらの内部に移動させます。`ProjectConfiguration`ファイルへのパスを変更し、それぞれのアプリケーションに対して必要な`.htaccess`ファイルのURL書き換え設定を作ります。

リンクヘルパー
--------------

ルーティングシステムに対して、テンプレート内では通常の`<a>`タグの代わりにリンクヘルパーを使うべきです。これをやっかいな問題と見なさず、むしろ、アプリケーションをきれいな状態に保ち維持しやすくするための機会として見てください。加えて、リンクヘルパーはとても便利で見逃せないショートカットをいくつか提供します。

### ハイパーリンク、ボタン、とフォーム

`link_to()`ヘルパーはすでにご存じのとおりです。このヘルパーはXHTML準拠のハイパーリンクを出力し、2つのパラメーター: クリック可能な要素とそれが指し示すリソースの内部URI、を必要とします。ハイパーリンクの代わりに、ボタンが欲しい場合、`button_to()`ヘルパーを使います。フォームも`action`属性の値を管理するヘルパーを持ちます。つぎの章でフォームについてさらに学ぶことになります。リスト9-7はリンクヘルパーのいくつかの例を示します。

リンク 9-7 - `<a>`、`<input>`、`<form>`タグのためのリンクヘルパー

    [php]
    // 文字列上のハイパーリンク
    <?php echo link_to('my article', 'article/read?title=Finance_in_France') ?>
     => <a href="/routed/url/to/Finance_in_France">my article</a>

    // 画像上のハイパーリンク
    <?php echo link_to(image_tag('read.gif'), 'article/read?title=Finance_in_France') ?>
     => <a href="/routed/url/to/Finance_in_France"><img src="/images/read.gif" /></a>

    // ボタンタグ
    <?php echo button_to('my article', 'article/read?title=Finance_in_France') ?>
     => <input value="my article" type="button"onclick="document.location.href='/routed/url/to/Finance_in_France';" />

    // フォームタグ
    <?php echo form_tag('article/read?title=Finance_in_France') ?>
     => <form method="post" action="/routed/url/to/Finance_in_France" />

絶対URL(`http://`で始まり、ルーティングシステムで無視される)とアンカーと同様に、リンクヘルパーは内部URIを受けとります。実際の世界のアプリケーションにおいて、内部URIは動的なパラメーターで作られます。リスト9-8はこれらすべての事例を示します。

リスト9-8 - リンクヘルパーが受けとるURL

    [php]
    // 内部のURI
    <?php echo link_to('my article', 'article/read?title=Finance_in_France') ?>
     => <a href="/routed/url/to/Finance_in_France">my article</a>

    // 動的なパラメーターを持つ内部URI
    <?php echo link_to('my article', 'article/read?title='.$article->getTitle()) ?>

    // アンカーを持つ内部URI
    <?php echo link_to('my article', 'article/read?title=Finance_in_France#foo') ?>
     => <a href="/routed/url/to/Finance_in_France#foo">my article</a>

    // 絶対URL
    <?php echo link_to('my article', 'http://www.example.com/foobar.html') ?>
     => <a href="http://www.example.com/foobar.html">my article</a>

### リンクヘルパーのオプション

7章で説明したように、ヘルパーは、連想配列もしくは文字列である、追加オプション引数を受けとります。リスト9-9で示されているように、これはリンクヘルパーにもあてはまります。

リスト9-9 - リンクヘルパーは追加オプションを受けとる

    [php]
    // 連想配列としての追加オプション
    <?php echo link_to('my article', 'article/read?title=Finance_in_France', array(
      'class'  => 'foobar',
      'target' => '_blank'
    )) ?>

    // 文字列としての追加オプション(同じ結果)
    <?php echo link_to('my article', 'article/read?title=Finance_in_France','class=foobar target=_blank') ?>
     => <a href="/routed/url/to/Finance_in_France" class="foobar" target="_blank">my article</a>

リンクヘルパーに対してsymfony固有のオプション(`confirm`と`popup`)の1つも追加できます。リスト9-10で示されるように、最初のオプションはリンクがクリックされたときにJavaScriptの確認ダイアログボックスが表示され、2番目のオプションは新しいウィンドウにリンクが開かれます。

リスト9-10 - リンクヘルパーのための`'confirm'`オプションと`'popup'`オプション

    [php]
    <?php echo link_to('アイテムを削除する', 'item/delete?id=123', 'confirm=Are you sure?') ?>
     => <a onclick="return confirm('よろしいですか？');"
           href="/routed/url/to/delete/123.html">delete item</a>

    <?php echo link_to('カートに追加する', 'shoppingCart/add?id=100', 'popup=true') ?>
     => <a onclick="window.open(this.href);return false;"
           href="/fo_dev.php/shoppingCart/add/id/100.html">カートに追加する</a>

    <?php echo link_to('カートに追加する', 'shoppingCart/add?id=100', array(
      'popup' => array('popupWindow', 'width=310,height=400,left=320,top=0')
    )) ?>
     => <a onclick="window.open(this.href,'popupWindow','width=310,height=400,left=320,top=0');return false;"
           href="/fo_dev.php/shoppingCart/add/id/100.html">カートに追加する</a>

これらのオプションを結びつけることができます。

### フェイクのGETとPOSTオプション

時にWeb開発者は実際にはPOSTを行うためにGETリクエストを使います。たとえば、つぎのURLを考えてみてください:

    http://www.example.com/index.php/shopping_cart/add/id/100

このリクエストは品物をショッピングカートのオブジェクトに追加することで、アプリケーションに含まれるデータを変更します。そしてデータはセッションもしくはデータベースに保存されます。このURLはブックマークされ、キャッシュされ、検索エンジンのインデックスに登録されます。すべての嫌なことがデータベースもしくはこのテクニックを使うWebサイトの評価指標に起こることを想像してみてください。当然のことながら、このリクエストはPOSTとして見なされるべきです。なぜなら検索エンジンのロボットはインデックスの上ではPOSTリクエストを行わないからです。

symfonyは`link_to()`ヘルパー、もしくは`button_to()`ヘルパーへの呼び出しを実際のPOSTリクエストに変換する方法を提供します。リスト9-11で示されるように、`post=true`オプションを追加するだけです。

リスト9-11 -リンク呼び出しをPOSTリクエストにする

    [php]
    <?php echo link_to('ショッピングカートに移動する', 'shoppingCart/add?id=100', 'post=true') ?>
     => <a onclick="f = document.createElement('form'); document.body.appendChild(f);
                    f.method = 'POST'; f.action = this.href; f.submit();return false;"
           href="/shoppingCart/add/id/100.html">ショッピングカートに移動する</a>

この`<a>`タグは`href`属性を持ち、検索エンジンのロボットなどの、JavaScriptサポートを持たないブラウザーはデフォルトのGETメソッドを行いながらリンクを辿ります。POSTメソッドだけに応答するように、つぎのようなコードをアクションの始めに追加することで、アクションの制限も行わなければなりません:

    [php]
    $this->forward404Unless($this->getRequest()->isMethod('post'));

このオプションによって独自の`<form>`タグが生成されるので、フォームに設置されたリンクの上でこのオプションが使われていないことを確認してください(訳注：`<form></form>`のなかにさらに`<form></form>`が生成されるのでフォームが入れ子になります)。

実際にデータを投稿するリンクをPOSTとしてタグ付けすることはよい習慣です。

### リクエストパラメーターをGET変数として強制する

ルーティングルールに従えば、パラメーターとして`link_to()`ヘルパーに渡された変数はパターンに変換されます。`routing.yml`ファイルで内部URIにマッチするルールが存在しない場合、リスト9-12で示されるように、デフォルトのルールでは`module/action?key=value`は`/module/action/key/value`に変換されます。

リスト9-12 - ルーティングルールのデフォルト

    [php]
    <?php echo link_to('my article', 'article/read?title=Finance_in_France') ?>
    => <a href="/article/read/title/Finance_in_France">my article</a>

実際にGETの構文を維持する必要がある場合--リクエストパラメーターを?key=value形式で渡す場合--`query_string`オプションのなかで、URLパラメーターの外部で強制される必要のある変数を設置する必要があります。
これがURLのアンカーにも衝突するので、これを内部URIに追加する代わりに`anchor`オプションに加えなければなりません。リスト9-13で示されるように、すべてのリンクヘルパーはこれらのオプションを受けとります。

リスト9-13 - query_string`オプションでGET変数を`強制する

    [php]
    <?php echo link_to('my article', 'article/read', array(
      'query_string' => 'title=Finance_in_France',
      'anchor' => 'foo'
    )) ?>
    => <a href="/article/read?title=Finance_in_France#foo">my article</a>

GET変数として表現されるリクエストパラメーターを持つURLはクライアントサイド上のスクリプト、サーバーサイド上の`$_GET`変数と`$_REQUEST`変数によって解釈されます。

>**SIDEBAR**
>**アセットヘルパー**
>
>7章でアセットヘルパーの`image_tag()`、`stylesheet_tag()`と`javascript_include_tag()`を紹介しました。これらのヘルパーによって画像、スタイルシート、JavaScriptファイルをレスポンスに含めることができます。これらのアセットへのパスはルーティングルールによって処理されません。これらは公開Webディレクトリの元に実際に設置されたレスポンスにリンクするからです。
>
>アセットに対してファイルの拡張子を記載する必要はありません。symfonyは自動的に`.png`、`.js`、もしくは`.css`の拡張子を画像、JavaScript、もしくはスタイルシートのヘルパー呼び出しに追加します。また、symfonyは`web/images/`ディレクトリ、`web/js/`ディレクトリと`web/css/`ディレクトリで自動的にこれらのアセットを探します。もちろん、特定のファイルフォーマットもしくは特定の場所からのファイルを含めたい場合、正式なファイル名もしくはファイルパスを引数として使います。そして、メディアファイルが明確な名前を持つ場合、symfonyがあなたの代わりに決定するので、`alt`属性を指定することを悩まずにすみます。
>
>     [php]
>     <?php echo image_tag('test') ?>
>     <?php echo image_tag('test.gif') ?>
>     <?php echo image_tag('/my_images/test.gif') ?>
>      => <img href="/images/test.png" alt="Test" />
>         <img href="/images/test.gif" alt="Test" />
>         <img href="/my_images/test.gif" alt="Test" />
>
>画像のサイズを修正するには`size`属性を使います。これは、ピクセル単位の、`x`で区切られた幅、高さを必要とします。
>
>     [php]
>     <?php echo image_tag('test', 'size=100x20')) ?>
>      => <img href="/images/test.png" alt="Test" width="100" height="20"/>
>
>(JavaScriptとスタイルシートに対して)アセットを`<head>`セクションのなかに含めたい場合、レイアウト内で`_tag()`バージョンを使う代わりに、テンプレートのなかで`use_stylesheet()`ヘルパーと`use_javascript()`ヘルパーを使います。これらのヘルパーはアセットをレスポンスに追加し、`</head>`タグがブラウザーに送られるまえにこれらのアセットはインクルードされます。

### 絶対パスを使う

リンクヘルパーとアセットヘルパーはデフォルトで相対パスを生成します。絶対パスへの出力を強制するには、リスト9-14で示されるように、`absolute`オプションを`true`に設定します。このテクニックはリンクをEメールのメッセージ、RSSフィード、APIのレスポンスに含めるために便利です。

リスト9-14 - 相対URLの代わりに絶対URLを取得する

    [php]
    <?php echo url_for('article/read?title=Finance_in_France') ?>
     => '/routed/url/to/Finance_in_France'
    <?php echo url_for('article/read?title=Finance_in_France', true) ?>
     => 'http://www.example.com/routed/url/to/Finance_in_France'

    <?php echo link_to('finance', 'article/read?title=Finance_in_France') ?>
     => <a href="/routed/url/to/Finance_in_France">finance</a>
    <?php echo link_to('finance', 'article/read?title=Finance_in_France','absolute=true') ?>
     => <a href=" http://www.example.com/routed/url/to/Finance_in_France">finance</a>

    // 同じことがアセットヘルパーにあてはまる
    <?php echo image_tag('test', 'absolute=true') ?>
    <?php echo javascript_include_tag('myscript', 'absolute=true') ?>

>**SIDEBAR**
>**メールヘルパー**
>
>今日において、Eメール収集ロボットがWebを徘徊するので、一日以内にスパムの餌食にならずにすむEメールアドレスを表示することはできません。symfonyが`mail_to`ヘルパーを提供する理由はそういうことです。
>
>`mail_to()`ヘルパーは2つのパラメーターをとります: 実際のEメールアドレスと表示される文字列です。追加オプションはHTMLでは全く読めない何かを出力する`encode`パラメーターを受けとります。ブラウザーはこれを理解できますがロボットは理解できません。
>
>     [php]
>     <?php echo mail_to('myaddress@mydomain.com', 'contact') ?>
>      => <a href="mailto:myaddress@mydomain.com'>contact</a>
>     <?php echo mail_to('myaddress@mydomain.com', 'contact', 'encode=true') ?>
>      => <a href="&#109;&#x61;... &#111;&#x6d;">&#x63;&#x74;... e&#115;&#x73;</a>
>
>エンコードされたEメールのメッセージはランダムな10進法と16進法のエンティティエンコーダによって変換された文字列で構成されます。このトリックは現在のアドレス収集するたいていのスパムボットを停止させますが、収集テクニックは急速に発展していることをご了承ください。

ルーティングの設定
------------------

ルーティングシステムは2つのことを行います:

  * モジュール/アクションとリクエストパラメーターを決定するために、入ってくるリクエストの外部URLを解釈し、内部URIに変換します。
  * リンクで使われている内部URIを外部URLの形式に整形します(リンクヘルパーを使っていることが前提)。

規約はルーティングルールのセットに基づいています。これらのルールはアプリケーションの`config/`ディレクトリに設置された`routing.yml`設定ファイルに保存されます。リスト9-15はすべてのsymfonyに搭載されたデフォルトのルーティングルールを示しています。

リスト9-15 - ルーティングルールのデフォルト(`frontend/config/routing.yml`)

    # デフォルトのルール
    homepage:
      url:   /
      param: { module: default, action: index }

    default_symfony:
      url:   /symfony/:action/*
      param: { module: default }

    default_index:
      url:   /:module
      param: { action: index }

    default:
      url:   /:module/:action/*

### ルールとパターン

ルーティングルールは外部URLと内部URI間の全単射の関係(bijective associations)です。典型的なルールはつぎのように構成されます:

  * ユニークなラベル。これは読みやすさと速さのためにあり、リンクヘルパーのために使われます
  * マッチするパターン(`url`キー)
  * リクエストパラメーターの値の配列(`param`キー)

パターンはワイルドカード(アスタリスクの`*`で表現される)と名前つきのワイルドカード(コロン、`:`で始まる)を含むことができます。名前つきのワイルドカードへのマッチはリクエストパラメーターの値になります。たとえば、リスト9-15で定義された`default`ルールは`/foo/bar`といったURLにマッチし、`module`パラメーターを`foo`に、`action`パラメーターを`bar`に設定します。そして`default_symfony`ルールにおいて、`symfony`はキーワードで、`action`は名前つきのワイルドカードパラメーターです。

>**NOTE**
>**symfony 1.1の新しい機能** 名前つきのワイルドカードはスラッシュもしくはドットで分離できるので、つぎのようなパターンを書けます:
>
>     my_rule:
>       url:   /foo/:bar.:format
>       param: { module: mymodule, action: myaction }
>
>この方法では、'foo/12.xml'のような外部URLは`my_rule`にマッチして`$bar=12`と`$format=xml`の2つのパラメーターを持つ`mymodule/myaction`を実行します。
>`sfPatternRouting`ファクトリ設定で`segment_separators`パラメーターを変更することで追加の区切り文字を追加できます(19章を参照)。

ルーティングシステムは`routing.yml`ファイルを上から順に解析し、最初にマッチした時点で止まります。これが独自のルール群をデフォルトのルールの上に追加しなければならない理由です。たとえば、URLの`/foo/123`はリスト9-16で定義されたルールの両方にマッチしますが、symfonyは最初`my_rule:`をテストして、ルールがマッチする場合、`default:`をテストしません。(`foo/123`アクションではなく)`123`に設定された`bar`を持つ`mymodule/myaction`アクションでリクエストは扱われます。

リスト9-16 - ルールは上から下へ順に解析される

    my_rule:
      url:   /foo/:bar
      param: { module: mymodule, action: myaction }

    # デフォルトのルール
    default:
      url:   /:module/:action/*

>**NOTE**
>新しいアクションが作られたとき、そのためのルーティングルールを作らなければならないということにはなりません。デフォルトの`module/action`パターンがあなたの用途に合う場合、`routing.yml`ファイルは忘れてください。しかしながら、アクションの外部URLをカスタマイズしたい場合、デフォルトのルールの上に新しいルールを追加します。

リスト9-17は`article/read`アクションに対する外部URL形式の変更プロセスを示しています。

リスト9-17 - `article/read`アクションに対して外部URL形式を変更する

    [php]
    <?php echo url_for('article/read?id=123') ?>
     => /article/read/id/123       // デフォルトのフォーマッティング

    // これを/article/123に変更するため、routing.ymlの始めに
    // 新しいルールを追加する
    article_by_id:
      url:   /article/:id
      param: { module: article, action: read }

問題はリスト9-17の`article_by_id`ルールは`article`モジュールのほかのすべてのアクションに対するデフォルトのルーティングを壊すことです。実際、`article/delete`のようなURLは`default`ルールの代わりにこのルールにマッチし、`delete`アクションの代わりに`delete`に設定された`id`をともなう`read`アクションを呼び出します。この問題を回避するには `article_by_id`ルールにパターンの制約を追加することで、ワイルドカードである`id`が整数のURLのときだけにマッチするように設定する必要があります。

### パターンの制約

URLが複数のルールにマッチするとき、制約もしくは要件(requirements)をパターンに追加することでルールを洗練させなければなりません。要件は正規表現のセットでマッチするルールのためにワイルドカードによってマッチされなければなりません。

たとえば、`id`パラメーターが整数であるURLだけにマッチするように`article_by_id`ルールを修正するには、リスト9-18で示されるように、ルールに一行追加します。

リスト9-18 - 要件(requirements)をルーティングルールに追加する

    article_by_id:
      url:   /article/:id
      param: { module: article, action: read }
      requirements: { id: \d+ }

`article/delete`のURLは`article_by_id`にはマッチしません。`'delete'`の文字列は要件を満たさないからです。それゆえ、ルーティングシステムはつぎのルールでマッチするものを探し続け、最後には`default`ルールを見つけます。

>**SIDEBAR**
>**パーマリンク**
>
>ルーティングのためのよいセキュリティのガイドラインは主キーを隠し、これらを可能なかぎり重要な文字列で置き換えることです。これらのIDよりもこれらのタイトルから記事にアクセスしたい場合はどうしますか？これを行うにはつぎのような外部URLになります:
>
>     http://www.example.com/article/Finance_in_France
>
>この範囲に対して、新しい`permalink`アクションを作成する必要があります。このアクションは`id`パラメーターの代わりに`slug`パラメーターを使い、新しいルールを追加します:
>
>     article_by_id:
>       url:          /article/:id
>       param:        { module: article, action: read }
>       requirements: { id: \d+ }
>
>     article_by_slug:
>       url:          /article/:slug
>       param:        { module: article, action: permalink }
>
>`permalink`アクションはタイトルからリクエストされた記事を決定する必要があるので、モデルは適切なメソッドを提供しなければなりません。
>
>     [php]
>     public function executePermalink($request)
>     {
>       $article = ArticlePeer::retrieveBySlug($request->getParameter('slug');
>       $this->forward404Unless($article);  // 記事がslugにマッチしない場合404を表示する
>       $this->article = $article;          // オブジェクトをテンプレートに渡す
>     }
>
>内部URIの正しいフォーマッティングを有効にするために、テンプレートのなかの`read`アクションへのリンクを`permalink`アクションへのリンクに置き換えることも必要です。
>
>     [php]
>     // つぎのコードを
>     <?php echo link_to('my article', 'article/read?id='.$article->getId()) ?>
>
>     // 以下のコードに置き換える
>     <?php echo link_to('my article', 'article/permalink?slug='.$article->getSlug()) ?>
>
>`requirements`の行のおかげで、`article_by_id`ルールが最初に現れたとしても、`/article/Finance_in_France`のような外部URLが`article_by_slug`ルールにマッチします。
>
>slugによって記事が読みとられるので、データベースのパフォーマンスを最適化するにはインデックスを`Article`モデルの記述内容の`slug`カラムに追加すべきです。

### デフォルト値を設定する

パラメーターが定義されていなくても、ルールを機能させるために名前つきのワイルドカードにデフォルト値を渡すことができます。デフォルト値を`param:`配列のなかで設定します。

たとえば、`id`パラメーターが設定されていない場合`article_by_id`ルールはマッチしません。リスト9-19で示されるように、強制することができます。

リスト9-19 - ワイルドカードに対してデフォルト値を設定する

    article_by_id:
      url:          /article/:id
      param:        { module: article, action: read, id: 1 }

デフォルトのパラメーターはパターン内で見つかるワイルドカードである必要はありません。リスト9-20において、`display`パラメーターはURLに表示されなくても`true`の値をとります。

リスト9-20 - リクエストパラメーターのためのデフォルト値を設定する

    article_by_id:
      url:          /article/:id
      param:        { module: article, action: read, id: 1, display: true }

注意深く見ると、パターン内で見つからない`module`変数と`action`変数に対して`article`と`read`がそれぞれのデフォルト値であることがわかります。

>**TIP**
>`sfRouting::setDefaultParameter()`メソッドを呼び出すことですべてのルーティングルールに対してデフォルトのパラメーターを定義できます。たとえば、デフォルトで`theme`パラメーターを`default`に設定するすべてのルールが欲しい場合、`$this->context->getRouting()->setDefaultParameter('theme', 'default');` をグローバルフィルターの1つに追加します。

### ルールの名前を利用してルーティングを加速する

リスト9-21で示されるように、ルールラベルが'at'記号(`@`)のまえに来る場合、リンクヘルパーはモジュール/アクションの組の代わりにルールラベルを受けとります。

リスト9-21 - モジュール/アクションの代わりにルールラベルを使う

    [php]
    <?php echo link_to('my article', 'article/read?id='.$article->getId()) ?>

    // つぎのように書くこともできる
    <?php echo link_to('my article', '@article_by_id?id='.$article->getId()) ?>

このトリックに関してよい点とわるい点があります、よい点はつぎのとおりです:

  * 内部URIの整形が速く行われます。symfonyはリンクにマッチするルールを見つけるためにすべてのルールを探す必要がないからです。ルーティングされたハイパーリンクをたくさん持つページにおいて、モジュール/アクションの組の代わりにルールラベルを使う場合、速度の押し上げは顕著です。
  * ルールラベルを使うことはアクションの背後にあるロジックを抽象化するための助けになります。アクション名を変更するがURLはそのままにする場合、`routing.yml`ファイルのなかで変更を行うだけで十分です。すべての`link_to()`呼び出しはさらに変更しなくても機能します。
  * 呼び出しのロジックはルール名で明らかです。モジュールとアクションが明確な名前を持つ場合でも、`article/display`よりも`@display_article_by_slug`を呼び出したほうがベターです。

一方で、わるい点は、新しいハイパーリンクを追加することが自明ではなくなることです。アクションに対してどのラベルが使われているのか解明するために`routing.yml`ファイルにつねに参照する必要があるからです。

最良の選択はプロジェクト次第です。結局は、あなた次第です。

>**TIP**
>テストの間(`dev`環境)、 ブラウザーの任意のリクエストに対してどのルールがマッチしたのかチェックしたい場合、"logs and msgs"セクションのWebデバッグツールバーを展開し、"logs and msgs"セクションのWebデバッグツールバー開き、"matched route XXX."と書かれている行を探してください。Webデバッグモードに関する詳細な情報は16章で知ることになります。

-

>**NOTE**
>**外部URLと内部URIの間の変換がキャッシュされるので、symfony 1.1の新しい機能** ルーティングのオペレーションは運用モードではるかに速いです。

### .html拡張子を追加する

以下の2つのURLを比較してください:

    http://myapp.example.com/article/Finance_in_France
    http://myapp.example.com/article/Finance_in_France.html

同じページであっても、ユーザー(とロボット)はURLなのでこれを違うものとして見るかも知れません。2番目のURLは静的なページの深くてよく整理されたWebディレクトリを呼び出します。静的なページは検索エンジンがインデックスを作成する方法を理解しているWebサイトの種類のものです。

リスト9-22で示されるように、サフィックスをルーティングシステムによって生成されたすべての外部URLに追加するには、`settings.yml`ファイル内の`suffix`の値を変更します。

リスト9-22 - すべてのURLに対してサフィックスを設定する(`frontend/config/settings.yml`)

    prod:
      routing:
        param:
          suffix: .html

デフォルトのサフィックスはピリオド(`.`)に設定されます。このことはあなたが接尾辞を指定しないかぎりルーティングシステムは接尾辞を追加しないことを意味します。 

時に、唯一のルーティングルールのためにサフィックスを指定する必要があります。その場合、リスト9-23で示されるように、接尾辞を`routing.yml`ファイルの関連する`url:`の行に直接書きます。グローバルな接尾辞は無視されます。

リスト9-23 - 1つのURLに対してサフィックスを設定する(`frontend/config/routing.yml`)

    article_list:
      url:          /latest_articles
      param:        { module: article, action: list }

    article_list_feed:
      url:          /latest_articles.rss
      param:        { module: article, action: list, type: feed }

### routing.ymlなしでルールを作成する

たいていの設定ファイルにあてはまることですが、`routing.yml`ファイルはルーティングルールを定義するための解決方法ですが、唯一の方法ではありません。アプリケーションの`config.php`ファイル、もしくはフロントコントローラースクリプト内で、しかし`dispatch()`を呼び出すまえに、PHPでルールを定義できます。なぜなら、このメソッドは現在のルーティングルールにしたがって実行するアクションを決定するからです。PHPでルールを定義することは、設定もしくはほかのパラメーターに依存する、動的なルールを作成することを許可することを意味します。

ルーティングルールを扱うオブジェクトは`sfPatternRouting`ファクトリです。`sfContext::getInstance()->getRouting()`を求めることでコードのすべての部分から利用できます。このオブジェクトの`prependRoute()`メソッドは`routing.yml`のなかで定義された既存のルールの上に新しいルールを追加します。このメソッドは4つのパラメーターを必要とします。このパラメーターはルールを定義するために必要なものと同じです: ルートのラベル、パターン、デフォルト値の連想配列、と要件のための別の連想配列です。たとえば、リスト9-18で示される`routing.yml`ルールの定義はリスト9-24で示されるPHPのコードと同等です。

>**NOTE**
>**symfony 1.1の新しい機能**: ルーティングクラスは`factories.yml`設定ファイルで設定可能です(デフォルトのルーティングクラスを変更するには、17章を参照)。この章では`sfPatternRouting`クラスを説明します。このクラスはデフォルトで設定されるルーティングルールです。

リスト9-24 - PHPでルールを定義する

    [php]
    sfContext::getInstance()->getRouting()->prependRoute(
      'article_by_id',                                  // ルートの名前
      '/article/:id',                                   // ルートパターン
      array('module' => 'article', 'action' => 'read'), // デフォルト値
      array('id' => '\d+'),                             // 要件
    );

Singletonの`sfPatternRouting`は手動でルートを扱うために便利なほかのメソッド、`clearRoutes()`、`hasRoutes()`、などを持ちます。もっと学ぶにはAPIドキュメント([http://www.symfony-project.org/api/1_1/](http://www.symfony-project.org/api/1_1/))を参照してください。

>**TIP**
>いったんこの本で説明された概念を十分に理解し始めたら、オンラインのAPIドキュメント、もっとベターなのはsymfonyのソースを眺めることで、フレームワークの理解を深めることができます。この本ではsymfonyの調整方法とパラメーターのすべては説明されていません。しかしながら、オンラインドキュメントは無制限です。

アクションのなかでルートを処理する
------------------------------

現在のルートについて情報を読みとりたい場合、たとえば"back to page xxx"リンクを用意するには、`sfPatternRouting`オブジェクトのメソッドを使うべきです。リスト9-25で示されるように、`getCurrentInternalUri()`メソッドによって返されたURIは、`link_to()`ヘルパーへの呼び出しのなかで使われます。

リスト9-25 - 現在のルートについて情報を読みとるために`sfRouting`オブジェクトを使う

    [php]
    // つぎのようなURLを求める場合
    http://myapp.example.com/article/21

    $routing = sfContext::getInstance()->getRouting();

    // つぎのarticle/readアクションを使う
    $uri = $routing->getCurrentInternalUri();
     => article/read?id=21

    $uri = $routing->getCurrentInternalUri(true);
     => @article_by_id?id=21

    $rule = $routing->getCurrentRouteName();
     => article_by_id

    // 現在のmodule/action名が必要なだけなら
    // これらが実際のリクエストパラメーターであることを覚えておく
    $module = $request->getParameter('module');
    $action = $request->getParameter('action');

内部URIを外部URLに変換する必要がある場合、テンプレートのなかで`url_for()`ヘルパーが行うように、リスト9-26で示されている`sfController`オブジェクトの`genUrl()`メソッドを使います。

リスト9-26 - 内部URIを変換するために`sfController`オブジェクトを使う

    [php]
    $uri = 'article/read?id=21';

    $url = $this->getController()->genUrl($uri);
     => /article/21

    $url = $this->getController()->genUrl($uri, true);
    => http://myapp.example.com/article/21

まとめ
----

ルーティング(routing)は外部URLの形式をよりユーザーフレンドリにするために設計された2つの方法を持つメカニズムです。それぞれのプロジェクトの1つのアプリケーションのURL内部でフロントコントローラーの名前を省略できるようにするにはURLの書き換え(URL rewriting)が必要です。ルーティングシステムが両方の方法で機能することを望むのであれば、URLをテンプレート内部に出力する必要があるたびにリンクヘルパーを使わなければなりません。`routing.yml`ファイルはルーティングシステムのルールを設定し、優先順位とルールの要件(requirements)を使います。`settings.yml`ファイルはフロントコントローラーの名前と外部URLで可能なプレフィックスの存在に関する追加設定を含みます。
