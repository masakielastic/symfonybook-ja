第10章 - フォーム
=================

テンプレートを書くとき、開発者の多くの時間はフォーム(form)に費やされます。このことにもかかわらず、フォームは一般的に貧弱に設計されます。デフォルト値、整形、検証、再投入と一般的なフォームの扱いなど多くのことに注意を払うことが必要なので、開発者のなかにはプロセスにおけるいくつかの重要な詳細事項を大まかにしか見ない人がいます。というわけで、symfonyはこのトピックに対して特別な注意を払うことにします。この章ではフォーム開発速度を上げる一方で、これらの多くの要件を自動化するツールについて説明します。

  * フォームヘルパーは、とりわけ日付、ドロップダウンのリスト、とリッチテキストなどの複雑な要素に対して、テンプレートのなかでフォーム入力を速く書く方法を提供します。
  * フォームがオブジェクトのプロパティを編集することに徹する場合、オブジェクトフォームヘルパーを利用することでテンプレートの編集作業が速くなります。
  * YAMLバリデーションファイルはフォームの検証と再投入を円滑にします。
  * バリデーターは入力を検証するために必要なコードをまとめます。symfonyはもっとも共通なニーズのためのバリデーターを搭載します。またカスタムバリデーターを追加することはとても簡単です。

フォームヘルパー
----------------

テンプレートによって、フォーム要素のHTMLタグがPHPコードに混ざることはよくあることです。symfonyのフォームヘルパーはこのタスクをシンプルにして`<input>`タグのあいだで`<?php echo`のタグを繰り返し展開することを避けることを目的としています。

### メインのフォームタグ

以前の章で説明したように、フォームを作成するには`form_tag()`ヘルパーを使わなければなりません。このヘルパーはパラメーターとして渡されたアクションをルーティングされたURLに変換するからです。2番目の引数は追加オプションをサポートします。たとえば、デフォルトの`method`を変更するには、デフォルトの`enctype`を変更するか、もしくはほかの属性を指定します。リスト10-1は例を示しています。

リスト10-1 - `form_tag()`ヘルパー

    [php]
    <?php echo form_tag('test/save') ?>
     => <form method="post" action="/path/to/save">

    <?php echo form_tag('test/save', 'method=get multipart=true class=simpleForm') ?>
     => <form method="get" enctype="multipart/form-data" class="simpleForm"action="/path/to/save">

フォームヘルパーを閉じる必要がないので、ソースコードのなかできれいに見えなくても、HTMLの`</form>`を使うべきです。

### 標準のフォーム要素

フォームヘルパーによって、フォームのなかのそれぞれの要素はデフォルトでname属性から推測されるid属性に渡されます。これは便利なだけの規約ではありません。標準のフォームヘルパーとそれらのオプションの全リストに関してはリスト10-2をご覧ください。

リスト10-2 - 標準のフォームヘルパーの構文

    [php]
    // テキストフィールド(入力)
    <?php echo input_tag('name', 'default value') ?>
     => <input type="text" name="name" id="name" value="default value" />

    // すべてのフォームヘルパーは追加オプションのパラメーターを受けとる
    // これによってカスタム属性を生成タグに追加できる
    <?php echo input_tag('name', 'default value', 'maxlength=20') ?>
     => <input type="text" name="name" id="name" value="default value" maxlength="20" />

    // 長いテキストフィールド(テキストエリア)
    <?php echo textarea_tag('name', 'default content', 'size=10x20') ?>
     => <textarea name="name" id="name" cols="10" rows="20">
          default content
        </textarea>

    // チェックボックス
    <?php echo checkbox_tag('single', 1, true) ?>
    <?php echo checkbox_tag('driverslicense', 'B', false) ?>
     => <input type="checkbox" name="single" id="single" value="1" checked="checked" />
        <input type="checkbox" name="driverslicense" id="driverslicense" value="B" />

    // ラジオボタン
    <?php echo radiobutton_tag('status[]', 'value1', true) ?>
    <?php echo radiobutton_tag('status[]', 'value2', false) ?>
     => <input type="radio" name="status[]" id="status_value1" value="value1" checked="checked" />
        <input type="radio" name="status[]" id="status_value2" value="value2" />

    // ドロップダウンリスト(選択)
    <?php echo select_tag('payment',
      '<option selected="selected">Visa</option>
       <option>Eurocard</option>
       <option>Mastercard</option>')
    ?>
     => <select name="payment" id="payment">
          <option selected="selected">Visa</option>
          <option>Eurocard</option>
          <option>Mastercard</option>
        </select>

    // 選択タグのためのオプションのリスト
    <?php echo options_for_select(array('Visa', 'Eurocard', 'Mastercard'), 0) ?>
     => <option value="0" selected="selected">Visa</option>
        <option value="1">Eurocard</option>
        <option value="2">Mastercard</option>

    // オプションのリストと結びつけられたドロップダウンヘルパーオプションのリスト
    <?php echo select_tag('payment', options_for_select(array(
      'Visa',
      'Eurocard',
      'Mastercard'
    ), 0)) ?>
     => <select name="payment" id="payment">
          <option value="0" selected="selected">Visa</option>
          <option value="1">Eurocard</option>
          <option value="2">Mastercard</option>
        </select>

    // オプション名を指定するため、連想配列を使う
    <?php echo select_tag('name', options_for_select(array(
      'Steve'  => 'Steve',
      'Bob'    => 'Bob',
      'Albert' => 'Albert',
      'Ian'    => 'Ian',
      'Buck'   => 'Buck'
    ), 'Ian')) ?>
     => <select name="name" id="name">
          <option value="Steve">Steve</option>
          <option value="Bob">Bob</option>
          <option value="Albert">Albert</option>
          <option value="Ian" selected="selected">Ian</option>
          <option value="Buck">Buck</option>
        </select>

    // 複数の選択を持つドロップダウンリスト(選択された値は配列が可能)
    <?php echo select_tag('payment', options_for_select(
      array('Visa' => 'Visa', 'Eurocard' => 'Eurocard', 'Mastercard' => 'Mastercard'),
      array('Visa', 'Mastercard'),
    ), array('multiple' => true))) ?>

     => <select name="payment[]" id="payment" multiple="multiple">
          <option value="Visa" selected="selected">Visa</option>
          <option value="Eurocard">Eurocard</option>
          <option value="Mastercard">Mastercard</option>
        </select>
    // 複数の選択を持つドロップダウンリスト(選択された値は配列が可能)
    <?php echo select_tag('payment', options_for_select(
      array('Visa' => 'Visa', 'Eurocard' => 'Eurocard', 'Mastercard' => 'Mastercard'),
      array('Visa', 'Mastercard')
    ), 'multiple=multiple') ?>
     => <select name="payment[]" id="payment" multiple="multiple">
          <option value="Visa" selected="selected">Visa</option>
          <option value="Eurocard">Eurocard</option>
          <option value="Mastercard" selected="selected">Mastercard</option>
        </select>

    // アップロードファイルフィールド
    <?php echo input_file_tag('name') ?>
     => <input type="file" name="name" id="name" value="" />

    // パスワードフィールド
    <?php echo input_password_tag('name', 'value') ?>
     => <input type="password" name="name" id="name" value="value" />

    // 隠しフィールド
    <?php echo input_hidden_tag('name', 'value') ?>
     => <input type="hidden" name="name" id="name" value="value" />

    // 投稿ボタン (テキストとして)
    <?php echo submit_tag('Save') ?>
     => <input type="submit" name="submit" value="Save" />

    // 投稿ボタン (画像として)
    <?php echo submit_image_tag('submit_img') ?>
     => <input type="image" name="submit" src="/images/submit_img.png" />

`submit_image_tag()`ヘルパーは同じ構文を使い`image_tag()`と同じ利点を持ちます。

>**NOTE**
>ラジオボタンに関して、デフォルトでは`id`属性は`name`属性の値ではなく、名前と値の組み合わせに設定されます。自動化された"別のものを選択したときに以前の選択を解除する"機能を手に入れるために同じ名前を持ついくつかのラジオボタンのタグを用意する必要があるからで、`id=name`の慣習によれば、厳密には禁止されている、ページ内で`id`属性を持つHTMLタグが存在することを意味します。

-

>**SIDEBAR**
>**フォーム投稿の扱いかた**
>
>フォームを通してユーザーによって投稿されたデータをどのように読みとりますか？リクエストパラメーターで利用できるので、アクションは値を取得するために`$this->getRequestParameter($elementName)`を呼び出すことが必要なだけです。
>
>フォームを表示して処理するためのよい習慣は同じアクションを使うことです。リクエストメソッド(GETもしくはPOST)に従い、フォームのテンプレートが呼び出されるもしくはフォームが処理されるとリクエストは別のアクションにリダイレクトされます。
>
>     [php]
>     // mymodule/actions/actions.class.phpにおいて
>     public function executeEditAuthor()
>     {
>       if ($this->getRequest()->getMethod() != sfRequest::POST)
>       {
>         // フォームを表示する
>         return sfView::SUCCESS;
>       }
>       else
>       {
>         // フォーム投稿を扱う
>         $name = $this->getRequestParameter('name');
>         ...
>         $this->redirect('mymodule/anotheraction');
>       }
>     }
>
>これを機能させるために、フォームのターゲットは表示するアクションと同じアクションでなければなりません。
>
>     [php]
>     // mymodule/templates/editAuthorSuccess.phpにおいて
>     <?php echo form_tag('mymodule/editAuthor') ?>
>
>     ...

symfonyは裏で非同期通信リクエストを行うために専用のフォームヘルパーを提供します。つぎの章では、Ajaxについて焦点を当てて、詳細な説明を行います。

### 日付入力ウィジェット

フォームは日付を読みとるためによく使われます。誤った書式の日付はフォーム投稿が失敗する主要な原因です。図10-1で示されるように、`rich`オプションを`true`に設定する場合、`input_date_tag()`ヘルパーはインタラクティブなJavaScriptのカレンダーのなかで入力するユーザーを助けします。

図10-1 - リッチな日付入力タグ

![リッチな日付入力タグ](http://github.com/masakielastic/symfonybook-ja/raw/master/images/F1001.png "リッチな日付入力タグ")

`rich`オプションが省略された場合、月、日にち、年の範囲で投入されたヘルパーは3つの`<select>`タグを出力します。これらのヘルパー(`select_day_tag()`、`select_month_tag()`と`select_year_tag()`)を呼び出すことでこれらのドロップダウンを個別に表示できます。これらの要素のデフォルト値は現在の日にち、月、年です。リスト10-3は日付入力ヘルパーを示してます。

リスト10-3 - 日付入力ヘルパー

    [php]
    <?php echo input_date_tag('dateofbirth', '2005-05-03', 'rich=true') ?>
     => a text input tag together with a calendar widget

    // つぎのヘルパーはDateFormヘルパーグループを必要とする
    <?php use_helper('DateForm') ?>

    <?php echo select_day_tag('day', 1, 'include_custom=日にちを選択する') ?>
    => <select name="day" id="day">
          <option value="">日にちを選択する</option>
          <option value="1" selected="selected">01</option>
          <option value="2">02</option>
          ...
          <option value="31">31</option>
        </select>

    <?php echo select_month_tag('month', 1, 'include_custom=月を選択する use_short_month=true') ?>
    => <select name="month" id="month">
          <option value="">月を選択する</option>
          <option value="1" selected="selected">Jan</option>
          <option value="2">Feb</option>
          ...
          <option value="12">Dec</option>
        </select>

    <?php echo select_year_tag('year', 2007, 'include_custom=年を選択する year_end=2010') ?>
     => <select name="year" id="year">
          <option value="">年を選択する</option>
          <option value="2006">2006</option>
          <option value="2007" selected="selected">2007</option>
          ...
        </select>

`input_date_tag()`ヘルパーが受け入れる日付の値はPHPの`strtotime()`関数によって認識されるものです。リスト10-4は利用できる書式を示し、リスト10-5は避けなければならない書式を示しています。

リスト10-4 - 日付ヘルパーで受け入れられる日付書式

    [php]
    // 立派に動作する
    <?php echo input_date_tag('test', '2006-04-01', 'rich=true') ?>
    <?php echo input_date_tag('test', 1143884373, 'rich=true') ?>
    <?php echo input_date_tag('test', 'now', 'rich=true') ?>
    <?php echo input_date_tag('test', '23 October 2005', 'rich=true') ?>
    <?php echo input_date_tag('test', 'next tuesday', 'rich=true') ?>
    <?php echo input_date_tag('test', '1 week 2 days 4 hours 2 seconds', 'rich=true') ?>

    // nullを返す
    <?php echo input_date_tag('test', null, 'rich=true') ?>
    <?php echo input_date_tag('test', '', 'rich=true') ?>

リスト10-5 - 日付ヘルパーで無効な日付書式

    [php]
    // 日付 zero = 01/01/1970
    <?php echo input_date_tag('test', 0, 'rich=true') ?>

    // 英語ではない日付書式は機能しない
    <?php echo input_date_tag('test', '01/04/2006', 'rich=true') ?>

### リッチなテキスト編集機能

TinyMCEとFCKEditorウィジェットの統合のおかげで、リッチなテキスト編集機能を`<textarea>`タグでも利用できます。図10-2で示されるように、これらはテキストを太文字、イタリック体、そのほかのスタイルとして整形するためのボタンつきのワードプロセッサのようなインターフェイスを提供します。

図10-2 - リッチなテキスト編集機能

![リッチなテキスト編集機能](http://github.com/masakielastic/symfonybook-ja/raw/master/images/F1002.png "リッチなテキスト編集機能")

両方のウィジェットとも手動でインストールする作業が必要です。2つのウィジェットのインストールの手続きは同じなので、ここではTinyMCEのリッチなテキスト編集機能だけを説明します。プロジェクトのWebサイト([http://tinymce.moxiecode.com/](http://tinymce.moxiecode.com/))からエディタをダウンロードし、一時フォルダーに解凍する必要があります。リスト10-6で示されるように、`tinymce/jscripts/tiny_mce/`ディレクトリをプロジェクトの`web/js/`ディレクトリにコピーし、`settings.yml`のなかでライブラリへのパスを定義します。

リスト10-6 - TinyMCEライブラリのパスをセットアップする

    all:
      .settings:
        rich_text_js_dir:  js/tiny_mce

セットアップが終了したら、`rich=true`オプションを追加することでテキストエリアのリッチなテキスト編集機能を利用できるように切り替えます。`tinymce_options`オプションを使うことでJavaScriptエディタに対してカスタムオプションを指定することもできます。リスト10-7を示してください。

リスト10-7 - リッチなテキストエリア

    [php]
    <?php echo textarea_tag('name', 'default content', 'rich=true size=10x20')) ?>
     => a rich text edit zone powered by TinyMCE
    <?php echo textarea_tag('name', 'default content', 'rich=true size=10x20 tinymce_options=language:"fr",theme_advanced_buttons2:"separator"')) ?>
    => a rich text edit zone powered by TinyMCE with custom parameters

### 国と言語の選択

国の選択フィールドを表示する必要がある場合を考えます。しかし国の名前はすべての言語とは同じではないので国のドロップダウンリストのオプションはユーザーのcultureにしたがって変わります(cultureに関する詳細な情報は13章を参照)。リスト10-8で示されるように、`select_country_tag()`ヘルパーがあなたの代わりにすべてを行います: これは国の名前を国際化し、標準的なISOの国コードを値として使います。

リスト10-8 - 国のタグヘルパーを選ぶ

    [php]
    <?php echo select_country_tag('country', 'AL') ?>
     => <select name="country" id="country">
          <option value="AF">アフガニスタン</option>
          <option value="AL" selected="selected">アルバニア</option>
          <option value="DZ">アルジェリア</option>
          <option value="AS">アメリカ領サモア</option>
      ...

`select_country_tag()`ヘルパーと同じように、`selext_language_tag()`ヘルパーはリスト10-9で示されるような言語のリストを表示します。 

リスト10-9 - 言語のタグヘルパーを選ぶ

    [php]
    <?php echo select_language_tag('language', 'en') ?>
     => <select name="language" id="language">
          ...
          <option value="elx">エラム語</option>
          <option value="en" selected="selected">英語</option>
          <option value="enm">中英語(1100-1500)</option>
          <option value="ang">古英語(ca.450-1100)</option>
          <option value="myv">エルジャ語</option>
          <option value="eo">エスペラント語</option>
          ...

オブジェクトのためのフォームヘルパー
------------------------------------

フォームの要素がオブジェクトのプロパティを編集するために使われるとき、標準のリンクヘルパーを書く作業が退屈になることがあります。たとえば、`Customer`オブジェクトの`telephone`属性を編集するには、つぎのように書くことになります:

    [php]
    <?php echo input_tag('telephone', $customer->getTelephone()) ?>
    => <input type="text" name="telephone" id="telephone" value="0123456789" />

属性の名前を繰り返すことを避けるために、symfonyはそれぞれのフォームヘルパーに対して代わりのオブジェクトフォームヘルパーを提供します。オブジェクトフォームヘルパーはオブジェクトとメソッド名からフォーム要素の名前とデフォルト値を推測します。以前の`input_tag()`はつぎのコードと同等です:

    [php]
    <?php echo object_input_tag($customer, 'getTelephone') ?>
    => <input type="text" name="telephone" id="telephone" value="0123456789" />

`object_input_tag()`に関して簡潔さは重要ではないかもしれません。しかしながら、すべての標準のフォームヘルパーは対応するオブジェクトフォームヘルパーを持ち、これらはすべて同じ構文を共有します。これはフォームの生成作業をとても簡単なものにします。scaffoldingと生成されたadministration(14章を参照)でオブジェクトフォームヘルパーが広範囲に使われる理由はそういうわけです。リスト10-10はヘルパーからのオブジェクトのリストです。

リスト10-10 - オブジェクトフォームヘルパーの構文

    [php]
    <?php echo object_input_tag($object, $method, $options) ?>
    <?php echo object_input_date_tag($object, $method, $options) ?>
    <?php echo object_input_hidden_tag($object, $method, $options) ?>
    <?php echo object_textarea_tag($object, $method, $options) ?>
    <?php echo object_checkbox_tag($object, $method, $options) ?>
    <?php echo object_select_tag($object, $method, $options) ?>
    <?php echo object_select_country_tag($object, $method, $options) ?>
    <?php echo object_select_language_tag($object, $method, $options) ?>

`object_password_tag()`ヘルパーは存在しません。ユーザーが以前入力したものに基づいて、デフォルト値をパスワードのタグに渡すことはよくない習慣だからです。

>**CAUTION**
>通常のフォームヘルパーとは異なり、`use_helper('Object')`を持つテンプレート内で`Object`ヘルパーグループの使用を明示的に宣言した場合のみオブジェクトフォームヘルパーは利用できます。

すべてのオブジェクトフォームヘルパーのなかでもっとも面白いのはドロップダウンリストに関連する`objects_for_select()`と`object_select_tag()`です。

### ドロップダウンをオブジェクトで投入する

`options_for_select()`ヘルパーは、ほかの標準ヘルパーで以前説明されたものですが、リスト10-11で示されるように、PHPの連想配列をオプションのリストに変換します。

リスト10-11 - `options_for_select()`をともなう配列に基づいたオプションのリストを作成する

    [php]
    <?php echo options_for_select(array(
      '1' => 'Steve',
      '2' => 'Bob',
      '3' => 'Albert',
      '4' => 'Ian',
      '5' => 'Buck'
    ), 4) ?>
     => <option value="1">Steve</option>
        <option value="2">Bob</option>
        <option value="3">Albert</option>
        <option value="4" selected="selected">Ian</option>
        <option value="5">Buck</option>

`Propel`のクエリの結果から得られた、`Author`クラスのオブジェクトの配列がすでに存在することを前提とします。この配列に基づいたオプションのリストを作りたい場合、リスト10-12で示されるように、それぞれのオブジェクトの`id`と`name`を読みとるためにループする必要があります。

リスト10-12 - `options_for_select()`を利用してオブジェクトの配列に基づいたオプションのリストを作成する

    [php]
    // アクションにおいて
    $options = array();
    foreach ($authors as $author)
    {
      $options[$author->getId()] = $author->getName();
    }
    $this->options = $options;

    // テンプレートにおいて
    <?php echo options_for_select($options, 4) ?>

この種の処理作業はよく行われるので、symfonyはこの作業を自動化するヘルパーを持ちます: `objects_for_select()`はオブジェクトの配列に直接基づいてオプションリストを作ります。ヘルパーは2つの追加パラメーターが必要です: `value`を読みとるために使われるメソッド名と生成される`<option>`タグのテキスト内容です。リスト10-12はつぎのシンプルなフォームと同等です:

    [php]
    <?php echo objects_for_select($authors, 'getId', 'getName', 4) ?>

これはスマートで速いですが、外部キーのカラムを処理するとき、symfonyはさらに掘り下げます。

### 外部キーのカラムに基づいてドロップダウンリストを作成する

外部キーのカラムが取得できる値は外部テーブルのレコードの主キーの値です。たとえば、`article`テーブルが`author`テーブルへの外部キーである`author_id`カラムを持つ場合、このカラムのための可能な値は`author`テーブルのすべてのレコードの`id`です。基本的には、記事の作者を編集するドロップダウンリストはリスト10-13のようになります。

リスト10-13 - `objects_for_select()`で外部キーに基づいたオプションのリストを作成する

    [php]
    <?php echo select_tag('author_id', objects_for_select(
      AuthorPeer::doSelect(new Criteria()),
      'getId',
      '__toString',
      $article->getAuthorId()
    )) ?>
    => <select name="author_id" id="author_id">
          <option value="1">Steve</option>
          <option value="2">Bob</option>
          <option value="3">Albert</option>
          <option value="4" selected="selected">Ian</option>
          <option value="5">Buck</option>
        </select>

`object_select_tag()`は単独ですべてを行います。これは外部テーブルの可能なレコードの名前で投入されたドロップダウンリストを表示します。ヘルパーはスキーマから外部テーブルと外部カラムを推測できるので、構文はとても簡潔です。リスト10-13はつぎのコードと同等です:

    [php]
    <?php echo object_select_tag($article, 'getAuthorId') ?>

`object_select_tag()`ヘルパーはパラメーターとして渡されるメソッド名に基づいて関連するピアクラスの名前(この例では`AuthorPeer`)を推測します。しかしながら、`related_class`オプションを3番目の引数に設定することで独自のクラスを指定できます。`<option>`タグのテキスト内容はレコード名で、オブジェクトクラスの`__toString()`メソッドの結果です(`$author->__toString()`メソッドが未定義の場合、主キーが代わりに使われます)。加えて、オプションのリストは空の基準値を持つ`doSelect()`メソッドで構成されます; これは作成日順に並べられたすべてのレコードを返します。特定の順番でレコードの部分集合だけを表示したい場合、この選択をオブジェクトの配列として返すピアクラス内でメソッドを作り、これを`peer_method`オプションに設定します。最後に、`include_blank`と`include_custom`オプションを設定することでドロップダウンリストのトップに、空白のオプションもしくはカスタムオプションを追加できます。リスト10-14は`object_select_tag()`ヘルパーに対するこれらの異なるオプションを示しています。

リスト10-14 - `object_select_tag()`ヘルパーのオプション

    [php]
    // 基本構文
    <?php echo object_select_tag($article, 'getAuthorId') ?>
    // AuthorPeer::doSelect(new Criteria())からリストを作成する

    // 可能な値を読みとるために使われるピアクラスを変更する
    <?php echo object_select_tag($article, 'getAuthorId', 'related_class=Foobar') ?>
    // FoobarPeer::doSelect(new Criteria())からリストを作成する

    // 可能な値を読みとるためにピアクラスを変更する
    <?php echo object_select_tag($article, 'getAuthorId','peer_method=getMostFamousAuthors') ?>
    // AuthorPeer::getMostFamousAuthors(new Criteria())からリストを作成する

    // リストのトップで<option value="">&nbsp;</option>を追加する
    <?php echo object_select_tag($article, 'getAuthorId', 'include_blank=true') ?>

    // リストのトップで<option value="">Choose an author</option>を追加する
    <?php echo object_select_tag($article, 'getAuthorId',
      'include_custom=Choose an author') ?>

### オブジェクトを更新する

オブジェクトヘルパーを利用してオブジェクトのプロパティを編集するための専用フォームはアクションのなかで扱うよりも簡単です。たとえば、`name`、`age`、`address`属性を持つ`Author`クラスのオブジェクトがある場合、フォームはリスト10-15のように書かれます。

リスト10-15 - オブジェクトヘルパーだけを持つフォーム

    [php]
    <?php echo form_tag('author/update') ?>
      <?php echo object_input_hidden_tag($author, 'getId') ?>
      名前: <?php echo object_input_tag($author, 'getName') ?><br />
      年齢:  <?php echo object_input_tag($author, 'getAge') ?><br />
      アドレス: <br />
             <?php echo object_textarea_tag($author, 'getAddress') ?>
    </form>

リスト10-16で示されるように、フォームが投稿されたときに、呼び出される`author`モジュールの`update`アクションはPropelによって生成された`fromArray()`修飾子によって簡単にオブジェクトを更新できます。

リスト10-16 - オブジェクトフォームヘルパーに基づいたフォーム投稿を扱う

    [php]
    public function executeUpdate ()
    {
      $author = AuthorPeer::retrieveByPk($this->getRequestParameter('id'));
      $this->forward404Unless($author);

      $author->fromArray($this->getRequest()->getParameterHolder()->getAll(),BasePeer::TYPE_FIELDNAME);
      $author->save();

      return $this->redirect('/author/show?id='.$author->getId());
    }

フォームのバリデーション
------------------------

6章ではリクエストパラメーターを検証するためにアクションクラスで`validateXXX()`メソッドを使う方法を説明しました。しかしながら、フォーム投稿を検証するこのテクニックを利用する場合、同じコードの一部を何度も書き直すことになります。symfonyはアクションクラスのなかのPHPコードの代わりに、YAMLファイルだけに依存する、フォームのバリデーションの代替テクニックを提供します。

フォームのバリデーション機能を実証するために、最初にリスト10-17で示されているサンプルのフォームを考えてみましょう。`name`、`email`、`age`、と`message`フィールドを持つ、古典的な問い合わせフォームです。

リスト10-17 - 問い合わせフォームのサンプル(`modules/contact/templates/indexSuccess.php`)

    [php]
    <?php echo form_tag('contact/send') ?>
      名前:    <?php echo input_tag('name') ?><br />
      Eメール:   <?php echo input_tag('email') ?><br />
      年齢Age:     <?php echo input_tag('age') ?><br />
      メッセージ: <?php echo textarea_tag('message') ?><br />
      <?php echo submit_tag() ?>
    </form>

フォームのバリデーションの原則はユーザーが不正なデータを入力しフォームに投稿した場合、つぎのページでエラーメッセージを表示することです。簡単な英語で、サンプルフォームに対して有効なデータを定義してみましょう:

  * `name`フィールドが必要です。2文字から100文字のテキストエントリーでなければなりません。
  * `email`フィールドが必要です。2文字から100文字までのテキストエントリーかつ有効なEメールアドレスでなければなりません。
  * `age`フィールドが必要です。0から120の整数でなければなりません。
  * `message`フィールドが必要です。

問い合わせフォームのためにもっと複雑な検証ルールを定義することができますが、バリデーションの可能性を実証するにはこれで十分です。

>**NOTE**
>フォームのバリデーションはサーバーサイドかつ/もしくはクライアントサイドで行われます。サーバーサイドのバリデーションは間違ったデータでデータベースが汚染されることを避けるために必須です。クライアントサイドのバリデーションはユーザーエクスペリエンスを大いに高めますが、オプションです。クライアントサイドのバリデーションはカスタムJavaScriptで行われます。

### バリデーター

上記の例において`name`フィールドと`email`フィールドが共通のバリデーションルールを共有していることがわかります。バリデーションルールのいくつかはWebフォームで頻繁に使われるのでsymfonyはこれらをバリデーターに実装するPHPコードにまとめます。バリデーター(validator)は`execute()`メソッドを提供するシンプルなクラスです。このメソッドはパラメーターとしてフィールドの値を必要とし、値が有効な場合は`true`を、そうでなければ`false`を返します。

symfonyはいくつかのバリデーターを搭載しています(この章の後のほうの"symfonyの標準的なバリデーター"で説明されます)が、今回は`sfStringValidator`を重点的にとり組んでみましょう。このバリデーターは入力が文字列であり、サイズが2つの指定された文字数の間に収まっていることをチェックします(`initialize()`メソッドを呼び出すときに定義されます)。`name`フィールドを検証するために求められるものがまさにこれです。リスト10-18は検証メソッドでこのバリデーターを使う方法を示しています。

リスト10-18 - 再利用可能なバリデーターでリクエストパラメーターを検証する(`modules/contact/action/actions.class.php`)

    [php]
    public function validateSend()
    {
      $name = $this->getRequestParameter('name');

      // nameフィールドが求められる
      if (!$name)
      {
        $this->getRequest()->setError('name', '名前のフィールドは空白であってはなりません');

        return false;
      }

      // nameフィールドは2文字から100文字の間のテキストエントリーでなければならない
      $myValidator = new sfStringValidator();
      $myValidator->initialize($this->getContext(), array(
        'min'       => 2,
        'min_error' => 'この名前は短すぎます(最小は2文字)',
        'max'       => 100,
        'max_error' => 'この名前は長すぎます(最大で100文字)',
      ));
      if (!$myValidator->execute($name, $error))
      {
        return false;
      }

      return true;
    }

ユーザーがリスト10-17のフォームで`name`フィールドの`a`の値を投稿する場合、`sfStringValidator`の`execute()`メソッドは`false`を返します(文字列の最少の長さが2文字だからです)。`validateSend()`メソッドが失敗し、`executeSend()`メソッドの代わりに`handleErrorSend()`メソッドが呼び出されます。

>**TIP**
>`sfRequest`オブジェクトの`setError()`メソッドはテンプレートに情報を渡すので、テンプレートはエラーメッセージを表示できます(この章の後の"エラーメッセージをフォームに表示する"のセクションで説明されます)。バリデーターはエラーを内部で設定するので、異なるバリデーションを行わない場合のために異なるエラーを定義できます。これが`sfStringValidator`の`min_error`と`max_error`初期化パラメーターの目的です。

上記の例で定義されたすべてのルールはバリデーターに翻訳できます:

  * `name`: `sfStringValidator` (`min=2`, `max=100`)
  * `email`: `sfStringValidator` (`min=2`, `max=100`)と`sfEmailValidator`
  * `age`: `sfNumberValidator` (`min=0`, `max=120`)

フィールドが求められるという事実はバリデーターによって処理されません。

### バリデーションファイル

`validateSend()`メソッドのバリデーターで連絡フォームの検証に簡単に実装できますが、多くのコードを繰り返すことを意味します。symfonyはフォームのためにバリデーションルールを定義する代わりの方法を提供し、この方法はYAMLを含みます。たとえば、リスト10-19は`name`フィールドのバリデーションルールの翻訳を示し、その結果はリスト10-18のものと同等です。

リスト10-19 - バリデーションファイル(`modules/contact/validate/send.yml`)

    fields:
      name:
        required:
          msg:       名前のフィールドは空白であってはなりません
        sfStringValidator:
          min:       2
          min_error: この名前は短すぎます。(最小で2文字)
          max:       100
          max_error: この名前は長すぎます。(最大で100文字)

バリデーションファイルにおいて、`fields`ヘッダーは、必要であれば検証されるフィールド、と値が存在するときにテストされるべきバリデーターの一覧を表示します。それぞれのバリデーターの値はバリデーターを手動で初期化するときに使うものと同じです。フィールドは必要なバリデーターの数だけ検証できます。

>**NOTE**
>バリデーターが失敗するときはバリデーションプロセスは停止しません。symfonyはすべてのバリデーターをテストし、少なくともそれらの1つが失敗した場合にバリデーションが失敗したことを宣言します。バリデーションファイルのルールのいくつかが失敗した場合でも、symfonyはまだ`validateXXX()`メソッドを探しそれを実行します。2つのバリデーションのテクニックはお互いを補っています。利点は複数の失敗をともなうフォームにおいて、すべてのエラーメッセージが表示されることです。

バリデーションファイルはモジュールの`validate/`ディレクトリに設置され、これらが検証しなければならないアクションの名前を取って名づけられます。たとえばリスト10-19は`validate/send.yml`という名前のファイルに保存されなければなりません。

### フォームを再表示する

デフォルトでは、symfonyはバリデーションの処理が失敗するときはいつもアクションクラスの`handleErrorSend()`メソッドを探す、もしくはメソッドが存在しない場合は`sendError.php`テンプレートを表示します。

失敗した検証をユーザーに知らせる通常の方法はエラーメッセージをともなうフォームを再表示することです。この目的のには、リスト10-20で示されるように、`handleErrorSend()`メソッドをオーバーライドし、フォームを表示するアクション(たとえば、`module/index`)へのリダイレクトで終わらせる必要があります。

リスト10-20 - フォームを再表示する(`modules/contact/actions/actions.class.php`)

    [php]
    class ContactActions extends sfActions
    {
      public function executeIndex()
      {
        //フォームを表示する
      }

      public function handleErrorSend()
      {
        $this->forward('contact', 'index');
      }

      public function executeSend()
      {
        //フォーム投稿を扱う
      }
    }

フォームを表示してフォーム投稿を扱うために同じアクションを選択する場合、リスト10-21で示されるように、`handleErrorSend()`メソッドは`sendSuccess.php`からフォームを再表示するために、単に`sfView::SUCCESS`を返します。

リスト10-21 - フォームを表示して処理する単独のアクション(`modules/contact/actions/actions.class.php`)

    [php]
    class ContactActions extends sfActions
    {
      public function executeSend()
      {
        if ($this->getRequest()->getMethod() != sfRequest::POST)
        {
          // テンプレートのためのデータを表示する

          // フォームを表示する
          return sfView::SUCCESS;
        }
        else
        {
          // 投稿を処理する
          ...
          $this->redirect('mymodule/anotheraction');
        }
      }
      public function handleErrorSend()
      {
        // テンプレートに対してデータを表示する

        // フォームを表示する
        return sfView::SUCCESS;
      }
    }

`executeSend()`メソッドと`handleErrorSend()`メソッドのなかで繰り返すことを避けるために、データを準備する必要のあるロジックをアクションクラスのprotectedとして定義されたメソッドにリファクタリングすることができます。

新しい設定によって、ユーザーが無効な名前を入力した場合、フォームは再び表示されますが、入力されたデータは失われ、エラーメッセージは失敗の理由を表示しません。最後の問題を解決するには、誤ったフィールドに近いエラーメッセージを挿入するために、フォームを表示するテンプレートを修正しなければなりません。

### フォームのなかでエラーメッセージを表示する

バリデーターパラメーターとして定義されたエラーメッセージはフィールドがバリデーションを失敗したときにリクエストに追加されます(リスト10-18で示されるように、`setError()`メソッドを用いてエラーを手動で追加できる)。`sfRequest`オブジェクトはエラーメッセージを読みとるために2つの便利なメソッド: `hasError()`と`getError()`を提供します。それぞれのメソッドはパラメーターとしてフィールドの名前を必要とします。加えて、1つもしくは多くのフォールドが`hasErrors()`メソッドで無効なデータを含むという事実が注目されるようにフォームのトップにアラートを表示できます。リスト10-22と10-23はこれらのメソッドの使いかたを示しています。

リスト10-22 - フォームのトップでエラーメッセージを表示する(`templates/indexSuccess.php`)

    [php]
    <?php if ($sf_request->hasErrors()): ?>
      <p>あなたが入力したデータは無効のようです。
      つぎのエラーを修正して再投稿して下さるようお願いします:</p>
      <ul>
      <?php foreach($sf_request->getErrors() as $name => $error): ?>
        <li><?php echo $name ?>: <?php echo $error ?></li>
      <?php endforeach; ?>
      </ul>
    <?php endif; ?>

リスト10-23 - エラーメッセージをフォーム内部に表示する(`templates/indexSuccess.php`)

    [php]
    <?php echo form_tag('contact/send') ?>
      <?php if ($sf_request->hasError('name')): ?>
        <?php echo $sf_request->getError('name') ?> <br />
      <?php endif; ?>
      Name:    <?php echo input_tag('name') ?><br />
      ...
      <?php echo submit_tag() ?>
    </form>

リスト10-23の`getError()`メソッドを使用して条件文を書くのは少し冗長です。`Validation`ヘルパーグループの使用を宣言することを前提とすると、symfonyが`getError()`メソッドを置き換えるために`form_error()`ヘルパーを提供する理由はそういうわけです。リスト10-24はこのヘルパーを使うことでリスト10-23を置き換えています。

リスト10-24 - 省略記法で、フォーム内部からエラーメッセージを表示する

    [php]
    <?php use_helper('Validation') ?>
    <?php echo form_tag('contact/send') ?>

               <?php echo form_error('name') ?><br />
      Name:    <?php echo input_tag('name') ?><br />
      ...
      <?php echo submit_tag() ?>
    </form>

`form_error()`ヘルパーはメッセージをより見やすくするためにそれぞれのエラーメッセージの前後に特別な文字を追加します。デフォルトでは、文字は下を指し示す矢印(`&darr;`エンティティに対応)ですが、`settings.yml`ファイルのなかで変更できます:

    all:
      .settings:
        validation_error_prefix:    ' &darr;&nbsp;'
        validation_error_suffix:    ' &nbsp;&darr;'

バリデーションを失敗した場合、フォームはエラーを正しく表示しますが、ユーザーによって入力されたデータは失われます。本当にユーザーフレンドリにするにはフォームを再投入する必要があります。

### フォームを再投入する

エラー処理は`forward()`メソッドを通して行われるので(リスト10-20で示される)、オリジナルのリクエストはまだアクセス可能で、ユーザーによって入力されたデータはリクエストパラメーターのなかに存在します。リスト10-25で示されるように、デフォルト値をそれぞれのフィールドに追加することでフォームを再投入できます。

リスト10-25 - 検証が失敗したときにフォームを再投入するためにデフォルト値を設定する(`templates/indexSuccess.php`)

    [php]
    <?php use_helper('Validation') ?>
    <?php echo form_tag('contact/send') ?>
               <?php echo form_error('name') ?><br />
      名前:    <?php echo input_tag('name', $sf_params->get('name')) ?><br />
               <?php echo form_error('email') ?><br />
      Eメール:   <?php echo input_tag('email', $sf_params->get('email')) ?><br />
               <?php echo form_error('age') ?><br />
      年齢:     <?php echo input_tag('age', $sf_params->get('age')) ?><br />
               <?php echo form_error('message') ?><br />
      メッセージ: <?php echo textarea_tag('message', $sf_params->get('message')) ?><br />
      <?php echo submit_tag() ?>
    </form>

繰り返しますが、これを書く作業はとても退屈です。symfonyは、YAMLバリデーションファイルにおいて、要素のデフォルト値の変更を行わずにフォームのすべてのフィールドを再投入する代替方法を提供します。フォームに対して`fillin:`機能を有効にするだけです。構文はリスト10-26で説明されています。

リスト10-26 - バリデーションが失敗したときにフォームを再投入するために`fillin`を有効にする(`validate/send.yml`)

    fillin:
      enabled: true  # フォームの再投入を有効にする
      param:
        name: test  # ページに1つのフォームのみが存在する場合のフォーム名
        skip_fields:   [email]  # これらのフィールドを再投入しない
        exclude_types: [hidden, password] # これらのフィールドタイプを再投入しない
        check_types:   [text, checkbox, radio, password, hidden] # これらを再投入する
        content_type:  html  # htmlは寛容なデフォルト。ほかのオプションはxmlとxhtml(xmlと同じだがxmlの最初の部分はなし)

デフォルトでは、自動の再投入はテキスト入力、チェックボックス、ラジオボタン、テキストエリア、と選択コンポーネント(シンプルなものと複数)に対して動作しますが、、パスワードもしくは隠しタグは再投入しません。`fillin`機能はファイルタグに対して動作しません。

>**NOTE**
>`fillin`機能はXMLのレスポンスの内容をユーザーに送る直前に解析することで機能します。デフォルトでは`fillin`はHTMLを出力します。
>
>`fillin`にXHTMLを出力させたい場合、`param: content_type: xml`を設定しなければなりません。レスポンスが厳密に有効なXHTMLのドキュメントではない場合`fillin`は機能しません。
>3番目に利用可能な`content_type`は`xhtml`で`xml`と同じですがIE6で奇妙な動作を引き起こすレスポンスからxmlのプロローグをとり除いてます。 

ユーザーが入力した値をフォームの入力に書き戻すまえにこれらを変換したい場合があります。リスト10-27で示されるように、変換を`conveters:`キーの下で定義すればエスケーピング、URLの書き換え、特別な文字をエンティティに変換する、関数を通して呼び出されたほかの変換などはフォームのフィールドに適用できます。

リスト10-27 - `fillin`のまえに入力を変換する(`validate/send.yml`)

    fillin:
      enabled: true
      param:
        name: test
        converters:         # 適用するコンバータ
          htmlentities:     [first_name, comments]
          htmlspecialchars: [comments]

### symfonyの標準のバリデーター 

symfonyはフォームに対して使える標準のバリデーターをいくつか持ちます:

  * `sfStringValidator`
  * `sfNumberValidator`
  * `sfEmailValidator`
  * `sfUrlValidator`
  * `sfRegexValidator`
  * `sfCompareValidator`
  * `sfPropelUniqueValidator`
  * `sfFileValidator`
  * `sfCallbackValidator`

それぞれのバリデーターはデフォルトのパラメーターとエラーメッセージのセットを持ちますが、`initialize()`バリデーターメソッドを通してYAMLファイル内で、簡単にオーバーライドできます。つぎのセクションでバリデーターを説明し、使いかたの例を示します。

#### 文字列のバリデーター

`sfStringValidator`によって文字に関連する制約をパラメーターに適用できるようになります。

    sfStringValidator:
      values:       [foo, bar]
      values_error: 認められる値はfooとbarのみです
      insensitive:  false  # trueの場合、値との比較は大文字と小文字を区別しない
      min:          2
      min_error:    少なくとも2文字入力してください
      max:          100
      max_error:    100文字以下で入力してください

#### 数字のバリデーター

`sfNumberValidator`パラメーターが数字であることを検証し、これによってサイズの制約を適用することができます。

    sfNumberValidator:
      nan_error:    整数を入力してください
      min:          0
      min_error:    値は最少で0でなければなりません
      max:          100
      max_error:    値は100以下でなければなりません

#### Eメールのバリデーター

`sfEmailValidator`はEメールアドレスとして有効なパラメーターが含まれるかを検証します。

    sfEmailValidator:
      strict:       true
      email_error:  このEメールアドレスは無効です

RFC822はEメールアドレスのフォーマットを定義します。しかしながら、これは一般的に受け入れられるフォーマットよりも寛容です。たとえば、RFCに従えば`me@localhost`は有効なEメールアドレスですが、おそらくは受信したくないでしょう。`strict`パラメーターを`true`に設定したとき、`name@domain.extenison`にマッチするEメールアドレスだけが有効です。`false`に設定したとき、RFC822がルールとして使われます。

#### URLバリデーター

`sfUrlValidator`はフィールドが正しいURLであるかチェックします。

    sfUrlValidator:
      url_error:    このURLは無効です

#### 正規表現のバリデーター

`sfRegexValidator`によって値をPerlと互換性のある正規表現のパターンとマッチできます。

    sfRegexValidator:
      match:        No
      match_error:  URLを含む投稿はスパムと見なされます
      pattern:      /http.*http/si

`match`パラメーターはリクエストパラメーターが有効なパターンにマッチする(値は`Yes`)、もしくは無効なパターン(値は`No`)にマッチするかを決定します。

#### 比較のバリデーター

`sfCompareValidator`は2つの異なるリクエストパラメーターが等しいかチェックします。パスワードをチェックするためにとても便利です。

    fields:
      password1:
        required:
          msg:      パスワードを入力してください
      password2:
        required:
          msg:      パスワードを再入力してください
        sfCompareValidator:
          check:    password1
          compare_error: 2つのパスワードが一致しません

`check`パラメーターは現在のフィールドがマッチしなければならないフィールドの名前を含みます。

#### Propel/Doctrine独自のバリデーター 

`sfPropelUniqueValidator`はリクエストパラメーターの値がすでにデータベースに存在していないかを検証します。ユニークインデックスのためにとても便利です。

    fields:
      nickname:
        sfPropelUniqueValidator:
          class:        User
          column:       login
          unique_error: このログインはすでに存在します。別のものを入力してください。

この例において、バリデーターは`login`カラムが検証するフィールドと同じ値を持つ`User`クラスのレコードに対してデータベースを調べます。

>**CAUTION**
>`sfPropelUniqueValidator`は競合条件の影響を受けやすいです。想像しにくいですが、複数ユーザーの環境において、結果は自身が返す瞬間を変えることがあります。重複の`INSERT`エラーを扱う準備もすべきです。

#### ファイルのバリデーター

`sfFileValidator`はフォーマット(mime-typeの配列)とサイズの制限をファイルのアップロードフィールドに適用します。

    fields:
      image:
        file:       True
        required:
          msg:      画像ファイルをアップロードしてください
        sfFileValidator:
          mime_types:
            - 'image/jpeg'
            - 'image/png'
            - 'image/x-png'
            - 'image/pjpeg'
          mime_types_error: PNGとJPEGの画像のみ許可されます
          max_size:         512000
          max_size_error:   最大のサイズは512KBです

`file`属性はフィールドに対して`True`に設定され、テンプレートはフォームを multipartとして宣言しなければならないことを覚えておいてください。

#### コールバックバリデーター

`sfCallbackValidator`はバリデーションをサードパーティの呼び出し可能なメソッドもしくはバリデーションを行う関数に委譲します。呼び出し可能なメソッドもしくは関数は`true`もしくは`false`を返さなければなりません。

    fields:
      account_number:
        sfCallbackValidator:
          callback:      is_numeric
          invalid_error: 数字を入力してください。
      credit_card_number:
        sfCallbackValidator:
          callback:      [myTools, validateCreditCard]
          invalid_error: 有効なクレジットカードの番号を入力してください。

コールバックメソッドもしくは関数は検証された値を最初のパラメーターとして受けとります。新しい完全なバリデータークラスを作成するよりも、既存のメソッドの機能を再利用するときにとても役立ちます。

>**TIP**
>この章の後のほうの"カスタムバリデーターを作成する"で説明されている独自のバリデーターを書くこともできます。

### 名前つきのバリデーター

バリデータークラスと設定を繰り返す必要であることがわかる場合、これを名前つきのバリデーターのもとでまとめることができます。連絡フォームの例において、emailフィールドは同じ`sfStringValidator`パラメーターを`name`フィールドとして必要とします。同じ設定を2回繰り返すことを避けるために、名前つきのバリデーターである`myStringValidator`を作成できます。これを行うために、`myStringValidator`ラベルを`validators:`ヘッダーの下に追加し、まとめたい名前つきのバリデーターの詳細な内容を持つ`class`キーと`param`キーを設定します。リスト10-28で示されるように、`fields`セクションの通常の名前つきバリデーターとまったく同じように、名前つきバリデーターを利用できます。

リスト10-28 - バリデーションファイルで名前つきバリデーターを再利用する(`validte/send.yml`)

    validators:
      myStringValidator:
        class: sfStringValidator
        param:
          min:       2
          min_error: このフィールドは短すぎます(最少で2文字)
          max:       100
          max_error: このフィールドは長すぎます(最大で100文字)

    fields:
      name:
        required:
          msg:       名前のフィールドは空白であってはなりません
        myStringValidator:
      email:
        required:
          msg:       Eメールのフィールドは空白であってはなりません
        myStringValidator:
        sfEmailValidator:
          email_error:  このEメールは無効です

### メソッドへのバリデーションを制限する

デフォルトでは、バリデーションファイルに設定されたバリデーターは、アクションがPOSTメソッドで呼び出されたときに実行されます。リスト10-29で示されるように、異なるメソッドに対して異なるバリデーションをできるようにするために、`methods`キーにおいて別の値を指定することでこの設定をグローバルもしくはフィールド単位でオーバーライドできます。

リスト10-29 - フィールドをテストするときを定義する(`validate/send.yml`)

    methods:         [post]     # これはデフォルトの設定

    fields:
      name:
        required:
          msg:       名前のフィールドは空白にはできません
        myStringValidator:
      email:
        methods:     [post, get] # グローバルメソッドの設定をオーバーライドする
        required:
          msg:       Eメールフィールドは空白にはできません
        myStringValidator:
        sfEmailValidator:
          email_error:  このEメールアドレスは無効です

### バリデーションファイルはどのようになるのか？

これまでのところ、バリデーションファイルの断片しか見てきませんでした。すべてを組み立てるとき、バリデーションのルールはYAMLで明確な変換を見つけます。リスト10-30はこの章の前の方で定義したすべてのルールに対応したサンプルの連絡フォームのための完全なバリデーションファイルを示しています。

リスト10-30 - 完全なバリデーションファイルのサンプル

    fillin:
      enabled:      true

    validators:
      myStringValidator:
        class: sfStringValidator
        param:
          min:       2
          min_error: このフィールドは短すぎます(最少で2文字)
          max:       100
          max_error: このフィールドは長すぎます(最大で100文字)

    fields:
      name:
        required:
          msg:       名前のフィールドは空白のままにできません
        myStringValidator:
      email:
        required:
          msg:       Eメールフィールドは空白のままにできません
        myStringValidator:
        sfEmailValidator:
          email_error:  このEメールアドレスは無効です
      age:
        sfNumberValidator:
          nan_error:    整数を入力してください
          min:          0
          min_error:    "あなたははまだ生まれていません。どのようにメッセージを送りたいのですか？"
          max:          120
          max_error:    "ちょっと、おばあちゃん、インターネットをサーフィンするには年をとりすぎていませんか？"
      message:
        required:
          msg:          メッセージフィールドを空白のままにできません

複雑なバリデーション
--------------------

バリデーションファイルは多くのニーズを満たしますが、バリデーションがとても複雑なときに、十分ではないかもしれません。この場合、アクションの`validateXXX()`メソッドに戻るもしくはつぎのセクションであなたの問題の解決方法が見つかります。

### カスタムバリデーターを作成する

それぞれのバリデーターは`sfValidator`クラスを拡張するクラスです。symfonyに搭載されたバリデータークラスがあなたのニーズに適していない場合、これをオートロードできる`lib/`ディレクトリの任意の場所で、新しいバリデータークラスを作成できます。構文はとてもシンプルです: バリデーターが実行されるとき、バリデーターの`execute()`メソッドが呼び出されます。`initialize()`メソッドでデフォルトの設定を定義することもできます。

`execute()`メソッドは最初のパラメーターとして検証する値と第2のパラメーターとして投げるエラーメッセージを受けとります。両方の値は参照として渡されるので、メソッドの範囲内からエラーメッセージを修正できます。

`initialize()`メソッドはYAMLファイルからContext Singletonとパラメーターの配列を受けとります。最初に親の`sfValidator`クラスの`initalize()`メソッドを呼び出さなければならず、そしてデフォルト値を設定します。

すべてのバリデーターは`$this->getParameterHolder()`によってアクセス可能なパラメーターホルダーを持ちます。

たとえば、文字列がスパムではないことをチェックするために`sfSpamValidator`を作りたい場合、リスト10-31で示されるコードを`sfSpamValidator.class.php`ファイルを追加します。`$value`が文字列の`'http'`を`max_url`倍多くの含むのかをチェックします。

リスト10-31 - カスタムバリデーターを作成する(`lib/sfSpamValidator.class.php`)

    [php]
    class sfSpamValidator extends sfValidator
    {
      public function execute (&$value, &$error)
      {
        // max_url=2に対して、正規表現は/http.*http/is
        $re = '/'.implode('.*', array_fill(0, $this->getParameter('max_url') + 1, 'http')).'/is';

        if (preg_match($re, $value))
        {
          $error = $this->getParameter('spam_error');

          return false;
        }

        return true;
      }

      public function initialize ($context, $parameters = null)
      {
        // 親を初期化する
        parent::initialize($context);

        // パラメーターのデフォルト値を設定する
        $this->setParameter('max_url', 2);
        $this->setParameter('spam_error', 'This is spam');

        // パラメーターを設定する
        $this->getParameterHolder()->add($parameters);

        return true;
      }
    }

バリデーターが`autoloadable`ディレクトリに追加されると同時に(そしてキャッシュがクリアされる)、リスト10-32で示されるように、バリデーションファイルでこれを使うことができます。

リスト10-32 - カスタムバリデーターを使う(`validate/send.yml`)

    fields:
      message:
        required:
          msg:          メッセージフィールドを空白にできません
        sfSpamValidator:
          max_url:      3
          spam_error:   このサイトを立ち去ってください、汚らわしいスパマーさん！

### フォームフィールドに対して配列構文を使う

PHPによってフォームフィールドに対して配列構文を利用できます。独自のフォームを書くとき、もしくはPropelのadministration(14章を参照)によって生成されたフォームを使うとき、リスト10-33のようなHTMLのコードで終わります。

リスト10-33 - 配列構文によるフォーム

    [php]
    <label for="story_title">タイトル:</label>
    <input type="text" name="story[title]" id="story_title" value="default value"
           size="45" />

バリデーションファイルで入力名を(角かっこで)そのまま使うと解析で引き起こされたエラーが投じられます。ここでの解決方法はリスト10-34で示されるように、`fields`セクションにおいて、角かっこ`[]`を中かっこ`{}`で置き換えることで、symfonyはあとでバリデーターに送られる名前の変換を考慮するようになります。

リスト10-34 - 配列構文を利用したフォームのためのバリデーションファイル

    fields:
      story{title}:
        required:     Yes

### 空のフィールド上でバリデーターを実行する

空の値の上で、必要ないフィールド上で、バリデーターを実行することが必要な場合があります。たとえば、ユーザーがパスワードを変更したい(したくない場合)にこの事例があてはまります。この場合、確認パスワードを入力しなければなりません。リスト10-35で例をご覧ください。

リスト10-35 - 2つのパスワードフィールドを持つフォームのためのバリデーションファイルのサンプル

    fields:
      password1:
      password2:
        sfCompareValidator:
          check:         password1
          compare_error: 2つのパスワードが一致しません

バリデーションの処理はつぎのように実行されます:

  * `password1` `== null`と`password2 == null`の場合:

    * `required`テストはパスします。
    * バリデーターは実行されません。
    * フォームは有効です。

  * `password2 == null`である一方で`password1`が`null`ではない場合:

    * `required`テストはパスします。
    * バリデーターは実行されません。
    * フォームは有効です。

`password1`が`not null`であるかどうか`password2`バリデーターを実行したい場合があります。幸いにも、`group`パラメーターのおかげでsymfonyバリデーターはこの事例に対処します。フィールドがグループ内に存在する場合、それが空ではなく同じグループのフィールドの1つが空ではないかバリデーターを実行します。

ですので、リスト10-36で示されているように設定を変更する場合、バリデーションプロセスは正しくふるまいます。

リスト10-36 - 2つのパスワードフィールドとグループによるフォームのためのバリデーションファイルのサンプル

    fields:
      password1:
        group:           password_group
      password2:
        group:           password_group
        sfCompareValidator:
          check:         password1
          compare_error: 2つのパスワードが一致しません

バリデーションの処理はつぎのように実行します:

  * `password1 == null`かつ`password2 == null`の場合:

    * `required`のテストがパスします。
    * バリデーターは実行されません。
    * フォームは有効です。

  * `password1 == null`かつ`password2 == 'foo'`の場合:

    * `required`テストはパスします。
    * `password2`は`not null`なので、バリデーターは実行され、失敗します。
    * エラーメッセージが`password2`にスローされます。

  * `password1 == 'foo'`で`password2 == null`の場合:

    * `required`テストがパスします。
    * `password1`が`not null`なので同じグループにある`password2`のためのバリデーターが実行され、失敗します。
    * エラーメッセージが`password2`にスローされます。

  * `password1 == 'foo'`で`password2 == 'foo'`の場合:

    * `required`テストがパスします。
    * `password2`が`not null`なので、バリデーターが実行され、パスします。
    * フォームは有効です。

まとめ
----

symfonyのテンプレートのなかでフォーム(form)を書く作業は標準のフォームヘルパーとスマートなオプションによって円滑になります。オブジェクトのプロパティを編集するためにフォームを設計するとき、バリデーションファイル(validation file)、バリデーションヘルパー、と再投入機能はフィールドの値の上での頑強でユーザーフレンドリなサーバーのコントロール機能を開発するために必要な作業量を減らします。そして、アクションクラスのなかでカスタムバリデーターを書くもしくは`validateXXX()`メソッドを作ることで、より複雑なバリデーションの需要も扱うことができます。
