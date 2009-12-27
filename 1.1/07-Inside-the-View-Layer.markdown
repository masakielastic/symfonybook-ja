第7章 - ビューレイヤーの内側
============================

ビュー(view)は特定のアクションと関連づけされた出力をレンダリングすることを引き受けます。symfonyにおいて、ビューはいくつかの部分から構成され、それぞれの部分はとり組む人達が簡単に修正できるように設計されています。

  * Webデザイナーは一般的にテンプレート(現在のアクションのデータのプレゼンテーション)と(すべてのページに共通なコードを含む)レイアウトにとり組みます。これらはHTMLで書かれ、大部分がヘルパーへの呼び出しである、PHPの埋め込みコードの塊(チャンク)です。
  * 再利用のために、開発者は通常テンプレートのコードのフラグメント(fragment - 断片)を部分テンプレート(partial - もしくはパーシャル)もしくはコンポーネント(component)にまとめます。これらはレイアウトの複数の領域に影響を与えるスロット(slot)とコンポーネントスロット(component)を使います。Webデザイナーもこれらのテンプレートのフラグメントにとり組むことができます。
  * 開発者は(レスポンスのプロパティとほかのインターフェイスの要素を設定する)YAML形式のビューの設定ファイルとレスポンスオブジェクトにとり組みます。テンプレート内で変数を扱うとき、クロスサイトスクリプティング(XSS - Cross-Site Scripting)のリスクを無視してはなりませんし、ユーザーのデータを安全に記録するために出力エスケーピングのテクニックをよく理解していることが求められます。

しかしながら、あなたの役割が何であれ、アクションの結果を表現する退屈な作業を速くする便利なツールを見ることになります。この章ではこれらすべてのツールをカバーします。

テンプレートを利用する
----------------------

リスト7-1はsymfonyの典型的なテンプレートを示します。HTMLコードと基本的なPHPコード、通常はアクション(`$this->name = 'foo';`)とヘルパーで定義された変数への呼び出しを含みます。

リスト7-1 - サンプルのindexSuccess.phpテンプレート

    [php]
    <h1>ようこそ</h1>
    <p>お帰りなさい、<?php echo $name ?>！</p>
    <ul>何をなさりたいですか？
      <li><?php echo link_to('最新の記事を読む', 'article/read') ?></li>
      <li><?php echo link_to('新しい記事を書き始める', 'article/write') ?></li>
    </ul>

4章で説明したように、テンプレートのなかではPHP開発者ではない人が読みやすいようにPHPの代替構文が望ましいです。テンプレートのなかではPHPコードは最小限に保つべきです。これらのファイルはアプリケーションのGUIを設計するために使われ、時々、別のチームによって作成とメンテナンスが行われ、アプリケーションのロジックではなくプレゼンテーションに特化されているからです。ロジックをアクション内部に保つことで、コードを重複せずに、1つのアクションを共有する複数のテンプレートを持つことが簡単になります。

### ヘルパー

ヘルパーはPHP関数でHTMLコードを返し、テンプレートのなかで使うことができます。リスト7-1において`link_to()`関数はヘルパーです。時々、テンプレート内でよく使われるコードのスニペット(断片)をまとめることで、ヘルパーは時間を節約します。たとえば、つぎのヘルパーのための関数定義は簡単に想像がつきます。

    [php]
    <?php echo input_tag('nickname') ?>
     => <input type="text" name="nickname" id="nickname" value="" />

上記のコードはリスト7-2のようになります。

リスト7-2 - サンプルのヘルパーの定義

    [php]
    function input_tag($name, $value = null)
    {
      return '<input type="text" name="'.$name.'" id="'.$name.'"value="'.$value.'" />';
    }

実際のところ、symfonyに組み込まれている`input_tag()`関数は、ほかの属性を`<input>`タグに追加する3番目のパラメーターを受けとるので、上記のコードよりも少々複雑です。完全な構文とオプションはAPIドキュメント([http://www.symfony-project.org/api/1_1/](http://www.symfony-project.org/api/1_1/))で確認できます。

たいていの場合、ヘルパーは賢いので長くて複雑なコードを書かずにすみます。

    [php]
    <?php echo auto_link_text('我々のサイトにお越しください www.example.com') ?>
     => 我々のサイトにお越しください <a href="http://www.example.com">www.example.com</a>

ヘルパーはテンプレートを書く作業工程を円滑にし、パフォーマンスとアクセシビリティの観点から最高のHTMLコードを生み出します。つねに無地のHTMLを使うことができますが、ヘルパーのほうがより速く書けます。

>**TIP**
>なぜヘルパーがsymfony内部のすべての場所で使われているcamelCase(キャメルケース)の規約ではなくアンダースコアの構文にしたがって命名されるのか疑問に思っていらっしゃるかもしれません。これはヘルパーが関数であり、PHPのすべてのコア関数がアンダースコアの構文の規約を利用するからです。

#### ヘルパーを宣言する

ヘルパーの定義を含むsymfonyのファイルはオートロードされません(これらがクラスではなく関数を含むからです)。ヘルパーは目的によって分類されます。たとえば、テキストを扱うすべてのヘルパー関数は`TextHelper.php`という名前のファイルで定義され、`Text`ヘルパーグループと呼ばれます。ですのでヘルパーをテンプレートのなかで使う必要がある場合、`use_helper()`関数でヘルパーを宣言することで最初の段階で関連のヘルパーグループをロードしなければなりません。リスト7-3は、`Text`ヘルパーグループの一部である、`auto_link_text()`ヘルパーを使うテンプレートを示しています。

リスト7-3 ヘルパーを使うことを宣言する

    [php]
    // このテンプレート内で特定のヘルパーグループを使う
    <?php use_helper('Text') ?>
    ...
    <h1>説明</h1>
    <p><?php echo auto_link_text($description) ?></p>

>**TIP**
>複数のヘルパーグループを宣言する必要がある場合、より多くの引数を`use_helper()`呼び出しに追加してください。たとえば、`Text`と`Javascript`ヘルパーグループの両方をテンプレートにロードするには、`<?php use_helper('Text', 'Javascript') ?>`を呼び出します。

いくつかのヘルパーは、すべてのテンプレートのなかで、宣言を行わずにデフォルトで利用できます。ヘルパーグループのヘルパーはつぎのようなものがあります:

  * `Helper`: ヘルパーのインクルージョンに必要(`use_helper()`関数は、実際、ヘルパーそのもの)
  * `Tag`: ほとんどすべてのヘルパーによって使われる、基本的なタグヘルパー 
  * `Url`: リンクとURLを管理するヘルパー
  * `Asset`: HTMLの`<head>`セクションの投入と外部アセット(画像、JavaScriptとCSS)への簡単なリンクを提供するヘルパー
  * `Partial`: テンプレートのフラグメントをインクルードできるようにするヘルパー
  * `Cache`: キャッシュされたコードのフラグメントを操作するヘルパー
  * `Form`: フォーム入力のヘルパー

すべてのテンプレートのためにデフォルトでロードされる、標準的なヘルパーのリストは`settings.yml`ファイルのなかで設定できます。`Cache`グループのヘルパーを使わない、もしくは`Text`グループのヘルパーをつねに使うことがわかっている場合、`standard_helpers`設定をそれぞれ変更します。これによってアプリケーションの動作を少し加速します。前のリストにある最初の4つのヘルパーグループ(`Helper`、`Tag`、`Url`、`Asset`)は削除できません。なぜなら、テンプレートエンジンが適切に動作するために必須だからです。結果として、標準ヘルパーのリストにも表示されません。

>**TIP**
>テンプレートの外部でヘルパーを使う必要がある場合、`sfLoader::loadHelpers($helpers)`を呼び出すことでどこからでもヘルパーグループをロードすることができます。`$helpers`はヘルパーグループの名前もしくはヘルパーグループ名の配列です。たとえば、アクション内で`auto_link_text()`を使いたい場合、`sfLoader::loadHelpers('Text')`を最初に呼び出す必要があります。

#### よく使われるヘルパー

ヘルパーが助けしてくれる機能に関連する、いくつかのヘルパーに関する詳細な内容は後の章で学ぶことになります。リスト7-4では、ヘルパーが返すHTMLコードと一緒に、よく使われるデフォルトのヘルパーの手短な一覧を示しています。

リスト7-4 - デフォルトの共通ヘルパー

    [php]
    // Helperグループ
    <?php use_helper('HelperName') ?>
    <?php use_helper('HelperName1', 'HelperName2', 'HelperName3') ?>

    // Tagグループ
    <?php echo tag('input', array('name' => 'foo', 'type' => 'text')) ?>
    <?php echo tag('input', 'name=foo type=text') ?>  // 代替のオプション構文
     => <input name="foo" type="text" />
    <?php echo content_tag('textarea', 'ダミーの内容', 'name=foo') ?>
     => <textarea name="foo">ダミーの内容</textarea>

    // Urlグループ
    <?php echo link_to('クリックしてください', 'mymodule/myaction') ?>
    => <a href="/route/to/myaction">クリックしてください</a>  // ルーティングの設定による

    // Assetグループ
    <?php echo image_tag('myimage', 'alt=foo size=200x100') ?>
     => <img src="/images/myimage.png" alt="foo" width="200" height="100"/>
    <?php echo javascript_include_tag('myscript') ?>
     => <script language="JavaScript" type="text/javascript" src="/js/myscript.js"></script>
    <?php echo stylesheet_tag('style') ?>
     => <link href="/stylesheets/style.css" media="screen" rel="stylesheet"type="text/css" />

symfonyにはほかの多くのヘルパーが存在し、それらすべてを説明するには一冊の本が必要になります。ヘルパーの最良のリファレンスはオンラインのAPIドキュメント([http://www.symfony-project.org/api/1_1/](http://www.symfony-project.org/api/1_1/))です。そこですべてのヘルパーの構文、オプションと例について十分な説明があります。

### 独自のヘルパーを追加する

symfonyはさまざまな目的のために多くのヘルパーを搭載していますが、APIドキュメントで必要なものが見つからない場合、新しいヘルパーを作りたいと思うでしょう。これはとても簡単に行うことができます。

ヘルパー関数(HTMLコードを返す通常のPHP関数)は`FooBarHelper.php`という名前のファイルに保存されます。`FooBar`はヘルパーグループの名前です。インクルードするために`use_helper('FooBar')`ヘルパーによってファイルが自動的に見つかるように、ファイルは`apps/frontend/lib/helper/`に保存されます(もしくはプロジェクトの`lib/`フォルダーの1つのもとで作られた`helper/`ディレクトリ)。

>**TIP**
>このシステムによって既存のsymfonyヘルパーをオーバーライドすることもできるようになります。たとえば、`Text`ヘルパーグループのすべてのヘルパーを再定義するために、 `TextHelper.php`ファイルを`apps/frontend/lib/helper/`ディレクトリに作ります。`use_helper('Text')`を呼び出すたびに、symfonyは固有のヘルパーよりあなたのヘルパーグループを使います。しかしつぎのことに気をつけてください: オリジナルのファイルがロードされない場合、そのファイルをオーバーライドするすべてのヘルパーグループの関数を再定義しなければなりません; さもなければ、いくつかのオリジナルのヘルパーはまったく利用できなくなります。

### ページのレイアウト

リスト7-1で示されているテンプレートは有効なXHTMLのドキュメントではありません。`DOCTYPE`の定義と`<html>`と`<body>`タグは見つかりません。これらは、アプリケーションのほかの場所に存在する、ページのレイアウトを含む`layout.php`ファイルに保存されるからです。このファイルは、グローバルレイアウト(global template)とも呼ばれますが、すべてのテンプレートのなかで繰り返しを避けるためにアプリケーションのすべてのページに共通なHTMLコードを保存します。テンプレートの内容は、レイアウトに統合されるか、考えを変える場合、レイアウトはテンプレートを"デコレート"(装飾)します。図7-1で説明されるように、これはDecoratorデザインパターンのアプリケーションです。

>**TIP**
>Decoratorとほかのデザインパターンについて詳しい情報はMartin Fowler(マーチン・ファウラー)が執筆したPatterns of Enterprise Application Architecture(Addison-Wesley、ISBN: 0-32112-742-0)をご覧ください(邦訳は「エンタープライズアプリケーションアーキテクチャパターン」翔泳社、2005年 ISBN 4798105538)。

図7-1 - レイアウト内でテンプレートをデコレートする

![レイアウト内でテンプレートをデコレートする](http://github.com/masakielastic/symfonybook-ja/raw/master/images/F0701.png "レイアウトでテンプレートをデコレートする")

リスト7-5は、アプリケーションの`templates/`ディレクトリのなかに設置された、デフォルトのページレイアウトを示しています。

リスト7-5 - デフォルトのレイアウト(`myproject/apps/frontend/templates/layout.php`)

    [php]
    <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
    <html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en" lang="en">
      <head>
        <?php include_http_metas() ?>
        <?php include_metas() ?>
        <?php include_title() ?>
        <link rel="shortcut icon" href="/favicon.ico" />
      </head>
      <body>
        <?php echo $sf_content ?>
      </body>
    </html>

`<head>`セクションに呼び出されたヘルパーはレスポンスオブジェクトとビューの設定から情報を取得します。`<body>`タグはテンプレートの結果を出力します。このレイアウトによって、デフォルトの設定とリスト7-1のサンプルのテンプレート、と処理されたビューはリスト7-6のようになります。

リスト7-6 - 組み立てられたレイアウト、ビューの設定、とテンプレート

    [php]
    <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
    <html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en" lang="en">
      <head>
        <meta http-equiv="content-type" content="text/html; charset=utf-8" />
        <meta name="title" content="symfony project" />
        <meta name="robots" content="index, follow" />
        <meta name="description" content="symfony project" />
        <meta name="keywords" content="symfony, project" />
        <title>symfonyのプロジェクト</title>
        <link rel="stylesheet" type="text/css" href="/css/main.css" />
        <link rel="shortcut icon" href="/favicon.ico">
      </head>
      <body>
        <h1>ようこそ</h1>
        <p>おかえり、<?php echo $name ?>!</p>
        <ul>何をなさいますか？
          <li><?php echo link_to('最近の記事を読む', 'article/read') ?></li>
          <li><?php echo link_to('新しい記事を書き始める', 'article/write') ?></li>
        </ul>
      </body>
    </html>

それぞれのアプリケーションに対してグローバルテンプレートを完全にカスタマイズできます。必要なHTMLコードを追加してください。このレイアウトはサイトのナビゲーション、ロゴなどを保持するためによく使われます。複数のレイアウトを保有することが可能で、それぞれのアクションに対して使われるレイアウトを選ぶことができます。今のところはJavaScriptとスタイルシートのインクルージョンは気にしないでください; この章の"ビューのコンフィギュレーション"のセクションで扱う方法を説明します。

### テンプレートのショートカット

テンプレートにおいて、いくつかのsymfonyの変数はつねに利用可能です。これらのショートカットは、symfonyのコアオブジェクトを通して、テンプレート内でもっとも共通に必要とされる情報にアクセスできます:

  * `$sf_context`: コンテキスト全体のオブジェクト(`sfContext`のインスタンス)
  * `$sf_request`: リクエストオブジェクト(`sfRequest`のインスタンス)
  * `$sf_params` : リクエストオブジェクトのパラメーター
  * `$sf_user`   : 現在のユーザーセッションのオブジェクト(`sfUser`のインスタンス)

以前の章で`sfRequest`と`sfUser`オブジェクトの便利なメソッドを詳細に説明しました。`$sf_request`と`$sf_user`変数を通してテンプレートのこれらのメソッドを実際に呼び出すことができます。たとえば、リクエストが`total`パラメーターを含む場合、その変数はつぎのようにテンプレート内で利用できます:

    [php]
    // 長いバージョン
    <?php echo $sf_request->getParameter('total') ?>

    // 短いバージョン
    <?php echo $sf_params->get('total') ?>

    // つぎのアクションコードと同等
    echo $request->getParameter('total')

コードのフラグメント
--------------------

いくつかのページでHTMLもしくはPHPコードを含めることが必要になることがよくあります。そのコードを繰り返すことを避けるには、多くの場合、PHPの`include()`ステートメントで十分です。 

たとえば、アプリケーションの多くのテンプレートがコードの同じフラグメントを使うことが必要な場合、グローバルテンプレートのディレクトリ(`myproject/apps/frontend/templates/`)にある、`myFragment.php`という名前のファイルにフラグメントを保存し、つぎのようにそのファイルをテンプレートにインクルードしてください:

    [php]
    <?php include(sfConfig::get('sf_app_template_dir').'/myFragment.php') ?>

しかしこれはフラグメントをまとめるにはあまりきれいな方法ではありません。たいていの場合、フラグメントとそれを含むさまざまなテンプレートのあいだで異なる変数名を持つ可能性があるからです。加えて、symfonyのキャッシュシステム(12章で説明)はインクルードを検出する方法を持たないので、コードのフラグメントはテンプレートから独立してキャッシュされません。symfonyは`include`を置き換えるために、代わりとなる3つのタイプの賢いコードのフラグメントを提供します;

  * ロジックが軽量の場合、テンプレートに渡すデータにアクセスするテンプレートファイルをインクルードするだけです。そのためには、部分テンプレート(partial)を使います。
  * ロジックが重い場合(たとえば、データモデルにアクセスして/もしくはセッションにしたがって内容を修正する場合)、プレゼンテーションをロジックから分離することが望ましいです。そのためには、コンポーネント(component)を使います。
  * コードのフラグメントでレイアウトの特定の部分を置き換えることを目的とする場合、すでに存在しているであろうデフォルトの内容に対して、スロット(slot)を使います。

>**NOTE**
>コンポーネントスロット(component slot)と呼ばれる、別のコードのフラグメントのタイプはフラグメントの性質が文脈に依存するときに使われます(たとえば、コードのフラグメントが任意のモジュールのアクションに対して異なることが必要な場合)。コンポーネントスロットはこの章の後のほうで説明します。

これらのコードのフラグメントのインクルードは`Partial`グループのヘルパーによって実現されます。これらのヘルパーは、初期化の宣言を行わずに、symfonyのどのテンプレートからも利用できます。

### 部分テンプレート

部分テンプレート(partial)は再利用可能なテンプレートコードの塊(チャンク)です。たとえば、情報公開を行うアプリケーションにおいて、記事を表示するテンプレートのコードは記事の詳細な内容のページに使われ、もっとも人気のある記事と最新の記事の一覧にも使われます。図7-2で示されるように、このコードは部分テンプレートのための完璧な候補です。

図7-2 - テンプレート内で部分テンプレートを再利用する

![テンプレート内で部分テンプレートを再利用する](http://github.com/masakielastic/symfonybook-ja/raw/master/images/F0702.png "テンプレート内で部分テンプレートを再利用する")

テンプレートのように、部分テンプレートは`templates/`ディレクトリに設置されたファイルで、これらはPHPが埋め込まれたHTMLコードを含みます。部分テンプレートのファイルの名前はつねにアンダースコア(`_`)で始まります。テンプレートが同じ`templates`フォルダーに設置されているので、これは部分テンプレートとテンプレートを区別するのに役立ちます。

部分テンプレートが同じモジュール、別のモジュール、もしくはグローバルな`templates/`ディレクトリにあったとしても、テンプレートは部分テンプレートをインクルードできます。リスト7-7に書かれているように、`include_partial()`ヘルパーを利用して部分テンプレートをインクルードして、モジュールと部分テンプレート名をパラメーターとして指定します(ただし先頭のアンダースコアと末尾の`.php`を省略します)。 

リスト7-7 - `mymodule`モジュールのテンプレートのなかで部分テンプレートをインクルードする

    [php]
    // frontend/modules/mymodule/templates/_mypartial1.php部分テンプレートをインクルードする
    // テンプレートと部分テンプレートは同じモジュールにあるので、
    // モジュール名を省略できる
    <?php include_partial('mypartial1') ?>

    // frontend/modules/foobar/templates/_mypartial2.php部分テンプレートをインクルードする
    // この場合モジュール名は必須
    <?php include_partial('foobar/mypartial2') ?>

    // frontend/templates/_mypartial3.php部分テンプレートをインクルードする
    // 'global'モジュールの一部として見なされる
    <?php include_partial('global/mypartial3') ?>

部分テンプレートはsymfonyの通常のヘルパーとテンプレートのショートカットにアクセスできます。しかし、部分テンプレートはアプリケーションのどこからでも呼び出すことができるので、部分テンプレートをインクルードするテンプレートを呼び出すアクションで定義された変数が明示的に引数として渡されないかぎり、部分テンプレートは変数に自動的にアクセスできません。たとえば、`$total`変数にアクセスできる部分テンプレートが欲しい場合、リスト7-8、7-9、7-10で示されるように、アクションは変数をテンプレートに渡さなければなりませんし、また、テンプレートは変数を`include_partial()`呼び出しの2番目の引数としてヘルパーに渡さなければなりません。

リスト7-8 - アクションが変数を定義する(`mymodule/actions/actions.class.php`)

    [php]
    class mymoduleActions extends sfActions
    {
      public function executeIndex()
      {
        $this->total = 100;
      }
    }

リスト7-9 - テンプレートが部分テンプレートに変数を渡す(`mymodule/templates/indexSuccess.php`)

    [php]
    <p>Hello, world!</p>
    <?php include_partial('mypartial', array('mytotal' => $total)) ?>

リスト7-10 - 部分テンプレートは変数を利用できる(`mymodule/templates/_mypartial.php`)

    [php]
    <p>合計: <?php echo $mytotal ?></p>

>**TIP**
>すべてのヘルパーはこれまで`<?php echo functionName() ?>`によって呼び出されてきました。通常のPHPの`include()`ステートメントと似たようなふるまいをするように、部分テンプレートのヘルパーは、`echo`することなく、`<?php include_partial() ?>`によって簡単に呼び出されます。実際に表示しないで部分テンプレートの内容を返す関数が必要な場合、代わりに`get_partial()`を使います。この章で説明されたすべての`include_`ヘルパーは`echo`ステートメントで一緒に呼び出すことができる`get_`で始まる対のヘルパーを持ちます。

>**TIP**
>**symfony 1.1の新しい機能**: テンプレートで終わる代わりに、アクションは部分テンプレートもしくはコンポーネントを返します。アクションクラスの`renderPartial()`と`renderComponent()`メソッドはコードの再利用性を促進します。加えて、それらは部分テンプレートのキャッシュ機能を利用します(12章を参照)。アクションのなかで定義された変数は、変数の連想配列をメソッドの2番目の引数として定義しないかぎり、自動的に部分テンプレート/コンポーネントに渡されます。
>
>     [php]
>     public function executeFoo()
>     {
>       // 何かを行う
>       $this->foo = 1234;
>       $this->bar = 4567;
>
>       return $this->renderPartial('mymodule/mypartial');
>     }
>
>この例では、部分テンプレートは`$foo`と`$bar`にアクセスできるようになります。アクションがつぎのような行で終わる場合:
>
>       return $this->renderPartial('mymodule/mypartial', array('foo' => $this->foo));
>
>部分テンプレートは`$foo`にのみアクセスできるようになります。

### コンポーネント

2章において、最初のサンプルスクリプトはプレゼンテーションからロジックを分離するために2つの部分に分割されました。MVCパターンがアクションとテンプレートに適用されるように、部分テンプレートをロジックとプレゼンテーションの部分に分割する必要があるかもしれません。その場合、コンポーネント(component)を使うべきです。

コンポーネントは、はるかに速く動くこと以外は、アクションと似ています。コンポーネントのロジックは、`action/components.class.php`ファイルに設置された`sfComponents`から継承したクラスに保存されます。そのプレゼンテーションは部分テンプレートに保存されます。`sfComponents`クラスのメソッドは、アクションと同じように、`execute`という単語で始まり、アクションが変数を渡す方法と同じように、変数をプレゼンテーションの対応物に渡すことができます。コンポーネントに対してプレゼンテーションとして役割を果たす部分テンプレートはコンポーネントによって(先頭の`execute`ではなく、代わりにアンダースコアで)命名されます。テーブル7-1はアクションとコンポーネントの間の命名規約を比較しています。

テーブル7-1 - アクションとコンポーネントの命名規約

規約                                |アクション         |コンポーネント
------------------------------------|-------------------|--------------
ロジックのファイル                    |`actions.class.php`  |`components.class.php`
ロジッククラスの拡張                |`sfActions`          |`sfComponents`
メソッドの命名方法                  |`executeMyAction()`  |`executeMyComponent()`
プレゼンテーションのファイルの命名方法|`myActionSuccess.php`|`_myComponent.php`

>**TIP**
>アクションのファイルを分割できるのと同様に、`sfComponents`クラスには`sfAction`クラスに該当する`sfComponent`クラスがあり、これによりアクションと同様の構文で一ファイル一コンポーネントが可能になります。

たとえば、任意の題目に対して最新のニュースの見出しを表示するサイドバーを持つことを仮定します。題目はユーザーのプロファイルに依存し、いくつかのページで再利用されます。ニュースの見出しを取得するために必要なクエリは単純な部分テンプレートで表示するには複雑すぎるので、これらをアクションのようなファイル、コンポーネントに移動させる必要があります。図7-3はこの例を図示しています。

リスト7-11と7-12で示された、この例に対して、コンポーネントは独自のモジュール(`news`)に保持されますが、ビューの機能上の観点から意味がある場合、コンポーネントとアクションを単独のモジュールに混ぜることができます。

図7-3 - テンプレート内でコンポーネントを使う

![テンプレート内でコンポーネントを使う](http://github.com/masakielastic/symfonybook-ja/raw/master/images/F0703.png "テンプレート内でコンポーネントを使う")

リスト7-11 - コンポーネントクラス(`modules/news/actions/components.class.php`)

    [php]
    <?php

    class newsComponents extends sfComponents
    {
      public function executeHeadlines()
      {
        $c = new Criteria();
        $c->addDescendingOrderByColumn(NewsPeer::PUBLISHED_AT);
        $c->setLimit(5);
        $this->news = NewsPeer::doSelect($c);
      }
    }

リスト7-12 - 部分テンプレート(`modules/news/templates/_headlines.php`)

    [php]
    <div>
      <h1>最新のニュース</h1>
      <ul>
      <?php foreach($news as $headline): ?>
        <li>
          <?php echo $headline->getPublishedAt() ?>
          <?php echo link_to($headline->getTitle(),'news/show?id='.$headline->getId()) ?>
        </li>
      <?php endforeach ?>
      </ul>
    </div>

では、コンポーネントがテンプレートのなかで必要になるたびに、つぎのように呼び出します:

    [php]
    <?php include_component('news', 'headlines') ?>

部分テンプレートのように、コンポーネントは連想配列の形式で追加パラメーターを受けとります。パラメーターはこれらの名前のもとで部分テンプレート内および`$this`オブジェクトを通してコンポーネントのなかで利用できます。例としてリスト7-13をご覧ください。

リスト7-13 - パラメーターをコンポーネントとテンプレートに渡す

    [php]
    // コンポーネントへの呼び出し
    <?php include_component('news', 'headlines', array('foo' => 'bar')) ?>

    // コンポーネント自身にて
    echo $this->foo;
     => 'bar'

    // _headlines.php部分テンプレートにて
    echo $foo;
     => 'bar'

通常のテンプレートのように、コンポーネント内部、もしくはグローバルレイアウト内部でコンポーネントをインクルードできます。アクションのように、コンポーネントの`execute`メソッドは変数を関連する部分テンプレートに渡し、同じショートカットにアクセスできます。しかし、似ている点はここまでです。コンポーネントはセキュリティもしくはバリデーションを処理せず、インターネットから呼び出すことはできませんし(呼び出せるのはアプリケーション自身からのみ)、さまざまなものを返すことはできません。それがコンポーネントがアクションよりも実行が速い理由です。

### スロット

部分テンプレートとコンポーネントは再利用のために優れたものです。しかし多くの場合、コードのフラグメントは複数の動的な領域を持つレイアウトの要件を満たすことが求められます。たとえば、カスタムタグをレイアウトの`<head>`セクションに追加することを考えます。レイアウトはアクションの内容に依存します。もしくは、レイアウトが主要な動的な領域を1つ持つことを想定します。その領域はアクションの結果によって内容が満たされます。加えて、レイアウトは別の小さな領域をたくさん持ちます。これらの領域はレイアウトで定義されたデフォルトの内容を持ちますが、テンプレートレベルでオーバーライドできます。

これらの状況に対して、解決方法はスロット(slot)です。基本的には、スロットはビューの要素(レイアウト、テンプレート、もしくは部分テンプレート)に設置できるプレースホルダーです。プレースホルダーの内容を満たすことは変数の設定に似ています。内容を満たすコードはレスポンスにグローバルに保存されるので、どこでもスロットを定義できます(レイアウト、テンプレート、もしくは部分テンプレート)。スロットをインクルードするまえにかならずスロットを定義してください。そしてレイアウトはテンプレートのあとで実行され(これはデコレーションプロセス)、部分テンプレートがテンプレート内部に呼び出されるときに部分テンプレートが実行されることを覚えておいてください。説明が抽象的でわかりにくいですか？例を見てみましょう。

1つのテンプレートと2つのスロットに対して1つの領域を持つレイアウトを想像してください。1つはサイドバー用で他はフッター用です。スロットの値はテンプレート内で定義されます。デコレーションプロセスの間、図7-4に示されるように、レイアウトのコードはテンプレートのコードを包み、スロットは事まえに定義された値で満たされます。サイドバーとフッターはメインのアクションに対して文脈依存の関係があります。これは複数の"穴"をともなうレイアウトを持つことと似ています。

図7-4 - テンプレートのなかで定義されたレイアウトのスロット

![テンプレートのなかで定義されたレイアウトのスロット](http://github.com/masakielastic/symfonybook-ja/raw/master/images/F0704.png "テンプレートのなかで定義されたレイアウトのスロット ")

いくつかのコードを読むことで理解が進みます。スロットを含めるには、`include_slot()`ヘルパーを使います。`has_slot()`ヘルパーはスロットが以前定義されていた場合に`true`を返します。そしておまけとしてフォールバックメカニズムを提供します。たとえば、リスト7-14で示されるように、レイアウトとデフォルトの内容のなかにある`'sidebar'`に対してプレースホルダーを定義してください。

リスト7-14 - 'sidebar'スロットをレイアウト内部でインクルードする

    [php]
    <div id="sidebar">
    <?php if (has_slot('sidebar')): ?>
      <?php include_slot('sidebar') ?>
    <?php else: ?>
      <!-- default sidebar code -->
      <h1>文脈上の領域</h1>
      <p>この領域はページのメインの内容と関連するリンクと情報を含みます。</p>

    <?php endif; ?>
    </div>

スロットが定義されていない場合、デフォルトの何らかの内容を表示することはとても日常的なことなので、`include_slot`ヘルパーはスロットが定義されたどうかを示すブール値を返します。リスト7-15はコードを簡略化するためにこの戻り値を考慮する方法を示しています。

リスト7-15 - レイアウトのなかで`'sidebar'`スロットをインクルードする

    [php]
    <div id="sidebar">
    <?php if (!include_slot('sidebar')): ?>
      <!-- default sidebar code -->
      <h1>文脈依存の領域</h1>
      <p>この領域はページのメインの内容に関連する
         リンクと情報を含みます。</p>
    <?php endif; ?>
    </div>

それぞれのテンプレートはスロットの内容を定義する機能を持ちます (実際には、部分テンプレートが行います)。スロットはHTMLのコードを保持することが想定されているので、symfonyはそれらを定義する便利な方法を提供します: リスト7-16のように、`slot()`と`end_slot()`ヘルパーの間のスロットのコードを書くだけです。

リスト7-16 - テンプレートのなかで`'sidebar'`スロットの内容をオーバーライドする

    [php]
    // ...

    <?php slot('sidebar') ?>
      <!-- 現在のテンプレートに対するカスタムサイドバーのコード -->
      <h1>ユーザーの詳細</h1>
      <p>名前:  <?php echo $user->getName() ?></p>
      <p>Eメール: <?php echo $user->getEmail() ?></p>
    <?php end_slot() ?>

スロットヘルパーに挟まれるコードはテンプレートの文脈内で実行されるので、このコードはアクション内部で定義されたすべての変数にアクセスできます。symfonyはこのコードの結果を自動的にレスポンスオブジェクトに設置します。これはテンプレート内部では表示されませんが、リスト7-14のような、将来の`include_slot()`呼び出しに対して利用できます。

スロットは文脈上の内容を表示することを目的とした領域を定義するためにとても便利です。これらは特定のアクションに対してHTMLコードをレイアウトに追加するために利用することも可能です。たとえば、最新のニュースのリストを表示するテンプレートがレイアウトの`<head>`部分のなかでリンクをRSSフィードに追加したい場合があります。これは`feed`スロットをレイアウトに追加してテンプレートのリストのなかでこのスロットをオーバーライドすることで簡単に実現されます。

スロットの内容がとても短い場合、これはまさにたとえば`title`スロットを定義する事例なので、リスト7-17で示されるように内容を`slot()`メソッドの2番目の引数として渡すことができます。

リスト7-17 - 短い値を定義するために`slot()`を使う

    [php]
    <?php slot('title', 'The title value') ?>

>**SIDEBAR**
>**テンプレートのフラグメントが見つかる場所**
>
>テンプレートにとり組む人達は通常はWebデザイナーで、symfonyについて詳しくないでしょうし、テンプレートのフラグメントはアプリケーション全体で拡散されているので、見つけることは困難でしょう。これらのいくつかのガイドラインによってsymfonyのテンプレートシステムにとり組む作業が快適になります。
>
>最初に、symfonyのプロジェクトは多くのディレクトリを含みますが、すべてのレイアウト、テンプレート、テンプレートのフラグメントのファイルは`templates/`という名前のディレクトリのなかに存在します。Webデザイナーに関しては、プロジェクトの構造をつぎのように小さくすることができます:
>
>
>     myproject/
>       apps/
>         application1/
>           templates/       # application 1のためのレイアウト
>           modules/
>             module1/
>               templates/   # module 1のためのテンプレートと部分テンプレート
>             module2/
>               templates/   # module 2のためのテンプレートと部分テンプレート
>             module3/
>               templates/   # module 3のためのテンプレートと部分テンプレート
>
>
>ほかのディレクトリはすべて無視されます。
>
>`include_partial()`に遭遇するとき、Webデザイナーは最初の引数だけが重要であることを理解する必要があります。この引数のパターンは`module_name/partical_name`で、これはプレゼンテーションのコードが`modules/module_name/templates/_partical_name.php`内で見つかることを意味します。
>
>`include_component()`ヘルパーに対して、モジュールと部分テンプレートの名前は最初の2つの引数です。残りに関しては、ヘルパーが何でありテンプレートでもっとも共通なヘルパーはどれかといった一般的な理解をしていればsymfonyのアプリケーションのためのテンプレートの設計を始めるには十分です。

ビューのコンフィギュレーション
------------

symfonyにおいて、ビューは2つの相異なる部分で構成されます:

  * アクションの結果のHTMLのプレゼンテーション(テンプレート、レイアウト、そしてテンプレートフラグメントに保存される)
  * 残りのすべて、以下の内容を含む:

    * メタ宣言: キーワード、説明、もしくはキャッシュの持続時間 
    * ページのタイトル: あなたのサイトのページを見つけるためにいくつものウィンドウを開くユーザーに対して役立つだけでなく、サイト検索のためのインデックス作成にも非常に重要。
    * ファイルのインクルージョン: JavaScriptとスタイルシートファイル。
    * レイアウト: アクションのなかにはカスタムレイアウト(ポップアップ、広告など)を必要とするもの、もしくはレイアウトをまったく必要としないもの(Ajaxアクションなど)が存在する。

ビューのなかにおいて、HTMLではないすべてのものはビューの設定(コンフィギュレーション)と呼ばれ、symfonyはこれを操作するために2つの方法を提供します。通常の方法は`view.yml`設定ファイルです。このファイルは値がコンテキスト、もしくはデータベースのクエリに依存しないときはつねに使われます。動的な値を設定する必要があるとき、代わりの方法は`sfResponse`オブジェクトの属性を通してビューの設定をアクション内部で直接設定することです。

>**NOTE**
>`sfResponse`オブジェクトと`view.yml`ファイルの両方を通してビューの設定パラメーターを設定した場合、`sfResponse`の定義が優先されます。

### view.ymlファイル

それぞれのモジュールはビューの設定を定義する`view.yml`ファイルを1つ持つことができます。これによって単独のファイル内部でモジュール全体とビューごとにビューの設定できます。`view.yml`ファイルの最初のレベルのキーはモジュールのビューの名前です。リスト7-18はビューのコンフィギュレーションの例を示します。

リスト7-18 - モジュールレベルの`view.yml`のサンプル

    editSuccess:
      metas:
        title: プロファイルを編集する

    editError:
      metas:
        title: プロファイルを編集している間に発生したエラー

    all:
      stylesheets: [my_style]
      metas:
        title: 私のWebサイト

>**CAUTION**
>`view.yml`ファイルのなかのメインキーはビューの名前で、アクションの名前ではないことに注意してください。復習として、ビューの名前はアクションの名前とアクションのサフィックスによって構成されます。たとえば、`edit`アクションが`sfView::SUCCESS`を返す場合(もしくはデフォルトアクションの接尾辞なので、何も返さない場合)、ビューの名前は`editSuccess`です。

モジュールのためのデフォルトの設定はモジュールの`view.yml`の`all:`キーのもとで定義されます。すべてのアプリケーションのビューに対するデフォルトの設定はアプリケーションの`view.yml`のなかで定義されます。繰り返しますが、設定カスケードの原則を認識してください。

  * `apps/frontend/modules/mymodules/config/view.yml`において、ビュー単位の定義は一つのビューだけに適用され、モジュールレベルの定義をオーバーライドします。
  * `apps/frontend/modules/mymodule/config/view.yml`において、`all:`定義はすべてのモジュールのアクションに適用され、アプリケーションレベルの定義をオーバーライドします。
  * `apps/frontend/config/view.yml`において、`default:`定義はアプリケーションのすべてのモジュールとすべてのアクションに適用されます。

>**TIP**
>モジュールレベルの`view.yml`ファイルはデフォルトでは存在しません。最初に、モジュールのためのビューの設定パラメーターを調整する必要があり、空の`view.yml`を`config/`ディレクトリのなかに作らなければなりません。

リスト7-5のデフォルトのテンプレートとリスト7-6の最後のレスポンスを見た後に、ヘッダーの値がどこから来るのか疑問に思うかもしれません。実際には、これらはビューのデフォルト設定であり、アプリケーションの`view.yml`で定義され、リスト7-19で示されます。

リスト7-19 - デフォルトのアプリケーションレベルのビュー設定(`apps/frontend/config/view.yml`)

    default:
      http_metas:
        content-type: text/html

      metas:
        #title:        symfony project
        #description:  symfony project
        #keywords:     symfony, project
        #language:     en
        robots:       index, follow

      stylesheets:    [main]

      javascripts:    []

      has_layout:     on
      layout:         layout

これらの設定はそれぞれ"ビューのコンフィギュレーション設定"のセクションで詳細に説明します。

### レスポンスオブジェクト

ビューレイヤーの一部ではありますが、レスポンスオブジェクトはしばしアクションによって修正されます。アクションは`getResponse()`メソッドを通して`sfResponse`と呼ばれるsymfonyのレスポンスオブジェクトにアクセスできます。リスト7-20は、アクションのなかでよく使われる`sfResponse`メソッドのいくつかのリストを示しています。

リスト7-20 アクションは`sfResponse`オブジェクトメソッドにアクセスできる

    [php]
    class mymoduleActions extends sfActions
    {
      public function executeIndex()
      {
        $response = $this->getResponse();

        // HTTPヘッダー
        $response->setContentType('text/xml');
        $response->setHttpHeader('Content-Language', 'en');
        $response->setStatusCode(403);
        $response->addVaryHttpHeader('Accept-Language');
        $response->addCacheControlHttpHeader('no-cache');

        // Cookie
        $response->setCookie($name, $content, $expire, $path, $domain);

        // メタ情報とページのヘッダー
        $response->addMeta('robots', 'NONE');
        $response->addMeta('keywords', 'foo bar');
        $response->setTitle('My FooBar Page');
        $response->addStyleSheet('custom_style');
        $response->addJavaScript('custom_behavior');
      }
    }

ここで示されるセッターメソッドに加えて、`sfReponse`クラスはレスポンスの属性の現在の値を返すゲッターを持ちます。

symfonyにおいてヘッダーのセッターはとても強力です。(`sfRenderingFilter`で)ヘッダーは可能なかぎり遅く送信されるので、望むだけ多く、そして遅く変更することができます。それらはとても便利なショートカットも提供します。たとえば、`setContentType()`を呼び出すときにcharsetを指定できない場合、symfonyは`settings.yml`ファイルで定義されたデフォルトのcharsetを自動的に追加します。

    [php]
    $response->setContentType('text/xml');
    echo $response->getContentType();
     => 'text/xml; charset=utf-8'

symfonyのレスポンスのステータスコードはHTTPの仕様と互換性があります。例外の場合はステータス500を返し、ページが見つからない場合はステータス404を返し、通常のページの場合はステータス200を返し、修正されていないページの場合はステータス304と共にシンプルなヘッダーに減らすことができる(詳細は12章で)、などです。しかし、`setStatusCode()`レスポンスメソッドを持つアクションのなかで独自のステータスコードを設定することで、これらのデフォルトをオーバーライドできます。カスタムコードとメッセージ、もしくは単なるカスタムコードを指定できます。この場合、symfonyはこのコードに対してもっとも共通したメッセージを追加します。

    [php]
    $response->setStatusCode(404, 'このページは存在しません');

>**TIP**
>ヘッダーを送るまえに、symfonyはこれらの名前を正規化します。`setHttpHeader()`の呼び出しのなかで`Contetn-Language`の代わりに`content-language`を書くことに悩む必要はありません。syfmonyは前者を理解し、自動的に後者に変換するからです。

### ビューのコンフィギュレーション設定

2種類のビューのコンフィギュレーション設定があることにお気づきかもしれません:

  * ユニークな値を持つもの(値は`view.yml`ファイルのなかの文字列で、レスポンスはそれらに対して`set`メソッドを使う)
  * 複数の値を持つもの(`view.yml`は配列を使用し、レスポンスは`add`メソッドを使う)

設定カスケードはユニークな値の設定を削除し複数の値の設定を集積することを覚えておいてください。この章を読み進めるとよりあきらかになります。

### メタタグのコンフィギュレーション

レスポンスの`<meta>`タグに書かれた情報はブラウザーには表示されませんが、ロボットと検索エンジンには役立ちます。これはすべてのページのキャッシュ設定もコントロールします。リスト7-19のように、これらのタグは`view.ymlのなかの`http_metas:`と`metas:`キーの下で定義するか、リスト7-21のように、アクションの`addHttpMeta()`と`addMeta()`のレスポンスメソッドでこれらのタグを定義します。

リスト7-21 - 「キー: 値」の組としてのメタの定義(`view.yml`)

    http_metas:
      cache-control: public

    metas:
      description:   Finance in France
      keywords:      finance, France

リスト7-22 - レスポンスアクションのレスポンス設定としてのメタの定義

    [php]
    $this->getResponse()->addHttpMeta('cache-control', 'public');
    $this->getResponse()->addMeta('description', 'Finance in France');
    $this->getResponse()->addMeta('keywords', 'finance, France');

デフォルトでは既存のキーを追加すると現在の内容が置き換えられます。HTTPメタタグに対して、`addHttpMeta()`メソッド(`setHttpHeader()`も同様)は3番目の値を設定することができ、falseを指定した場合は既存の値を置き換えるのではなく値を追加します。

    [php]
    $this->getResponse()->addHttpMeta('accept-language', 'en');
    $this->getResponse()->addHttpMeta('accept-language', 'fr', false);
    echo $this->getResponse()->getHttpHeader('accept-language');
     => 'en, fr'

これらのメタタグが最終的なドキュメントに表示されるように、`include_http_metas()`と`include_metas()`ヘルパーは`<head>`セクションのなかで呼び出されなければなりません(これはデフォルトのレイアウトの場合; リスト7-5を参照)。適切な`<meta>`タグを出力するためにsymfonyはすべての`view.yml`ファイル(リスト7-18で示されているデフォルトのものを含む)とレスポンスの属性から自動的に設定を集約します。リスト7-21の例はリスト7-23で示されているような状態で終わります。

リスト7-23 - 最終的なページ内のメタタグの出力

    [php]
    <meta http-equiv="content-type" content="text/html; charset=utf-8" />
    <meta http-equiv="cache-control" content="public" />
    <meta name="robots" content="index, follow" />
    <meta name="description" content="Finance in France" />
    <meta name="keywords" content="finance, France" />

おまけとして、レスポンスのHTTPヘッダーは、レイアウトのなかで`include_http_metas()`ヘルパーを持たないもしくはレイアウトをまったく持たない場合、`http-metas:`の定義にも影響されます。たとえば、ページをプレーンなテキストとして送る必要がある場合、つぎのように`view.yml`を定義します:

    http_metas:
      content-type: text/plain

    has_layout: false

### タイトルのコンフィギュレーション

ページタイトルは検索エンジンのインデックス作業の重要な部分です。タブブラウジングを提供するモダンなブラウザーでも非常に便利です。HTMLにおいて、タイトルはページのタグとメタ情報の両方であるので、`view.yml`ファイルは`title:`キーを`metas:`キーの子として探します。リスト7-24は`view.yml`のタイトル定義を示し、リスト7-25はアクション定義を示します。

リスト7-24 - タイトルの定義(`view.yml`)

    indexSuccess:
      metas:
        title: Three little piggies

リスト7-25 - アクションのなかのタイトルの定義 -- 動的なタイトルを可能にする

    [php]
    $this->getResponse()->setTitle(sprintf('%d little piggies', $number));

最終的なドキュメントの`<head>`セクションにおいて、`<title>`タグタイトルの定義は、`include_metas()`ヘルパーが存在する場合は`<meta name="title">`タグを、`include_title()`ヘルパーが存在する場合は`<title>`タグを設定します。両ほうが含まれる場合(リスト7-5のデフォルトのレイアウトなど)、タイトルはドキュメントのソース内で2回表示されますが(リスト7-6を参照)、これは無害です。

### ファイルのインクルードのコンフィギュレーション

リスト7-26が示すように、特定のスタイルシートもしくはJavaScriptファイルをビューに追加することは簡単です。

リスト7-26 - アセットファイルのインクルード

    // view.ymlにおいて
    indexSuccess:
      stylesheets: [mystyle1, mystyle2]
      javascripts: [myscript]

    [php]
    // アクションにおいて
    $this->getResponse()->addStylesheet('mystyle1');
    $this->getResponse()->addStylesheet('mystyle2');
    $this->getResponse()->addJavascript('myscript');

    // テンプレートにおいて
    <?php use_stylesheet('mystyle1') ?>
    <?php use_stylesheet('mystyle2') ?>
    <?php use_javascript('myscript') ?>
    
それぞれの場合、引数はファイル名です。ファイルがロジカルな拡張子を持つ場合(`.css`はスタイルシート用、`.js`はJavaScriptファイル用)、これを省略することができます。ファイルがロジカルな位置(`/css/`はスタイルシート用で、`/js/`はJavaScriptファイル用)を持つ場合、同様に省略できます。symfonyは正しい拡張子、もしくは位置を理解するのに十分賢いです。

メタとタイトルの定義とは異なり、ファイルインクルージョンの定義はテンプレートのヘルパーもしくは含まれるレイアウトを必要としません。このことは以前の設定はテンプレートとレイアウトの内容が何であれ、リスト7-27のHTMLコードを出力することを意味します。

リスト7-27 - ファイルのインクルードの結果 -- レイアウトのなかではヘルパー呼び出しに対して必要ない

    [php]
    <head>
    ...
    <link rel="stylesheet" type="text/css" media="screen" href="/css/mystyle1.css" />
    <link rel="stylesheet" type="text/css" media="screen" href="/css/mystyle2.css" />
    <script language="javascript" type="text/javascript" src="/js/myscript.js">
    </script>
    </head>

>**NOTE**
>レスポンスのなかでのスタイルシートとJavaScriptのインクルージョンは`sfCommonFilter`という名前のフィルターによって実行されます。これはレスポンスの`<head>`タグを探し、`</head>`を丁度閉じるまえに`<link>`タグと`<script>`タグを追加します。レイアウトもしくはテンプレートのなかに`<head>`タグが存在しない場合、インクルージョンは行われないことを意味します。

設定カスケードの原則が適用されるので、アプリケーションの`view.yml`のなかで定義された任意のファイルインクルージョンはアプリケーションのすべてのページで現れます。リスト7-28、7-29、7-30はこの原則のお手本を示しています。

リスト7-28 - アプリケーションの`view.yml`のサンプル

    default:
      stylesheets: [main]

リスト7-29 - モジュールの`view.yml`のサンプル

    indexSuccess:
      stylesheets: [special]

    all:
      stylesheets: [additional]

リスト7-30 - `indexSuccess`ビューの結果

    [php]
    <link rel="stylesheet" type="text/css" media="screen" href="/css/main.css" />
    <link rel="stylesheet" type="text/css" media="screen" href="/css/additional.css" />
    <link rel="stylesheet" type="text/css" media="screen" href="/css/special.css" />

より高いレベルで定義されたファイルを除外することが必要がある場合、リスト7-31で示されるように、より低いレベルの定義で、マイナス記号(`-`)をファイルの名前のまえに追加します。

リスト7-31 - モジュールの`view.yml`のサンプルはアプリケーションレベルで定義されたファイルをとり除く

    indexSuccess:
      stylesheets: [-main, special]

    all:
      stylesheets: [additional]

すべてのスタイルシートもしくはJavaScriptファイルを除外するには、リスト7-32で示されるように、`-*`をファイル名として使います。

リスト7-32 - アプリケーションレベルで定義されたすべてのファイルを除外するモジュールの`view.yml`のサンプル

    indexSuccess:
      stylesheets: [-*]
      javascripts: [-*]

リスト7-33で示されるように、ファイルをインクルードする位置(最初もしくは最後の位置)を強制するためにより正確な追加のパラメーターを定義することができます。これはスタイルシートとJavaScriptの両方に対して機能にします。

リスト7-33 - インクルードされたアセットの位置を定義する

    // view.ymlにおいて
    indexSuccess:
      stylesheets: [special: { position: first }]

    [php]
    // アクションにおいて
    $this->getResponse()->addStylesheet('special', 'first');
    
    // テンプレートにおいて
    <?php use_stylesheet('special', 'first') ?>

**symfony 1.1の新しい機能**: リスト7-34で示されるように、結果の`<link>`もしくは`<script>`のタグが指定された正しい位置を参照するように、アセットファイル名の変形を回避することを決めることもできます。

リスト7-34 - そのままの名前でスタイルシートのインクルージョン

    // the view.ymlにて
    indexSuccess:
      stylesheets: [main, paper: { raw_name: true }]

    [php]
    // アクションにて
    $this->getResponse()->addStylesheet('main', '', array('raw_name' => true));
    
    // テンプレートにて
    <?php use_stylesheet('main', '', array('raw_name' => true)) ?>
    
    // ビューの結果
    <link rel="stylesheet" type="text/css" media="print" href="main" />

リスト7-35で示されるように、スタイルシートのインクルージョンに対してmediaを指定するために、デフォルトのスタイルシートのタグのオプションを変更できます。

リスト7-35 - mediaをともなうスタイルシートのインクルージョン

    // view.ymlにおいて
    indexSuccess:
      stylesheets: [main, paper: { media: print }]

    [php]
    // アクションにおいて
    $this->getResponse()->addStylesheet('paper', '', array('media' => 'print'));
    
    // テンプレートにおいて
    <?php use_stylesheet('paper', '', array('media' => 'print')) ?>
    
    // 結果のビュー
    <link rel="stylesheet" type="text/css" media="print" href="/css/paper.css" />

### レイアウトのコンフィギュレーション

Webサイトのグラフィカルな表にしたがって、いくつかのレイアウトを持ちます。古典的なWebサイトは少なくとも2つ持ちます: デフォルトのレイアウトとポップアップのレイアウトです。

デフォルトのレイアウトが`myproject/apps/frontend/templates/layout.php`であることはすでに見ました。追加のレイアウトは同じグローバルな`templates/`ディレクトリに追加しなければなりません。`frontend/templates/my_layout.php`ファイルでビューが欲しい場合、リスト7-36で示されるつぎの構文を使います。 

リスト7-36 - レイアウトの定義

    // view.ymlにおいて
    indexSuccess:
      layout: my_layout

    [php]
    // アクションにおいて
    $this->setLayout('my_layout');
    
    // テンプレートにおいて
    <?php decorate_with('my_layout') ?>

ビューのなかにはレイアウトがまったく必要のないものがあります(たとえば、RSSフィード上のプレーンテキストのページ)。この場合、リスト7-37で示されるように、`has_layout`を`false`に設定してください。

リスト7-37 - レイアウトの除去

    // `view.yml`にて
    indexSuccess:
      has_layout: false

    [php]
    // アクションにて
    $this->setLayout(false);
    
    // テンプレートにて
    <?php decorate_with(false) ?>

>**NOTE**
>デフォルトではAjaxのアクションのビューはレイアウトを持ちません。

コンポーネントスロット
----------------------

ビューのコンポーネントとビューの設定の力を組み合わせることでビューの開発に新しい観点がもたらされます: コンポーネントスロット(component slot)システムです。再利用性とレイヤーの分離に焦点を当てるスロットの代替物です。ですので、コンポーネントスロットはスロットよりも構造的ですが、実行は少し遅いです。

スロットのように、コンポーネントスロットはビューの要素内で宣言できる名前つきのプレースホルダーです。違いは入力コードを決定する方法にあります。スロットに対して、コードは別のビューの要素のなかに設定されます; コンポーネントスロットに対して、コードはコンポーネントの実行から由来し、このコンポーネントの名前はビューの設定から由来します。アクションのなかでコンポーネントスロットを見た後にこれらの理解がより深まります。

コンポーネントスロットのプレースホルダーを設定するには、`include_component_slot()`ヘルパーを使います。この関数はラベルをパラメーターとして必要とします。たとえば、アプリケーションの`layout.php`ファイルが文脈上のサイドバーを含む場合を想像してください。リスト7-38はコンポーネントスロットのヘルパーがインクルードされる方法を示しています。

リスト7-38 - 'sidebar'という名前のコンポーネントをインクルードする

    [php]
    ...
    <div id="sidebar">
      <?php include_component_slot('sidebar') ?>
    </div>

コンポーネントスロットのラベルとコンポーネントの名前の間の対応をビューの設定のなかで定義します。たとえば、アプリケーションの`view.yml`のなかの、`components`ヘッダーの下で`sidebar`コンポーネントに対してデフォルトのコンポーネントを設定してください。キーはコンポーネントスロットのラベルです; 値はモジュールとコンポーネントの名前を含む配列でなければなりません。リスト7-39は例を示しています。

リスト7-37 - 'sidebar'スロットコンポーネントのデフォルトを定義する(`frontend/config/view.yml`)

    default:
      components:
        sidebar:  [bar, default]

レイアウトが実行されたとき、`sidebar`コンポーネントスロットは`bar`モジュールに設置された`barComponents`クラスの`executeDefault()`メソッドの結果で内容が満たされ、このメソッドは`modules/bar/templates/`に設置された`_default.php`部分テンプレートを表示します。

設定カスケードによって任意のモジュールに対してこの設定をオーバーライドできます。たとえば、`user`モジュールにおいて、ユーザー名とユーザーが公開した記事の数を表示する文脈上のコンポーネントが欲しいことがあります。この場合、リスト7-40で示されるように、モジュールの`view.yml`のなかでサイドバースロットの設定を特化します。

リスト7-38 - 'sidebar'スロットコンポーネントを特化する(`frontend/modules/user/config/view.yml`)

    all:
      components:
        sidebar:  [bar, user]

このスロットを処理するコンポーネントの定義はリスト7-41のようなものになります。

リスト7-41 - 'sidebar'スロットによって使われるコンポーネント(`modules/bar/actions/components.class.php`)

    [php]
    class barComponents extends sfComponents
    {
      public function executeDefault()
      {
      }

      public function executeUser()
      {
        $this->current_user = $this->getUser()->getCurrentUser();
        $c = new Criteria();
        $c->add(ArticlePeer::AUTHOR_ID, $this->current_user->getId());
        $this->nb_articles = ArticlePeer::doCount($c);
      }
    }

リスト7-42はこれら2つのコンポーネントのためのビューを示しています。

リスト7-42 - 'sidebar'スロットコンポーネントによって使われる部分テンプレート(`modules/bar/templates/`)

    [php]
    // _default.php
    <p>この領域はコンテキスト上の情報を含みます。</p>

    // _user.php
    <p>ユーザー名: <?php echo $current_user->getName() ?></p>
    <p><?php echo $nb_articles ?> articles published</p>

コンポーネントスロットはパンくずリスト、コンテキスト上のナビゲーション、あらゆる種類の動的な挿入のために使われます。コンポーネントとして、これらはグローバルレイアウトと通常のテンプレート、もしくはほかのコンポーネントのなかで使用できます。スロットのコンポーネントを設定するコンフィギュレーションは最後に呼び出されたアクションのコンフィギュレーションからつねに除外されます。

任意のモジュールに対してコンポーネントスロットの利用を一時中止する必要がある場合、リスト7-43で示されるように、空のモジュール/コンポーネントを宣言します。

リスト7-43 - コンポーネントスロットを無効にする(`view.yml`)

    all:
      components:
        sidebar:  []

出力エスケーピング機能
----------------------

動的なデータをテンプレートに挿入するとき、データの統合性について気をつけなければなりません。たとえば、データが匿名ユーザーから入力されたフォームから来た場合、クロスサイトスクリプティング(XSS - Cross-Site Scripting)攻撃を開始する悪意のあるスクリプトを含んでいるリスクが存在する可能性があります。出力データに含まれるHTMLタグが無害になるように、出力データをエスケープできることが必須です。

例として、ユーザーが入力フィールドをつぎのような値で満たすことを想像してください:

    [php]
    <script>alert(document.cookie)</script>

警告なしでこの値をechoする場合、JavaScriptはすべてのブラウザーで実行され、アラートを表示するよりはるかに危険な攻撃を許してしまいます。値をつぎのようにするために、表示するまえに値をエスケープしなければならない理由はそういうわけです:

    [php]
    &lt;script&gt;alert(document.cookie)&lt;/script&gt;

`htmlspecialchars()`の呼び出しで信頼性のないすべての値をとり囲むことで出力を手動でエスケープできますが、このアプローチは繰り返し作業がとても多くエラーが起こりがちです。代わりに、symfonyは出力エスケーピング(output escaping)と呼ばれる特別なシステムを提供します。これは自動的にテンプレート内のすべての変数出力をエスケープします。これはアプリケーションの`settings.yml`内の単純なパラメーターで有効になります。

### 出力エスケーピング機能を有効にする

出力エスケーピング機能は`settings.yml`のなかでアプリケーションに対してグローバルに設定されます。2つのパラメーターが出力エスケーピングの動作方法をコントロールします: 変数がビューに対して利用できるようにする方法を戦略が決定し、メソッドはデータに適用されるデフォルトのエスケーピング機能です。

基本的に、出力エスケーピング機能を有効にするために必要なことは、リスト7-42で示されるように、`escaping_strategy`パラメーターをデフォルト値である`off`の代わりに`on`に設定することです。

リスト7-42 - 出力エスケーピング機能を有効にする(`frontend/config/settings.yml`)

    all:
      .settings:
        escaping_strategy: on
        escaping_method:   ESC_SPECIALCHARS

これは`htmlspeciachars()`をデフォルトですべての変数の出力に追加します。たとえば、つぎのようにアクションで`test`変数を定義することを想像してみましょう:

    [php]
    $this->test = '<script>alert(document.cookie)</script>';

出力エスケーピング機能を有効にした場合、この変数をテンプレートのなかでechoするとエスケープされたデータが出力されます:

    [php]
    echo $test;
     => &lt;script&gt;alert(document.cookie)&lt;/script&gt;

また、すべてのテンプレートは`$sf_data`変数にアクセスすることができます。これはエスケープされたすべての変数を参照するコンテナオブジェクトです。ですのでつぎのようにテスト変数を出力することもできます:

    [php]
    echo $sf_data->get('test');
    => &lt;script&gt;alert(document.cookie)&lt;/script&gt;

>**TIP**
>`$sf_data`オブジェクトはArrayインターフェイスを実装するので、`$sf_data->get('myvariable')`を使う代わりに、`$sf_data['myvariable']`を呼び出すことでエスケープされた値をとり出すことができます。しかし、これは本当の配列ではないので、`print_r()`のような関数は期待どおりに動きません。

`$sf_data`によってエスケープされていない、もしくは生のデータにもアクセスできます。変数を信用することが前提になりますが、このことは変数がブラウザーに解釈されることを意図したHTMLコードを保存するときに便利です。生のデータを出力することが必要なとき、`getRaw()`メソッドを呼び出します。

    [php]
    echo $sf_data->getRaw('test');
     => <script>alert(document.cookie)</script>

実際にHTMLとして解釈されるHTMLを含む変数を必要とするたびに生のデータにアクセスしなければなりません。テンプレートをインクルードするためにデフォルトのレイアウトが`$sf_content`の代わりに`$sf_data->getRaw('sf_content')`を使う理由がこれで理解できたでしょう。出力エスケーピングが有効になったときに`$sf_content`が壊れるからです。

`escaping_strategy`が`off`のとき、`$sf_data`はまだ利用できますが、つねに名前のデータを返します。

>**TIP**
>symfony 1.0では`escaping_strategy`に対して2つの別の値がありました。`bc`は現在`off`に、`both`は`on`に置き換えられました。これらの値を使用してもまだ機能しますが、エラーをロギングすることになります。

### エスケーピングヘルパー

エスケーピングヘルパー(escaping helper)はエスケープされたバージョンの入力を返す機能です。これらは`settings.yml`ファイルのなかでデフォルトの`escaping_method`として、もしくはビューのなかで特定の値のためのエスケーピングメソッドを指定するために提供されます。つぎのエスケーピングヘルパーが利用できます:

  * `ESC_RAW`: 値をエスケープしません。
  * `ESC_SPECIALCHARS`: PHP関数の`htmlspecialchars()`を入力に適用する。
  * `ESC_ENTITIES`: 引用形式として`ENT_QUOTES`でPHP関数の`htmlentities()`を入力に適用します。
  * `ESC_JS`: HTMLとして使われる予定のJavaScriptの文字列のなかに入れられる値をエスケープする。これはJavaScriptの利用によってHTMLが動的に変更されるものをエスケープするために便利です。
  * `ESC_JS_NO_ENTITIES`: エンティティを追加しないがJavaScriptの文字列のなかに入れられる値をエスケープする。これはダイアログボックスを利用して値を表示する場合に便利です(たとえば、`javascript:alert(myString);`のなかで使われる`myString`変数)。

### 配列とオブジェクトをエスケープする

出力エスケーピングは文字列だけでなく、配列とオブジェクトに対しても機能します。オブジェクトもしくは配列である値はそれらのエスケープされた状態をそれらの子供に渡します。`escaping_strategy`を`on`に設定したと仮定すると、リスト7-43はエスケーピングカスケードのお手本を示しています。

リスト7-45 - エスケーピング機能は配列とオブジェクトに対しても作用する

    [php]
    // クラスの定義
    class myClass
    {
      public function testSpecialChars($value = '')
      {
        return '<'.$value.'>';
      }
    }

    // アクションにおいて
    $this->test_array = array('&', '<', '>');
    $this->test_array_of_arrays = array(array('&'));
    $this->test_object = new myClass();

    // テンプレートにおいて
    <?php foreach($test_array as $value): ?>
      <?php echo $value ?>
    <?php endforeach; ?>
     => &amp; &lt; &gt;
    <?php echo $test_array_of_arrays[0][0] ?>
     => &amp;
    <?php echo $test_object->testSpecialChars('&') ?>
     => &lt;&amp;&gt;

実際のところ、テンプレート内部の変数はあなたが期待した型ではありません。出力エスケーピングシステムはこれらを"デコレート"して、特別なオブジェクトに変換します:

    [php]
    <?php echo get_class($test_array) ?>
     => sfOutputEscaperArrayDecorator
    <?php echo get_class($test_object) ?>
     => sfOutputEscaperObjectDecorator

これはいくつかの通常のPHP関数(`array_shift()`、`print_r()`などの)がエスケープされた配列上ではもはや機能しない理由を説明しています。しかし、これらはまだ`[]`を使うことでアクセスすることが可能で、`foreach`を使うことで展開され、これらは`count()`で正しい結果を返します(`count()`はPHP 5.2かそれ以降で動作します)。そしてテンプレートにおいて、ともかくデータはリードオンリーなので、たいていのアクセスは実際に動作するメソッドを通したものになります。

`$sf_data`オブジェクトを通して生のデータをとり出す方法がまだあります。加えて、エスケープされたオブジェクトのメソッドは追加パラメーター: エスケーピングメソッドを受けとるために変更されます。テンプレートの変数を表示するたびに代わりのエスケーピングメソッド、もしくはエスケーピングを無効にする`ESC_RAW`ヘルパーを選ぶことができます。リスト7-46の例をご覧ください。

リスト7-46 - エスケープされたオブジェクトのメソッドは追加パラメーターを受けとる

    [php]
    <?php echo $test_object->testSpecialChars('&') ?>
    => &lt;&amp;&gt;
    // つぎの3行は同じ値を返す
    <?php echo $test_object->testSpecialChars('&', ESC_RAW) ?>
    <?php echo $sf_data->getRaw('test_object')->testSpecialChars('&') ?>
    <?php echo $sf_data->get('test_object', ESC_RAW)->testSpecialChars('&') ?>
     => <&>

テンプレート内のオブジェクトを処理する場合、追加パラメーターのトリックを多く利用することになります。これがメソッド呼び出し上で生のデータを得るための最速の方法だからです。

>**CAUTION**
>出力エスケーピングを有効にした場合、symfonyの通常の変数もエスケープされます。`$sf_user`、`$sf_request`、`$sf_param`と`$sf_context`はそのまま機能しますが、これらが持つメソッドの呼び出しに`ESC_RAW`を最後の引数として追加しないかぎり、メソッドはエスケープされたデータを返すことを覚えておいてください。

>**TIP**
>**symfony 1.1の新しい機能**: たとえXSSがもっとも共通のセキュリティ上の弱点の1つであったとしても、これは唯一の弱点ではありません。CSRFはとても有名なのでsyfmonyはアプリケーションを保護するためのシンプルな方法を提供します。詳細な情報に関しては6章の"CSRFフィルター"という名前の読み物をお読みください。

まとめ
----

プレゼンテーションレイヤーを操作するためにあらゆる種類のツールを利用できます。ヘルパー(helper)のおかげで、テンプレート(template)が秒単位で作られます。レイアウト(layout)、部分テンプレート(partial - もしくはパーシャル)、コンポーネント(component)とコンポーネントスロット(component slot)によってモジュール性と再利用性の両ほうがもたらされます。ビュー(view)の設定においてたいていのページのヘッダーを扱うために速く書けるYAMLを利用します。設定カスケード(configuration cascade)によってビューごとにすべての設定を定義しないですみます。動的なデータに依存するプレゼンテーションのすべての修正のために、アクションは`sfResponse`オブジェクトにアクセスできます。そして出力エスケーピングシステム(output escaping system)のおかげで、ビューはクロスサイトスクリプティング(XSS - Cross-Site Scripting)攻撃から安全です。
