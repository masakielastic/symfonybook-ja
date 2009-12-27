第4章 - ページの作り方の基本
============================

興味深いことに、プログラマが新しい言語もしくはフレームワークを学ぶとき、最初に学ぶチュートリアルは"Hello,world!"をスクリーン上に表示させることです。人工知能の分野におけるあらゆる試みが今のところ貧弱な会話能力の結果で終わっているので、コンピュータを世界全体に挨拶できる何かと考えるのは奇妙なことです。しかしながら、symfonyはほかのプログラムよりも愚かではないので、その証明として、symfonyで"Hello,`<Your Name Here>`"と表示するページを作ることができます。

この章ではモジュールの作り方をお教えします。モジュール(module)とはページを分類する構造上の要素です。MVCパターンなので、アクションとテンプレートに分割されたページを作る方法も学びます。リンクとフォームはWebの基本的なインタラクションです。これらをテンプレートに挿入してアクションで扱う方法を理解します。

モジュールのスケルトンを作成する
--------------------------------

2章で説明したように、symfonyはページをモジュールに分類します。ページを作るまえに、モジュールを作成する必要があります。モジュールはsymfonyが認識できるファイル構造を持つ最初は内容を持たない骨組みです。

symfonyのコマンドラインはモジュールの作成作業を自動化します。アプリケーションの名前とモジュールの名前を引数として`init-module`タスクを呼び出す必要があるだけです。前の章で、`myapp`アプリケーションを作りました。`mymodule`モジュールをこの`myapp`アプリケーションに追加するために、つぎのコマンドを入力します:

    > cd ~/myproject
    > symfony init-module myapp mymodule

    >> dir+      ~/myproject/apps/myapp/modules/mymodule
    >> dir+      ~/myproject/apps/myapp/modules/mymodule/actions
    >> file+     ~/myproject/apps/myapp/modules/mymodule/actions/actions.class.php
    >> dir+      ~/myproject/apps/myapp/modules/mymodule/config
    >> dir+      ~/myproject/apps/myapp/modules/mymodule/lib
    >> dir+      ~/myproject/apps/myapp/modules/mymodule/templates
    >> file+     ~/myproject/apps/myapp/modules/mymodule/templates/indexSuccess.php
    >> dir+      ~/myproject/apps/myapp/modules/mymodule/validate
    >> file+     ~/myproject/test/functional/myapp/mymoduleActionsTest.php
    >> tokens    ~/myproject/test/functional/myapp/mymoduleActionsTest.php
    >> tokens    ~/myproject/apps/myapp/modules/mymodule/actions/actions.class.php
    >> tokens    ~/myproject/apps/myapp/modules/mymodule/templates/indexSuccess.php

`actions/`、`config/`、`lib/`、`templates/`と`validate/`ディレクトリは別にして、このコマンドは3つのファイルだけ作りました。`test/`フォルダー内部に存在するファイルはユニットテストに関係しますが、15章まで気にする必要はありません。`actions.class.php`(リスト4-1で示されている)は`default`モジュールの初期ページに転送(フォワード)します。`templates/indexSuccess.php`ファイルは空です。

リスト4-1 - 生成されたデフォルトのアクション(`actions/actions.class.php`)

    [php]
    <?php

    class mymoduleActions extends sfActions
    {
      public function executeIndex()
      {
        $this->forward('default', 'module');
      }
    }

>**NOTE**
>実際の`actions.class.php`ファイルを見ると、多くのコメントを含めて、これら数行よりも多くのものが見つかります。なぜならsymfonyはあなたのプロジェクトのドキュメントを作るためにPHP形式のコメントを使い、phpDocumentor([http://www.phpdoc.org/](http://www.phpdoc.org/))と互換性のあるそれぞれのクラスファイルを用意することを推奨しているからです。

新しいそれぞれのモジュールに対して、symfonyはデフォルトの`index`アクションを作ります。これは`executeIndex`と呼ばれるアクションメソッドと`indexSuccess.php`と呼ばれるテンプレートファイルから構成されます。プレフィックスの`execute`とサフィックスの`Success`の意味は6章と7章で説明します。それまでの間はこの命名方法を規約として考えることにします。つぎのURLをブラウザーに入力することで対応するページ(図4-1で再現)を見ることができます:

    http://localhost/myapp_dev.php/mymodule/index

この章ではデフォルトの`index`アクションを使わないので、`actions.class.php`ファイルから`executeIndex()`メソッドを削除し、`templates/`ディレクトリから`indexSuccess.php`ファイルを削除できます。

>**NOTE**
>symfonyはコマンドライン以外にもモジュールを初期化するほかの方法を提供します。これらの1つはあなた自身でディレクトリを作ることです。多くの場合、アクションとモジュールのテンプレートは任意のテーブルのデータを操作するために作られます。テーブルからレコードを作成する、読みとる、更新する、削除するために必要なコードはしばしば同じなので、symfonyはこのコードを生成するためにscaffoldingと呼ばれるメカニズムを提供します。このテクニックに関するより詳細な情報は14章を参照してください。

図4-1 - 生成されたデフォルトのindexページ

![生成されたデフォルトのindexページ](http://github.com/masakielastic/symfonybook-ja/raw/master/images/F0401.jpg "生成されたデフォルトのindexページ")

ページを追加する
----------------

symfonyにおいて、ページの背後にあるロジックはアクションに保存され、プレゼンテーションはテンプレートに保存されます。ロジックをともなわないページでも空のアクションを必要とします。

### アクションを追加する

"Hello, world!"のページは`myAction`アクションを通してアクセスできます。リスト4-2で示されているように、このアクションを作成するには、`executeMyAction`メソッドを`mymoduleActions`クラスに追加します。

リスト4-2 - アクションの追加は実行メソッドをアクションクラスに追加することと似ている(`actions/actions.class.php`)

    [php]
    <?php

    class mymoduleActions extends sfActions
    {
      public function executeMyAction()
      {
      }
    }

アクションのメソッド名はつねに`execute``Xxx``()`で、名前の2番目の部分は最初が大文字であるアクションの名前です。

では、つぎのURLをリクエストしてください:

    http://localhost/myapp_dev.php/mymodule/myAction

`myActionSuccess.php`テンプレートが見つからないことをsymfonyが訴えます。これは正常なことです; symfonyにおいて、ページはつねにアクションとテンプレートで構成されます。

>**CAUTION**
>URL(ドメイン名ではありません)と同じように、symfonyは大文字と小文字を区別します(PHPはメソッド名の大文字と小文字を区別しません)。このことは`executemyaction()`メソッドもしくは`executeMyaction()`メソッドを追加してブラウザーで`myAction`を呼び出すと、symfonyが404エラーを返すことを意味します。

-

>**SIDEBAR**
>URLはレスポンスの一部
>
>symfonyは実際のアクションの名前とアクションを呼び出すために必要なURLの形式を完全に分離できるようにするルーティングシステムを持ちます。このメカニズムによってURLをあたかもレスポンスの一部のようにカスタム形式に整えることができます。もはやファイル構造やリクエストパラメーターによって制限されることはありません; アクションのためのURLを望むフレーズで表示できます。たとえば、通常、articleという名前のモジュールのindexアクションへの呼び出しはつぎのようになります:
>
>     http://localhost/myapp_dev.php/article/index?id=123
>
>このURLはデータベースから任意の記事をとり出します。この例では、ヨーロッパのセクションのとりわけフランスのファイナンスについて議論している記事(`id=123`)がとり出されます。しかしながら、`routing.yml`設定ファイルを少し変更することでURLを完全に異なる形式で書けます:
>
>     http://localhost/articles/europe/france/finance.html
>
>検索エンジンにわかりやすいURLになるだけでなく、ユーザーにとっても重要です。ユーザーは、カスタムクエリを行うために、つぎのようにアドレスバーを擬似コマンドラインとして使えます:
>
>     http://localhost/articles/tagged/finance+france+euro
>
>symfonyはユーザーのためにスマートURLを解析し生成する方法を知っています。ルーティングシステムは自動的にスマートURLからリクエストパラメーターをはがし、アクションがこれらを利用できるようにします。レスポンスに含まれるハイパーリンクの形式を整えるので、これらは"スマート"に見えます。この機能について9章で詳しく学ぶことになります。
>
>全体的に、このことはあなたのアプリケーションのアクションを命名する方法は、アクションを呼び出すために使われるURLの見た目ではなく、アプリケーションのアクションの機能によって影響されることを意味します。アクションの名前はアクションが実際に行うことを説明し、不定詞の動詞(`show`、`list`、`edit`など)であることがよくあります。アクションの名前は全体的にエンドユーザーに対して見えないので、明確な名前(`listByName`もしくは`showWithComments`など)を使うことを躊躇しないでください。アクションの機能を説明するコードのコメントを書かずにすむことに加えて、コードがはるかに読みやすくなります。

### テンプレートを追加する

アクションはそれ自身をレンダリングすることをテンプレートに求めます。テンプレートはモジュールの`templates/`ディレクトリに設置されたファイルで、アクションとアクションのサフィックスで命名されます。アクションのデフォルトの接尾辞は"success"なので、`myAction`アクションのために作られるテンプレートファイルは`myActionSuccess.php`という名前になります。

テンプレートはプレゼンテーション用のコードだけを含むことを前提としているので、これらをできるかぎり小さなPHPコードとして維持してください。当然のことながら、"Hello, world!"を表示するページはリスト4-3のようなシンプルなテンプレートを持つことができます。

リスト4-3 - テンプレート(`mymodule/templates/myActionSuccess.php`)

    [php]
    <p>Hello, world!</p>

テンプレートのなかでPHPコードを実行する必要がある場合、リスト4-4で示すように、通常のPHP構文は避けるべきです。代わりに、PHPプログラマではない人でも理解できるように、リスト4-5で示されるPHPの代替構文を使うテンプレートを書きます。最終的なコードを正しくインデントするだけでなく、アクションの複雑なPHPコードを維持するための助けになります。なぜなら制御文(`if`、`foreach`、`while`など)だけが代替構文を持つからです。

リスト4-4 - アクションの用途にはよいが、テンプレートの用途にはよくない通常のPHP構文(`templates/showSuccess.php`)

    [php]
    <p>Hello, world!</p>
    <?php

    if ($test)
    {
      echo "<p>".time()."</p>";
    }

    ?>

リスト4-5 - テンプレート用のPHPの代替構文のよい例(`templates/showSuccess.php`)

    [php]
    <p>Hello, world!</p>
    <?php if ($test): ?>
    <p><?php echo time(); ?></p>
    <?php endif; ?>

>**TIP**
>テンプレートの構文が十分に読みやすいものか確認するためのよい経験則はPHPもしくは波かっこでechoされるHTMLコードを含まないことです。また多くの場合、`<?php`で展開するとき、同じ行の`?>`で閉じることです。

### テンプレートにアクションからの情報を渡す

アクションの仕事はすべての複雑な計算、データのとり出し、とテストの実施、echoもしくはテストされるテンプレートに対して変数を設定することです。symfonyは(アクションの`$this->variableName`経由でアクセスされる)アクションクラスの属性が(`$variableName`経由で)グローバルな名前空間内のテンプレートに直接アクセスできるようにします。リスト4-6と4-7はテンプレートにアクションからの情報を渡す方法を示しています。

リスト4-6 - テンプレートがアクションの属性を利用できるようにアクションを設定する(`mymodule/actions/actions.class.php`)

    [php]
    <?php

    class mymoduleActions extends sfActions
    {
      public function executeMyAction()
      {
        $today = getdate();
        $this->hour = $today['hours'];
      }
    }

リスト4-7 - テンプレートはアクションの属性に直接アクセスできる(`templates/showSuccess.php`)

    [php]
    <p>Hello, world!</p>
    <?php if ($hour >= 18): ?>
    <p>Or should I say good evening? It is already <?php echo $hour ?>.</p>
    <?php endif; ?>

>**NOTE**
>アクションの変数をセットアップしなくてもテンプレートはすでにごく一部のデータにアクセスできます。すべてのテンプレートは`$sf_context`、`$sf_request`、`$sf_params`、`$sf_user`オブジェクトのメソッドを呼び出すことができます。これらは現在のコンテキスト、リクエスト、リクエストパラメーター、セッションに関連するデータを含みます。まもなくこれらを効率的に使う方法を学ぶことになります。

ユーザーからの情報をフォームで集める
------------------------------------

フォームはユーザーから情報を得るためのよい方法です。HTMLのフォームとフォームの要素を書く作業は、とりわけXHTML準拠にしたい場合、やりづらいことがあります。リスト4-8で示されるように、通常の方法でフォームの要素をsymfonyのテンプレートに含めることができますが、symfonyはこのタスクをより簡単にするヘルパーを提供します。

リスト4-8 - 通常のHTMLコードをテンプレートに含めることができる

    [php]
    <p>こんにちは！</p>
    <?php if ($hour >= 18): ?>
    <p>それともこんばんはと言うほうがよろしいでしょうか？もう<?php echo $hour ?>時です。</p>
    <?php endif; ?>
    <form method="post" action="/myapp_dev.php/mymodule/anotherAction">
      <label for="name">お名前は？</label>
      <input type="text" name="name" id="name" value="" />
      <input type="submit" value="Ok" />
    </form>

ヘルパー(helper)とはsymfonyによって定義されたPHP関数で、テンプレートの範囲内で使われることを目的としています。ヘルパーはHTMLコードの出力を行いあなた自身でHTMLコードよりも速く書けます。symfonyのヘルパーを利用することで、リスト4-9で示されたコードがリスト4-8と同じ結果を得ることができます。

リスト4-9 - HTMLタグよりもヘルパーを利用したほうが速くて簡単

    [php]
    <p>こんにちは？</p>
    <?php if ($hour >= 18): ?>
    <p>それともこんばんはと言うほうがよろしいでしょうか？もう<?php echo $hour ?>時です。</p>
    <?php endif; ?>
    <?php echo form_tag('mymodule/anotherAction') ?>
      <?php echo label_for('name', 'お名前は？') ?>
      <?php echo input_tag('name') ?>
      <?php echo submit_tag('Ok') ?>
    </form>

>**SIDEBAR**
>ヘルパーはあなたをお助けします
>
>リスト4-9の例で、ヘルパー版のコードを書くほうがHTMLコードを直接書くよりも本当に速くないと思うのであれば、つぎのコードを考えてみてください:
>
>
    [php]
    <?php
    $card_list = array(
      'VISA' => 'Visa',
      'MAST' => 'MasterCard',
      'AMEX' => 'American Express',
      'DISC' => 'Discover');
    echo select_tag('cc_type', options_for_select($card_list, 'AMEX'));
    ?>
>
>
>これはつぎのHTMLを出力します:
>
>     [php]
>     <select name="cc_type" id="cc_type">
>       <option value="VISA">Visa</option>
>       <option value="MAST">MasterCard</option>
>       <option value="AMEX" selected="selected">American Express</option>
>       <option value="DISC">Discover</option>
>     </select>
>
>テンプレートのなかでヘルパーを利用する利点はコードを書く時間の理論値、コードの明瞭性と簡潔性です。支払う唯一の代償は学習時間と`<?php echo ?>`を書く時間です。学習の方はこの本を読み終わるまでに終わり、`<?php echo ?>`を書く作業は好きなテキストエディタでショートカットを用意すればすみます。テンプレート内でsymfonyのヘルパーを使わずに従来の方法でHTMLを書くことはできますが、その方法では多くの時間が失われ面白くなくなります。

短縮型の開始タグ(`<?=`は`<?php echo`と同じ)の使用はプロフェッショナルなWebアプリケーションに対して非推奨であることに注意してください。運用のWebサーバーは複数のスクリプト言語を理解できるので結果として混乱する可能性があるからです。加えて短縮型の開始タグはPHPのデフォルトの設定では機能せず、有効にするためにサーバーを調整する必要があるからです(訳注：short_open_tagディレクティブをonに設定する)。結局のところ、XMLとバリデーションを処理しなければならないとき、XMLにおいて`<?`は特別な意味を持つので不十分な表記です。

フォームの操作を説明するにはまるごと1章必要です。symfonyはフォームの操作を簡単にするために、多くのツール、大部分はヘルパーですが、を提供します。これらのヘルパーは10章で学ぶことになります。

別のアクションにリンクする
--------------------------

アクションの名前とそれを呼び出すURLが完全に分離されていることはご存じのとおりです。ですのでリスト4-10のようにテンプレートのなかで`anotherAction`アクションへのリンクを作る場合、リンクはデフォルトのルーティングのみで動作します。URLの表示方法の変更をあとで決める場合、ハイパーリンクを変更するにはすべてのテンプレートを再吟味する必要があります。

リスト4-10 - 古典的な方法による、ハイパーリンク

    [php]
    <a href="/myapp_dev.php/mymodule/anotherAction?name=anonymous">
      私の名前は教えません
    </a>

このやっかいな問題を避けるには、アプリケーションのアクションへのハイパーリンクを作る`link_to`ヘルパーをつねに使うべきです。リスト4-11はハイパーリンクのヘルパーの使いかたを示しています。

リスト4-11 - `link_to()`ヘルパー(`templates/***Success.php`)

    [php]
    <p>こんにちは！</p>
    <?php if ($hour >= 18): ?>
    <p>もしくはこんばんはと言ったほうがよろしいでしょうか？もう<?php echo $hour ?>時です。</p>
    <?php endif; ?>
    <?php echo form_tag('mymodule/anotherAction') ?>
      <?php echo label_for('name', 'お名前は？') ?>
      <?php echo input_tag('name') ?>
      <?php echo submit_tag('Ok') ?>
      <?php echo link_to('名前を教えない','mymodule/anotherAction?name=anonymous') ?>
    </form>

ルーティングルールを変更するとき、すべてのテンプレートが正しくふるまいURLをルールにしたがって再び整形すること以外、結果のHTMLは以前のものと同じになります。

`link_to()`ヘルパーは、多くのヘルパーと同じように、特別なオプションと追加のタグ属性のために別の引数を受けとります。リスト4-12はオプション引数の例と結果のHTMLを示します。オプション引数は連想配列もしくは空白によって区切られた`key=value`の組を表示するシンプルな文字列です。

リスト4-12 - たいていのヘルパーはオプション引数を受けとる(`templates/***Success.php`)

    [php]
    // 連想配列としてのオプション引数
    <?php echo link_to('名前を教えない', 'mymodule/anotherAction?name=anonymous',
      array(
        'class'    => 'special_link',
        'confirm'  => 'よろしいですか？',
        'absolute' => true
    )) ?>

    // 文字列としてのオプション引数
    <?php echo link_to('名前を教えない', 'mymodule/anotherAction?name=anonymous',
      'class=special_link confirm=Are you sure? absolute=true') ?>

    // 両方の呼び出しは同じものを出力する
     => <a class="special_link" onclick="return confirm('Are you sure?');"
        href="http://localhost/myapp_dev.php/mymodule/anotherAction/name/anonymous">
        名前を教えない</a>

HTMLタグを出力するsymfonyのヘルパーを使うとき、オプション引数に(リスト4-12の例の`class`属性のような)追加のタグ属性を挿入できます。これらの属性を"速くて汚い"HTML 4.0の方法(ダブルクォートなし)で書くこともできますが、symfonyはすばらしく整形されたXHTMLでそれらを出力します。それがヘルパーがHTMLを書くより速い別の理由です。

>**NOTE**
>追加の解析と変換が必要なので、文字列の構文は配列よりも若干遅いです。

フォームヘルパーのように、リンクヘルパーとオプションは数多く存在します。9章でこれらを詳しく説明します。

リクエストから情報を入手する
----------------------------

ユーザーが送信した情報がフォーム経由(通常はPOSTリクエスト)もしくはURL経由(GETリクエスト)のどちらでも、`sfActions`オブジェクトの`getRequestParameter()`メソッドを用いてアクションから関連するデータをとり出すことができます。リスト4-13は、`anotherAction`メソッドで、`name`パラメーターの値をとり出す方法を示しています。

リスト4-13 - データをアクションのリクエストパラメーターから取得する(`mymodule/actions/actions.class.php`)

    [php]
    <?php

    class mymoduleActions extends sfActions
    {
      ...

      public function executeAnotherAction()
      {
        $this->name = $this->getRequestParameter('name');
      }
    }

データの操作方法がシンプルであるなら、リクエストパラメーターをとり出すためにアクションを使う必要もありません。テンプレートが`$sf_params`という名前のオブジェクトにアクセスできるからです。アクションの`getRequestParameter()`と同様に、`$sf_params`はリクエストパラメーターをとり出す`get()`メソッドを提供します。

`executeAnotherAction()`が空の場合、リスト4-14は`anotherActionSuccess.php`テンプレートが同じ`name`パラメーターをとり出す方法を示しています。

リスト4-14 - リクエストパラメーターをテンプレートから直接取得する(`templates/***Success.php`)

    [php]
    <p>Hello, <?php echo $sf_params->get('name') ?>!</p>

>**NOTE**
>なぜ、代わりに`$_POST`、`$_GET`、`$_REQUEST`変数を使わないのでしょうか？なぜならURLは異なるようにフォーマットされるため(たとえば`?`や`=`をともなわない`http://localhost/articles/europe/france/finance.html`)、通常のPHP変数はそれ以上機能しないので、ルーティングシステムのみがリクエストパラメーターをとり出すことができるからです。悪意のあるコードのインジェクションを防止するために、入力フィルタリングを追加したい場合、これはすべてのリクエストパラメーターを1つのクリーンなパラメーターホルダーに保持する場合のみ実現できます。

`$sf_params`オブジェクトは単に同等のゲッターを配列に渡すよりも強力です。たとえば、リクエストパラメーターの存在を確認することだけを試したい場合、リスト4-15のように、実際の値を`get()`で試す代わりに`$sf_params->has()`メソッドで簡単に利用できます。

リスト4-15 - テンプレートのなかでリクエストパラメーターを試す(`templates/***Success.php`)

    [php]
    <?php if ($sf_params->has('name')): ?>
      <p>こんにちは、<?php echo $sf_params->get('name') ?>さん！</p>
    <?php else: ?>
      <p>こんにちは、John Doeさん！</p>
    <?php endif; ?>

読者のなかにはこれを一行で書けることをすでに推測している人がいるでしょう。symfonyのたいていのゲッターメソッドに関しては、アクションの`getRequestParameter()`メソッドとテンプレートの`$sf_params->get()`メソッド(実際には、同じオブジェクトの同じメソッドの呼び出し)の両方とも2番目の引数としてリクエストパラメーターが存在しない場合に使われるデフォルト値を受けとります。

    [php]
    <p>こんにちは、<?php echo $sf_params->get('name', 'John Doe') ?>さん！</p>

まとめ
----

symfonyにおいて、ページはアクションとテンプレートで構成されます。アクションは`actions/actions.class.php`ファイルのメソッドでプレフィックスが`execute`で、テンプレートは`templates/`ディレクトリ内のファイルで、通常は`Success.php`で終わります。これらはアプリケーション内の機能にしたがって、モジュールに分類されます。ヘルパーによってテンプレートを円滑に書けます。ヘルパー(helper)とはsymfonyによって提供されるHTMLコードを返す関数です。そして、必要に応じて整形されるURLはレスポンスの一部として考えることが必要です。ですのでアクションの命名もしくはリクエストパラメーターの読みとりにおいてURLを直接参照することは避けるべきです。

これらの基本的な原則を理解したら、すでにアプリケーション全体を書けます。しかしながらこれだけでは作業が長すぎます。アプリケーションの開発過程の間に実現しなければならないほとんどすべてのタスクは別のsymfonyの機能によって1つもしくは別の方法で円滑になります・・・今のところこの本が終わらない理由です。
