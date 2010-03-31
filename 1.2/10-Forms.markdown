第10章 - フォーム
=================

>**Caution**
>この章では symfony 1.0 でフォームを実装する方法を説明しています。互換性のために admin ジェネレーターの機能はまだこのシステムを利用するので、symfony 1.2 でもこの章の情報も価値があります。しかしながら、symfony 1.2 で新しいプロジェクトを始めるのであれば、新しいフォームフレームワークの詳細について「symfony 
Forms in Action」の本も読むべきです。

テンプレートを書くとき、開発者の多くの時間はフォーム (form) に費やされます。このことにもかかわらず、一般的にフォームは貧弱に設計されます。デフォルト値、整形、バリデーション、再投入と一般的なフォームの扱いなど多くのことに注意を払うことが必要なので、開発者のなかにはプロセスにおけるいくつかの重要な詳細事項を大まかにしか見ない人がいます。というわけで、symfonyはこのトピックに対して特別な注意を払うことにします。この章ではフォームの開発速度を上げる一方で、これらの多くの要件を自動化するツールについて説明します。

  * フォームヘルパーは、とりわけ日付、ドロップダウンのリスト、とリッチテキストなどの複雑な要素に対して、テンプレートのなかでフォームを速く書く方法を提供します。
  * フォームがオブジェクトのプロパティを編集することに徹する場合、オブジェクトフォームヘルパーを利用することでテンプレートの編集作業が速くなります。
  * YAML バリデーションファイルはフォームのバリデーションと再投入を円滑にします。
  * バリデーターは入力をバリデートするために必要なコードをまとめます。symfony はもっとも共通なニーズのためのバリデーターを搭載します。またカスタムバリデーターを追加することはとても簡単です。

フォームヘルパー
----------------

テンプレートによって、フォーム要素の HTML タグが PHP コードに混ざることはよくあることです。symfony のフォームヘルパーはこのタスクをシンプルにして `<input>` タグのあいだで `<?php echo` のタグを繰り返し展開することを避けることを目的としています。

### メインのフォームタグ

以前の章で説明したように、フォームを作るには `form_tag()` ヘルパーを使わなければなりません。このヘルパーがパラメーターとして渡されたアクションをルーティングURLに変換するからです。2番目の引数は追加オプションをサポートします。たとえば、デフォルトの `method` を変更するには、デフォルトの `enctype` を変更するか、もしくはほかの属性を指定します。リスト10-1は例を示しています。

リスト10-1 - `form_tag()` ヘルパー

    [php]
    <?php echo form_tag('test/save') ?>
     => <form method="post" action="/path/to/save">

    <?php echo form_tag('test/save', 'method=get multipart=true class=simpleForm') ?>
     => <form method="get" enctype="multipart/form-data" class="simpleForm"action="/path/to/save">

フォームヘルパーを閉じる必要がないので、ソースコードのなかできれいに見えなくても、HTML の `</form>`を使うべきです。

### 標準のフォーム要素

デフォルトでは、フォームヘルパーによって、フォームのなかのそれぞれの要素は name 属性から推測される id 属性に渡されます。これは便利なだけの規約ではありません。標準のフォームヘルパーとそれらのオプションの全リストに関してはリスト10-2をご覧ください。

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

    // 複数の選択肢を持つドロップダウンリスト (選択された値は配列が可能)
    <?php echo select_tag('payment', options_for_select(
      array('Visa' => 'Visa', 'Eurocard' => 'Eurocard', 'Mastercard' => 'Mastercard'),
      array('Visa', 'Mastercard'),
    ), array('multiple' => true))) ?>

     => <select name="payment[]" id="payment" multiple="multiple">
          <option value="Visa" selected="selected">Visa</option>
          <option value="Eurocard">Eurocard</option>
          <option value="Mastercard">Mastercard</option>
        </select>
    // 複数の選択肢を持つドロップダウンリスト (選択された値は配列が可能)
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

`submit_image_tag()` ヘルパーは同じ構文を使い `image_tag()` と同じ利点を持ちます。

>**NOTE**
>ラジオボタンに関して、デフォルトでは `id` 属性は `name` 属性の値ではなく名前と値の組み合わせに設定されます。自動化された「別のものを選択したときに以前の選択を解除する」機能を手に入れるために同じ名前を持ついくつかのラジオボタンのタグを用意する必要があるからで、`id=name` の慣習によればページのなかで `id` 属性を持つ HTML タグが存在することを意味します。これは厳密には禁止されています。

-

>**SIDEBAR**
>**フォーム投稿の扱いかた**
>
>フォームを通してユーザーによって投稿されたデータをどのように読みとりますか？リクエストパラメーターで利用できるので、アクションは値を得るために `$request->getParameter($elementName)` を呼び出すことが必要なだけです。
>
>フォームを表示して処理するためのよい習慣は同じアクションを使うことです。リクエストメソッド (GET もしくは POST) に従い、フォームのテンプレートが呼び出されるもしくはフォームが処理されるとリクエストは別のアクションにリダイレクトされます。
>
>     [php]
>     // mymodule/actions/actions.class.phpにおいて
>     public function executeEditAuthor(sfWebRequest $request)
>     {
>       if (!$this->getRequest()->isMethod('post'))
>       {
>         // フォームを表示する
>         return sfView::SUCCESS;
>       }
>       else
>       {
>         // フォーム投稿を扱う
>         $name = $request->getParameter('name');
>         ...
>         $this->redirect('mymodule/anotheraction');
>       }
>     }
>
>これを機能させるには、フォームのターゲットは表示するアクションと同じアクションでなければなりません。
>
>     [php]
>     // mymodule/templates/editAuthorSuccess.phpにおいて
>     <?php echo form_tag('mymodule/editAuthor') ?>
>
>     ...

symfony は裏で非同期通信リクエストを行うために専用のフォームヘルパーを提供します。つぎの章では、Ajax について焦点を当てて、詳細な説明を行います。

### 日付入力ウィジェット

フォームは日付を読みとるためによく使われます。誤った書式の日付はフォーム投稿が失敗する主な原因です。図10-1で示されるように、`rich` オプションを `true` にセットする場合、`input_date_tag()` ヘルパーはインタラクティブな JavaScript カレンダーのなかで入力するユーザーを助けます。

図10-1 - リッチな日付入力タグ

![リッチな日付入力タグ](http://github.com/masakielastic/symfonybook-ja/raw/master/images/F1001.png "リッチな日付入力タグ")

`rich` オプションが省略された場合、月、日にち、年の範囲で投入されたヘルパーは3つの `<select>` タグを出力します。これらのヘルパー (`select_day_tag()`、`select_month_tag()` と `select_year_tag()`) を呼び出すことでこれらのドロップダウンを個別に表示できます。これらの要素のデフォルト値は現在の日にち、月、年です。リスト10-3は日付入力ヘルパーを示してます。

リスト10-3 - 日付入力ヘルパー

    [php]
    <?php echo input_date_tag('dateofbirth', '2005-05-03', 'rich=true') ?>
     => a text input tag together with a calendar widget

    // つぎのヘルパーは DateForm ヘルパーグループを必要とする
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

`input_date_tag()` ヘルパーが受け入れる日付の値は PHP 関数の `strtotime()` によって認識されるものです。リスト10-4は利用できる書式を示し、リスト10-5は避けなければならない書式を示しています。

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

TinyMCE と FCKEditor ウィジェットの統合機能のおかげで、`<textarea>` タグでもリッチなテキスト編集機能を利用できます。図10-2で示されるように、これらはテキストを太文字、イタリック体、そのほかのスタイルとして整形するためのボタンつきのワードプロセッサのようなインターフェイスを提供します。

図10-2 - リッチなテキスト編集機能

![リッチなテキスト編集機能](http://github.com/masakielastic/symfonybook-ja/raw/master/images/F1002.png "リッチなテキスト編集機能")

両方のウィジェットとも手動でインストールする作業が必要です。2つのウィジェットのインストールの手続きは同じなので、ここでは TinyMCE のリッチなテキスト編集機能だけを説明します。プロジェクトの公式サイト ([http://tinymce.moxiecode.com/](http://tinymce.moxiecode.com/)) からエディタをダウンロードし、一時フォルダーで解凍する必要があります。リスト10-6で示されるように、`tinymce/jscripts/tiny_mce/` ディレクトリをプロジェクトの `web/js/` ディレクトリにコピーし、`settings.yml` のなかでライブラリへのパスを定義します。

リスト10-6 - TinyMCE ライブラリのパスをセットアップする

    all:
      .settings:
        rich_text_js_dir:  js/tiny_mce

セットアップが終了したら、テキストエリアのリッチなテキスト編集機能を利用できるように `rich=true` オプションを追加して切り替えます。JavaScript エディタに `tinymce_options` オプションを渡すことでカスタムオプションを指定することもできます。リスト10-7を示してください。

リスト10-7 - リッチなテキストエリア

    [php]
    <?php echo textarea_tag('name', 'default content', 'rich=true size=10x20') ?>
     => a rich text edit zone powered by TinyMCE
    <?php echo textarea_tag('name', 'default content', 'rich=true size=10x20 tinymce_options=language:"fr",theme_advanced_buttons2:"separator"') ?>
    => a rich text edit zone powered by TinyMCE with custom parameters

### 国、言語と通貨の選択

国の選択フィールドを表示する必要がある場合を考えます。しかし国の名前はすべての言語とは同じではないので国のドロップダウンリストのオプションはユーザー culture にしたがって変わります (culture に関する詳細な情報は13章を参照)。リスト10-8で示されるように、`select_country_tag()` ヘルパーがあなたの代わりにすべてを行います: これは国の名前を国際化し、ISO の標準国コードを値として使います。

リスト10-8 - 国のタグヘルパーを選ぶ

    [php]
    <?php echo select_country_tag('country', 'AL') ?>
     => <select name="country" id="country">
          <option value="AF">アフガニスタン</option>
          <option value="AL" selected="selected">アルバニア</option>
          <option value="DZ">アルジェリア</option>
          <option value="AS">アメリカ領サモア</option>
      ...

`select_country_tag()` ヘルパーと同じように、`selext_language_tag()` ヘルパーはリスト10-9で示されるような言語のリストを表示します。 

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

3番目のヘルパーは `select_currency_tag()` ヘルパーで、リスト10-10のような、通貨の一覧を表示します。

リスト10-10 - 通貨タグヘルパーを選ぶ

    [php]
    <?php echo select_currency_tag('currency', 'EUR') ?>
     => <select name="currency" id="currency">
          ...
          <option value="ETB">エチオピアビル</option>
          <option value="ETD">エチオピアドル</option>
          <option value="EUR" selected="selected">ユーロ</option>
          <option value="XBA">欧州複合単位(EURCO)</option>
          <option value="XEU">欧州通貨単位</option>
          ...

>**NOTE**
>3つのすべてのヘルパーはオプションの配列である3番目の引数を受けとります。これは表示されるオプションを特定のセット: `array('countries' => array ('FR', 'DE'))` のように制限できます。`select_language_tag()` に対するオプションの名前は `languages` で `select_currency_tag()` に対するオプションの名前は `currencies` です。
>たいていの場合、セットを既知でサポートされる値のリスト、とりわけ期限切れの項目を含む可能性があるリスト、に制限することに意味があります。
>
>`select_currency_tag()` はオプションに表示されるものに影響を与える `display` という名前の追加オプションを提供します。これは `symbol`、`code` もしくは `name` の1つに設定できます。

オブジェクトのためのフォームヘルパー
---------------------------------------

フォームの要素がオブジェクトのプロパティを編集するために使われるとき、標準のリンクヘルパーを書く作業が退屈になることがあります。たとえば、`Customer` オブジェクトの `telephone` 属性を編集するには、つぎのように書くことになります:

    [php]
    <?php echo input_tag('telephone', $customer->getTelephone()) ?>
    => <input type="text" name="telephone" id="telephone" value="0123456789" />

属性の名前を繰り返すことを避けるために、symfony はそれぞれのフォームヘルパーに対して代わりのオブジェクトフォームヘルパーを提供します。オブジェクトフォームヘルパーはオブジェクトとメソッドの名前からフォーム要素の名前とデフォルト値を推測します。以前の`input_tag()` はつぎのコードと同等です:

    [php]
    <?php echo object_input_tag($customer, 'getTelephone') ?>
    => <input type="text" name="telephone" id="telephone" value="0123456789" />

`object_input_tag()` に関して簡潔さは重要ではないかもしれません。しかしながら、すべての標準のフォームヘルパーは対応するオブジェクトフォームヘルパーを持ち、これらはすべて同じ構文を共有します。これはフォームの生成作業をとても楽にします。scaffolding と生成された administration (14章を参照) でオブジェクトフォームヘルパーが広範囲に使われる理由はそういうわけです。リスト10-11はヘルパーからのオブジェクトのリストです。

リスト10-11 - オブジェクトフォームヘルパーの構文

    [php]
    <?php echo object_input_tag($object, $method, $options) ?>
    <?php echo object_input_date_tag($object, $method, $options) ?>
    <?php echo object_input_hidden_tag($object, $method, $options) ?>
    <?php echo object_textarea_tag($object, $method, $options) ?>
    <?php echo object_checkbox_tag($object, $method, $options) ?>
    <?php echo object_select_tag($object, $method, $options) ?>
    <?php echo object_select_country_tag($object, $method, $options) ?>
    <?php echo object_select_language_tag($object, $method, $options) ?>

`object_password_tag()` ヘルパーは存在しません。以前ユーザーが入力した値をもとに、デフォルト値をパスワードのタグに渡すことはよくない習慣だからです。

>**CAUTION**
>通常のフォームヘルパーとは異なり、`use_helper('Object')` を持つテンプレート内で `Object` ヘルパーグループの使用を明示的に宣言した場合のみオブジェクトフォームヘルパーは利用できます。

すべてのオブジェクトフォームヘルパーのなかでもっとも面白いのはドロップダウンリストに関連する `objects_for_select()` と `object_select_tag()` です。

### ドロップダウンをオブジェクトで投入する

`options_for_select()` ヘルパーは、ほかの標準ヘルパーで以前説明されたものですが、リスト10-12で示されるように、PHP の連想配列をオプションのリストに変換します。

リスト10-12 - `options_for_select()` を利用して配列に基づくオプションのリストを作成する

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

すでに、`Propel` のクエリの結果から得られた、`Author` クラスのオブジェクトの配列が存在することを前提とします。この配列をもとにオプションのリストを作りたい場合、リスト10-13で示されるように、それぞれのオブジェクトの `id` と `name` を読みとるためにループする必要があります。

リスト10-13 - `options_for_select()` を使ってオブジェクト配列をもとにオプションのリストを作る

    [php]
    // アクションのなか
    $options = array();
    foreach ($authors as $author)
    {
      $options[$author->getId()] = $author->getName();
    }
    $this->options = $options;

    // テンプレートのなか
    <?php echo options_for_select($options, 4) ?>

この種の処理作業はよく行われるので、symfony はこの作業を自動化するヘルパーを用意します: `objects_for_select()` はオブジェクトの配列から直接オプションリストを作ります。ヘルパーは2つの追加パラメーターが必要です: `value` を読みとるために使われるメソッド名と生成される `<option>` タグのテキスト内容です。リスト10-13はつぎのシンプルなフォームと同等です:

    [php]
    <?php echo objects_for_select($authors, 'getId', 'getName', 4) ?>

これはスマートで速く書けますが、外部キーのカラムを処理するとき、symfony はさらに掘り下げます。

### 外部キーのカラムをもとにドロップダウンリストを作る

外部キーのカラムが取得できる値は外部テーブルのレコードの主キーの値です。たとえば、`article` テーブルが `author` テーブルへの外部キーである `author_id` カラムを持つ場合、このカラムで利用可能な値は `author` テーブルのすべてのレコードの `id` です。基本的には、記事の作者を編集するドロップダウンリストはリスト10-14のようになります。

リスト10-14 - `objects_for_select()` を使って外部キーをもとにオプションのリストを作る

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

`object_select_tag()` は単独ですべてを行います。これは外部テーブルの可能なレコードの名前で投入されたドロップダウンリストを表示します。ヘルパーはスキーマから外部テーブルと外部カラムを推測できるので、構文はとても簡潔です。リスト10-13はつぎのコードと同等です:

    [php]
    <?php echo object_select_tag($article, 'getAuthorId') ?>

`object_select_tag()` ヘルパーはパラメーターとして渡されるメソッドの名前をもとに関連するピアクラスの名前 (この例では `AuthorPeer`) を推測します。しかしながら、`related_class` オプションを3番目の引数に設定することで独自のクラスを指定できます。`<option>` タグのテキスト内容はレコード名で、オブジェクトクラスの `__toString()` メソッドの結果です (`$author->__toString()` メソッドが未定義の場合、主キーが代わりに使われます)。加えて、オプションのリストは空の基準値を持つ`doSelect()` メソッドで構成されます; これは作成日順に並べられたすべてのレコードを返します。特定の順番でレコードの部分集合だけを表示したい場合、この選択をオブジェクトの配列として返すピアクラスのなかでメソッドを作り、これを `peer_method` オプションに設定します。最後に、`include_blank` と `include_custom` オプションを設定することでドロップダウンリストのトップに、空白のオプションもしくはカスタムオプションを追加できます。リスト10-15は `object_select_tag()` ヘルパーに対するこれらの異なるオプションを示しています。

リスト10-15 - `object_select_tag()` ヘルパーのオプション

    [php]
    // 基本構文
    <?php echo object_select_tag($article, 'getAuthorId') ?>
    // AuthorPeer::doSelect(new Criteria()) からリストを作る

    // 可能な値を読みとるために使われるピアクラスを変更する
    <?php echo object_select_tag($article, 'getAuthorId', 'related_class=Foobar') ?>
    // FoobarPeer::doSelect(new Criteria()) からリストを作る

    // 可能な値を読みとるためにピアクラスを変更する
    <?php echo object_select_tag($article, 'getAuthorId','peer_method=getMostFamousAuthors') ?>
    // AuthorPeer::getMostFamousAuthors(new Criteria()) からリストを作る

    // リストのトップで <option value="">&nbsp;</option> を追加する
    <?php echo object_select_tag($article, 'getAuthorId', 'include_blank=true') ?>

    // リストのトップで <option value="">Choose an author</option> を追加する
    <?php echo object_select_tag($article, 'getAuthorId',
      'include_custom=Choose an author') ?>

### オブジェクトを更新する

オブジェクトヘルパーを利用してオブジェクトのプロパティを編集するための専用フォームはアクションのなかで扱うよりも簡単です。たとえば、`name`、`age`、`address` 属性を持つ `Author` クラスのオブジェクトがある場合、フォームはリスト10-16のように書きます。

リスト10-16 - オブジェクトヘルパーだけを持つフォーム

    [php]
    <?php echo form_tag('author/update') ?>
      <?php echo object_input_hidden_tag($author, 'getId') ?>
      名前: <?php echo object_input_tag($author, 'getName') ?><br />
      年齢:  <?php echo object_input_tag($author, 'getAge') ?><br />
      アドレス: <br />
             <?php echo object_textarea_tag($author, 'getAddress') ?>
    </form>

リスト10-17で示されるように、フォームが投稿されたときに、呼び出される `author` モジュールの `update` アクションは Propel によって生成された `fromArray()` メソッドによって簡単にオブジェクトを更新できます。

リスト10-17 - オブジェクトフォームヘルパーに基づくフォーム投稿を扱う

    [php]
    public function executeUpdate(sfWebRequest $request)
    {
      $author = AuthorPeer::retrieveByPk($request->getParameter('id'));
      $this->forward404Unless($author);

      $author->fromArray($this->getRequest()->getParameterHolder()->getAll(),BasePeer::TYPE_FIELDNAME);
      $author->save();

      return $this->redirect('/author/show?id='.$author->getId());
    }

フォームのバリデーション
------------------------

>**NOTE**
>このセクションで説明されている機能は symfony 1.1 で廃止され `sfCompat10` プラグインを有効にした場合のみに動作します。

6章ではリクエストパラメーターをバリデートするためにアクションクラスで `validateXXX()` メソッドを使う方法を説明しました。しかしながら、フォーム投稿をバリデートするこのテクニックを利用する場合、同じコードの一部を何度も書き直すことになります。symfony はアクションクラスのなかの PHP コードの代わりに、YAML ファイルだけに依存する、フォームのバリデーションの代替テクニックを提供します。

フォームのバリデーション機能を実証するために、最初にリスト10-18で示されているサンプルのフォームを考えてみましょう。`name`、`email`、`age` と `message`フィールドを持つ、古典的な問い合わせフォームです。

リスト10-18 - 問い合わせフォームのサンプル (`modules/contact/templates/indexSuccess.php`)

    [php]
    <?php echo form_tag('contact/send') ?>
      名前:    <?php echo input_tag('name') ?><br />
      Eメール:   <?php echo input_tag('email') ?><br />
      年齢:     <?php echo input_tag('age') ?><br />
      メッセージ: <?php echo textarea_tag('message') ?><br />
      <?php echo submit_tag() ?>
    </form>

フォームのバリデーションの原則はユーザーが不正なデータを入力しフォームに投稿した場合、つぎのページでエラーメッセージを表示することです。簡単な英語で、サンプルフォームに対して有効なデータを定義してみましょう:

  * `name` フィールドが必要です。2文字から100文字のテキストエントリーでなければなりません。
  * `email` フィールドが必要です。2文字から100文字までのテキストエントリーかつ有効なEメールアドレスでなければなりません。
  * `age` フィールドが必要です。0から120の整数でなければなりません。
  * `message`フィールドが必要です。

問い合わせフォームのためにもっと複雑なバリデーションルールを定義することができますが、バリデーションの可能性を実証するにはこれで十分です。

>**NOTE**
>フォームのバリデーションはサーバーサイドかつ/もしくはクライアントサイドで行われます。サーバーサイドのバリデーションは間違ったデータでデータベースが汚染されることを避けるために必須です。クライアントサイドのバリデーションはユーザーエクスペリエンスを大いに高めますが、オプションです。クライアントサイドのバリデーションはカスタム JavaScript で行われます。

### バリデーター

上記の例において `name` フィールドと `email` フィールドが共通のバリデーションルールを共有していることがわかります。バリデーションルールのいくつかは Web フォームで頻繁に使われるので symfony はこれらのバリデーターを実装する PHP コードにまとめます。バリデーター (validator) は `execute()` メソッドを提供するシンプルなクラスです。このメソッドはパラメーターとしてフィールドの値を必要とし、値が有効な場合は `true`を、そうでなければ `false` を返します。

symfony はいくつかのバリデーターを搭載しています (この章のあとのほうの"symfonyの標準バリデーター"で説明します) が、今回は `sfStringValidator` を重点的にとり組んでみましょう。このバリデーターは入力が文字列であり、サイズが2つの指定文字数の間に収まっていることをチェックします (`initialize()` メソッドを呼び出すときに定義されます)。`name` フィールドをバリデートするために求められるものがまさにこれです。リスト10-19はバリデーションメソッドでこのバリデーターを使う方法を示しています。

リスト10-19 - 再利用可能なバリデーターでリクエストパラメーターをバリデートする (`modules/contact/action/actions.class.php`)

    [php]
    public function validateSend(sfWebRequest $request)
    {
      $name = $request->getParameter('name');

      // name フィールドが求められる
      if (!$name)
      {
        $this->getRequest()->setError('name', '名前のフィールドは空白であってはなりません');

        return false;
      }

      // name フィールドは2文字から100文字の間のテキストエントリーでなければならない
      $myValidator = new sfStringValidator($this->getContext(), array(
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

ユーザーがリスト10-18のフォームで `name` フィールドの `a` の値を投稿する場合、`sfStringValidator` の `execute()` メソッドは `false` を返します (文字列の最少の長さが2文字だからです)。`validateSend()` メソッドが失敗し、`executeSend()` メソッドの代わりに `handleErrorSend()` メソッドが呼び出されます。
ユーザーがリスト10-18のフォームで `name` フィールドの `a` の値を投稿する場合、`sfStringValidator` の `execute()` メソッドは `false` を返します (文字列の最少の長さが2文字だからです)。`validateSend()` メソッドが失敗し、`executeSend()` メソッドの代わりに `handleErrorSend()` メソッドが呼び出されます。

>**TIP**
>`sfRequest` オブジェクトの `setError()` メソッドはテンプレートに情報を渡すので、テンプレートはエラーメッセージを表示できます (この章の後の「エラーメッセージをフォームに表示する」のセクションで説明されます)。バリデーターはエラーを内部で設定するので、異なるバリデーションを行わない場合のために異なるエラーを定義できます。これが `sfStringValidator` の `min_error` と `max_error` 初期化パラメーターの目的です。

上記の例で定義されたすべてのルールはバリデーターに翻訳できます:

  * `name`: `sfStringValidator` (`min=2`, `max=100`)
  * `email`: `sfStringValidator` (`min=2`, `max=100`)と`sfEmailValidator`
  * `age`: `sfNumberValidator` (`min=0`, `max=120`)

フィールドが求められるという事実はバリデーターによって処理されません。

### バリデーションファイル

`validateSend()` メソッドのバリデーターで問い合わせフォームのバリデーションを簡単に実装できますが、多くのコードを繰り返すことを意味します。symfony はフォームのためにバリデーションルールを定義する YAML を使う代替方法を提供します。たとえば、リスト10-20は`name` フィールドのバリデーションルールの翻訳を示し、その結果はリスト10-19のものと同等です。

リスト10-20 - バリデーションファイル (`modules/contact/validate/send.yml`)

    fields:
      name:
        required:
          msg:       名前のフィールドは空白であってはなりません
        sfStringValidator:
          min:       2
          min_error: この名前は短すぎます。(最小で2文字)
          max:       100
          max_error: この名前は長すぎます。(最大で100文字)

バリデーションファイルにおいて、`fields` ヘッダーは、必要であればバリデートされるフィールド、と値が存在するときにテストされるべきバリデーターの一覧を表示します。それぞれのバリデーターの値はバリデーターを手動で初期化するときに使うものと同じです。フィールドは必要なバリデーターの数だけバリデートできます。

>**NOTE**
>バリデーターが失敗するときはバリデーションプロセスは停止しません。symfony  はすべてのバリデーターをテストし、少なくともそれらの1つが失敗した場合にバリデーションが失敗したことを宣言します。バリデーションファイルのルールのいくつかが失敗した場合でも、symfony はまだ `validateXXX()`メソッドを探しそれを実行します。2つのバリデーションのテクニックはお互いを補い合っています。利点は複数の失敗をともなうフォームにおいて、すべてのエラーメッセージが表示されることです。

バリデーションファイルはモジュールの `validate/`ディレクトリに設置され、これらがバリデートしなければならないアクションの名前を取って名づけられます。たとえばリスト10-19は `validate/send.yml` という名前のファイルに保存されなければなりません。

### フォームを再表示する

デフォルトでは、symfony はバリデーションの処理が失敗するときはいつもアクションクラスの `handleErrorSend()` メソッドを探す、もしくはメソッドが存在しない場合は `sendError.php` テンプレートを表示します。

失敗したバリデーションをユーザーに知らせる通常の方法はエラーメッセージをともなうフォームを再表示することです。この目的のには、リスト10-21で示されるように、`handleErrorSend()` メソッドをオーバーライドし、フォームを表示するアクション (たとえば、`module/index`) へのリダイレクトで終わらせる必要があります。

リスト10-21 - フォームを再表示する (`modules/contact/actions/actions.class.php`)

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
        //フォーム投稿を処理する
      }
    }

フォームを表示してフォーム投稿を処理するために同じアクションを選ぶ場合、リスト10-22で示されるように、`handleErrorSend()` メソッドは `sendSuccess.php` からフォームを再表示するために、単に `sfView::SUCCESS` を返します。

リスト10-22 - フォームを表示して処理する単独のアクション (`modules/contact/actions/actions.class.php`)

    [php]
    class ContactActions extends sfActions
    {
      public function executeSend()
      {
        if (!$this->getRequest()->isMethod('post'))
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

`executeSend()` メソッドと `handleErrorSend()` メソッドのなかで繰り返すことを避けるために、データを用意する必要のあるロジックをアクションクラスの protected として定義されたメソッドにリファクタリングすることができます。

新しい設定によって、ユーザーが無効な名前を入力した場合、フォームは再び表示されますが、入力されたデータは失われ、エラーメッセージは失敗の理由を表示しません。最後の問題を解決するには、誤ったフィールドに近いエラーメッセージを挿入するために、フォームを表示するテンプレートを修正しなければなりません。

### フォームのなかでエラーメッセージを表示する

バリデーターパラメーターとして定義されたエラーメッセージはフィールドがバリデーションを失敗したときにリクエストに追加されます (リスト10-18で示されるように、`setError()`メソッドを用いてエラーを手動で追加できる)。`sfRequest` オブジェクトはエラーメッセージを読みとるために2つの便利なメソッド: `hasError()` と `getError()`を提供します。それぞれのメソッドはパラメーターとしてフィールドの名前を必要とします。加えて、1つもしくは多くのフォールドが無効なデータを含むという事実が注目されるように `hasErrors()` メソッドでアラートをフォームのトップに表示できます。リスト10-23と10-24はこれらのメソッドの使いかたを示しています。

リスト10-23 - フォームのトップでエラーメッセージを表示する (`templates/indexSuccess.php`)

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

リスト10-24 - エラーメッセージをフォーム内部に表示する (`templates/indexSuccess.php`)

    [php]
    <?php echo form_tag('contact/send') ?>
      <?php if ($sf_request->hasError('name')): ?>
        <?php echo $sf_request->getError('name') ?> <br />
      <?php endif; ?>
      Name:    <?php echo input_tag('name') ?><br />
      ...
      <?php echo submit_tag() ?>
    </form>

リスト10-23の `getError()` メソッドを使って条件文を書くのは少し冗長です。`Validation` ヘルパーグループの使用を宣言することを前提とすると、symfony が `getError()` メソッドを置き換えるために `form_error()` ヘルパーを提供する理由はそういうわけです。リスト10-25はこのヘルパーを使うことでリスト10-24を置き換えています。

リスト10-25 - 省略記法でフォーム内部からエラーメッセージを表示する

    [php]
    <?php use_helper('Validation') ?>
    <?php echo form_tag('contact/send') ?>

               <?php echo form_error('name') ?><br />
      名前:    <?php echo input_tag('name') ?><br />
      ...
      <?php echo submit_tag() ?>
    </form>

`form_error()` ヘルパーはメッセージをより見やすくするためにそれぞれのエラーメッセージの前後に特別な文字を追加します。デフォルトでは、文字は下を指し示す矢印 (`&darr;` エンティティに対応)ですが、`settings.yml` ファイルで変更できます:

    all:
      .settings:
        validation_error_prefix:    ' &darr;&nbsp;'
        validation_error_suffix:    ' &nbsp;&darr;'

バリデーションを失敗した場合、フォームはエラーを正しく表示しますが、ユーザーによって入力されたデータは失われます。本当にユーザーフレンドリーにするにはフォームを再投入する必要があります。

### フォームを再投入する

エラー処理は `forward()` メソッドを通して行われるので(リスト10-21で示される)、オリジナルのリクエストはまだアクセス可能で、ユーザーによって入力されたデータはリクエストパラメーターのなかに存在します。リスト10-26で示されるように、デフォルト値をそれぞれのフィールドに追加することでフォームを再投入できます。

リスト10-26 - バリデーションが失敗したときにフォームを再投入するためにデフォルト値を設定する (`templates/indexSuccess.php`)

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

繰り返しますが、これを書く作業はとても退屈です。YAML バリデーションファイルにおいて、symfony は要素のデフォルト値を変更せずにフォームのすべてのフィールドを再投入する代替方法を提供します。フォームに対して `fillin:` 機能を有効にするだけです。構文はリスト10-27で説明されています。

リスト10-27 - バリデーションが失敗したときにフォームを再投入するために `fillin` を有効にする (`validate/send.yml`)

    fillin:
      enabled: true  # フォームの再投入を有効にする
      param:
        name: test  # ページに1つのフォームのみが存在する場合のフォーム名
        skip_fields:   [email]  # これらのフィールドを再投入しない
        exclude_types: [hidden, password] # これらのフィールドタイプを再投入しない
        check_types:   [text, checkbox, radio, password, hidden] # これらを再投入する
        content_type:  html  # htmlは寛容なデフォルト。ほかのオプションは xml と xhtml (xml と同じだが xml の最初の部分はなし)

デフォルトでは、自動の再投入はテキスト入力、チェックボックス、ラジオボタン、テキストエリアと選択コンポーネント(シンプルなものと複数)に対して動作しますが、パスワードもしくは隠しタグは再投入しません。`fillin` 機能はファイルタグに対して動作しません。

>**NOTE**
>`fillin` 機能は XML のレスポンス内容をユーザーに送信するに解析することで機能します。デフォルトでは `fillin` は HTML を出力します。
>
>`fillin` に XHTML を出力させたい場合、`param: content_type: xml` を設定しなければなりません。レスポンスが厳密に有効な XHTML ドキュメントではない場合 `fillin`は機能しません。
>3番目に利用可能な `content_type` は `xhtml` で `xml` と同じですがIE6で奇妙な動作を引き起こすレスポンスからxmlのプロローグをとり除きます。 

ユーザーが入力した値をフォームの入力に書き戻すまえにこれらを変換したい場合があります。リスト10-28で示されるように、`conveters:` キーの下で変換方法を定義すればエスケーピング、URL の書き換え、特別な文字をエンティティに変換する、関数を通して呼び出されたほかの変換などはフォームのフィールドに適用できます。

リスト10-28 - `fillin` のまえに入力を変換する (`validate/send.yml`)

    fillin:
      enabled: true
      param:
        name: test
        converters:         # 適用するコンバータ
          htmlentities:     [first_name, comments]
          htmlspecialchars: [comments]

### symfony の標準バリデーター 

symfony はフォーム用の標準バリデーターをいくつか備えています:

  * `sfStringValidator`
  * `sfNumberValidator`
  * `sfEmailValidator`
  * `sfUrlValidator`
  * `sfRegexValidator`
  * `sfCompareValidator`
  * `sfPropelUniqueValidator`
  * `sfDoctrineUniqueValidator`
  * `sfFileValidator`
  * `sfCallbackValidator`
  * `sfDateValidator`

それぞれのバリデーターはデフォルトのパラメーターとエラーメッセージのセットを持ちますが、
YAML ファイルのなかで、`initialize()` バリデーターメソッドを通して簡単にオーバーライドできます。
つぎのセクションでバリデーターを説明し、使いかたの例を示します。

#### 文字列バリデーター

`sfStringValidator` によって文字に関連する制約をパラメーターに適用できるようになります。

    sfStringValidator:
      values:       [foo, bar]
      values_error: 認められる値は foo と bar のみです
      insensitive:  false  # true の場合、値との比較は大文字と小文字を区別しない
      min:          2
      min_error:    少なくとも2文字入力してください
      max:          100
      max_error:    100文字以下で入力してください

#### 数字バリデーター

`sfNumberValidator` パラメーターが数字であることをバリデートし、これによってサイズの制約を適用することができます。

    sfNumberValidator:
      nan_error:    整数を入力してください
      min:          0
      min_error:    値は最少で0でなければなりません
      max:          100
      max_error:    値は100以下でなければなりません

#### メールバリデーター

`sfEmailValidator` はメールアドレスとして有効なパラメーターが含まれるかをバリデートします。

    sfEmailValidator:
      strict:       true
      email_error:  このメールアドレスは無効です

RFC822 はメールアドレスのフォーマットを定義します。しかしながら、これは一般的に受け入れられるフォーマットよりも寛容です。たとえば、RFC に従えば `me@localhost` は有効なEメールアドレスですが、おそらくは受信したくないでしょう。`strict` パラメーターを `true` にセットするとき、`name@domain.extenison` にマッチするメールアドレスだけが有効です。`false` にセットしするとき、RFC822 がルールとして使われます。

#### URL バリデーター

`sfUrlValidator` はフィールドが正しい URL であるかチェックします。

    sfUrlValidator:
      url_error:    この URL は無効です

#### 正規表現バリデーター

`sfRegexValidator` によって値を Perl と互換性のある正規表現のパターンでチェックできます。

    sfRegexValidator:
      match:        No
      match_error:  URL を含む投稿はスパムと見なされます
      pattern:      /http.*http/si

`match` パラメーターはリクエストパラメーターが有効なパターンにマッチする (値は `Yes`)、もしくは無効なパターン (値は `No`) にマッチするかを決定します。

#### 比較バリデーター

`sfCompareValidator` は2つの異なるリクエストパラメーターを比較します。パスワードをチェックするためにとても便利です。

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

`check` パラメーターは現在のフィールドがマッチしなければならないフィールドの名前を含みます。

デフォルトでは、バリデーターはパラメーターが等しいことをチェックします。`operator` パラメーターを指定することでこのふるまいを変更できます。利用可能な演算子はつぎのとおりです: `>`、`>=`、`<`、`<=`、`==` と `!=`。

#### Propel/Doctrine 独自のバリデーター 

`sfPropelUniqueValidator` はすでにリクエストパラメーターの値がデータベースに存在していないかをバリデートします。ユニークインデックスのためにとても便利です。

    fields:
      nickname:
        sfPropelUniqueValidator:
          class:        User
          column:       login
          unique_error: このログインはすでに存在します。別のものを入力してください。

この例において、バリデーターは `login` カラムがバリデートするフィールドと同じ値を持つ `User` クラスのレコードに対してデータベースを調べます。

>**CAUTION**
>`sfPropelUniqueValidator` は競合条件の影響を受けやすいです。想像しにくいですが、複数ユーザーの環境において、結果は自身が返す瞬間を変えることがあります。重複の `INSERT` エラーを処理する準備もすべきです。

-

>**NOTE**
>プロジェクト用に Doctrine ORM を有効にした場合、上記の Propel 用に説明した同じ方法で
>`sfDoctrineUniqueValidator` を使うこともできます。  

#### ファイルバリデーター

`sfFileValidator` はファイルのアップロードフィールドにフォーマット (mime-type の配列) とサイズの制限を適用します。

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

フィールドに対して `file` 属性は `True` にセットされ、テンプレートはフォームを multipart として宣言しなければならないことを覚えておいてください。

#### コールバックバリデーター

`sfCallbackValidator` はバリデーションをサードパーティのコールバックメソッドもしくはバリデーションを行う関数に委譲します。コールバックメソッドもしくは関数は `true` もしくは `false` を返さなければなりません。

    fields:
      account_number:
        sfCallbackValidator:
          callback:      is_numeric
          invalid_error: 数字を入力してください。
      credit_card_number:
        sfCallbackValidator:
          callback:      [myTools, validateCreditCard]
          invalid_error: 有効なクレジットカードの番号を入力してください。

コールバックメソッドもしくは関数はバリデートされた値を最初のパラメーターとして受けとります。新しい完全なバリデータークラスを作成するよりも、既存のメソッドの機能を再利用するときにとても役立ちます。

>**TIP**
>この章の後のほうの「カスタムバリデーターを作成する」のセクションで説明されている独自のバリデーターを書くこともできます。

#### 日付バリデーター

`sfDateValidator` は投稿された日付が有効かつ/もしくは指定範囲にあることをチェックします。

    sfDateValidator:
      date_error:    You have entered an invalid date
      compare:       "2007-05-01"
      operator:      ">="  #defaults to "==" if not supplied
      compare_error: "Enter a date later than 1st May 2007"

>**TIP**
>この章の後の「カスタムバリデーターを作成する」のセクションで説明されているように、独自バリデーターを書くこともできます。

### 名前つきバリデーター

バリデータークラスと設定を繰り返す必要があることがわかっている場合、これを名前つきバリデーターのもとでまとめることができます。連絡フォームの例において、email フィールドは同じ `sfStringValidator` パラメーターを `name` フィールドとして必要とします。同じ設定を2回繰り返すことを避けるために、名前つきのバリデーターである `myStringValidator` を作成できます。これを行うために、`myStringValidator` ラベルを `validators:` ヘッダーの下に追加し、まとめたい名前つきのバリデーターの詳細な内容を持つ `class` キーと `param` キーをセットします。リスト10-29で示されるように、`fields` セクションの通常の名前つきバリデーターとまったく同じように、名前つきバリデーターを利用できます。

リスト10-29 - バリデーションファイルで名前つきバリデーターを再利用する (`validte/send.yml`)

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
          email_error:  このメールは無効です

### メソッドへのバリデーションを制限する

デフォルトでは、バリデーションファイルに設定されたバリデーターは、アクションが POST メソッドで呼び出されたときに実行されます。リスト10-30で示されるように、異なるメソッドに対して異なるバリデーションをできるようにするために、`methods` キーで別の値を指定することでこの設定をグローバルもしくはフィールド単位でオーバーライドできます。

リスト10-30 - フィールドをテストするときを定義する (`validate/send.yml`)

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
          email_error:  このメールアドレスは無効です

### バリデーションファイルはどのようになるのか？

これまでのところ、バリデーションファイルの断片しか見てきませんでした。すべてを組み立てるとき、バリデーションのルールは YAML で明確な変換を見つけます。リスト10-31はこの章の前の方で定義したすべてのルールに対応するサンプルの問い合わせフォームのための完全なバリデーションファイルを示しています。

リスト10-31 - バリデーションファイルの完全なサンプル

    fillin:
      enabled:      true

    validators:
      myStringValidator:
        class: sfStringValidator
        param:
          min:       2
          min_error: このフィールドは短すぎます (最少で2文字)
          max:       100
          max_error: このフィールドは長すぎます (最大で100文字)

    fields:
      name:
        required:
          msg:       名前のフィールドは空白にはできません
        myStringValidator:
      email:
        required:
          msg:       メールフィールドは空白にはできません
        myStringValidator:
        sfEmailValidator:
          email_error:  このメールアドレスは無効です
      age:
        sfNumberValidator:
          nan_error:    整数を入力してください
          min:          0
          min_error:    "まだあなたは生まれていません。どのようにメッセージを送りたいのですか？"
          max:          120
          max_error:    "ちょっと、おばあちゃん、インターネットをサーフィンするには年をとりすぎていませんか？"
      message:
        required:
          msg:          メッセージフィールドは空白にはできません

複雑なバリデーション
--------------------

バリデーションファイルは多くのニーズを満たしますが、バリデーションがとても複雑なときに十分ではないことがあります。この場合、アクションの `validateXXX()` メソッドに戻るもしくはつぎのセクションであなたの問題の解決方法が見つかります。

### カスタムバリデーターを作成する

それぞれのバリデーターは `sfValidator` クラスを継承するクラスです。symfony に搭載されたバリデータークラスがあなたのニーズに適していない場合、これをオートロードできる `lib/` ディレクトリの任意の場所で、新しいバリデータークラスを作成できます。構文はとてもシンプルです: バリデーターが実行されるとき、バリデーターの `execute()` メソッドが呼び出されます。`initialize()` メソッドでデフォルト設定を定義することもできます。

`execute()` メソッドは最初のパラメーターとしてバリデートする値と2番目のパラメーターとして投げるエラーメッセージを受けとります。両方の値は参照として渡されるので、メソッドの範囲内からエラーメッセージを修正できます。

`initialize()` メソッドはYAMLファイルから Context Singleton とパラメーターの配列を受けとります。最初に親の `sfValidator` クラスの `initalize()` メソッドを呼び出さなければならず、そしてデフォルト値を設定します。

すべてのバリデーターは `$this->getParameterHolder()` によってアクセス可能なパラメーターホルダーを持ちます。

たとえば、文字列がスパムではないことをチェックするために `sfSpamValidator` を作りたい場合、リスト10-32で示されるコードを `sfSpamValidator.class.php` ファイルを追加します。`$value` が文字列の `'http'` を `max_url` 倍多くの含むのかをチェックします。

リスト10-32 - カスタムバリデーターを作成する (`lib/sfSpamValidator.class.php`)

    [php]
    class sfSpamValidator extends sfValidator
    {
      public function execute(&$value, &$error)
      {
        // max_url=2 に対して、正規表現は /http.*http/is
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

バリデーターが `autoloadable` ディレクトリに追加されると同時に (そしてキャッシュがクリアされる)、リスト10-33で示されるように、バリデーションファイルでこれを使うことができます。

リスト10-33 - カスタムバリデーターを使う (`validate/send.yml`)

    fields:
      message:
        required:
          msg:          メッセージフィールドを空白にできません
        sfSpamValidator:
          max_url:      3
          spam_error:   このサイトを立ち去ってください、汚らわしいスパマーさん！

### フォームフィールドに対して配列構文を使う

PHP によってフォームフィールドに対して配列構文を利用できます。独自のフォームを書くとき、もしくは Propel の administration (14章を参照) の生成フォームを使うとき、リスト10-34のような HTML のコードで終わります。

リスト10-34 - 配列構文によるフォーム

    [php]
    <label for="story_title">タイトル:</label>
    <input type="text" name="story[title]" id="story_title" value="default value"
           size="45" />

バリデーションファイルで入力名を(角かっこで)そのまま使うと解析で引き起こされたエラーが投じられます。ここでの解決方法はリスト10-35で示されるように、`fields`セクションにおいて、角かっこ `[]` を波かっこ `{}` で置き換えることで、symfony はあとでバリデーターに送られる名前の変換を考慮するようになります。

リスト10-35 - 配列構文を使ったフォームのためのバリデーションファイル

    fields:
      story{title}:
        required:     Yes

### 空のフィールド上でバリデーターを実行する

空の値の上で、必要ないフィールド上で、バリデーターを実行することが必要な場合があります。たとえば、ユーザーがパスワードを変更したい場合 (したくない場合) にこの事例があてはまります。この場合、確認パスワードを入力しなければなりません。リスト10-36で例をご覧ください。

リスト10-36 - 2つのパスワードフィールドを持つフォーム用のバリデーションファイルのサンプル

    fields:
      password1:
      password2:
        sfCompareValidator:
          check:         password1
          compare_error: 2つのパスワードが一致しません

バリデーションの処理はつぎのように実行されます:

  * `password1` `== null` と `password2 == null` の場合:

    * `required` テストは通ります。
    * バリデーターは実行されません。
    * フォームは有効です。

  * `password2 == null` である一方で `password1` が `null` ではない場合:

    * `required` テストは通ります。
    * バリデーターは実行されません。
    * フォームは有効です。

`password1` が `not null` であるかどうか `password2` バリデーターを実行したい場合があります。幸いにも、`group` パラメーターのおかげで symfony バリデーターはこの事例に対処します。フィールドがグループのなかに存在する場合、それが空ではなく同じグループのフィールドの1つが空ではないかバリデーターを実行します。

ですので、リスト10-36で示されているように設定を変更する場合、バリデーションプロセスは正しくふるまいます。

リスト10-36 - 2つのパスワードフィールドとグループによるフォーム用のバリデーションファイルのサンプル

    fields:
      password1:
        group:           password_group
      password2:
        group:           password_group
        sfCompareValidator:
          check:         password1
          compare_error: 2つのパスワードが一致しません

バリデーションの処理はつぎのように実行します:

  * `password1 == null` かつ `password2 == null` の場合:

    * `required` のテストは通ります。
    * バリデーターは実行されません。
    * フォームは有効です。

  * `password1 == null` かつ `password2 == 'foo'` の場合:

    * `required` テストは合格します。
    * `password2` は `not null` なので、バリデーターは実行され、失敗します。
    * エラーメッセージが `password2` に投げられます。

  * `password1 == 'foo'` で `password2 == null` の場合:

    * `required` テストは合格します。
    * `password1` が `not null` なので同じグループにある `password2` のためのバリデーターが実行され、失敗します。
    * エラーメッセージが `password2` に投げられます。

  * `password1 == 'foo'` で `password2 == 'foo'` の場合:

    * `required` テストは合格します。
    * `password2` が `not null` なので、バリデーターが実行され、合格します。
    * フォームは有効です。

まとめ
----

symfony のテンプレートのなかでフォーム (form) を書く作業は標準のフォームヘルパーとスマートなオプションによって円滑になります。オブジェクトのプロパティを編集するためにフォームを設計するとき、バリデーションファイル (validation file)、バリデーションヘルパーと再投入機能は頑強でユーザーフレンドリーなサーバーのフィールド値コントロール機能を開発するために必要な作業量を減らします。そして、アクションクラスのなかでカスタムバリデーターを書くもしくは `validateXXX()` メソッドを作ることで、より複雑なバリデーションのニーズに対応することができます。
