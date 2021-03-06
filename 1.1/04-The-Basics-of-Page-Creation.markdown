第4章 - ページの作り方の基本
============================

興味深いことに、プログラマが新しい言語もしくはフレームワークを学ぶとき、最初に学ぶチュートリアルは"Hello,world!"をスクリーンに表示させることです。人工知能の分野におけるあらゆる試みが今のところ貧弱な会話能力の結果で終わっていることをふまえると、コンピュータを世界全体に挨拶できる何かと考えるのは奇妙なことです。しかしながら、symfonyはほかのプログラムよりも愚かではなく、その証明として、symfonyで"Hello,`<あなたのお名前>`"と表示するページを作ることができます。

この章ではモジュールの作り方をお教えします。モジュール(module)とはページを分類する構造上の要素です。MVCパターンなので、アクションとテンプレートに分割されたページの作り方も学びます。リンクとフォームはWebの基本的なインタラクションです。これらをテンプレートに挿入してアクションで処理する方法を理解します。

モジュールのスケルトンを作成する
-------------------------------

2章で説明したように、symfonyはページをモジュールに分類します。ページを作るまえに、モジュールを作成する必要があります。モジュールはsymfonyが認識できるファイル構造を持つ骨組みで最初は内容を持ちません。

symfonyコマンドはモジュールの作成作業を自動化します。アプリケーションの名前とモジュールの名前を引数として`generate:module`タスクを呼び出すことだけが必要です。前の章で作成した`frontend`アプリケーションに`content`モジュールを追加するには、つぎのコマンドを入力します:

    > cd ~/myproject
    > php symfony generate:module frontend content

    >> dir+      ~/myproject/apps/frontend/modules/content/actions
    >> file+     ~/myproject/apps/frontend/modules/content/actions/actions.class.php
    >> dir+      ~/myproject/apps/frontend/modules/content/templates
    >> file+     ~/myproject/apps/frontend/modules/content/templates/indexSuccess.php
    >> file+     ~/myproject/test/functional/frontend/contentActionsTest.php
    >> tokens    ~/myproject/test/functional/frontend/contentActionsTest.php
    >> tokens    ~/myproject/apps/frontend/modules/content/actions/actions.class.php
    >> tokens    ~/myproject/apps/frontend/modules/content/templates/indexSuccess.php

`actions/`と`templates/`ディレクトリは別にして、このコマンドは3つのファイルだけ作りました。`test/`フォルダーのなかに存在するファイルは機能テストに関係しますが、15章まで気にする必要はありません。`actions.class.php`(リスト4-1で示されている)は`default`モジュールの初期ページに転送(フォワード)します。`templates/indexSuccess.php`ファイルは空です。

リスト4-1 - 生成されたデフォルトのアクション(`actions/actions.class.php`)

    [php]
    <?php

    class contentActions extends sfActions
    {
      public function executeIndex()
      {
        $this->forward('default', 'module');
      }
    }

>**NOTE**
>実際の`actions.class.php`ファイルを見ると、多くのコメントを含めて、これら数行よりも多くのものが見つかります。なぜならsymfonyはプロジェクトのドキュメントを作るためにPHP形式のコメントを使い、phpDocumentorツール([http://www.phpdoc.org/](http://www.phpdoc.org/))と互換性のあるクラスファイルを用意することを推奨しているからです。

新しいそれぞれのモジュールに対して、symfonyはデフォルトの`index`アクションを作ります。これは`executeIndex`と呼ばれるアクションメソッドと`indexSuccess.php`と呼ばれるテンプレートファイルから構成されます。プレフィックスの`execute`とサフィックスの`Success`の意味は6章と7章で説明します。それまでの間はこの命名方法を規約として考えることにします。つぎのURLをブラウザーに入力すると対応ページ(図4-1で再現)を見ることができます:

    http://localhost/frontend_dev.php/content/index

この章ではデフォルトの`index`アクションを使わないので、`actions.class.php`ファイルから`executeIndex()`メソッドを、`templates/`ディレクトリから`indexSuccess.php`ファイルを削除できます。

>**NOTE**
>symfonyはコマンドライン以外にもモジュールを初期化するほかの方法を提供します。その1つはあなた自身でディレクトリを作ることです。多くの場合、アクションとモジュールのテンプレートは任意のテーブルのデータを操作するために作られます。テーブルからレコードを作成する、読みとる、更新する、削除するために必要なコードはしばしば同じなので、symfonyはこのコードを生成するためのメカニズムを提供します。このテクニックに関する詳細な情報は14章を参照してください。

図4-1 - 生成されたデフォルトのindexページ

![生成されたデフォルトのindexページ](http://github.com/masakielastic/symfonybook-ja/raw/master/images/F0401.jpg "生成されたデフォルトのindexページ")

ページを追加する
----------------

symfonyにおいて、ページの背後にあるロジックはアクションに保存され、プレゼンテーションはテンプレートに保存されます。ロジックをともなわないページでも空のアクションを必要とします。

### アクションを追加する

"Hello, world!"のページは`show`アクションを通してアクセスできます。リスト4-2で示されているように、このアクションを作るには、`contentActions`クラスに`executeShow`メソッドを追加します。

リスト4-2 - アクションを追加するにはアクションクラスに実行メソッドを追加する(`actions/actions.class.php`)

    [php]
    <?php

    class contentActions extends sfActions
    {
      public function executeShow()
      {
      }
    }

アクションのメソッド名はつねに`executeXxx()`で、名前の2番目の部分は大文字で始まるクションの名前です。

では、つぎのURLをリクエストしてください:

    http://localhost/frontend_dev.php/content/show

symfonyが`showSuccess.php`テンプレートが見つからないことを訴えます。これは正常なことです; symfonyにおいて、ページはつねにアクションとテンプレートで構成されます。

>**CAUTION**
>URL(ドメイン名ではありません)と同じように、symfonyは大文字と小文字を区別します(PHPのメソッド名は大文字と小文字を区別しません)。このことはブラウザーで`sHow`を呼び出すと、symfonyが404エラーを返すことを意味します。

-

>**SIDEBAR**
>URLはレスポンスの一部
>
>symfonyは実際のアクションの名前とアクションを呼び出すために必要なURLの形式を完全に分離できるようにするルーティングシステムを備えています。このメカニズムによってURLをあたかもレスポンスの一部のようにカスタム形式で整えることができます。ファイル構造やリクエストパラメーターによって制限されることはもはやありません; アクション用のURLを望むフレーズで表示できます。たとえば、通常、articleモジュールのindexアクションを呼び出すにはURLはつぎのようになります:
>
>     http://localhost/frontend_dev.php/article/index?id=123
>
>このURLではデータベースから任意の記事をとり出します。この例では、ヨーロッパのセクションのとりわけフランスのファイナンスを議論している記事(`id=123`)がとり出されます。しかしながら、`routing.yml`設定ファイルを少し変更することでURLを完全に異なる形式で書けます:
>
>     http://localhost/articles/europe/france/finance.html
>
>検索エンジンにわかりやすいURLになるだけでなく、ユーザーにとっても重要です。ユーザーは、カスタムクエリを行うために、つぎのようにアドレスバーを擬似コマンドラインとして使うことができます:
>
>     http://localhost/articles/tagged/finance+france+euro
>
>symfonyはユーザーのためにスマートURLを解析し生成する方法を知っています。ルーティングシステムはスマートURLからリクエストパラメーターを自動的に読みとり、アクションがこれらを利用できるようにします。レスポンスに含まれるハイパーリンクの形式が整えられるので、これらは"スマート"に見えます。この機能は9章で詳しく学ぶことになります。
>
>全体的に、このことはアプリケーションのアクションを命名する方法は、アクションを呼び出すために使われるURLの見た目ではなく、アプリケーションのアクションの機能によって影響されることを意味します。アクションの名前はアクションが実際に行うことを説明し、不定形の動詞(`show`、`list`、`edit`など)であることがよくあります。アクションの名前は全体的にエンドユーザーには見えないので、明確な名前(`listByName`もしくは`showWithComments`など)を使うことをためらわないでください。アクションの機能を説明するコードのコメントを書かずにすむことに加えて、コードがはるかに読みやすくなります。

### テンプレートを追加する

アクションは自分自身をレンダリングすることをテンプレートに要求します。テンプレートはモジュールの`templates/`ディレクトリに設置されたファイルで、アクションの名前とアクションのサフィックスをつなげて命名されます。アクションのデフォルトの接尾辞は"success"なので、`show`アクションのために作られるテンプレートファイルは`showSuccess.php`という名前になります。

テンプレートはプレゼンテーション用のコードだけを含むことを前提としているので、これらをできるかぎり小さなPHPコードとして維持してください。当然のことながら、"Hello, world!"を表示するページはリスト4-3のようなシンプルなテンプレートを持つことができます。

リスト4-3 - テンプレート(`content/templates/showSuccess.php`)

    [php]
    <p>Hello, world!</p>

テンプレートのなかでPHPコードを実行する必要がある場合、リスト4-4で示すように、通常のPHP構文は避けるべきです。代わりに、PHPプログラマではない人でも理解できるように、リスト4-5で示されるPHPの代替構文を使うテンプレートを書きます。最終的なコードを正しくインデントするだけでなく、アクションの複雑なPHPコードを維持するための助けになります。なぜなら制御文(`if`、`foreach`、`while`など)だけが代替構文を持つからです。

リスト4-4 - アクションにはよいが、テンプレートにはよくない通常のPHP構文(`templates/showSuccess.php`)

    [php]
    <p>Hello, world!</p>
    <?php

    if ($test)
    {
      echo "<p>".time()."</p>";
    }

    ?>

リスト4-5 - テンプレート用の代替のPHP構文のよい例(`templates/showSuccess.php`)

    [php]
    <p>Hello, world!</p>
    <?php if ($test): ?>
      <p><?php echo time(); ?></p>
    <?php endif; ?>

>**TIP**
>テンプレートの構文が十分に読みやすいものか確認するためのよい経験則はPHPもしくは波かっこでechoされるHTMLコードを含まないことです。また多くの場合、`<?php`で展開するとき、同じ行の`?>`で閉じることです。

### テンプレートにアクションからの情報を渡す

アクションの仕事はすべての複雑な計算、データのとり出し、とテストの実施、echoもしくはテストされるテンプレートに対して変数を設定することです。symfonyは(アクションの`$this->variableName`経由でアクセスされる)アクションクラスのプロパティが(`$variableName`経由で)グローバルな名前空間内のテンプレートに直接アクセスできるようにします。リスト4-6と4-7はテンプレートにアクションからの情報を渡す方法を示しています。

リスト4-6 - テンプレートがアクションのプロパティを利用できるようにアクションを設定する(`content/actions/actions.class.php`)

    [php]
    <?php

    class contentActions extends sfActions
    {
      public function executeShow()
      {
        $today = getdate();
        $this->hour = $today['hours'];
      }
    }

リスト4-7 - テンプレートはアクションのプロパティに直接アクセスできる(`templates/showSuccess.php`)

    [php]
    <p>Hello, world!</p>
    <?php if ($hour >= 18): ?>
      <p>Or should I say good evening? It is already <?php echo $hour ?>.</p>
    <?php endif; ?>

省略形式の開始タグ(`<?=`は`<?php echo`と同じ)の使用はプロフェッショナルなWebアプリケーションでは非推奨であることに注意してください。運用のWebサーバーが複数のスクリプト言語を混同する可能性があるからです。加えて省略形式の開始タグはPHPのデフォルト設定では機能せず、有効にするためにサーバーを調整する必要があるからです(訳注：short_open_tagディレクティブをonに設定する)。結局のところ、XMLとバリデーションを処理しなければならないとき、XMLにおいて`<?`は特別な意味を持つので不十分な表記です。

>**NOTE**
>アクションの変数をセットアップしなくてもデフォルトでテンプレートはデータのごく一部にアクセスできます。すべてのテンプレートは`$sf_request`, `$sf_params`、`$sf_response`、と`$sf_user`オブジェクトのメソッドを呼び出すことができます。これらのオブジェクトは現在のリクエスト、リクエストパラメーター、セッションに関連するデータを格納します。まもなくこれらを効率的に使う方法を学ぶことになります。

別のアクションにリンクする
--------------------------

アクションの名前とそれを呼び出すURLが完全に分離されていることはご存じのとおりです。ですのでリスト4-10のようにテンプレートのなかで`update`アクションへのリンクを作る場合、リンクはデフォルトのルーティングのみで機能します。URLの表示方法の変更をあとで決める場合、ハイパーリンクを変更するにはすべてのテンプレートを再吟味する必要があります。

リスト4-10 - 古典的なハイパーリンク

    [php]
    <a href="/frontend_dev.php/content/update?name=anonymous">
      私の名前は教えません
    </a>

このやっかいな問題を避けるには、アプリケーションのアクションへのハイパーリンクを作る`link_to`ヘルパーをつねに使うべきです。そしてURLの部分を生成したい場合、`url_for()`が求めるヘルパーです。

ヘルパー(helper)とはsymfonyで定義されたテンプレートのなかでHTMLコードを出力するPHP関数です。HTMLコードを直接書くよりこれを使うほうが速く書けます。リスト4-11はハイパーリンクヘルパーの使いかたを示しています。

リスト4-11 - `link_to()`と`url_for()`ヘルパー(`templates/***Success.php`)

    [php]
    <p>こんにちは！</p>
    <?php if ($hour >= 18): ?>
    <p>もしくはこんばんはと言ったほうがよろしいでしょうか？現在<?php echo $hour ?>時です。</p>
    <?php endif; ?>
    <form method="post" action="<?php echo url_for('content/update') ?>">
      <label for="name">お名前は？</label>
      <input type="text" name="name" id="name" value="" />
      <input type="submit" value="Ok" />
      <?php echo link_to('名前を教えない','content/update?name=anonymous') ?>
    </form>

ルーティングルールを変更しても、すべてのテンプレートは正しくふるまい、ルールにしたがってURLを再び整形すること以外結果のHTMLは以前のものと同じになります。

symfonyはフォームの操作をずっと簡単になるツールをたくさん提供するので、フォームの操作はそれだけで1つの章の説明をする必要があります。10章でこれらのヘルパーについて詳細な内容を学ぶことになります。

`link_to()`ヘルパーは、多くのヘルパーと同じように、特別なオプションと追加のタグ属性用の別の引数を受けとります。リスト4-12はオプション引数の例と結果のHTMLを示しています。オプション引数は連想配列もしくは空白文字で区切られた`key=value`の組を表示するシンプルな文字列です。

リスト4-12 - たいていのヘルパーはオプション引数を受けとる(`templates/***Success.php`)

    [php]
    // 連想配列としてのオプション引数
    <?php echo link_to('名前を教えない', 'content/update?name=anonymous',
      array(
        'class'    => 'special_link',
        'confirm'  => 'よろしいですか？',
        'absolute' => true
    )) ?>

    // 文字列としてのオプション引数
    <?php echo link_to('名前を教えない', 'content/update?name=anonymous',
      'class=special_link confirm=Are you sure? absolute=true') ?>

    // 両方の呼び出しは同じものを出力する
     => <a class="special_link" onclick="return confirm('Are you sure?');"
        href="http://localhost/frontend_dev.php/content/update/name/anonymous">
        名前を教えない</a>

HTMLタグを出力するsymfonyのヘルパーを使うとき、オプション引数に(リスト4-12の例の`class`属性のような)追加のタグ属性を挿入できます。これらの属性を"速くて汚い"HTML 4.0の方法(ダブルクォートなし)で書くこともできますが、symfonyはこれらをすばらしく整形されたXHTMLの形式で出力します。それがHTMLよりもヘルパーを速く書ける別の理由です。

>**NOTE**
>追加の解析と変換が必要なので、文字列の構文は配列よりも若干遅いです。

symfonyのすべてのヘルパーのように、リンクヘルパーとオプションは数多く存在します。9章でこれらを詳しく説明します。

リクエストから情報を入手する
----------------------------

ユーザーが送信した情報がフォーム経由(通常はPOSTリクエスト)もしくはURL経由(GETリクエスト)のどちらでも、`sfRequest`オブジェクトの`getParameter()`メソッドを使用してアクションから関連データをとり出すことができます。リスト4-13は、`update`メソッドで`name`パラメーターの値をとり出す方法を示しています。

リスト4-13 - データをアクションのリクエストパラメーターから取得する(`content/actions/actions.class.php`)

    [php]
    <?php

    class contentActions extends sfActions
    {
      // ...

      public function executeUpdate($request)
      {
        $this->name = $request->getParameter('name');
      }
    }

利便性に関して、すべての`executeXxx()`メソッドは最初の引数として現在の`sfRequest`オブジェクトを受けとります。

データの操作がシンプルであるなら、リクエストパラメーターをとり出すためにアクションを使う必要もありません。テンプレートが`$sf_params`オブジェクトにアクセスできるからです。アクションの`getParameter()`と同様に、`$sf_params`はリクエストパラメーターをとり出す`get()`メソッドを提供します。

`executeUpdate()`が空の場合、リスト4-14は`updateSuccess.php`テンプレートが同じ`name`パラメーターをとり出す方法を示しています。

リスト4-14 - リクエストパラメーターをテンプレートから直接取得する(`templates/***Success.php`)

    [php]
    <p>Hello, <?php echo $sf_params->get('name') ?>!</p>

>**NOTE**
>代わりに`$_POST`、`$_GET`、`$_REQUEST`変数を使わないのはなぜでしょうか？URLは異なるようにフォーマットされるため(たとえば`?`や`=`をともなわない`http://localhost/articles/europe/france/finance.html`)、通常のPHP変数はそれ以上機能しないので、ルーティングシステムのみがリクエストパラメーターをとり出すことができるからです。悪意のあるコードのインジェクションを防止するために入力フィルタリングを追加したい場合、すべてのリクエストパラメーターを1つのクリーンなパラメーターホルダーに格納することで実現できます。

`$sf_params`オブジェクトは配列に同等のゲッターを単に渡すよりも強力です。たとえば、リクエストパラメーターの存在を確認したい場合、リスト4-15のように、`get()`で実際の値を試す代わりに`$sf_params->has()`メソッドが簡単に使えます。

リスト4-15 - テンプレートのなかでリクエストパラメーターを試す(`templates/***Success.php`)

    [php]
    <?php if ($sf_params->has('name')): ?>
      <p>こんにちは、<?php echo $sf_params->get('name') ?>さん！</p>
    <?php else: ?>
      <p>こんにちは、John Doeさん！</p>
    <?php endif; ?>

読者のなかにはこれを一行で書けることを推測している人がいることでしょう。symfonyのたいていのゲッターメソッドに関しては、アクションの`$request->getParameter()`メソッドとテンプレートの`$sf_params->get()`メソッド(実際には、同じオブジェクトの同じメソッドの呼び出し)の両方とも2番目の引数としてリクエストパラメーターが存在しない場合に使われるデフォルト値を受けとります。

    [php]
    <p>こんにちは、<?php echo $sf_params->get('name', 'John Doe') ?>さん！</p>

まとめ
----

symfonyにおいて、ページはアクション(action)とテンプレート(template)で構成されます。アクションは`actions/actions.class.php`ファイルのメソッドでプレフィックスは`execute`です。テンプレートは`templates/`ディレクトリのなかのファイルで、通常は`Success.php`で終わります。これらはアプリケーションの機能にしたがって、モジュール(module)に分類されます。ヘルパーによってテンプレートを書く作業が円滑になります。ヘルパー(helper)とはsymfonyによって提供されるHTMLコードを返す関数です。そして、必要に応じて整形されるURLはレスポンスの一部として考えることが必要です。ですのでアクションの命名もしくはリクエストパラメーターの読みとりにおいてURLを直接参照することは避けるべきです。

これらの基本原則を理解したら、すでにアプリケーション全体を書けます。しかしこれだけでは作業が長すぎます。アプリケーションの開発過程で成し遂げなければならないほとんどすべてのタスクはsymfonyの別の機能によって1つもしくは別の方法で円滑に行われます・・・今のところこの本が終わらない理由です。
