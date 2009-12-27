第11章 - Ajaxの統合
===================

クライアントサイド上でのインタラクション、複雑な視覚効果(イフェクト)、非同期通信はWeb 2.0のアプリケーションにおいて共通の機能です。JavaScriptを必要とするこれらの機能の実装に関して、コードを手書きするのはやっかいでデバッグに時間がかかることはよくあります。幸いにして、symfonyはヘルパーの完全なセットを持つテンプレート内部のJavaScriptの多くの共通部分を自動化します。JavaScriptのコードを書かずに多くのクライアントサイドのふるまいを実現できます。開発者は実現したいイフェクトだけに集中していればよく、symfonyが複雑な構文と互換性問題を処理します。

この章ではクライアントサイドのスクリプト作成を円滑にするためにsymfonyが提供するツールについて説明します:

  * 基本的なJavaScriptヘルパーは、DOM(Document Object Model)要素を更新するもしくはリンクでスクリプトを起動させるために標準規格に準拠している`<script>`タグを出力します。
  * Prototypeはsymfonyに統合されたJavaScriptライブラリです。これはJavaScriptコアに新しい関数とメソッドを追加することでクライアントサイドのスクリプト開発を加速します。
  * Ajaxヘルパーによってユーザーはリンクをクリックする、フォームを投稿する、もしくはフォーム要素を修正することでページのいくつかの部分を更新できるようになります。
  * これらのヘルパーの多くのオプションは、とりわけコールバック関数を利用することで、よりすばらしい柔軟性とパワーも提供します。
  * script.aculo.usもsymfonyに統合されているもう一つのJavaScriptライブラリです。script.aculo.usはインターフェイスとユーザーエクスペリエンスを強化する動的な視覚効果を追加します。
  * JSON(JavaScript Object Notation)はサーバーとクライアントスクリプトのあいだでコミュニケーションを行うために使われる標準規格です。
  * 前述のすべての要素を結びつけるクライアントサイドのインタラクションはsymfonyのアプリケーションで実現可能です。オートコンプリート(自動入力補完)、ドラッグドロップ、ソート可能なリスト、編集可能なテキストはPHPコードの1行、symfonyのヘルパーの呼び出しですべて実装できます。

基本的なJavaScriptヘルパー
--------------------------

JavaScriptはクロスブラウザーの互換性が欠如していたため、プロフェッショナルなWebアプリケーションで実際に使うものはほとんどないと長い間見なされてきました。今日において、互換性の問題は(ほとんど)解決され、いくつかの頑強なライブラリによって、膨大な行数のコード作成と膨大な時間のデバッグ作業を行わなくてもJavaScriptで複雑なインタラクションをプログラミングできます。もっとも人気のある先進技術はAjax(asynchronous JavaScript and XML)と呼ばれます。これはこの章の"Ajaxヘルパー"のセクションで説明します。

逆説的にも、この章ではjavaScriptのコードはほんのわずかしか見ません。symfonyがクライアントサイドのスクリプティングへの独自の方法を持つからです: symfonyはJavaScriptのふるまいをヘルパーにまとめて抽象化するので、テンプレートはJavaScriptのコードをまったく表示せずに終わります。ふるまいをページの要素に追加するにはPHPコードが1行必要ですが、このヘルパー呼び出しがJavaScriptのコード出力を行います。生成されたレスポンスを検査すると、すべてがカプセル化された複雑性を持つことがあきらかになります。ヘルパーが、ブラウザーの一貫性、複雑な制限のある事例、拡張性などを対処するので、これらが含むJavaScriptコードの総量が極めて重要になる可能性があります。それゆえ、この章はJavaScriptのイフェクトを実現するためにJavaScriptを使わないやりかたをお教えします。

`Javascript`ヘルパーグループの使用を宣言しているのであれば、ここで説明されるすべてのヘルパーは、テンプレートのなかで利用できます。

    [php]
    <?php use_helper('Javascript') ?>

すぐに学ぶように、これらのヘルパーのなかにはHTMLコードやJavaScriptコードを出力するものがあります。

### テンプレートのなかのJavaScript

XHTMLにおいて、JavaScriptコードブロックはCDATA宣言で囲まなければなりません。しかし、複数のJavaScriptコードブロックを必要とするページを書く作業はすぐに退屈になります。symfonyが文字列をXHTML準拠の`<script>`タグに変換する`javascript_tag()`ヘルパーを提供するのはそういうわけです。リスト11-1はこのヘルパーの使いかたのお手本を示しています。

リスト11-1 - `javascript_tag()`ヘルパーによるJavaScriptの挿入

    [php]
    <?php echo javascript_tag("
      function foobar()
      {
      ...
      }
    ") ?>
     => <script type="text/javascript">
        //<![CDATA[
          function foobar()
          {
            ...
          }
        //]]>
        </script>

しかし、コードブロックよりも、JavaScriptのもっとも共通の使いかたは、特定のスクリプトを起動させるハイパーリンクにあります。リスト11-2で示されるように、`link_to_function()`ヘルパーはまさにこれを行います。

リスト11-2 - `link_to_function()`ヘルパーでJavaScriptのリンクを発動させる

    [php]
    <?php echo link_to_function('クリックしてください！', "alert('foobar')") ?>
     => <a href="#" onClick="alert('foobar'); return none;">クリックしてください！</a>

`link_to()`ヘルパーと同じように、3番目の引数で`<a>`タグにオプションを追加できます。

>**NOTE**
>`link_to()`ヘルパーが`button_to()`の兄弟を持つのと同様に、`button_to_function()`ヘルパーを呼び出すことでボタン(`<input type="button">`)からJavaScriptを発動させることができます。そしてクリック可能なイメージが望ましい場合、`link_to_function(image_tag('myimage'), "alert('foobar')")`を呼び出します。

### DOM要素を更新する

動的なインターフェイスにおける共通タスクの1つはページ要素の更新です。通常書くコードはリスト11-3で示されるようなものです。

リスト11-3 - JavaScript要素を更新する

    [php]
    <div id="indicator">データ処理の開始</div>
    <?php echo javascript_tag("
      document.getElementById("indicator").innerHTML =
        "<strong>データ処理の完了</strong>";
    ") ?>

この目的のために、symfonyはHTMLではなくJavaScriptを生み出すヘルパーを提供します。このヘルパーは`update_element_function()`と呼ばれます。リスト11-4は使いかたを示しています。

リスト11-4 - JavaScript要素を`update_element_function()`ヘルパーで更新する

    [php]
    <div id="indicator">データ処理の開始</div>
    <?php echo javascript_tag(
      update_element_function('indicator', array(
        'content'  => "<strong>データ処理の終了</strong>",
      ))
    ) ?>

少なくとも実際のJavaScriptコードと同じぐらいの長さがあるので、なぜこのヘルパーがとりわけ便利なのかに戸惑っているかもしれません。本当の問題は読みやすさです。たとえば、要素の前あとで内容を挿入したい場合、要素を更新する代わりに除去したい場合、もしくはある種の条件にしたがって何も行いたくない場合があります。そのような場合において、JavaScriptコードはやや散乱しますが、リスト11-5で示されるように、`update_element_function()`はテンプレートをとても読みやすいように維持します。

リスト11-5 - `update_element_function()`ヘルパーのオプション

    [php]
    // 'indicator'要素の直後に内容を差し込む
    update_element_function('indicator', array(
      'position' => 'after',
      'content'  => "<strong>データの章が完了</strong>",
    ));

    // $conditionがtrueの場合のみ、'indicator'の前の要素を除去する
    update_element_function('indicator', array(
      'action'   => $condition ? 'remove' : 'empty',
      'position' => 'before',
    ))

ヘルパーによってテンプレートはJavaScriptコードよりも理解しやすくなります。そして似たようなふるまいに対して単独の構文があります。これはヘルパーの名前がとても長い理由でもあります: コメントを追加しなくても、コードの名前そのものが説明になります。

### グレイスフルデグラデーション(Graceful Degradation)

`<noscript>`タグによってJavaScriptをサポートしないブラウザーが表示するHTMLコードを指定できます。symfonyはまったく逆のことを行うヘルパーでこれを補完します: symfonyがコードを保証するので、JavaScriptを実際にサポートするブラウザーだけがコードを実行します。リスト11-6で示されるように、`if_javascript()`と`end_if_javascript()`ヘルパーはグレースフルデグラデーション(graceful degradation)をサポートする(訳注：フォールトトレラントシステムとも言い換えできる)アプリケーションの作成を円滑にします。

リスト11-6 - グレースフルデグラデーションを可能にする`if_javascript()`ヘルパーを使う

    [php]
    <?php if_javascript(); ?>
      <p>JavaScriptは有効です。</p>
    <?php end_if_javascript(); ?>

    <noscript>
      <p>JavaScriptは有効ではありません。</p>
    </noscript>

>**NOTE**
>`if_javascript()`と`end_if_javascript()`ヘルパーを呼び出すときに`echo`を含む必要はありません。

Prototype
---------

Prototypeは偉大なJavaScriptライブラリです。クライアントスクリプト言語の可能性を広げ、いつも夢見ているけれど見つからない関数を追加し、DOMを操作する新しいメカニズムを提供します。プロジェクトのWebサイトは [http://prototypejs.org/](http://prototypejs.org/)です。

Prototypeのファイルはsymfonyのフレームワークに搭載されており、symfonyの新規プロジェクトの`web/sf/prototype`ディレクトリでアクセスできます。このことはアクションにつぎのコードを追加することでPrototypeを利用できるようになることを意味します:

    [php]
    $prototypeDir = sfConfig::get('sf_prototype_web_dir');
    $this->getResponse()->addJavascript($prototypeDir.'/js/prototype');

もしくは`view.yml`ファイルにつぎのコードを追加することでも利用できます:

    all:
      javascripts: [%SF_PROTOTYPE_WEB_DIR%/js/prototype]

>**NOTE**
>つぎのセクションで説明するsymfonyのAjaxヘルパーはPrototypeに依存しているので、Prototypeライブラリの1つを使い始めると同時にPrototypeは自動的にインクルードされます。これはテンプレートが`_remote`ヘルパーを呼び出す場合、PrototypeのJavaScriptをレスポンスに手動で追加する必要がないことを意味します。

いったんPrototypeライブラリがロードされると、JavaScriptコアに追加されるすべての新しい関数が使えるようになります。この本の目的はこれらすべてを説明することではないのですが、つぎのWebサイトでPrototypeのよいドキュメントが見つかります:

  * Particletree: [http://particletree.com/features/quick-guide-to-prototype/](http://particletree.com/features/quick-guide-to-prototype/)
  * Sergio Pereira: [http://www.sergiopereira.com/articles/prototype.js.html](http://www.sergiopereira.com/articles/prototype.js.html)
  * Script.aculo.us: [http://wiki.script.aculo.us/scriptaculous/show/Prototype](http://wiki.script.aculo.us/scriptaculous/show/Prototype)

JavaScriptに追加されるPrototype関数の1つはドル関数の`$()`です。基本的に、この関数は`document.getElementById()`のシンプルなショートカットですが、もう少し強力です。この使いかたの例に関してはリスト11-7をご覧ください。

リスト11-7 - JavaScriptでIDによって要素を取得する`$()`関数を使う

    [php]
    node = $('elementID');

    // つぎのコードと同じ
    node = document.getElementById('elementID');

    // 一度に複数の要素を取得することも可能で
    // この場合、結果はDOM要素の配列である
    nodes = $('firstDiv', 'secondDiv');

PrototypeはJavaScriptコアが本当に欠如する関数を提供します。たとえば、引数として渡されるクラスを持つすべてのDOM要素の配列を返す関数です:

    [php]
    nodes = document.getElementByClassName('myclass');

しかしながら、これはほとんど使わないでしょう。Prototypeが二重ドル、`$$()`と呼ばれるもっと強力な関数を提供するからです。この関数はCSSセレクタに基づいてDOM要素の配列を返します。ですので以前の呼び出しはつぎのようにも書けます:

    [php]
    nodes = $$('.myclass');

CSSセレクタの力のおかげで、クラスとIDでDOMを解析できるので、親子関係と前後関係が以前のXPathの表記で行うよりもはるかに簡単です。これらすべてを結びつける複雑なセレクタで要素にアクセスすることもできます:

    [php]
    nodes = $$('body div#main ul li.last img > span.legend');

Prototypeによって提供された構文強化の最後の例はeach配列イテレータです。PHPと同等の簡潔さを提供します。JavaScriptに匿名関数とクロージャを定義する機能を追加します。JavaScriptを手作業で実装するさいに多く使います。

    [php]
    var vegetables = ['Carrots', 'Lettuce', 'Garlic'];
    vegetables.each(function(food) { alert('I love ' + food); });

Prototypeを使ったJavaScriptプログラミングは手作業で行うよりはるかに面白く、このライブラリはsymfonyの一部でもあるので、関連ドキュメントを読むのに数分しかかかりません。

Ajaxヘルパー
------------

ページ要素を更新したい場合、リスト11-5のようにJavaScriptではなくサーバーによって実行されるPHPスクリプトで行う場合にはどうしたらよいでしょうか？これによってサーバーのレスポンスにしたがってページの一部を変更する機会が提供されます。リスト11-8で示されるように、`remote_function()`ヘルパーはまさにこれを行います。

リスト11-8 - `remote_function()`ヘルパーを使う

    [php]
    <div id="myzone"></div>
    <?php echo javascript_tag(
      remote_function(array(
        'update'  => 'myzone',
        'url'     => 'mymodule/myaction',
      ))
    ) ?>

>**NOTE**
>通常の`url_for()`のように、`url`パラメーターは内部URI(`module/action?key1=value1&...`)もしくはルーティングルール名のどちらかを格納します。

呼び出されたとき、このスクリプトは`mymodule/myaction`アクションのリクエストのレスポンスによってidがmyzoneである要素を更新します。この種のインタラクションはAjaxと呼ばれ、非常にインタラクティブなWebアプリケーションのなか心です。Wikipedia([http://en.wikipedia.org/wiki/AJAX](http://en.wikipedia.org/wiki/AJAX))ではつぎのように説明されています:

舞台裏のサーバーで小さな量のデータを交換することで、AjaxはWebページをより応答性の高いものにするのでユーザーが変更をするたびにWebページ全体をリロードする必要がありません。これはWebページのインタラクティビティ、スピードとユーザービリティを増大させることを意味します。

Ajaxは`XMLHttpRequest`に依存します。`XMLHttpRequest`は`hidden`フレームのように振る舞うJavaScriptオブジェクトで、サーバーリクエストとWebページの残りの部分の操作からこのオブジェクトを更新できます。このオブジェクトはとても低いレベルで、異なるブラウザーが異なる方法で処理するので、通常はAjaxリクエストの処理を手作業で行うことは長い部分のコードを書くことを意味します。幸いにして、PrototypeはAjaxを処理するすべてのコードをカプセル化し、よりシンプルなAjaxオブジェクトを提供するので、symfonyはこのオブジェクトを頼りにします。いったんAjaxヘルパーをテンプレートのなかで使うとPrototypeライブラリが自動的にロードされるのはそういうわけです。

>**CAUTION**
>リモートアクションのURLが現在のページが同じドメインに所属しない場合Ajaxヘルパーは機能しません。この制限はセキュリティのために存在し、回避できないブラウザーの制限に依存します。

Ajaxインタラクションは3つの部分から構成されます: 呼び出し元(リンク、ボタン、フォーム、もしくはアクションを立ち上げるためにユーザーが操作するコントロール機能)、サーバーアクション、アクションのレスポンスを表示するページの領域です。リモートアクションがクライアントサイド上のJavaScript関数によって処理されるデータを返す場合、もっと複雑なインタラクションを開発できます。symfonyはテンプレートにAjaxインタラクションを挿入するために複数のヘルパーを提供します。これらすべての名前にはremoteという単語が含まれます。これらは共通の構文も共有します。構文はすべてのAjaxパラメーターを持つ連想配列です。AjaxヘルパーはJavaScriptではなくHTMLコードを出力することに注意してください。

>**SIDEBAR**
>**Aaxアクションはいかが？**
>
>リモート関数と呼ばれるアクションは通常のアクションです。これらはルーティングに従い、`return`をともなうレスポンスをレンダリングするためにビューを決定し、変数をテンプレートに渡し、ほかのアクションと同様にモデルを変更することができます。
>
>しかしながら、つぎのようにAjaxを通して呼び出されたとき、アクションは`true`を返します:
>
>     [php]
>     $isAjax = $this->getRequest()->isXmlHttpRequest();
>
>symfonyはアクションがAjaxコンテキストのなかにあることを知っており、それに対応してレスポンスを処理できます。そのために、デフォルトで、開発環境ではAjaxアクションはWebデバッグツールバーを含みません。また、これらはデコレーションプロセスをスキップします(これらのテンプレートはデフォルトでレイアウトに含まれません)。デコレートされたAjaxビューが欲しい場合、`view.yml`モジュールのファイルのなかでこのビューに対してhas_layout: true`を明示的に指定する必要があります。
>
>Ajaxインタラクションにおいて応答性は重大なので、レスポンスがあまり複雑ではない場合、ビューを作成することは避け、代わりにアクションから直接レスポンスを返すのはよいアイディアかもしれません。ですのでテンプレートをスキップしてAjaxリクエストを加速するためにアクションのなかで`renderText()`メソッドを使うことができます。
>
>**symfony 1.1の新しい機能**: たいていのAjaxアクションは部分テンプレートを単にインクルードするテンプレートになります。Ajaxレスポンスのコードは初期ページを表示するためにすでに使われているからです。一行コードだけのテンプレートを作ることを回避するために、アクションは`renderPartial()`メソッドを使うことができます。このメソッドは部分テンプレートの再利用性、それらのキャッシュ機能、と`renderText()`メソッドの速さを活用します。
>
>     [php]
>     public function executeMyAction()
>     {
>       // 何かを行う
>       return $this->renderPartial('mymodule/mypartial');
>     }
>

### Ajaxリンク

AjaxリンクはWeb 2.0のアプリケーションで利用できるAjaxインタラクションを広く共有します。`link_to_remote()`ヘルパーは、予想どおり、リモート機能を呼び出すリンクを出力します。リスト11-9で示されるように構文は`link_to()`とよく似ています(2番目のパラメーターがAjaxオプションの連想配列であること以外)。

リスト11-9 - `link_to_remote()`ヘルパーによるAjaxリンク

    [php]
    <div id="feedback"></div>
    <?php echo link_to_remote('この投稿を削除する', array(
        'update' => 'feedback',
        'url'    => 'post/delete?id='.$post->getId(),
    )) ?>

この例では、`'この投稿を削除する'`のリンクをクリックするとバックグランドで`post/delete`アクションの呼び出しが行われます。サーバーによって変えされるレスポンスは`id`が`feedback`である要素のなかに現れます。このプロセスは図11-1で説明されます。

図11-1 - ハイパーリンクでリモート更新を実行する

![ハイパーリンクでリモート更新を実行する](http://github.com/masakielastic/symfonybook-ja/raw/master/images/F1101.png "ハイパーリンクでリモート更新を実行する")

リスト11-10で示されるように、文字列の代わりに画像を使い、`module/action`の内部URLの代わりにルール名を使い、オプションを3番目の引数の`<a>`タグに追加することができます。

リスト11-10 - `link_to_remote()`ヘルパーのオプション

    [php]
    <div id="emails"></div>
    <?php echo link_to_remote(image_tag('refresh'), array(
        'update' => 'emails',
        'url'    => '@list_emails',
    ), array(
        'class'  => 'ajax_link',
    )) ?>

### Ajax駆動のフォーム

Webフォームはよく別のアクションを呼び出しますが、これによってページ全体がリフレッシュされます。このフォームに対して`link_to_function()`が対応するものはサーバーのレスポンスをともなうページ要素だけを更新するフォーム投稿になります。これは`form_remote_tag()`ヘルパーが行うことで、構文はリスト11-11で示されます。

リスト11-11 - `form_remote_tag()`ヘルパーをともなうAjaxフォーム

    [php]
    <div id="item_list"></div>
    <?php echo form_remote_tag(array(
        'update'   => 'item_list',
        'url'      => 'item/add',
    )) ?>
      <label for="item">Item:</label>
      <?php echo input_tag('item') ?>
      <?php echo submit_tag('Add') ?>
    </form>

`form_remote_tag()`は通常の`form_tag()`ヘルパーのように`<form>`を開きます。このフォームを投稿することで、`item`フィールドをリクエストパラメーターとして、バックグラウンドで`item/add`アクションにPOSTリクエストを行います。リスト11-2で描かれているように、レスポンスは`item_list`要素の内容を置き換えます。通常の`</form>`閉じタグでAjaxフォームを閉じてください。

図11-2 - リモート更新をフォームで実行する

![フォームでリモート更新を実行する](http://github.com/masakielastic/symfonybook-ja/raw/master/images/F1102.png "フォームでリモート更新を実行する")

>**CAUTION**
>Ajaxフォームはmultipartになることはできません。これは`XMLHttpRequest`オブジェクトの制限です。このことはAjaxフォームを通してファイルのアップロードを処理できないことを意味します。しかしながら、次善策があります。たとえば、`XMLHttpRequest`の代わりに隠し`iframe`を使うことです。

ページモードとAjaxモードの両方でフォームが機能するようにしたい場合、最善の解決方法はフォームを通常のものと同じように定義することです。しかしながらそのためには、通常の投稿ボタンに加えて、Ajaxでフォームを投稿する2番目のボタン(`<input type="button"/>`)が必要です。symfonyはこのボタンを`submit_to_remote()`と呼びます。これはグレースルフルデグラデーションを行うAjaxインタラクションを開発するための助けになります。リスト11-12で例をご覧ください。

リスト11-12 - 通常の投稿とAjaxの投稿によるフォーム

    [php]
    <div id="item_list"></div>
    <?php echo form_tag('@item_add_regular') ?>
      <label for="item">Item:</label>
      <?php echo input_tag('item') ?>
      <?php if_javascript(); ?>
        <?php echo submit_to_remote('ajax_submit', 'Add in Ajax', array(
            'update'   => 'item_list',
            'url'      => '@item_add',
        )) ?>
      <?php end_if_javascript(); ?>
      <noscript>
        <?php echo submit_tag('Add') ?>
      </noscript>
    </form>

リモート投稿タグと通常の投稿タグを結びつけた使いかたの別の例は記事を編集するフォームです。これはAjaxのプレビューボタンと通常の投稿を行う投稿ボタンを提供します。

>**NOTE**
>ユーザーがEnterキーを押すとき、フォームはメインの`<form>`タグで定義されたアクションを使って投稿されます。この例では通常のアクションです。

モダンなフォームは投稿されたときだけでなく、フィールドの値がユーザーによって更新されたときも反応します。そのためにsymfonyでは`observe_field()`ヘルパーを使うことができます。リスト11-13は提案機能を開発するためにこのヘルパーを使う例を示しています: `item`フィールドに文字を入力するたびそれをトリガーとしてAjaxが呼び出されitem_suggestion要素が更新されます。

リスト11-13 - フィールドの値を`observe_field()`で変更するときにリモート関数を呼び出す

    [php]
    <?php echo form_tag('@item_add_regular') ?>
      <label for="item">Item:</label>
      <?php echo input_tag('item') ?>
      <div id="item_suggestion"></div>
      <?php echo observe_field('item', array(
          'update'   => 'item_suggestion',
          'url'      => '@item_being_typed',
      )) ?>
      <?php echo submit_tag('Add') ?>
    </form>

`@item_being_typed`ルールで書かれたモジュール/アクションはフォームを投稿しなくてもユーザーがobservedフィールド(`item`)の値を変更するたびに呼び出されます。アクションは`value`リクエストパラメーターから`item`の現在の値を取得することができるようになります。observedフィールド以外のほかのフィールドの値を渡したい場合、`with`パラメーターのなかでそのフィールドをJavaScriptの式として記述できます。たとえば、アクションが`param`パラメーターを受けとるようにしたい場合、リスト11-14で示されるような`observe_field()`ヘルパーを書きます。

リスト11-14 - `with`オプションでリモートアクションに独自のパラメーターを渡す

    [php]
    <?php echo observe_field('item', array(
        'update'   => 'item_suggestion',
        'url'      => '@item_being_typed',
        'with'     => "'param=' + value",
    )) ?>

このヘルパーはHTML要素の出力を行わず、代わりにパラメーターとして渡された要素のためのふるまいを出力します。この章の後のほうでふるまいを割り当てるJavaScriptヘルパーの例をより多く見ることになります。

フォームのフィールドのすべてを観測したい場合、`observe_form`ヘルパーを使うべきです。フォームフィールドの1つが修正されるたびにこのヘルパーはリモート関数を呼び出します。

### 定期的にリモート関数を呼び出す

大事なことを言い忘れましたが、`periodically_call_remote()`ヘルパーは数秒毎に実行されるAjaxインタラクションです。これはHTMLコントロール機能をともないませんが、ページ全体のふるまいとしてバックグランドで透過的に実行されます。これはマウスの位置を追跡する、ラージテキストエリアの内容を自動保存するなどのために非常に便利です。リスト11-15はこのヘルパーの使いかたの例を示しています。

リスト11-15 - `periodically_call_remote()`で周期的にリモート関数を呼び出す

    [php]
    <div id="notification"></div>
    <?php echo periodically_call_remote(array(
        'frequency' => 60,
        'update'    => 'notification',
        'url'       => '@watch',
        'with'      => "'param=' + \$F('mycontent')",
    )) ?>

2つのリモート関数呼び出しの間を待つ秒数(`frequency`)を指定しない場合、デフォルト値の10秒が使われます。`with`パラメーターはJavaScriptで評価されるので、`$F()`関数などのPrototype関数を使うことができます。

リモートコールパラメーター
------------------------

以前のセクションで説明されたすべてのAjaxヘルパーは、`update`パラメーターと`url`パラメーターに加えて、ほかの値を受けとることができます。Ajaxパラメーターの連想配列はリモートコールとそれらのレスポンス処理のふるまいを変更し調整することができます。

### レスポンス状態にしたがって異なる要素を更新する

リモートアクションが失敗した場合、リモートヘルパーは成功したレスポンスによって更新された要素以外の別の要素の更新を選択できます。この目的のために、`update`パラメーターの値を連想配列に分割し、`success`と`failure`の場合に更新する要素に対して異なる値を設定します。たとえば1つのページと1つのエラーのフィードバックの領域内にたくさんのAjaxインタラクションが存在する場合、これは大いに役立ちます。リスト11-16は条件つきの更新の扱いかたのお手本を示しています。

リスト11-16 - 条件つきの更新を行う

    [php]
    <div id="error"></div>
    <div id="feedback"></div>
    <p>Hello, World!</p>
    <?php echo link_to_remote('この投稿を削除する', array(
        'update'   => array('success' => 'feedback', 'failure' => 'error'),
        'url'      => 'post/delete?id='.$post->getId(),
    )) ?>

>**TIP**
>HTTPエラーコード(500、404と2XXの範囲にないすべてのコード)は、`sfView::ERROR`を返すアクションではなく、failureの場合の更新を実行します。Ajaxがエラーになったことを伝えるアクションを作りたい場合、アクションは`$this->getResponse()->setStatusCode(404)`もしくはそれに類するものを呼び出さなければなりません。

### 位置にしたがって要素を更新する

`update_element_function()`ヘルパーと同様に、`position`パラメーターを追加することで特定の要素に更新する要素を相対的なものとして指定できます。リスト11-17は例を示しています。

リスト11-17 - レスポンスの位置を変更するために`position`パラメーターを使う

    [php]
    <div id="feedback"></div>
    <p>Hello, World!</p>
    <?php echo link_to_remote('この投稿を削除する', array(
        'update'   => 'feedback',
        'url'      => 'post/delete?id='.$post->getId(),
        'position' => 'after',
    )) ?>

これは`feedback`要素の後にAjax呼び出しのレスポンスを挿入することができます; すなわち、`<div>`と`<p>`のあいだです。このメソッドによって、いくつかのAjax呼び出しを行い、`update`要素のあとで累積するレスポンスを見ることができます。

`position`パラメーターはつぎの値を受けとることができます:

  * `before`: 要素の前
  * `after`: 要素の後
  * `top`: 要素の内容の一番上
  * `bottom`: 要素の内容の一番下

### 条件にしたがって要素を更新する

リスト11-18で示されるように、リモートコールは、`XMLHttpRequest`を実際に投稿するまえに、ユーザーが確認できるようにする追加パラメーターを受けとることができます。

リスト11-18 - リモート関数を呼び出すまえに確認を求める`confirm`パラメーターを使う

    [php]
    <div id="feedback"></div>
    <?php echo link_to_remote('この投稿を削除する', array(
        'update'   => 'feedback',
        'url'      => 'post/delete?id='.$post->getId(),
        'confirm'  => 'よろしいですか？',
    )) ?>

"よろしいですか？"を示すJavaScriptのダイアログボックスはユーザーがリンクをクリックしたときにポップアップを行い、`post/delete`アクションはユーザーがOKをクリックして選択を確認する場合だけに呼び出されます。

リスト11-19で示されるように、`condition`パラメーターを設定した場合、条件つきでリモートコールがブラウザーサイド(JavaScript)で実行されるようにすることも可能です。

リスト11-19 - クライアントサイドのテストにしたがって条件つきでリモート関数を呼び出す

    [php]
    <div id="feedback"></div>
    <?php echo link_to_remote('この投稿を削除する', array(
        'update'    => 'feedback',
        'url'       => 'post/delete?id='.$post->getId(),
        'condition' => "$('elementID') == true",
    )) ?>

### Ajaxリクエストメソッドを決定する

デフォルトでは、AjaxリクエストはPOSTメソッドで構成されます。データを修正しないAjax呼び出しを行いたい場合、もしくはAjax呼び出しの結果として組み込みのバリデーションを持つフォームを表示したい場合、GETするAjaxリクエストメソッドを変更することが必要な場合があります。リスト11-20で示されるように、メソッドオプションはAjaxリクエストを変更します。

リスト11-20 - Ajaxリクエストメソッドを変更する

    [php]
    <div id="feedback"></div>
    <?php echo link_to_remote('この投稿を削除する', array(
        'update'    => 'feedback',
        'url'       => 'post/delete?id='.$post->getId(),
        'method'    => 'get',
    )) ?>

### スクリプトの実行を許可する

Ajax呼び出しのレスポンスコード(`update`要素に挿入され、サーバーによって送られてきたコード)がJavaScriptを含む場合、デフォルトでこれらのスクリプトが実行されないことに驚くかもしれません。これはリモート攻撃のリスクを減らし開発者がレスポンスのなかに存在するコードを確実に知っているときのみスクリプトの実行を可能にすることが目的です。

リモートレスポンスのなかでスクリプト実行を行う機能を`script`オプションによって明確に宣言する必要があるのはそういうわけです。リスト11-21はリモートレスポンスから実行できるJavaScriptを宣言するAjax呼び出しの例を示しています。

リスト11-21 - Ajaxのレスポンスでスクリプトの実行を許可する

    [php]
    <div id="feedback"></div>
    <?php
      // post/deleteアクションのレスポンスがJavaScriptを含む場合、
      // ブラウザーで実行されることを許可する
      echo link_to_remote('この投稿を削除する', array(
        'update' => 'feedback',
        'url'    => 'post/delete?id='.$post->getId(),
        'script' => true,
    )) ?>

リモートテンプレートが(`remote_function()`といった)Ajaxヘルパーを格納する場合、これらのPHP関数はJavaScriptコードを生成し、`'script' => true`オプションを追加しないかぎりこれらは実行されないことに注意してください。

>**NOTE**
>たとえリモートレスポンスに対してスクリプトの実行を有効にしたとしても、生成コードをチェックするコードをリモートコードスクリプトが実際に見るわけではありません。スクリプトは実行されますがコードには現れません、奇妙なことかもしれませんが、この動作は全く持って正常なものです。

### コールバック機能を作成する

Ajaxインタラクションの重大な欠点は更新領域が実際に更新されるまでユーザーには見えないことです。遅いネットワーク、もしくはサーバー障害の場合、実際は処理されなかったにもかかわらず、ユーザーは自分のアクションが反映されたと信じるかもしれません。Ajaxインタラクションでユーザーのイベントを通知することが重要であるのはそういうわけです。

デフォルトでは、さまざまなJavaScriptコールバックが(進行インディケータなどに対して)起動される間それぞれのリモートリクエストは非同期プロセスです。すべてのコールバックは`request`オブジェクトにアクセスできます。`request`オブジェクトは`XMLHtppRequest`メソッドを持ちます。コールバックはAjaxインタラクションのイベントに対応します:

  * `before`: リクエストが初期化される前
  * `after`: リクエストが初期化された直あとでロードされる前
  * `loading`: リモートレスポンスがブラウザーによってロードされているとき
  * `loaded`: ブラウザーがリモートレスポンスをロードすることをを終わらせたとき
  * `interactive`: ロードが終了していなくても、ユーザーがリモートレスポンスと相互作用するとき
  * `success`: `XMLHttpRequest`が完結し、HTTPステータスコードが2XXの範囲にあるとき
  * `failure`: `XMLHttpRequest`が完結し、HTTPステータスコードが2XXの範囲にないとき
  * `404`: リクエストが404ステータスを返すとき
  * `complete`: `XMLHttpRequest`が完結したとき(`success`もしくは`failure`がある場合は、それらよりあとで起動します)

たとえば、リモートコールが初期化されレスポンスが一度受信されたとき、これを隠すために、ローディングインディケータを表示することはよくあることです。これを実現するには、リスト11-22で示されるように、Ajax呼び出しに`loading`と`complete`パラメーターを追加します。

リスト11-22 - アクティビティインディケータの表示と隠匿を行うためにAjaxコールバックを使う

    [php]
    <div id="feedback"></div>
    <div id="indicator">読み込み中...</div>
    <?php echo link_to_remote('この投稿を削除する', array(
        'update'   => 'feedback',
        'url'      => 'post/delete?id='.$post->getId(),
        'loading'  => "Element.show('indicator')",
        'complete' => "Element.hide('indicator')",
    )) ?>

`show`メソッドと`hide`メソッドは、JavaScriptのElementオブジェクトと同様に、Prototypeの便利なそのほかの追加機能です。

視覚効果を作成する
------------------

Webページで`<div>`要素を表示したり隠したりする以上のことをできるようにするために、symfonyはscript.aculo.usライブラリの視覚効果を統合します。視覚効果の構文は[http://script.aculo.us/](http://script.aculo.us/)のwikiでよいドキュメントが見つかります。基本的には、script.aculo.usライブラリは複雑な視覚効果を実現するためにDOMを操作するJavaScriptオブジェクトと関数を提供します。リスト11-23でいくつかの例をご覧ください。結果はWebページの特定領域上の視覚的なアニメーションなので、これらが実際に何をしているのかを理解するために自分で効果を確かめることをお勧めします。script.aclulo.usの公式サイトは動的な効果のアイディアを得られるギャラリーを提供します。

リスト11-23 - Script.acuo.usによるJavaScriptの視覚効果

    [php]
    // 'my_field'要素をハイライトする
    Effect.Highlight('my_field', { startcolor:'#ff99ff', endcolor:'#999999' })

    // 要素をブラインドダウンする
    Effect.BlindDown('id_of_element');

    // 要素をフェードアウェイする
    Effect.Fade('id_of_element', { transition: Effect.Transitions.wobble })

symfonyは、`Javascript`ヘルパーグループの一部でもある、`visual_effect()`ヘルパーで`JavaScriptの`Effect`オブジェクトをカプセル化します。リスト11-24で示されるように、このヘルパーは通常のリンクで使えるJavaScriptを出力します。

リスト11-24 - `visual_effect()`ヘルパーによるテンプレートでの視覚効果

    [php]
    <div id="secret_div" style="display:none">最初からずっと存在していました！</div>
    <?php echo link_to_function(
      '秘密のdivを表示する',
      visual_effect('appear', 'secret_div')
    ) ?>
    // はEffect.Appear('secret_div')を呼び出す

リスト11-25で示されるように、`visual_effects()`ヘルパーはAjaxコールバックでも利用できます。このヘルパーはリスト11-22のようなアクティビティインディケータを表示しますが、視覚的にはより満足のゆくものです。`indicator`要素はAjax呼び出しが始まるときに次第に表示され、レスポンスが到達したときに次第に消えます。加えて、`feedback`要素は、ユーザーの注目をこの部分のウィンドウに引きつけるために、リモートコールによって更新された後にハイライトされます。

リスト11-25 - Ajaxコールバックにおける視覚効果

    [php]
    <div id="feedback"></div>
    <div id="indicator" style="display: none">ロード中...</div>
    <?php echo link_to_remote('この投稿を削除する', array(
        'update'   => 'feedback',
        'url'      => 'post/delete?id='.$post->getId(),
        'loading'  => visual_effect('appear', 'indicator'),
        'complete' => visual_effect('fade', 'indicator').
                      visual_effect('highlight', 'feedback'),
    )) ?>

コールバックのなかで視覚効果を連結することでこれらを結びつける方法に注目してください。

JSON
----

JSON(JavaScript Object Notation)は軽量のデータ交換フォーマットです。基本的には、オブジェクトの情報を運ぶJavaScriptハッシュ(リスト11-26の例を参照)にすぎません。しかし、JSONはAjaxインタラクションに対して2つのすばらしい利点があります: JavaScriptのなかで読みやすく、Webレスポンスのサイズを減らすことができます。

リスト11-26 - JavaScriptでのJSONオブジェクトのサンプル

    var myJsonData = {"menu": {
      "id": "file",
      "value": "File",
      "popup": {
        "menuitem": [
          {"value": "New", "onclick": "CreateNewDoc()"},
          {"value": "Open", "onclick": "OpenDoc()"},
          {"value": "Close", "onclick": "CloseDoc()"}
        ]
      }
    }}

AjaxアクションがさらなるJavaScript処理に対して呼び出し元のページへ構造化データを返す必要がある場合、JSONはレスポンスのためのよいフォーマットです。たとえば、1つのAjax呼び出しが呼び出し元のページでいくつかの要素を更新する場合において非常に便利です。

たとえば、リスト11-27のような呼び出し元のページを想像してください。このページは更新することが必要かもしれない2つの要素を持ちます。1つのリモートヘルパーは、ページ要素の両方ではなく片方(`title`もしくは`name`)だけを更新できます。

リスト11-27 - 複数のAjaxを更新するテンプレートのサンプル

    [php]
    <h1 id="title">基本的な手紙</h1>
    <p>親愛なる<span id="name">name_here</span>へ、</p>
    <p>あなたのEメールは受信され近いうちに返信されます。</p>
    <p>敬具、</p>

両方を更新するために、Ajaxレスポンスがつぎの配列を含むJSONのデータになることができるか想像してください:

     [["title", "My basic letter"], ["name", "Mr Brown"]]

それからリモートコールは、JavaScriptのちょっとした助けによって、簡単にこのレスポンスを解釈し、列のなかのフィールドを更新できます。リスト11-28のコードはこの効果を得るためにリスト11-27のテンプレートに追加できるものを示しています。

リスト11-28 - リモートレスポンスから複数の要素を更新する

    [php]
    <?php echo link_to_remote('文字列をリフレッシュする', array(
      'url'      => 'publishing/refresh',
      'complete' => 'updateJSON(request)'
    )) ?>

    <?php echo javascript_tag("
    function updateJSON(ajax)
    {
      json = ajax.responseJSON;
      for (var i = 0; i < json.length; i++)
      {
         Element.update(json[i][0], json[i][1]);
      }
    }
    ") ?>

`complete`コールバックはAjaxレスポンスにアクセス可能でこれをサードパーティの関数に渡すことができます。このカスタム`updateJSON()`関数は`responseJSON`を通して得られたリスポンスからJSONのイテレーションを行い、それぞれの配列のメンバーに対して、`second`パラメーターの内容を持つ最初のパラメーターによって名づけられた要素を更新します。

リスト11-29は`publishing/refresh`アクションがJSONのレスポンスを返す方法を示してます。

リスト11-29 - JSONのデータを返すアクションのサンプル

    [php]
    class publishingActions extends sfActions
    {
      public function executeRefresh()
      {
        $this->getResponse()->setHttpHeader('Content-Type','application/json; charset=utf-8');
        $output = '[["title", "My basic letter"], ["name", "Mr Brown"]]';
        return $this->renderText('('.$output.')');
      }

content typeである`application/json`を利用することでPrototypeはレスポンスのbodyとして渡されるJSONを評価できます。これはヘッダーを通してJSONを返すよりも望ましいです。HTTPヘッダーはサイズの点で制限されておりいくつかのブラウザーはボディを持たないレスポンスに苦しむからです。
`->renderText()`を利用すればテンプレートファイルは使われないので、よりベターなパフォーマンスが得られます。

JSONはWebアプリケーションのあいだで標準規格となりました。Webサービスは、サーバーよりもクライアントでのサービス統合を可能にするために(マッシュアップ)、XMLよりもJSON形式のレスポンスをよく提示します。サーバーとJavaScript関数の間のコミュニケーションのためにどちらのフォーマットを使うべきか悩んでいる場合、おそらくJSONが最善の方法です。

>**TIP**
>バージョン5.2以降、PHPは`json_encode()`と`json_decode()`の2つの関数を提供します。これらによってPHP配列とJSON構文のあいだで変換できます。逆もしかりです([http://www.php.net/manual/ref.json.php](http://www.php.net/manual/ref.json.php))。これらはJSON配列(と一般的なAjax)を円滑にしてネイティブで読みやすいPHPコードの使用を有効にします:
>
>     [php]
>     $output = array("title" => "My basic letter", "name" => "Mr Brown");
>     return $this->renderText(json_encode($output));

複雑なインタラクションをAjaxで実行する
--------------------------------------

symfonyのAjaxヘルパーのなかには、単独の呼び出しで複雑なインタラクションを組み立てるいくつかのツールも見つかります。これらのツールによって複雑なJavaScriptのコードをともなわずにデスクトップのアプリケーションのようなインタラクション(ドラッグドロップ、オートコンプリート、ライブ編集)によってユーザーエクスペリエンスを強化できます。つぎのセクションでは複雑なインタラクションのためのヘルパーを説明し、シンプルな例を示します。追加パラメーターと調整方法はscript.aculo.usのドキュメントで説明されます。

>**CAUTION**
>複雑なインタラクションが可能であれば、これらが自然に感じられるように調整を行うプレゼンテーションのための追加時間が必要です。これらがユーザーエクスペリエンスを強化することを確信するときのみこれらをご利用ください。これらがユーザーを混乱させるリスクが存在するのであればこれらを避けてください。

### オートコンプリート

ユーザーが入力している間にユーザーエントリーにマッチする単語リストを表示するテキスト入力コンポーネントはオートコンプリート(autocompletetion)と呼ばれます。`input_auto_complete_tag()`と呼ばれる単独のヘルパーによって、リスト11-30で示された例のように、リモートアクションがHTMLのアイテムとしてフォーマットされたレスポンスを返す条件の下で、この効果を実現できます。

リスト11-30 - オートコンプリートのタグと互換性のあるレスポンスの例

    <ul>
      <li>suggestion1</li>
      <li>suggestion2</li>
      ...
    </ul>

つぎのリスト11-31で示される例のように、通常のテキスト入力を処理するようにテンプレートにヘルパーを挿入します。

リスト11-31 - テンプレートのなかでオートコンプリートタグヘルパーを使う

    [php]
    <?php echo form_tag('mymodule/myaction') ?>
      筆者の名前を見つける:
      <?php echo input_auto_complete_tag('author', 'default name',
        'author/autocomplete',
        array('autocomplete' => 'off'),
        array('use_style'    => true)
      ) ?>
      <?php echo submit_tag('Find') ?>
    </form>

ユーザーが`author`フィールドに文字を入力するたびにこれは`author/autocomplete`アクションを呼び出します。アクションが`author`のリクエストパラメーターにしたがってマッチ可能な単語リストを決定し、リスト11-30と同じようなフォーマットでこれらを返すように設計するのはあなた次第です。図11-3で示されるように、ヘルパーは`author`タグの下でリストを表示し、提示された項目の1つをクリックする、もしくはキーボードで選択すると入力が補完されます。

図11-3 - オートコンプリートの例

![オートコンプリートの例](http://github.com/masakielastic/symfonybook-ja/raw/master/images/F1103.png "オートコンプリートの例")

`input_auto_complete_tag()`ヘルパーの5番目の引数はつぎのパラメーターを受けとることができます:

  * `use_style`: レスポンスが自動的に一覧を表示するスタイル。
  * `frequency`: 呼び出し周期の頻度(デフォルトは0.4秒)。
  * `indicator`: オートコンプリートのロードが始まり完了した場合に消えるときに表示されるインディケータのid。
  * tokens: トークン化されたインクリメンタルなオートコンプリートを許可するため。たとえば、このパラメーターを`,`に設定してユーザーが`jane, george`を入力すると、アクションは`'george'`の値だけを受けとります。

### ドラッグアンドドロップ

要素をマウスで掴む、動かす、どこかで放すといった機能はデスクトップのアプリケーションでは親しまれていますが、Webブラウザーにおいてはまれです。無地のJavaScriptにおいてこのようなふるまいをコードに書く作業がとても非常に複雑だからです。幸いにして、symfonyにおいてこの機能を実現するには1行のコードだけ必要です。

symfonyは`draggable_element()`と`drop_receiving_element()`の2つのヘルパーを提供します。これらをふるまいの修飾子としてみなすことができます; これらはこれらが扱う要素にオブザーバーと機能を追加します。ドラッグ可能な要素に対して要素をドラッグ可能な要素もしくは受けとる要素として宣言するためにこれらを使います。ドラッグ可能な要素はこれをマウスでクリックすることで掴み取ることができます。マウスボタンが放されるまで、要素を動かす、もしくはウィンドウを越えてドラッグされます。`ドラッグ可能な要素が放されたとき、受けとる要素はリモート関数を呼び出します。リスト11-32は要素を受けとるショッピングカートとこのタイプのインタラクションのお手本を示しています。

リスト11-32 - ショッピングカートでドラッグ可能な要素とドロップを受けとる要素

    [php]
    <ul id="items">
      <li id="item_1" class="food">にんじん</li>
      <?php echo draggable_element('item_1', array('revert' => true)) ?>
      <li id="item_2" class="food">りんご</li>
      <?php echo draggable_element('item_2', array('revert' => true)) ?>
      <li id="item_3" class="food">オレンジ</li>
      <?php echo draggable_element('item_3', array('revert' => true)) ?>
    </ul>
    <div id="cart">
      <p>カートが空です</p>
      <p>ここにある品物をドラッグしてカートに追加してください</p>
    </div>
    <?php echo drop_receiving_element('cart', array(
      'url'        => 'cart/add',
      'accept'     => 'food',
      'update'     => 'cart',
    )) ?>

未注文の商品リストのそれぞれの項目をマウスで掴み取ることが可能で、ウィンドウのなかであればドラッグできます。放されたとき、これらは元の位置に戻ります。`cart`要素の上で放されたとき、`cart/add`アクションのリモート呼び出しが発動します。`id`リクエストパラメーターを見ることで、アクションは`cart`要素にどの品物がドロップされたのか決定できます。リスト11-32は現実のショッピングセッションをシミュレートしています: 品物を掴み、カートで放します。会計に進みます。

>**TIP**
>リスト11-32において、ヘルパーが修正する要素の直後にヘルパーが書かれていますが、これは必須要件ではありません。テンプレートの終わりですべての`draggalbe_element()`と`drop_receiving_element()`ヘルパーを明解に分けることも可能です。重要なのはヘルパー呼び出しの最初の引数で、これはふるまいを受けとる要素の識別子を指定します。

`draggable_element()`ヘルパーはつぎのパラメーターを受けとります:

  * `revert`: `true`に設定した場合、要素は放されたときに元の場所に戻ります。任意の関数を参照することも可能で、ドラッグ操作の終了後に呼び出されます。
  * `ghosting`: 要素をクローンしてそのクローンをドラッグし、クローンがドロップされるまで、所定の位置に留めます。
  * `snap`: `snap`: `false`に設定した場合、スナッピングされません。そうでなければドラッグ可能な要素は間隔がxとyであるグリッドの交点だけになります。この場合、オプションは`xy`もしくは`[x,y]`もしくは`function(x,y){return [x,y]}`の形式をとります。

`drop_receiving_element()`ヘルパーはつぎのパラメーターを受けとります:

  * `accept`: 文字列もしくはCSSクラスを記述する文字列の配列。要素はこれらのCSSクラスを1つもしくは複数持つドラッグ可能な要素だけを受けとります。
  * `hoverclass`: ドラッグ可能な受けとり要素をある要素の上にドラッグするときにその要素に追加されるCSSクラス

### ソート可能なリスト

`draggable`要素によって可能となる別の機能としては、マウスで項目を動かすことによるリストのソート機能があります。`sortable_element()`ヘルパーはソート可能なふるまいを項目に追加します。リスト11-33はこの機能を実装するよい例です。

リスト11-33 -ソート可能なリストの例

    [php]
    <p>何が一番好きですか？</p>
    <ul id="order">
      <li id="item_1" class="sortable">にんじん</li>
      <li id="item_2" class="sortable">りんご</li>
      <li id="item_3" class="sortable">オレンジ</li>
      // どのみち誰も芽キャベツが好きではない
      <li id="item_4">芽キャベツ</li>
    </ul>
    <div id="feedback"></div>
    <?php echo sortable_element('order', array(
      'url'    => 'item/sort',
      'update' => 'feedback',
      'only'   => 'sortable',
    )) ?>

`sortable_element()`ヘルパーのマジックによって、`<ul>`要素はソート可能になります。このことはその子要素をドラッグアンドドロップで再び並べ替えることができることを意味します。リストを並べ替えるためにユーザーが項目をドラッグして放すたびに、Ajaxリクエストはつぎのパラメーターによって設定されます:

    POST /sf_sandbox/web/frontend_dev.php/item/sort HTTP/1.1
      order[]=1&order[]=3&order[]=2&_=

十分に並べ替えられたリストは配列として渡されます(`orde[$rank]=$id`の形式で、`$rank`は`0`で始まり、`$id`はリスト要素の`id`プロパティのアンダースコア(`_`)のあとで来るものに基づきます)。`id`プロパティのソート可能な要素(この例では`order`)はパラメーターの配列を名づけるために使われます。

`sortable_element()`ヘルパーはつぎのパラメーターを受けとります:

  * `only`: CSSクラスを記述する文字列もしくは文字列の配列。このクラスをともなうsortable要素の子要素だけが動かせる。
  * `hoverclass`: マウスが要素の上にホーバーされたときにその要素に追加されるCSSクラス。
  * `overlap`: 項目がインラインに表示される場合は`horizontal`に設定、1行につき1つの項目があるとき(例の場合)、`vertical`に設定します。
  * `tag`: リストの順序が`<li>`要素のセットではない場合、ドラッグ可能なものにするためにソート可能な子要素を決定しなければなりません(たとえば、`div`もしくは`dl`)。

>**TIP**
>symfony 1.1以降では、`url`オプションをともなわずに`sortable_element()`ヘルパーを使うこともできます。ソートにおいてAJAXリクエストは行われません。`save`ボタンもしくはその他へのAJAX呼び出しを延期したい場合に便利です。

### その場で編集する

より多くのWebアプリケーションは、フォームにおいて内容を再表示することなく、ページ上でユーザーがページの内容を直接編集することを許可しています。インタラクションの原則はシンプルであることです。テキストブロックはユーザーがその上にマウスをホーバーするときにハイライトされます。ユーザーがブロック内側でクリックすると、プレーンテキストはブロックのテキストで入力されたテキストエリアに変換され、保存ボタンが表示されます。ユーザーはテキストエリアの内側でテキストを編集することが可能で、いったんそれを保存すると、テキストエリアとテキストはプレーンな形式で表示されます。symfonyにおいて、`input_in_place_editor_tag()`ヘルパーによってこの編集可能なふるまいを要素に追加できます。リスト11-34はこのヘルパーの使いかたのお手本を示しています。

リスト11-34 - 編集可能なテキストの例

    [php]
    <div id="edit_me">このテキストを編集できます</div>
    <?php echo input_in_place_editor_tag('edit_me', 'mymodule/myaction', array(
      'cols'        => 40,
      'rows'        => 10,
    )) ?>

ユーザーが編集可能なテキストをクリックしたとき、編集可能で、テキストで満たされたテキスト入力エリアによって置き換えられます。フォームが投稿されたとき、編集された値のセットを`value`パラメーターとして保有する`mymodule/myaction`アクションがAjaxのなかに呼び出されます。アクションの結果は編集可能な要素を更新します。これはとても速く書くことが可能でとても強力です。

`input_in_place_editor_tag()`ヘルパーはつぎのパラメーターを受けとります:

  * `cols`と`rows`: 編集のために表示されるテキスト入力領域のサイズ(`rows`が1より大きい場合は`<textarea>`になります)
  * `loadTextURL`: 編集するテキストを表示するために呼び出されるアクションのURIです。編集可能な要素が特別な整形方法を利用することかつユーザーが整形なしのテキストを編集できるようにしたい場合にこれは便利です。
  * `save_text`と`cancel_text`: 保存リンク(デフォルトは"ok")とキャンセルリンク(デフォルトは"cancel")上のテキスト。

多くのオプションが利用できます。オプションのドキュメントとそれらの効果に関しては [http://mir.aculo.us/2007/7/17/in-place-editing-the-summer-2007-rewrite/](http://mir.aculo.us/2007/7/17/in-place-editing-the-summer-2007-rewrite/)を参照してください。

まとめ
----

クライアントサイドのふるまいを手に入れるためにテンプレートのなかでJavaScriptを書くことに疲れましたら、JavaScriptヘルパーは代わりのシンプルな方法を提供します。これらのヘルパーは基本的なリンクのふるまいと要素の更新を自動化するだけでなく、すぐにAjaxインタラクションを開発する方法も提供します。強力なPrototypeによる構文強化とscript.aculo.usによるすばらしい視覚効果の助けによって、複雑なインタラクションを実現するには、数行のコードを書くことだけが必要です。

高度にインタラクティブなアプリケーションはsymfonyで静的なページを作ることと同じぐらい簡単なので、ほとんどすべてのデスクトップアプリケーションのインタラクションはWebアプリケーションで利用できると考えることができます。
