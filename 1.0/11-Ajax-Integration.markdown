第11章 - Ajaxの統合
===================

クライアントサイド上でのインタラクション、複雑な視覚効果、非同期通信はWeb 2.0のアプリケーションにおいて共通の機能です。JavaScriptを必要とするこれらの機能は、コードを手書きするのはやっかいでデバッグに時間がかかることはよくあります。幸いにして、symfonyはヘルパーの完全なセットをともなうテンプレート内部のJavaScriptの多くの共通部分を自動化します。一行のJavaScriptのコードがなくても多くのクライアントサイドのふるまいを実現できます。開発者は実現したい効果だけに集中していればよく、symfonyが複雑な構文と互換性問題を処理します。

この章ではクライアントサイドのスクリプトを書く作業を円滑にするためにsymfonyによって提供されたツールについて説明します:

  * 基本的なJavaScriptヘルパーは、DOM(Document Object Model)要素を更新するもしくはリンクでスクリプトを起動させるために標準規格に準拠している`<script>`タグを出力します。
  * Prototypeはsymfonyに統合されたJavaScriptのライブラリです。これはJavaScriptのコアに新しい関数とメソッドを追加することでクライアントサイドのスクリプト開発を加速します。
  * Ajaxヘルパーによってユーザーはリンクをクリックする、フォームを投稿する、もしくはフォーム要素を修正することでページのいくつかの部分を更新できるようになります。
  * これらのヘルパーの多くのオプションは、とりわけコールバック関数を利用することで、よりすばらしい柔軟性とパワーも提供します。
  * script.aculo.usもsymfonyに統合された別のJavaScriptのライブラリです。script.aculo.usはインターフェイスとユーザーエクスペリエンスを強化する動的な視覚効果を追加します。
  * JSON(JavaScript Object Notation)はサーバーとクライアントスクリプトのあいだでコミュニケーションを行うために使われる標準規格です。
  * 前述のすべての要素を結びつけるクライアントサイドのインタラクションはsymfonyのアプリケーションで実現可能です。オートコンプリート、ドラッグドロップ、ソート可能なリスト、編集可能なテキストはPHPコードの1行、symfonyのヘルパーへの呼び出し、ですべて実装できます。

基本的なJavaScriptヘルパー
--------------------------

JavaScriptはクロスブラウザーの互換性が欠如していたため、プロフェッショナルなWebアプリケーションで実際に使うものはほとんどないと長い間見なされてきました。今日において、互換性の問題は(ほとんど)解決され、いくつかの頑強なライブラリによって、膨大な行数のコーディングと膨大な時間のデバッグを行わなくてもJavaScriptで複雑なインタラクションをプログラミングできます。もっとも人気のある先進技術はAjax(asynchronous JavaScript and XML)と呼ばれます。これはこの章の"Ajaxヘルパー"のセクションで説明します。

逆説的にも、この章ではjavaScriptのコードはほんのわずかしか見ません。symfonyがクライアントサイドのスクリプティングへの独自の方法を持つからです: symfonyはJavaScriptのふるまいをヘルパーにまとめて抽象化するので、テンプレートはJavaScriptのコードをまったく表示せずに終わります。開発者のために、ふるまいをページの要素に追加するにはPHPのコードが1行必要ですが、このヘルパー呼び出しがJavaScriptのコードの出力を行い、生成されたレスポンスを検査すると、すべてがカプセル化された複雑性を持つことがあきらかになります。ヘルパーが、ブラウザーの一貫性、複雑な制限のある事例、拡張性などを扱うので、これらが含むJavaScriptコードの総量が極めて重要になる可能性があります。それゆえ、この章はJavaScriptで開発するために使う効果を実現するためのJavaScriptの使いかたをお教えします。

`Javascript`ヘルパーグループの使用を宣言しているのであれば、ここで説明されるすべてのヘルパーは、テンプレートのなかで利用できます。

    [php]
    <?php use_helper('Javascript') ?>

すぐに学ぶように、これらのヘルパーのなかにはHTMLのコードやJavaScriptのコードを出力するものがあります。

### テンプレート内のJavaScript

XHTMLにおいて、JavaScriptのコードブロックはCDATA宣言で囲まなければなりません。しかし、複数のJavaScriptのコードブロックを必要とするページを書く作業はすぐに退屈になります。symfonyが文字列をXHTML準拠の`<script>`タグに変換する`javascript_tag()`ヘルパーを提供するのはそういうわけです。リスト11-1はこのヘルパーの使いかたのお手本を示しています。

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

しかし、コードブロックよりも、JavaScriptのもっとも共通の使いかたは、特定のスクリプトを起動させるハイパーリンクのなかにあります。リスト11-2で示されるように、`link_to_function()`ヘルパーはまさにこれを行います。

リスト11-2 - JavaScriptを`link_to_function()`ヘルパーをともなうリンクで発動させる

    [php]
    <?php echo link_to_function('クリックしてください！', "alert('foobar')") ?>
     => <a href="#" onClick="alert('foobar'); return none;">クリックしてください！</a>

`link_to()`ヘルパーと同じように、オプションを3番目の引数の`<a>`タグに追加できます。

>**NOTE**
>`link_to()`ヘルパーが`button_to()`の兄弟を持つのと同様に、`button_to_function()`ヘルパーを呼び出すことでボタン(`<input type="button">`)からJavaScriptを発動させることができます。そしてクリック可能なイメージが望ましい場合、`link_to_function(image_tag('myimage'), "alert('foobar')")`を呼び出します。

### DOM要素を更新する

動的なインターフェイスにおける共通のタスクの1つはページの要素の更新です。通常書くコードはリスト11-3で示されるようなものです。

リスト11-3 - JavaScriptの要素を更新する

    [php]
    <div id="indicator">データ処理の開始</div>
    <?php echo javascript_tag("
      document.getElementById("indicator").innerHTML =
        "<strong>データ処理の完了</strong>";
    ") ?>

この目的のために、symfonyはHTMLではなくJavaScriptを生み出すヘルパーを提供します。このヘルパーは`update_element_function()`と呼ばれます。リスト11-4は使いかたを示しています。

リスト11-4 - JavaScriptの要素を`update_element_function()`ヘルパーで更新する

    [php]
    <div id="indicator">データ処理の開始</div>
    <?php echo javascript_tag(
      update_element_function('indicator', array(
        'content'  => "<strong>データ処理の終了</strong>",
      ))
    ) ?>

少なくとも実際のJavaScriptのコードと同じぐらいの長さがあるので、なぜこのヘルパーがとりわけ便利なのかに戸惑っているかもしれません。本当の問題は読みやすさです。たとえば、要素の前あとで内容を挿入したい場合、要素を更新する代わりに除去したい場合、もしくはある種の条件にしたがって何も行いたくない場合があります。そのような場合において、JavaScriptのコードはやや散乱しますが、リスト11-5で示されるように、`update_element_function()`はテンプレートをとても読みやすいように維持します。

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

ヘルパーによってテンプレートはJavaScriptのコードよりも理解しやすくなります。そして似たようなふるまいに対して単独の構文があります。これはヘルパーの名前がとても長い理由でもあります: 追加コメントなしで、コードの名前そのものが説明になります。

### グレイスフルデグラデーション(Graceful Degradation)

`<noscript>`タグによってJavaScriptのサポートを持たないブラウザーが表示するHTMLコードを指定できます。symfonyはまったく逆のことを行うヘルパーでこれを補完します: symfonyがコードを保証するので、JavaScriptを実際にサポートするブラウザーだけがコードを実行します。リスト11-6で示されるように、`if_javascript()`と`end_if_javascript()`ヘルパーはグレースフルデグラデーション(graceful degradation)をサポートする(訳注：フォールトトレラントシステムとも言い換えできる)アプリケーションの作成を円滑にします。

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

Prototypeは偉大なJavaScriptのライブラリです。クライアントスクリプト言語の可能性を広げ、いつも夢見ているけれど見つからない関数を追加し、DOMを操作する新しいメカニズムを提供します。プロジェクトのWebサイトは [http://prototypejs.org/](http://prototypejs.org/)です。

Prototypeのファイルはsymfonyのフレームワークに搭載されており、すべての新しいsymfonyのプロジェクトの`web/sf/prototype`ディレクトリにおいて、アクセスできます。このことはつぎのコードをアクションに追加することでPrototypeを利用できるようになることを意味します:

    [php]
    $prototypeDir = sfConfig::get('sf_prototype_web_dir');
    $this->getResponse()->addJavascript($prototypeDir.'/js/prototype');

もしくはつぎのコードを`view.yml`ファイルに追加することでも利用できます:

    all:
      javascripts: [%SF_PROTOTYPE_WEB_DIR%/js/prototype]

>**NOTE**
>つぎのセクションで説明するsymfonyのAjaxヘルパーはPrototypeに依存しているので、Prototypeのライブラリの1つを使い始めると同時にPrototypeは自動的にインクルードされます。これはテンプレートが`_remote`ヘルパーを呼び出す場合、PrototypeのJavaScriptをレスポンスに手動で追加する必要がないことを意味します。

いったんPrototypeのライブラリがロードされると、JavaScriptのコアに追加されるすべての新しい関数を利用できます。この本の目的はこれらすべてを説明することではないのですが、Web上でPrototypeについてつぎのWebサイトのようなよいドキュメントが見つかります:

  * Particletree: [http://particletree.com/features/quick-guide-to-prototype/](http://particletree.com/features/quick-guide-to-prototype/)
  * Sergio Pereira: [http://www.sergiopereira.com/articles/prototype.js.html](http://www.sergiopereira.com/articles/prototype.js.html)
  * Script.aculo.us: [http://wiki.script.aculo.us/scriptaculous/show/Prototype](http://wiki.script.aculo.us/scriptaculous/show/Prototype)

JavaScriptに追加されるPrototypeの関数の1つはドル関数の`$()`です。基本的に、この関数は`document.getElementById()`へのシンプルなショートカットですが、もう少し強力です。この使いかたの例に関してはリスト11-7をご覧ください。

リスト11-7 - JavaScriptでIDによって要素を取得する`$()`関数を使う

    [php]
    node = $('elementID');

    // つぎのコードと同じ
    node = document.getElementById('elementID');

    // 一度に複数の要素を取得することも可能で
    // この場合、結果はDOM要素の配列である
    nodes = $('firstDiv', 'secondDiv');

PrototypeはJavaScriptのコアが本当に欠如する関数を提供します。たとえば、引数として渡されるクラスを持つすべてのDOM要素の配列を返す関数です:

    [php]
    nodes = document.getElementByClassName('myclass');

しかしながら、これはほとんど使わないでしょう。Prototypeが二重ドル、`$$()`と呼ばれるもっと強力な関数を提供するからです。この関数はCSSセレクタに基づいてDOM要素の配列を返します。ですので以前の呼び出しをつぎのようにも書けます:

    [php]
    nodes = $$('.myclass');

CSSセレクタの力のおかげで、クラスとIDでDOMを解析できるので、親子関係と前後関係が以前のXPathの表記で行うよりもはるかに簡単です。これらすべてを結びつける複雑なセレクタで要素にアクセスすることもできます:

    [php]
    nodes = $$('body div#main ul li.last img > span.legend');

Prototypeによって提供された構文強化の例の最後の1つはeach配列イテレータです。PHPと同等の簡潔さを提供します。匿名関数とクロージャを定義する機能をJavaScriptに追加します。JavaScriptを手作業で実装する場合に多く使います。

    [php]
    var vegetables = ['Carrots', 'Lettuce', 'Garlic'];
    vegetables.each(function(food) { alert('I love ' + food); });

Prototypeを使うJavaScriptのプログラミングは手作業で行うよりはるかに面白く、このライブラリはsymfonyの一部でもあるので、関連ドキュメントを読むには数分しかかかりません。

Ajaxヘルパー
------------

ページの要素を更新したい場合、リスト11-5のようにJavaScriptではなくサーバーによって実行されるPHPのスクリプトで行う場合にはどうしたらよいでしょうか？これによってサーバーのレスポンスにしたがってページの部分を変更する機会が提供されます。リスト11-8で示されるように、`remote_function()`ヘルパーはまさにこれを行います。

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
>通常の`url_for()`のように、`url`パラメーターは内部のURI(`module/action?key1=value1&...`)もしくはルーティングルール名のどちらかを含みます。

呼び出されたとき、このスクリプトは`mymodule/myaction`アクションのリクエストのレスポンスによってidがmyzoneである要素を更新します。この種のインタラクションはAjaxと呼ばれ、非常にインタラクティブなWebアプリケーションのなか心です。Wikipedia([http://en.wikipedia.org/wiki/AJAX](http://en.wikipedia.org/wiki/AJAX))ではつぎのように説明されています:

舞台裏のサーバーで小さな量のデータを交換することで、AjaxはWebページをより反応性の高いものにするのでユーザーが変更をするたびに全体のWebページをリロードする必要がありません。これはWebページのインタラクティビティ、スピードとユーザービリティを増大させることを意味します。

Ajaxは`XMLHttpRequest`に依存します。`XMLHttpRequest`は`hidden`フレームのように振る舞うJavaScriptのオブジェクトで、サーバーリクエストとWebページの残りの部分の操作からこのオブジェクトを更新できます。このオブジェクトはとても低いレベルで、異なるブラウザーが異なる方法で処理するので、通常はAjaxリクエストの処理を手作業で行うことは長い部分のコードを書くことを意味します。幸いにして、PrototypeはAjaxを処理するすべてのコードをカプセル化し、よりシンプルなAjaxオブジェクトを提供するので、symfonyはこのオブジェクトを頼りにします。いったんAjaxヘルパーをテンプレートのなかで使うとPrototypeライブラリが自動的にロードされるのはそういうわけです。

>**CAUTION**
>リモートアクションのURLが現在のページが同じドメインに所属しない場合Ajaxヘルパーは機能しません。この制限はセキュリティの理由のために存在し、回避できないブラウザーの制限に依存します。

Ajaxのインタラクションは3つの部分から構成されます: コーラー(リンク、ボタン、フォーム、もしくはアクションを立ち上げるためにユーザーが操作するコントロール機能)、サーバーアクション、アクションのレスポンスを表示するページの領域です。リモートアクションがクライアントサイド上のJavaScript関数によって処理されるデータを返す場合、もっと複雑なインタラクションを開発できます。symfonyはAjaxのインタラクションを、すべてが`remote`を名前に含む、テンプレートに挿入するために複数のヘルパーを提供します。これらは共通の構文も共有します。構文はすべてのAjaxパラメーターを持つ連想配列です。AjaxヘルパーはJavaScriptではなくHTMLコードを出力することに注意してください。

>**SIDEBAR**
>**Aaxアクションはいかが？**
>
>リモート関数と呼ばれるアクションは通常のアクションです。これらはルーティングに従い、`return`をともなうレスポンスをレンダリングするためにビューを決定し、変数をテンプレートに渡し、ほかのアクションと同様にモデルを変更することができます。
>
>しかしながら、Ajaxを通して呼び出されたとき、アクションは`true`をつぎの呼び出しに対して返します:
>
>     [php]
>     $isAjax = $this->getRequest()->isXmlHttpRequest();
>
>symfonyはアクションがAjaxの文脈のなかにあることを知っており、それに応じてレスポンスに処理を適合させることができます。そのために、デフォルトで、Ajaxのアクションは開発環境ではWebデバッグツールバーを含みません。また、これらはデコレーションプロセスをスキップします(これらのテンプレートはデフォルトでレイアウトに含まれません)。デコレートされたAjaxのビューが欲しい場合、`view.yml`モジュールのファイルのなかでこのビューに対してhas_layout: true`を明示的に指定する必要があります。
>
>もう1つだけ言っておきます: Ajaxのインタラクションにおいて応答性は重大なので、レスポンスがあまり複雑ではない場合、ビューを作成することは避け、代わりにアクションから直接レスポンスを返すことはよいアイディアかもしれません。ですのでテンプレートをスキップしてAjaxのリクエストを加速するためにアクションのなかで`renderText()`メソッドを使うことができます。

### Ajaxのリンク

AjaxのリンクはWeb 2.0のアプリケーションで利用できるAjaxのインタラクションを広く共有します。`link_to_remote()`ヘルパーは、予想どおり、リモート機能を呼び出すリンクを出力します。リスト11-9で示されるように構文は`link_to()`のものととても似ています(2番目のパラメーターがAjaxオプションの連想配列であること以外)。

リスト11-9 - `link_to_remote()`ヘルパーによるAjaxリンク

    [php]
    <div id="feedback"></div>
    <?php echo link_to_remote('この投稿を削除する', array(
        'update' => 'feedback',
        'url'    => 'post/delete?id='.$post->getId(),
    )) ?>

この例では、`'この投稿を削除する'`のリンクをクリックするとバックグランドのなかで`post/delete`アクションへの呼び出しが行われます。サーバーによって返されたレスポンスは`id`が`feedback`である要素のなかに現れます。このプロセスは図11-1で説明されます。

図11-1 - ハイパーリンクでリモートの更新を実行する

![ハイパーリンクでリモート更新を実行する](http://github.com/masakielastic/symfonybook-ja/raw/master/images/F1101.png "ハイパーリンクでリモートの更新を実行する")

リスト11-10で示されるように、文字列の代わりに画像を使い、内部の`module/action`のURLの代わりにルール名を使い、オプションを3番目の引数の`<a>`タグに追加することができます。

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

Webフォームはよく別のアクションを呼び出しますが、これによってページ全体がリフレッシュされます。このフォームに対して`link_to_function()`が対応するものはサーバーのレスポンスをともなうページの要素だけを更新するフォーム投稿になります。これは`form_remote_tag()`ヘルパーが行うことで、構文はリスト11-11で示されます。

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

`form_remote_tag()`は通常の`form_tag()`ヘルパーのように`<form>`を開きます。このフォームを投稿することで、`item`フィールドをリクエストパラメーターとして、バックグラウンドでPOSTリクエストを`item/add`アクションに行います。リスト11-2で描かれているように、レスポンスは`item_list`要素の内容を置き換えます。通常の`</form>`閉じタグでAjaxフォームを閉じてください。

図11-2 - リモート更新をフォームで実行する

![フォームでリモート更新を実行する](http://github.com/masakielastic/symfonybook-ja/raw/master/images/F1102.png "フォームでリモート更新を実行する")

>**CAUTION**
>Ajaxのフォームはmultipartになることはできません。これは`XMLHttpRequest`オブジェクトの制限です。このことはAjaxフォームを通してファイルのアップロードを処理できないことを意味します。しかしながら、次善策があります。たとえば、`XMLHttpRequest`の代わりに隠し`iframe`を使うことです。

ページモードとAjaxモードの両方でフォームが機能するようにしたい場合、最前の解決方法はフォームを通常のものと同じように定義することです。しかしながらそのためには、通常の投稿ボタンに加えて、Ajaxでフォームを投稿する2番目のボタン(`<input type="button"/>`)が必要です。symfonyはこのボタンを`submit_to_remote()`と呼びます。これはグレースルフルデグラデーションを行うAjaxのインタラクションを開発するための助けになります。リスト11-12で例をご覧ください。

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
>ユーザーがEnterキーを押すとき、フォームはメインの`<form>`タグで定義されたアクションを使用して投稿されます。この例では通常のアクションです。

モダンなフォームは投稿されたときだけでなく、フィールドの値がユーザーによって更新されたときも反応します。symfonyでは、そのために`observe_field()`ヘルパーを使うことができます。リスト11-13は提案機能を開発するためにこのヘルパーを使う例を示しています: `item`フィールドに入力されたそれぞれの文字はページの`item_suggestion`要素をリフレッシュするAjax呼び出しを呼び出します。

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

`@item_being_typed`ルールで書かれたモジュール/アクションはフォームを投稿しなくてもユーザーがobservedフィールド(`item`)の値を変更するたびに呼び出されます。アクションは`value`リクエストパラメーターから現在の`item`の値を取得することができます。observedフィールドの値以外のほかのものを渡したい場合、渡したいものを`with`パラメーターのなかのJavaScriptの表記として指定できます。たとえば、`param`パラメーターを取得するアクションが欲しい場合、リスト11-14で示されるように、`observe_field()`ヘルパーを書いてください。

リスト11-14 - 独自のパラメーターを`with`オプションでリモートアクションに渡す

    [php]
    <?php echo observe_field('item', array(
        'update'   => 'item_suggestion',
        'url'      => '@item_being_typed',
        'with'     => "'param=' + value",
    )) ?>

このヘルパーはHTML要素の出力を行わず、代わりにパラメーターとして渡された要素のためのふるまいを出力します。この章の後のほうでふるまいを割り当てるJavaScriptヘルパーの例をより多く見ることになります。

フォームのフィールドのすべてを観測したい場合、`observe_form`ヘルパーを使うべきです。フォームフィールドの1つが修正されるたびにこのヘルパーはリモート関数を呼び出します。

### 定期的にリモート関数を呼び出す

大事なことを言い忘れましたが、`periodically_call_remote()`ヘルパーは数秒毎に実行されるAjaxのインタラクションです。これはHTMLのコントロールに添付されませんが、全体のページのふるまいとして、バックグランドで透過的に実行されます。これはマウスの位置を追跡する、ラージテキストエリアの内容を自動保存するなどのために非常に便利です。リスト11-15はこのヘルパーの使いかたの例を示しています。

リスト11-15 - `periodically_call_remote()`で周期的にリモート関数を呼び出す

    [php]
    <div id="notification"></div>
    <?php echo periodically_call_remote(array(
        'frequency' => 60,
        'update'    => 'notification',
        'url'       => '@watch',
        'with'      => "'param=' + \$F('mycontent')",
    )) ?>

リモート関数への2つの呼び出しを待つ秒数(`frequency`)を指定しない場合、デフォルト値の10秒が使われます。`with`パラメーターはJavaScriptで評価されるので、`$F()`関数といった、Prototypeの関数を使うことができます。

リモート呼び出しパラメーター
------------------------

以前のセクションで説明されたすべてのAjaxヘルパーは、`update`パラメーターと`url`パラメーターに加えて、ほかの値を取ることができます。Ajaxパラメーターの連想配列はリモート呼び出しとそれらのレスポンスの処理のふるまいを変更し調整することができます。

### レスポンスの状態にしたがって異なる要素を更新する

リモートアクションが失敗した場合、リモートヘルパーは、成功したレスポンスによって更新された要素以外の、別の要素の更新を選択できます。この目的のために、`update`パラメーターの値を連想配列に分割し、`success`と`failure`の場合に更新する要素に対して異なる値を設定します。たとえば1つのページと1つのエラーのフィードバックの領域内にたくさんのAjaxのインタラクションが存在する場合、これは大いに役立ちます。リスト11-16は条件つきの更新の扱いかたのお手本を示しています。

リスト11-16 - 条件つきの更新を扱う

    [php]
    <div id="error"></div>
    <div id="feedback"></div>
    <p>Hello, World!</p>
    <?php echo link_to_remote('この投稿を削除する', array(
        'update'   => array('success' => 'feedback', 'failure' => 'error'),
        'url'      => 'post/delete?id='.$post->getId(),
    )) ?>

>**TIP**
>HTTPのエラーコード(500、404と2XXの範囲にないすべてのコード)は、`sfView::ERROR`を返すアクションではなく、failureの場合の更新を実行します。Ajaxのfailureを返すアクションを作りたい場合、アクションは`$this->getResponse()->setStatusCode(404)`もしくは似たようなものを呼び出さなければなりません。

### 位置にしたがって要素を更新する

`update_element_function()`ヘルパーと同様に、`position`パラメーターを追加することで特定の要素に相対的なものとして更新する要素を指定できます。リスト11-17は例を示しています。

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

`position`パラメーターはつぎの値を取ることができます:

  * `before`: 要素の前
  * `after`: 要素の後
  * `top`: 要素の内容の一番上
  * `bottom`: 要素の内容の一番下

### 条件にしたがって要素を更新する

リスト11-18で示されるように、リモート呼び出しは、`XMLHttpRequest`を実際に投稿するまえに、ユーザーが確認できるようにする追加パラメーターを取ることができます。

リスト11-18 - リモート関数を呼び出すまえに確認を求める`confirm`パラメーターを使う

    [php]
    <div id="feedback"></div>
    <?php echo link_to_remote('この投稿を削除する', array(
        'update'   => 'feedback',
        'url'      => 'post/delete?id='.$post->getId(),
        'confirm'  => 'よろしいですか？',
    )) ?>

"よろしいですか？"を示すJavaScriptのダイアログボックスはユーザーがリンクをクリックしたときにポップアップを行い、`post/delete`アクションはユーザーがOKをクリックして選択を確認する場合だけに呼び出されます。

リスト11-19で示されるように、`condition`パラメーターを提供した場合、リモート呼び出しはブラウザーサイド(JavaScript)で実行されるテストによって調整されます。

リスト11-19 - クライアントサイドのテストにしたがって条件つきでリモート関数を呼び出す

    [php]
    <div id="feedback"></div>
    <?php echo link_to_remote('この投稿を削除する', array(
        'update'    => 'feedback',
        'url'       => 'post/delete?id='.$post->getId(),
        'condition' => "$('elementID') == true",
    )) ?>

### Ajaxのリクエストメソッドを決定する

デフォルトでは、AjaxのリクエストはPOSTメソッドで構成されます。データを修正しないAjaxの呼び出しを行いたい場合、もしくはAjaxの呼び出しの結果として組み込みのバリデーションを持つフォームを表示したい場合、GETするAjaxのリクエストメソッドを変更することが必要な場合があります。リスト11-20で示されるように、メソッドオプションはAjaxのリクエストを変更します。

リスト11-20 - Ajaxのリクエストメソッドを変更する

    [php]
    <div id="feedback"></div>
    <?php echo link_to_remote('この投稿を削除する', array(
        'update'    => 'feedback',
        'url'       => 'post/delete?id='.$post->getId(),
        'method'    => 'get',
    )) ?>

### スクリプトの実行を許可する

Ajaxの呼び出しのレスポンスコード(`update`要素に挿入され、サーバーによって送られてきたコード)がJavaScriptを含む場合、デフォルトでこれらのスクリプトが実行されないことに驚くかもしれません。これはリモート攻撃のリスクを減らし開発者がレスポンス内に存在するコードを確実に知っているときのみスクリプトの実行を可能にすることが目的です。

リモートレスポンスのなかでスクリプトの実行を行う機能を`script`オプションによって明確に宣言する必要があるのはそういうわけです。リスト11-21はリモートレスポンスから実行できるJavaScriptを宣言するAjaxの呼び出しの例を示しています。

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

リモートテンプレートが(`remote_function()`といった)Ajaxヘルパーを含む場合、これらのPHP関数はJavaScriptのコードを生成し、`'script' => true`オプションを追加しないかぎりこれらは実行されないことに注意してください。

>**NOTE**
>リモートレスポンスに対してスクリプトの実行を有効にしても、生成したコードをチェックするツールを使わないのであれば、リモートコードでコードを実際に見ることはありません。スクリプトは実行されますが、コードには現れません、奇妙なことかもしれませんが、このふるまいは完全に普通です。

### コールバック機能を作成する

Ajaxのインタラクションの重大な欠点は更新する領域が実際に更新されるまでユーザーには見えないことです。遅いネットワーク、もしくはサーバー障害の場合、実際は処理されなかったにもかかわらず、ユーザーは自分のアクションが反映されたと信じるかもしれません。Ajaxのインタラクションでユーザーのイベントを通知することが重要であるのはそういうわけです。

デフォルトでは、さまざまなJavaScriptのコールバックが(進行インディケータなどに対して)起動される間それぞれのリモートリクエストは非同期プロセスです。すべてのコールバックは`request`オブジェクトにアクセスできます。`request`オブジェクトは`XMLHtppRequest`を保持します。コールバックはAjaxのインタラクションのイベントに対応します:

  * `before`: リクエストが初期化される前
  * `after`: リクエストが初期化された直あとでロードされる前
  * `loading`: リモートレスポンスがブラウザーによってロードされているとき
  * `loaded`: ブラウザーがリモートレスポンスをロードすることをを終わらせたとき
  * `interactive`: ロードが終了していなくても、ユーザーがリモートレスポンスと相互作用するとき
  * `success`: `XMLHttpRequest`が完結し、HTTPステータスコードが2XXの範囲にあるとき
  * `failure`: `XMLHttpRequest`が完結し、HTTPステータスコードが2XXの範囲にないとき
  * `404`: リクエストが404ステータスを返すとき
  * `complete`: `XMLHttpRequest`が完結したとき(それらが存在する場合、success`もしくは`failure`のあとで起動します)

たとえば、リモート呼び出しが初期化されレスポンスが一度受信されたとき、これを隠すために、ローディングインディケータを表示することはよくあることです。これを実現するには、リスト11-22で示されるように、`loading`と`complete`パラメーターをAjaxの呼び出しに追加します。

リスト11-22 - アクティビティインディケータの表示と隠匿を行うためにAjaxのコールバックを使う

    [php]
    <div id="feedback"></div>
    <div id="indicator">読み込み中...</div>
    <?php echo link_to_remote('この投稿を削除する', array(
        'update'   => 'feedback',
        'url'      => 'post/delete?id='.$post->getId(),
        'loading'  => "Element.show('indicator')",
        'complete' => "Element.hide('indicator')",
    )) ?>

`show`メソッドと`hide`メソッドは、JavaScriptのElementオブジェクトと同様に、Prototypeのそのほかの便利な追加機能です。

視覚効果を作成する
------------------

Webページで`<div>`要素を表示したり隠したりする以上のことをできるようにするために、symfonyはscript.aculo.usのライブラリの視覚効果を統合します。視覚効果の構文は[http://script.aculo.us/](http://script.aculo.us/)のwikiでよいドキュメントが見つかります。基本的には、script.aculo.usのライブラリは複雑な視覚効果を実現するためにDOMを操作するJavaScriptのオブジェクトと関数を提供します。リスト11-23でいくつかの例をご覧ください。結果はWebページの特定領域上の視覚的なアニメーションなので、これらが実際に何をしているのかを理解するために自分で効果を確かめることをお勧めします。script.aclulo.usのWebサイトは動的な効果のアイディアを得ることができるギャラリーを提供します。

リスト11-23 - Script.acuo.usによるJavaScriptの視覚効果

    [php]
    // 'my_field'要素をハイライトする
    Effect.Highlight('my_field', { startcolor:'#ff99ff', endcolor:'#999999' })

    // 要素をブラインドダウンする
    Effect.BlindDown('id_of_element');

    // 要素をフェードアウェイする
    Effect.Fade('id_of_element', { transition: Effect.Transitions.wobble })

symfonyは、`Javascript`ヘルパーグループの一部でもある、`visual_effect()`ヘルパーで`JavaScriptの`Effect`オブジェクトをカプセル化します。リスト11-24で示されるように、このヘルパーは通常のリンクで使えるJavaScriptを出力します。

リスト11-24 `visual_effect()`ヘルパーによるテンプレートでの視覚効果

    [php]
    <div id="secret_div" style="display:none">最初からずっと存在していました！</div>
    <?php echo link_to_function(
      '秘密のdivを表示する',
      visual_effect('appear', 'secret_div')
    ) ?>
    // はEffect.Appear('secret_div')への呼び出しを行う

リスト11-25で示されるように、`visual_effects()`ヘルパーはAjaxのコールバックでも利用できます。このヘルパーはリスト11-22のようなアクティビティインディケータを表示しますが、視覚的にはより満足のゆくものです。`indicator`要素はAjaxの呼び出しが始まるときに次第に表示され、レスポンスが到達したときに次第に消えます。加えて、`feedback`要素は、ユーザーの注目をこの部分のウィンドウに引きつけるために、リモート呼び出しによって更新されていた後にハイライトされます。

リスト11-25 - Ajaxのコールバックにおける視覚効果

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

JSON(JavaScript Object Notation)は軽量のデータ交換フォーマットです。基本的には、オブジェクトの情報を運ぶJavaScriptのハッシュ(リスト11-26の例を参照)にすぎません。しかし、JSONはAjaxのインタラクションに対して2つのすばらしい利点があります: JavaScriptのなかで読みやすく、Webレスポンスのサイズを減らすことができます。

リスト11-26 - JavaScriptのなかのサンプルのJSONオブジェクト

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

AjaxのアクションがさらなるJavaScriptの処理に対してコーラー(caller)のページへ構造化データを返す必要がある場合、JSONはレスポンスのためのよいフォーマットです。たとえば、1つのAjaxの呼び出しがコーラーのページでいくつかの要素を更新する場合において非常に便利です。

たとえば、リスト11-27のようなコーラーのページを想像してください。このページは更新することが必要かもしれない2つの要素を持ちます。1つのリモートヘルパーは、ページの要素の両方ではなく片方(`title`もしくは`name`)だけを更新できます。

リスト11-27 - 複数のAjaxを更新するためのサンプルのテンプレート

    [php]
    <h1 id="title">基本的な手紙</h1>
    <p>親愛なる<span id="name">name_here</span>へ、</p>
    <p>あなたのEメールは受信され近いうちに返信されます。</p>
    <p>Sincerely,</p>

両方を更新するために、Ajaxのレスポンスがつぎの配列を含むJSONのヘッダーになることができるか想像してください:

     [["title", "My basic letter"], ["name", "Mr Brown"]]

それからリモート呼び出しは、JavaScriptのちょっとした助けによって、簡単にこのレスポンスを解釈し、列のなかのフィールドを更新できます。リスト11-28のコードはこの効果を得るためにリスト11-27のテンプレートに追加できるものを示しています。

リスト11-28 - リモートレスポンスから複数の要素を更新する

    [php]
    <?php echo link_to_remote('文字列をリフレッシュする', array(
      'url'      => 'publishing/refresh',
      'complete' => 'updateJSON(request, json)'
    )) ?>

    <?php echo javascript_tag("
    function updateJSON(request, json)
    {
      var nbElementsInResponse = json.length;
      for (var i = 0; i < nbElementsInResponse; i++)
      {
         Element.update(json[i][0], json[i][1]);
      }
    }
    ") ?>

`complete`コールバックはレスポンスの`json`ヘッダーにアクセス可能でこれをサードパーティの関数に渡すことができます。このカスタム`updateJSON()`関数はJSONヘッダーのイテレーションを行い、それぞれの配列のメンバーに対して、`second`パラメーターの内容を持つ最初のパラメーターによって名づけられた要素を更新します。

リスト11-29は`publishing/refresh`アクションがJSONのレスポンスを返す方法を示してます。

リスト11-29 - JSONのヘッダーを返すサンプルのアクション

    [php]
    class publishingActions extends sfActions
    {
      public function executeRefresh()
      {
        $output = '[["title", "基本的な手紙"], ["name", "ブラウンさん "]]';
        $this->getResponse()->setHttpHeader("X-JSON", '('.$output.')');

        return sfView::HEADER_ONLY;
      }

HTTPプロトコルはJSONをレスポンスヘッダーに保存することを許可します。レスポンスは内容を持たないので、アクションはこれをヘッダーのみとして即座に送ります。これはビューレイヤーを完全に回避し、`renderText()`と同じぐらい速いですが、より小さなレスポンスをともないます。

>**CAUTION**
>リスト11-29で示されるアプローチにはいくつかの制限があります: HTTPヘッダーの最大サイズです。公式の制限はありませんが、大きなヘッダーは十分に転送されないもしくはブラウザーによって解釈されないことがあります。このことは、JSONの配列が大きい場合、リモートアクションは、JavaScript配列としてJSONを持つ、通常のレスポンスを返すことを意味します。

JSONはWebアプリケーションのあいだで標準規格となりました。Webサービスは、サーバーよりもクライアント上でのサービスの統合(マッシュアップ)を可能にするために、XMLよりもJSON形式のレスポンスをよく提示します。サーバーとJavaScript関数の間のコミュニケーションのためにどちらのフォーマットを使うべきか悩んでいる場合、おそらくJSONはあなたの最善の方法です。

>**TIP**
>バージョン5.2以降、PHPは`json_encode()`と`json_decode()`の2つの関数を提供します。これらによってPHPとJSONの構文のあいだで配列を変換できるようになります。逆もしかりです([http://www.php.net/manual/ref.json.php](http://www.php.net/manual/ref.json.php))。これらは一般的なJSON配列とAjaxの統合を円滑にします。

複雑なインタラクションをAjaxで実行する
--------------------------------------

symfonyのAjaxヘルパーのなかには、単独の呼び出しで複雑なインタラクションを組み立てるいくつかのツールも見つかります。これらのツールによって複雑なJavaScriptのコードをともなわずにデスクトップのアプリケーションのようなインタラクション(ドラッグドロップ、オートコンプリート、ライブ編集)によってユーザーエクスペリエンスを強化できます。つぎのセクションでは複雑なインタラクションのためのヘルパーを説明し、シンプルな例を示します。追加パラメーターと調整方法はscript.aculo.usのドキュメントで説明されます。

>**CAUTION**
>複雑なインタラクションが可能であれば、これらが自然に感じられるように調整を行うプレゼンテーションのための追加時間が必要です。これらがユーザーエクスペリエンスを強化することを確信するときのみこれらをご利用ください。これらがユーザーを混乱させるリスクが存在するのであればこれらを避けてください。

### オートコンプリート

ユーザーが入力している間にユーザーのエントリーをマッチする単語のリストを表示するテキスト入力コンポーネントはオートコンプリート(autocompletetion)と呼ばれます。`input_auto_complete_tag()`と呼ばれる単独のヘルパーによって、リモートアクションがリスト11-30で示された例と似たようなHTMLのアイテムとしてフォーマットされたレスポンスを返す条件の下で、この効果を実現できます。

リスト11-30 - オートコンプリートのタグと互換性のあるレスポンスの例

    <ul>
      <li>suggestion1</li>
      <li>suggestion2</li>
      ...
    </ul>

リスト11-31で示されているつぎの例のように、通常のテキスト入力を処理するようにヘルパーをテンプレートのなかに挿入してください。

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

ユーザーが`author`フィールドに文字を入力するたびにこれは`author/autocomplete`アクションを呼び出します。アクションが`author`のリクエストパラメーターにしたがって可能なマッチのリストを決定し、リスト11-30と同じようなフォーマットでこれらを返すように設計するのはあなた次第です。図11-3で示されるように、ヘルパーは`author`タグの下でリストを表示し、提示された項目の1つをクリックする、もしくはキーボードで選択すると入力が補完されます。

図11-3 - オートコンプリートの例

![オートコンプリートの例](http://github.com/masakielastic/symfonybook-ja/raw/master/images/F1103.png "オートコンプリートの例")

`input_auto_complete_tag()`ヘルパーの3番目の引数はつぎのパラメーターを取ることができます:

  * `use_style`: レスポンスが自動的に一覧を表示するスタイル。
  * `frequency`: 周期的な呼び出しの頻度(デフォルトは0.4秒)。
  * `indicator`: オートコンプリートのローディングが始まり完了した場合に消えるときに表示されるインディケータのid。
  * tokens: トークン化されたインクリメンタルなオートコンプリートを許可するため。たとえば、このパラメーターを`,`に設定してユーザーが`jane, george`を入力すると、アクションは`'george'`の値だけを受けとります。

### ドラッグアンドドロップ

要素をマウスで掴む、動かす、どこかで放すといった機能はデスクトップのアプリケーションでは親しまれていますが、Webブラウザーにおいてはまれです。無地のJavaScriptにおいてこのようなふるまいをコードに書く作業がとても非常に複雑だからです。幸いにして、symfonyにおいてこの機能を実現するには1行だけ必要です。

symfonyは`draggable_element()`と`drop_receiving_element()`の2つのヘルパーを提供します。これらをふるまいの修飾子としてみなすことができます; これらはオブザーバと機能をこれが扱う要素に追加します。ドラッグ可能な要素に対して要素をドラッグ可能な要素もしくは受けとる要素として宣言するためにこれらを使います。ドラッグ可能な要素はこれをマウスでクリックすることでつかみ取ることができます。マウスボタンが放されるまで、要素を動かす、もしくはウィンドウを越えてドラッグされます。`ドラッグ可能な要素が放されたとき、受けとる要素はリモート関数を呼び出します。リスト11-32は要素を受けとるショッピングカードとこのタイプのインタラクションのお手本を示しています。

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

並べ替えをされていないリストのそれぞれの項目をマウスでつかみ取ることが可能で、ウィンドウを越えてドラッグされます。放されたとき、これらは元の位置に戻ります。`cart`要素を越えて放されたとき、`cart/add`アクションへのリモート呼び出しを発動させます。`id`リクエストパラメーターを見ることで、アクションはどの品物が`cart`要素にドロップされたのか決定できます。リスト11-32は本当のショッピングセッションをシミュレートしています: 品物をつかみとり、カートで放します。会計に進みます。

>**TIP**
>リスト11-32において、ヘルパーが修正する要素の直後にヘルパーが書かれますが、これは要件ではありません。テンプレートの終わりですべての`draggalbe_element()`と`drop_receiving_element()`ヘルパーをとても上手に分類できました。重要なことはヘルパー呼び出しの最初の引数で、これはふるまいを受けとる要素の識別子を指定します。

`draggable_element()`ヘルパーはつぎのパラメーターを受け入れます:

  * `revert`: `true`に設定した場合、要素は放されたときに元の場所に戻ります。ドラッグが終了したとき、任意の関数参照にもなり、呼び出されます。
  * `ghosting`: 要素をクローンしてそのクローンをドラッグし、クローンがドロップされるまで、所定の位置に留めます。
  * `snap`: `false`に設定した場合、スナッピングは起きません。さもなければ、`x`と`y`の間隔のグリッドの交点だけに`draggable`がドラッグされます。そしてこの場合、`xy`もしくは`[x,y]`もしくは`function(x,y){return [x,y]}`の形式をとります。

`drop_receiving_element()`ヘルパーはつぎのパラメーターを受けとります:

  * `accept`: 文字列もしくはCSSクラスを記述する文字列の配列。要素はこれらのCSSクラスを1つもしくは複数持つドラッグ可能な要素だけを受けとります。
  * `hoverclass`: ドラッグ可能な受けとり要素をある要素の上にドラッグするときにその要素に追加されるCSSクラス

### ソート可能なリスト

`draggable`要素によって提供されたほかの可能性はマウスで項目を動かすことでリストをソートする機能です。`sortable_element()`ヘルパーはソート可能なふるまいを項目に追加します。リスト11-33はこの機能を実装するよい例です。

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

`sortable_element()`ヘルパーのマジックによって、`<ul>`要素はソート可能になります。このことはその子要素をドラッグアンドドロップで再び並べ替えることができることを意味します。リストを並べ替えるためにユーザーが項目をドラッグして放すたびに、Ajaxリクエストはつぎのパラメーターによって構成されます:

    POST /sf_sandbox/web/frontend_dev.php/item/sort HTTP/1.1
      order[]=1&order[]=3&order[]=2&_=

十分に並べ替えられたリストは配列として渡されます(`orde[$rank]=$id`のフォーマットで、`$rank`は`0`で始まり、`$id`はリスト要素の`id`プロパティのアンダースコア(`_`)のあとで来るものに基づきます)。`id`プロパティのソート可能な要素(この例では`order`)はパラメーターの配列を名づけるために使われます。

`sortable_element()`ヘルパーはつぎのパラメーターを受けとります:

  * `only`: CSSクラスを記述する文字列もしくは文字列の配列。このクラスをともなうsortable要素の子要素だけが動かせる。
  * `hoverclass`: マウスが要素の上にホーバーされたときにその要素に追加されるCSSクラス。
  * `overlap`: 項目がインラインに表示される場合は`horizontal`に設定、1行につき1つの項目があるとき(例の場合)、`vertical`に設定します。
  * `tag`: リストの順序が`<li>`要素のセットではない場合、ドラッグ可能なものにするためにソート可能な子要素を決定しなければなりません(たとえば、`div`もしくは`dl`)。

### その場で編集する

より多くのWebアプリケーションは、フォームにおいて内容を再表示することなく、ページ上でユーザーがページの内容を直接編集することを許可しています。インタラクションの原則はシンプルであることです。テキストのブロックはユーザーがその上にマウスをホーバーするときにハイライトされます。ユーザーがブロックの内側でクリックする場合、プレーンテキストはブロックのテキストで入力されたテキストエリアに変換され、保存ボタンが表示されます。ユーザーはテキストエリアの内側でテキストを編集することが可能で、いったんそれを保存すると、テキストエリアとテキストはプレーンなフォームに表示されます。symfonyにおいて、`input_in_place_editor_tag()`ヘルパーによってこの編集可能なふるまいを要素に追加できます。リスト11-34はこのヘルパーの使いかたのお手本を示しています。

リスト11-34 - 編集可能なテキストの例

    [php]
    <div id="edit_me">このテキストを編集できます</div>
    <?php echo input_in_place_editor_tag('edit_me', 'mymodule/myaction', array(
      'cols'        => 40,
      'rows'        => 10,
    )) ?>

ユーザーが編集可能なテキストをクリックしたとき、編集可能で、テキストで満たされたテキスト入力エリアによって置き換えられます。フォームが投稿されたとき、`mymodule/myaction`アクションは`value`パラメーターとしての編集された値のセットを伴ってAjaxのなかに呼び出されます。アクションの結果は編集可能な要素を更新します。これはとても速く書くことが可能でとても強力です。

`input_in_place_editor_tag()`ヘルパーはつぎのパラメーターを受けとります:

  * `cols`と`rows`: 編集のために表示されるテキスト入力領域のサイズ(`rows`が1より大きい場合は`<textarea>`になります)
  * `loadTextURL`: 編集するテキストを表示するために呼び出されるアクションのURIです。編集可能な要素が特別な整形方法を利用することかつユーザーが整形なしのテキストを編集できるようにしたい場合にこれは便利です。
  * `save_text`と`cancel_text`: 保存リンク(デフォルトは"ok")とキャンセルリンク(デフォルトは"cancel")上のテキスト。

まとめ
----

クライアントサイドのふるまいを手に入れるためにテンプレートのなかでJavaScriptを書くことに疲れましたら、JavaScriptヘルパーは代わりのシンプルな方法を提供します。これらのヘルパーは基本的なリンクのふるまいと要素の更新を自動化するだけでなく、すぐにAjaxのインタラクションを開発する方法も提供します。強力なPrototypeによる構文強化とscript.aculo.usによるすばらしい視覚効果の助けによって、複雑なインタラクションを実現するには、数行のコードを書くことだけが必要です。

高度にインタラクティブなアプリケーションはsymfonyで静的なページを作ることと同じぐらい簡単なので、ほとんどすべてのデスクトップアプリケーションのインタラクションはWebアプリケーションで利用できると考えることができます。
