第17章 - symfonyを拡張する
==========================

結局のところ、symfonyのふるまいを変える必要があります。特定のクラスが振る舞う方法を修正するもしくは独自のカスタム機能を追加する必要があるにせよ、変更作業は必然です。symfonyが予想できない固有の要望を顧客が持つからです。実際のところ、この状況はよくありふれたことなのでsymfonyは既存のクラスを拡張するメカニズムを提供します。このメカニズムはミックスイン(mixin)と呼ばれます。ファクトリ(factory)の設定を利用してsymfonyのコアクラスをあなた独自のクラスに置き換えることもできます。いったん拡張機能(エクステンション)を作れば、プラグインとして簡単にパッケージにできるので、ほかのアプリケーションもしくはほかのsymfonyのユーザーが再利用できます。

ミックスイン
------------

PHPの現在の制約のなかで、もっとも悩ましいことは複数のクラスを継承できないことです。別の制約は既存のクラスに新しいメソッドを追加するもしくは既存のメソッドをオーバーライドできないことです。これら2つの制約を緩和し、symfonyを本当に拡張できるようにするため、symfonyは`sfMixer`と呼ばれるクラスを導入します。調理器具に関連する機能ではありませんが、ミックスインの概念はオブジェクト指向のプログラミングで見つかります(訳注：Rubyなど)。ミックスインとは拡張するためにクラスにミックスされるメソッドもしくは関数のグループです。

### 多重継承を理解する

多重継承は複数のクラスを拡張するためのクラスの機能で、これらのクラスプロパティとメソッドを継承します。例を考えてみましょう。リスト17-1のように、それぞれが独自のプロパティとメソッドを持つ`Story`クラスと`Book`クラスを想像してください。

リスト17-1 - 2つの例のクラス

    [php]
    class Story
    {
      protected $title = '';
      protected $topic = '';
      protected $characters = array();

      public function __construct($title = '', $topic = '', $characters = array())
      {
        $this->title = $title;
        $this->topic = $topic;
        $this->characters = $characters;
      }

      public function getSummary()
      {
        return $this->title.', a story about '.$this->topic;
      }
    }

    class Book
    {
      protected $isbn = 0;

      function setISBN($isbn = 0)
      {
        $this->isbn = $isbn;
      }

      public function getISBN()
      {
        return $this->isbn;
      }
    }

`ShortStory`クラスは`Story`クラスを継承し、`ComputerBook`クラスは`Book`クラスを継承し、論理的に、`Novel`クラスは`Story`クラスと`Book`クラスを継承して、これらすべてのメソッドを利用します。不幸なことに、PHPでは多重継承は不可能です。リスト17-2のように`Novel`宣言を書くことはできません。

リスト17-2 - PHPでは多重継承は不可能である

    [php]
    class Novel extends Story, Book
    {
    }

    $myNovel = new Novel();
    $myNovel->getISBN();

`Novel`クラスが2つのクラスを拡張する代わりに2つのインターフェイスを実装できる可能性がありますが、この方法では親クラスで実際にクラスを書くことができません。

### クラスをミックスする

`sfMixer`クラスは問題に対して別のアプローチをとります。クラスが適切なフックを含むという前提のもとで、既存のクラスを受けとり、後天的にこれを拡張します。この手順では2つのステップを含みます。

  * クラスを拡張可能なものとして宣言する
  * クラス宣言の後に拡張機能(もしくはミックスイン)を登録する

リスト17-3は`sfMixer`クラスで`Novel`クラスを実装する方法を示しています。

リスト17-3 - `sfMixer`を通して多重継承は可能である

    [php]
    class Novel extends Story
    {
      public function __call($method, $arguments)
      {
        return sfMixer::callMixins();
      }
    }

    sfMixer::register('Novel', array('Book', 'getISBN'));
    $myNovel = new Novel();
    $myNovel->getISBN();

クラスの1つ(`Story`)はメインの親として選択され、1つのクラスからのみ継承できるというPHPの機能と一致しています。`Novel`クラスは`__call()`メソッドに設置されたコードによって拡張可能なものとして宣言されます。ほかのクラスのメソッド(`Book`)は`sfMixer::register()`への呼び出しによってあとで`Novel`クラスに追加されます。つぎのセクションではこのプロセスをわかりやすく説明します。

`Novel`クラスの`getISBN()`メソッドが呼び出されたとき、あたかもリスト17-2で定義されたクラスのようにすべてのことが起こります。それをシミュレートするのが`__call()`マジックメソッドと`sfMixer`staticメソッドであること以外は。`getISBN()`メソッドは`Novel`クラスにミックスされます。

>**SIDEBAR**
>ミックスインを使うとき
>
>多くの場合でsymfonyのミックスインのメカニズムは便利です。以前説明したような多重継承をシミュレートすることもこれらのメカニズムの1つです。
>
>宣言したあとでメソッドを変更するためにミックスインを利用できます。たとえば、グラフィックライブラリを作成するとき、おそらくは線を表す`Line`オブジェクトを実装することになります。このオブジェクトは4つの属性(端の座標)とそれ自身をレンダリングする`draw()`メソッドを持ちます。`ColoredLine`オブジェクトは同じプロパティとメソッドを持ちますが、色を指定する追加の`color`属性を持ちます。さらに、`ColoredLine`オブジェクトの`draw()`メソッドは、オブジェクトの色に関して、シンプルな`Line`のメソッドとは少し異なります。色を処理するグラフィカルな要素の機能を`ColoredElement`クラスにパッケージにすることができました。これによってほかのグラフィカルな要素のために`color`メソッドを再利用できるようになります(`Dot`、`Polygon`など)。この場合、`ColoredLine`オブジェクトの理想的な実装は、ミックスインした`ColoredElement`クラスからのメソッドで`Line`クラスの拡張です。最後の`draw()`メソッドは`Line`からのオリジナルのメソッドと`ColoredElement`からのメソッドとのミックスになります。
>
>ミックスインは新しいメソッドを既存のクラスに追加する方法とみなすこともできます。たとえば、`sfActions`と呼ばれるsymfonyのアクションクラスはsymfony内部で定義されます。PHPの制約の一つは初期に宣言したあとで`sfActions`の定義を変更できないことです。アプリケーションの1つだけに`sfActions`にカスタムメソッドを追加したい場合があるかもしれません。たとえば、リクエストを特別なWebサービスに転送することです。この目的のために、PHPだけでは不十分ですが、ミックスインのメカニズムは完璧な解決方法を提供します。

### クラスを拡張可能なものとして宣言する

クラスを拡張できるように宣言するには、1つもしくは複数の"フック"(hook)をコードに挿入しなければなりません。`sfMixer`クラスはフックによってあとでコードを識別できます。これらのフックは`sfMixer::callMixins()`メソッドへの呼び出しです。多くのsymfonyのクラス、`sfRequest`、`sfResponse`、`sfController`、`sfUser`、`sfAction`などにフックがすでに含まれています。

フックは望む拡張性の程度に応じて、クラスの異なる部分に設置できます:

  * 新しいメソッドをクラスに追加できるようにするため、リスト17-4で示されているように、フックを`__call()`メソッドに挿入して、結果を返さなければなりません。

リスト17-4 - クラスに新しいメソッドを取得する機能を与える

    [php]
    class SomeClass
    {
      public function __call($method, $arguments)
      {
        return sfMixer::callMixins();
      }
    }

  * 既存のメソッドの動作方法を変えるために、リスト17-5で示されているように、フックをメソッドの内側に挿入しなければなりません。ミックスインのクラスによって追加されたコードはフックが設置された場所で実行されます。

リスト17-5 - メソッドに変更できる機能を与える

    [php]
    class SomeOtherClass
    {
      public function doThings()
      {
        echo "I'm working...";
        sfMixer::callMixins();
      }
    }

メソッドに複数のフックを追加できます。この場合、フックを命名しなければなりませんので、リスト17-6で示されているように、どのフックがあとで実行されるのかを定義することができます。名前つきのフックはフックの名前をパラメーターとして`callMixins()`を呼び出します。実行されるミックスインのコードの場所を伝えるために、ミックスインを登録するときに、この名前があとで使われます。

リスト17-6 - メソッドは複数のフックを含むことができる。この場合フックに名前をつけなければならない

    [php]
    class AgainAnotherClass
    {
      public function doMoreThings()
      {
        echo "I'm ready.";
        sfMixer::callMixins('beginning');
        echo "I'm working...";
        sfMixer::callMixins('end');
        echo "I'm done.";
      }
    }

もちろん、リスト17-7が示すように、クラスを作るテクニックを新しく拡張可能なメソッドに割り当てる機能に結びつけることができます。

リスト17-7 - クラスはさまざまな方法で拡張可能である

    [php]
    class BicycleRider
    {
      protected $name = 'John';

      public function getName()
      {
        return $this->name;
      }

      public function sprint($distance)
      {
        echo $this->name."は".$distance."メートルをダッシュする\n";
        sfMixer::callMixins(); // sprint()メソッドは拡張可能
      }

      public function climb()
      {
        echo $this->name.'はよじ登る';
        sfMixer::callMixins('slope'); // climb()はここで拡張可能
        echo $this->name.'は山頂に到達する';
        sfMixer::callMixins('top'); // そしてここでも
      }

      public function __call($method, $arguments)
      {
        return sfMixer::callMixins(); // BicyleRiderクラスは拡張可能
      }
    }

>**CAUTION**
>拡張可能なものとして宣言されたクラスだけが`sfMixer`によって拡張可能です。このことはこのサービスを"購読"(subscribe)しなかったクラスを拡張するためにこのメカニズムを利用できないことを意味します。

### 拡張機能を登録する

拡張機能を既存のフックに登録するには、`sfMixer::register()`メソッドを使います。最初の引数は拡張する要素で、2番目の引数はPHPで呼び出し可能なもので、ミックスインを表します。

最初の引数の形式は拡張したいものに依ります:

  * クラスを拡張する場合、クラスの名前を使います。
  * 匿名のフックでメソッドを拡張する場合、`class::method`パターンを使います。
  * 名前つきのフックでメソッドを拡張する場合、`class::method:hook`パターンを使います。

リスト17-8はリスト17-7で定義されたクラスを拡張することでこの原則を説明しています。拡張されたオブジェクトは自動的に最初の引数として`Mixin`メソッドに渡されます(もちろん、拡張されたメソッドがstaticであることは除きます)。ミックスインメソッドはオリジナルのメソッド呼び出しのパラメーターも利用します。

リスト17-8 - 拡張機能を登録する

    [php]
    class Steroids
    {
      protected $brand = 'foobar';

      public function partyAllNight($bicycleRider)
      {
        echo $bicycleRider->getName()."は夜にダンスをして時間を費やす。\n";
        echo "ありがとう".$brand."！\n";
      }

      public function breakRecord($bicycleRider, $distance)
      {
        echo "誰も".$distance."メートルより速い記録を出したことはない！\n";
      }

      static function pass()
      {
        echo "。そして先頭集団の半分を超す。\n";
      }
    }

    sfMixer::register('BicycleRider', array('Steroids', 'partyAllNight'));
    sfMixer::register('BicycleRider:sprint', array('Steroids', 'breakRecord'));
    sfMixer::register('BicycleRider:climb:slope', array('Steroids', 'pass'));
    sfMixer::register('BicycleRider:climb:top', array('Steroids', 'pass'));

    $superRider = new BicycleRider();
    $superRider->climb();
    => Johnはよじ登る。そして先頭集団の半分を超す。
    => Johnは山頂に到達する。そして先頭集団の半分を超す。
    $superRider->sprint(2000);
    => Johnは2000メートルをダッシュする
    => 誰も2000メートルよりも速い記録を出したことがない！
    $superRider->partyAllNight();
    => Johnは夜にダンスをして時間を費やす。
    => ありがとうfoobar！

拡張メカニズムはメソッドを追加するだけではありません。`partyAllNight()`メソッドは`Steroids`クラスの属性を利用します。`Steroids`クラスのメソッドで`BicycleRider`クラスを拡張するとき、`BicycleRider`オブジェクトの内側に新しい`Steroids`インスタンスを実際に作ります。

>**CAUTION**
>同じ名前を持つ2つのメソッドを既存のクラスに追加できません。`__call()`メソッド内の`callMixins()`呼び出しがミックスインのメソッド名をキーとして使うからです、同じように、クラスのメソッドと同じ名前のメソッドをそのクラスに追加することもできません。ミックスインのメカニズムが`__call()`マジックメソッドに依存しており、この特殊なケースにおいては、これは決して呼び出しされないからです。

`register()`呼び出しの2番目の引数はPHPで呼び出し可能なので、`class::method`配列か`object->method`配列か、関数名になることができます。リスト17-9の例をご覧ください。

リスト17-9 - PHPで呼び出し可能なものはmixer拡張機能として登録できる

    [php]
    // PHPで呼び出し可能なものとしてクラスメソッドを使う
    sfMixer::register('BicycleRider', array('Steroids', 'partyAllNight'));

    // PHPで呼び出し可能なものとしてオブジェクトメソッドを使う
    $mySteroids = new Steroids();
    sfMixer::register('BicycleRider', array($mySteroids, 'partyAllNight'));

    // PHPで呼び出し可能なものとして関数を使う
    sfMixer::register('BicycleRider', 'die');

拡張機能のメカニズムは動的であり、すでにオブジェクトをインスタンス化していても、クラスでさらなる拡張機能を利用できることを意味します。リスト17-10の例をご覧ください。

リスト17-10 - 拡張機能のメカニズムは動的でインスタンス化のあとでも行われれる

    [php]
    $simpleRider = new BicycleRider();
    $simpleRider->sprint(500);
    => Johnは500メートルをダッシュする
    sfMixer::register('BicycleRider:sprint', array('Steroids', 'breakRecord'));
    $simpleRider->sprint(500);
    => Johnは500メートルをダッシュする
    => 誰も500メートルより速い記録を出したことがない！

### より精密に拡張する

`sfMixer::callMixins()`のインストラクションは実際には少し手の込んだものへのショートカットです。これは現在のオブジェクトと現在のメソッドのパラメーターを渡しながら、自動的に登録されたミックスインのリストをループしてこれらを1つずつ呼び出しています。要するに、`sfMixer::callMixins()`呼び出しはおよそリスト17-11のようにふるまいます。

リスト17-11 - `callMixin()`は登録されたミックスインをループして、これらを実行する

    [php]
    foreach (sfMixer::getCallables($class.':'.$method.':'.$hookName) as $callable)
    {
      call_user_func_array($callable, $parameters);
    }

ほかのパラメーターを渡したい場合もしくは戻り値で特別な何かをしたい場合、ショートカットのメソッドを利用する代わりに`foreach`ループを明示的に書けます。ミックスインがクラスに統合される例についてリスト17-12をご覧ください。

リスト17-12 - カスタムループで`callMixin()`を置き換える

    [php]
    class Income
    {
      protected $amount = 0;

      public function calculateTaxes($rate = 0)
      {
        $taxes = $this->amount * $rate;
        foreach (sfMixer::getCallables('Income:calculateTaxes') as $callable)
        {
          $taxes += call_user_func($callable, $this->amount, $rate);
        }

        return $taxes;
      }
    }

    class FixedTax
    {
      protected $minIncome = 10000;
      protected $taxAmount = 500;

      public function calculateTaxes($amount)
      {
        return ($amount > $this->minIncome) ? $this->taxAmount : 0;
      }
    }

    sfMixer::register('Income:calculateTaxes', array('FixedTax', 'calculateTaxes'));

>**SIDEBAR**
>Propelのビヘイビアー
>
>以前8章で説明しましたが、Propelのビヘイビアー(behavior)は特殊なミックスインです: これらはPropelが生成したオブジェクトを拡張します。例を見てみましょう。
>
>Propelのオブジェクトはすべてが`delete()`メソッドを持つデータベースのテーブルに対応します。`delete`メソッドはデータベースから関連するレコードを削除します。しかし、`Invoice`クラスに対して、レコードを削除できないようにするために、データベース内部のレコードを維持できるように`delete()`メソッドを変更して、`is_deleted`属性の値をtrueに変更したい場合があります。通常のオブジェクトの読みとりメソッド(`doSelect()`、`retrieveByPk()`)は`is_deleted`が`false`であるとレコードとみなすだけです。`forceDelete()`と呼ばれる別のメソッドを追加する必要があります。このメソッドによってレコードを本当に削除できます。実際、これらすべての修正は`ParanoidBehavior`と呼ばれる新しいクラスにまとめられます。最後の`Invoice`クラスはPropelの`BaseInvoice`クラスを拡張し、ミックスインされた`ParanoidBehaviorMixin`のメソッドを持ちます。
>
>ビヘイビアーはPropelオブジェクト上のミックスインです。実際に、symfonyにおいて"ビヘイビアー"という用語は複数の内容をカバーします: ミックスインはプラグインとしてパッケージになります。丁度述べた`ParanoidBehavior`クラスは`sfPropelParanoidBehaivorPlugin`と呼ばれる実在するsymfonyのプラグインに対応します。インストールとこのプラグインの使いかたに関する詳細な内容はsymfonyの公式サイトのwiki([http://trac.symfony-project.org/wiki/sfPropelParanoidBehaviorPlugin](http://trac.symfony-project.org/wiki/sfPropelParanoidBehaviorPlugin))を参照してください。
>
>ビヘイビアーに関して最後の一言です: これらをサポートできるようにするには、生成されたPropelのオブジェクトは多くのフックを含まなければなりません。ビヘイビアーを使わない場合、これらが実行を遅くしてパフォーマンスにペナルティを課す可能性があります。フックがデフォルトで有効になっていないのはそういうわけです。ビヘイビアーのサポートを有効にするには、最初に`propel.ini`ファイルのなかで`propel.builder.addBehaviors`プロパティを`true`に設定してモデルをリビルドしなければなりません。

ファクトリ
----------

ファクトリ(factory)は特定のタスクのためのクラスの定義です。symfonyはコントローラーやセッション機能などのコア機能についてファクトリに依存します。たとえば、symfonyが新しいリクエストオブジェクトを作る必要がある場合、symfonyはこの目的のために使うクラスの名前のためにファクトリの定義を検索します。リクエストのためのデフォルトのファクトリの定義は`sfWebRequest`クラスで、symfonyはリクエストを処理するためにこのクラスのオブジェクトを作ります。ファクトリの定義を使う利点はsymfonyのコア機能をとても簡単に変更できることです: ファクトリの定義を変更し、symfonyは自身の代わりにカスタムリクエストクラスを使います。

ファクトリの定義は`factories.yml`設定ファイルに保存されます。リスト17-3はデフォルトのファクトリの定義ファイルを示します。それぞれの定義はオートロードされるクラスの名前と(オプションとしての)パラメーターの一式から構成されます。たとえば、セッションストレージのファクトリ(`storage:`キーの下に設定する)は一貫したセッションを可能にするためにクライアントコンピュータ上で作成されたCookieに名前をつける`session_name`パラメーターを使います。

リスト17-13 - デフォルトのファクトリファイル(`myapp/config/factories.yml`)

    cli:
      controller:
        class: sfConsoleController
      request:
        class: sfConsoleRequest

    test:
      storage:
        class: sfSessionTestStorage

    #all:
    #  controller:
    #    class: sfFrontWebController
    #
    #  request:
    #    class: sfWebRequest
    #
    #  response:
    #    class: sfWebResponse
    #
    #  user:
    #    class: myUser
    #
    #  storage:
    #    class: sfSessionStorage
    #    param:
    #      session_name: symfony
    #
    #  view_cache:
    #    class: sfFileCache
    #    param:
    #      automaticCleaningFactor: 0
    #      cacheDir:                %SF_TEMPLATE_CACHE_DIR%

ファクトリを変更する最良の方法はデフォルトのファクトリから継承した新しいクラスを作り、新しいメソッドをそのクラスに追加することです。たとえば、ユーザーセッションのファクトリは`myUser`クラス(`myapp/lib/`ディレクトリに設置)に設定され、`sfUser`クラスから継承します。既存のファクトリを利用するには同じメカニズムを使います。リスト17-14はリクエストオブジェクトのための新しいファクトリの例を示しています。

リスト17-14 - ファクトリをオーバーライドする

    [php]
    //オートロードされたディレクトリ内でmyRequest.class.phpを作る
    // たとえばmyapp/lib/において
    <?php

    class myRequest extends sfRequest
    {
      // あなたのコードをここに
    }

    // factories.ymlでクラスをリクエストファクトリとして宣言する
    all:
      request:
        class: myRequest

ほかのフレームワークへのブリッジ
------------------------------

サードパーティのクラスによって提供された機能が必要な場合、このクラスを`lib/`ディレクトリの1つにコピーしたくない場合、おそらくはsymfonyがファイルを探す通常の場所の外側にそのクラスをインストールすることになります。この場合、クラスを利用するには、オートロードを利用するsymfonyのブリッジ機能を使わないかぎり、`requre`ステートメントを手動でコードに含めることになります。

symfonyは(まだ)すべてのためのツールを提供していません。PDFジェネレーター、Google MapsのAPI、PHPによるLucene検索エンジンの実装など、おそらくZend Frameworkからいくつかのライブラリが必要になります。PHPで直接イメージを操作する、Eメールを読むためにPOP3アカウントに接続する、コンソールのインターフェイスを設計することなどを行いたい場合、eZcomponentsからライブラリを選ぶことがあるかもしれません。幸いにして、正しい設定を定義をすると、これらのライブラリからのコンポーネントはsymfonyで正常に動作します。

(PEAR経由でサードパーティのライブラリをインストールしないかぎり)最初に宣言するものはライブラリのrootディレクトリへのパスです。これはアプリケーションの`settings.yml`設定ファイルで行われます:

    .settings:
      zend_lib_dir:   /usr/local/zend/library/
      ez_lib_dir:     /usr/local/ezcomponents/

それから、symfonyでオートロードが失敗するとき考慮するライブラリを指定することでオートロードのルーチンを拡張します:

    .settings:
      autoloading_functions:
        - [sfZendFrameworkBridge, autoload]
        - [sfEzComponentsBridge,  autoload]

この設定は`autoload.yml`で定義されたルールとは異なります(このルールに関する詳しい情報は19章を参照)。`autoloading_functions`設定はブリッジクラスを指定し、`autoload.yml`ファイルは検索のためのパスとルールを指定します。つぎの内容はロードされていないクラスの新しいオブジェクトを作るときに起きることを説明しています:

  1. symfonyのオートロード機能(`sfCore::spAutoload()`)は最初`autoload.yml`ファイルで宣言されたパスでクラスを探します。
  2. 何も見つからない場合、`sf_autoloading_functions`設定で宣言されたコールバックメソッドが順番に呼び出されます。これらの1つが`true`を返すまで続きます
  3. `sfZendFrameworkBridge::autoload()`
  4. `sfEzComponentsBridge::autoload()`
  5. これらが`false`を返す場合、PHP 5.0.Xを利用しているのであればsymfonyはクラスが存在しないことを伝える例外を投じ、PHP 5.1を利用している場合、エラーはPHP自身によって作られます。

このことはほかのフレームワークコンポーネントがオートロードメカニズムから恩恵を受けることを意味し、独自の環境よりもこれらを簡単に利用できます。たとえば、PHPのLucene検索エンジンと同等のものを実装するためにZend Frameworkの`Zend_Search`コンポーネントを使いたい場合、つぎのように書かなければなりません:

    [php]
    require_once 'Zend/Search/Lucene.php';
    $doc = new Zend_Search_Lucene_Document();
    $doc->addField(Zend_Search_Lucene_Field::Text('url', $docUrl));
    ...

symfonyとZend Frameworkのブリッジ機能によって、上記のコードはよりシンプルになります。つぎのように書くだけです:

    [php]
    $doc = new Zend_Search_Lucene_Document(); // クラスがオートロードされた
    $doc->addField(Zend_Search_Lucene_Field::Text('url', $docUrl));
    ...

利用可能なブリッジ機能は`$sf_symfony_lib_dir/addon/bridge/`ディレクトリに保存されます。

プラグイン
----------

1つのsymfonyアプリケーションの1つのために開発したコードピースの再利用がおそらく必要になります。このコードのピースを単独のクラスのパッケージにすることができるのであれば、問題ありません: クラスを別のアプリケーションの`lib/`フォルダーの1つに設置すればオートローダが残りを引き受けます。しかし、コードが複数のファイルに散在している場合、たとえば、administrationジェネレーター用の完全に新しいテーマ、もしくは好みの視覚効果を自動化するJavaScriptファイルとヘルパーの組み合わせなどの場合、ファイルをコピーするだけの方法は最良の解決方法ではありません。

プラグインはいくつかのファイルにまたがるコードをパッケージにする方法と、いくつかのプロジェクトをまたがってこのコードを再利用する方法を提供します。プラグインのなかで、クラス、フィルター、ミックスイン、ヘルパー、設定、タスク、モジュール、スキーマ、モデルの拡張、フィクスチャ、Webアセットなどをパッケージにすることができます。プラグインをインストール、アップグレード、アンインストールする方法は簡単です。これらは`.tgz`アーカイブ、PEARパッケージ、もしくはコードリポジトリとして配布可能で、コードのリポジトリから簡単にチェックアウトできます。PEARのパッケージとなったプラグインは依存関係の管理機能を利用し、アップグレードと自動検出が簡単です。symfonyのロードメカニズムはプラグインを考慮し、プラグインによって提供される機能はあたかもフレームワークの一部であるかのようにプロジェクトで利用できます。

ですので、プラグインは基本的にsymfonyプロジェクトのために拡張機能をパッケージにしたものです。プラグインによってアプリケーションを越えて独自コードを再利用できるだけでなく、別の投稿者によって開発されたものも再利用可能でsymfonyコアにサードパーティの拡張機能を追加できます。

### symfonyのプラグインを見つける

symfonyの公式サイトにはプラグイン専用のページが存在しており、つぎのURLからアクセスできます:

    http://www.symfony-project.org/plugins/

そこにあるそれぞれのプラグインのリストは詳細なインストールの手引きとドキュメントが一緒になった独自のページを持ちます。

これらのプラグインのなかにはコミュニティから投稿されたもの、symfonyのコア開発者からもたらされたものがあります。後者には、つぎのようなものが見つかります:

  * `sfFeedPlugin`: RSSとAtomフィードの操作を自動化する
  * `sfThumbnailPlugin`: たとえばアップロードされたイメージのためにサムネイルを作る。
  * `sfMediaLibraryPlugin`: メディアのアップロードと管理を可能にします。リッチテキスト内部でイメージの編集を可能にするリッチテキストエディタのための拡張機能を含む
  * `sfShoppingCartPlugin`: ショッピングカートの運用を可能にする
  * `sfPagerNavigationPlugin`: `sfPager`オブジェクトに基づいた古典的でAjaxによるページャーコントロールを提供する
  * `sfGuardPlugin`: 認証、承認とsymfonyの標準的なセキュリティ機能を上回る別のユーザーの管理機能を提供する
  * `sfPrototypePlugin`: prototypeとscript.aculo.usのJavaScriptファイルをスタンドアロンのライブラリとして提供する
  * `sfSuperCachePlugin`: できるかぎりWebサーバーが速くなるように、webディレクトリのrootの元の`cache`ディレクトリでページを書く
  * `sfOptimizerPlugin`: 運用環境において実行が速くなるようにアプリケーションのコードを最適化する(詳細はつぎの章を参照)
  * `sfErrorLoggerPlugin`: データベースにすべての404エラーと500エラーをログに記録しこれらのエラーを閲覧するためのadministrationモジュールを提供する
  * `sfSslRequirementPlugin`: アクションのためのSSL暗号化サポートを提供する

公式サイトはビヘイビアー(behavior)と呼ばれる、Propelオブジェクトを拡張するために設計されたプラグインも提示します。これらのなかで、つぎのものが見つかります:

  * `sfPropelParanoidBehaviorPlugin`: オブジェクトの削除を無効にして、`deleted_at`カラムの更新で置き換える
  * `sfPropelOptimisticLockBehaviorPlugin`: Propelオブジェクト用にオプティミスティックロック(楽観的ロック)を実装する

公式サイトの専用ページを定期的に確認すべきです。いつも新しいプラグインが追加され、これらはWebアプリケーションのプログラミングの多くの面にとても便利なショートカットをもたらしてくれます。

公式サイトの専用ページは別にして、プラグインを配布するほかの方法はダウンロードのためのプラグインアーカイブを提供することと、PEARチャンネルでプラグインをホストすること、もしくは公開のバージョンコントロールリポジトリに保存することです。

### プラグインをインストールする

プラグインのインストール作業はプラグインパッケージの作成方法によって異なります。つねに`README`ファイルかつ・もしくはプラグインのダウンロードのページ上のインストールの手引きを参照してください。また、プラグインのインストールした後につねにsymfonyのキャッシュをクリアしてください。

プラグインはプロジェクト単位でインストールされます。つぎのセクションで説明されるすべての方法ではすべてのプラグインのファイルを`myproject/plugins/pluginName/`ディレクトリに設置します。

### PEARプラグイン

公式サイトの専用ページの一覧に記載されているプラグインはPEARパッケージとして搭載されています。これらのプラグインをインストールするには、リスト17-15で示されるように、`plugin-install`タスクに完全なURLを渡します。

リスト17-15 - 公式サイトの専用ページからプラグインをインストールする

    > cd myproject
    > php symfony plugin-install http://plugins.symfony-project.com/pluginName
    > php symfony cc

代わりの方法として、プラグインをダウンロードしてディスクからインストールすることもできます。この場合、リスト17-16で示されるように、チャンネル名をパッケージアーカイブへの絶対パスに置き換えてください。

リスト17-16 - ダウンロードしたPEARパッケージからプラグインをインストールする

    > cd myproject
    > php symfony plugin-install /home/path/to/downloads/pluginName.tgz
    > php symfony cc

プラグインのなかにはPEARチャンネルでホストされているものがあります。リスト17-17で示されるように、`plugin-install`タスクでこれらをインストールします。チャンネル名を入力しておくことも忘れないでください。

リスト17-17 - プラグインをPEARチャンネルからインストールする

    > cd myproject
    > php symfony plugin-install channelName/pluginName
    > php symfony cc

これら3つのインストール方法はすべてPEARパッケージを利用するので、"PEARプラグイン"という用語は公式サイトのwiki、PEARチャンネル、もしくはダウンロードしたPEARパッケージからインストールされたプラグインを説明するために区別なく使われます。

#### アーカイブのプラグイン

プラグインのなかには単純にファイルのアーカイブとしてやってくるものがあります。それらをインストールするには、アーカイブをプロジェクトの`plugins/`ディレクトリに解凍します。プラグインが`web/`サブディレクトリを含む場合、リスト17-18で示されるように、このディレクトリをコピーするかプロジェクトの`web/`ディレクトリにシンボリックリンクを作ることを忘れないでください。最後に、キャッシュをクリアすることを忘れないでください。

リスト17-18 - プラグインをアーカイブからインストールする

    > cd plugins
    > tar -zxpf myPlugin.tgz
    > cd ..
    > ln -sf plugins/myPlugin/web web/myPlugin
    > php symfony cc

#### バージョン管理システムのリポジトリからプラグインをインストールする

プラグインはときにバージョン管理システム用の独自のソースコードリポジトリを持つことがあります。`plugins/`ディレクトリのなかでチェックアウトするだけでこれらのプラグインをインストールできますが、プロジェクト自身がバージョン管理システムの管理下にある場合、この作業によって問題が引き起こされる可能性があります。

代わりの方法として、プラグインを外部依存のライブラリとして宣言することが可能で、すべてのプロジェクトのソースコードを更新するとプラグインのソースコードも更新されます。たとえば、Subversionは`svn:externals`プロパティで外部依存を保存します。ですので、リスト17-19で示されているように、このプロパティを編集してソースコードをあとで更新することでプラグインを追加できます。

リスト17-19 - ソースのバージョン管理リポジトリからプラグインをインストールする

    > cd myproject
    > svn propedit svn:externals plugins
      pluginName   http://svn.example.com/pluginName/trunk
    > svn up
    > php symfony cc

>**NOTE**
>プラグインが`web/`ディレクトリを含む場合、アーカイブのプラグインに関しては同じようにそのディレクトリへのシンボリックリンクを作らなければなりません。

#### プラグインモジュールを有効にする

プラグインのなかにはモジュール全体を含むものがあります。プラグインモジュールと古典的なモジュールの違いはプラグインモジュールが`myproject/myapp/modules/`ディレクトリに現れないことだけです(簡単にアップグレードできる状態を保つため)。リスト17-20で示されるように、`settings.yml`ファイルのなかでこれらを有効にしなければなりません。

リスト17-20 - プラグインモジュールを有効にする(`myapp/config/settings.yml`)

    all:
      .settings:
        enabled_modules:  [default, sfMyPluginModule]

これはプラグインモジュールを必要としないアプリケーションが誤ってそのプラグインを利用できるように設定する状況を避けるためです。その状況ではセキュリティの欠陥を公開してしまう可能性があります。`frontend`モジュールと`backend`モジュールを提供するプラグインを考えてください。`frontend`モジュールは`frontend`アプリケーション専用として、`backend`モジュールは`backend`アプリケーション専用として有効にする必要があります。プラグインモジュールがデフォルトで有効にされない理由はそういうわけです。

>**TIP**
>defaultモジュールはデフォルトで唯一有効なモジュールです。これは本当のプラグインモジュールではありません。フレームワークの`$sf_symfony/data_dir/modules/default/`に所属するからです。これは初期ページと、404エラー用のデフォルトページとクレデンシャルが必要なエラーページを提供するモジュールです。symfonyのデフォルトページを使いたくない場合、このモジュールを`enabled_modules`設定から除外します。

#### インストールしたプラグインの一覧を表示する

プロジェクトの`plugins/`ディレクトリをざっと見るとプラグインがインストールされている場所がわかります。そして`plugin-list`タスクはより詳細な情報を示します: バージョン番号とインストールしたそれぞれのプラグインのチャンネル名です。

リスト17-21 - インストールされたプラグインの一覧

    > cd myproject
    > php symfony plugin-list

    Installed plugins:
    sfPrototypePlugin               1.0.0-stable # pear.symfony-project.com (symfony)
    sfSuperCachePlugin              1.0.0-stable # pear.symfony-project.com (symfony)
    sfThumbnail                     1.1.0-stable # pear.symfony-project.com (symfony)

#### プラグインのアップグレードとアンインストール

PEARのプラグインをアンインストールするには、リスト17-22で示されるように、プロジェクトのrootディレクトリから`plugin-uninstall`タスクを呼び出します。プラグインの名前にインストールしたチャンネルをプレフィックスとして追加しなければなりません(このチャンネルを決めるために`plugin-list`タスクを使います)。

リスト17-22 - プラグインをアンインストールする

    > cd myproject
    > php symfony plugin-uninstall pear.symfony-project.com/sfPrototypePlugin
    > php symfony cc

>**TIP**
>チャンネルのなかにはエイリアスを持つものがあります。たとえば、`pear.symfony-project.com`チャンネルは`symfony`とも見なされます。このことはリスト17-22のように`php symfony plugin-uninstall symfony/sfPrototypePlugin`を呼び出すだけで`sfPrototypePlugin`をアンインストールできることを意味します。

アーカイブからインストールしたプラグインもしくはSVNリポジトリからインストールしたプラグインをアンインストールするには、プロジェクトの`plugins/`と`web/`ディレクトリからプラグインファイルを手動で削除してキャッシュをクリアします。

プラグインをアップグレードするには、`plugin-upgrade`タスク(PEARプラグインの場合)もしくは`svn update`を実行します(バージョン管理システムのリポジトリからプラグインを入手した場合)。アーカイブからインストールしたプラグインは簡単にアップグレードできません。

### プラグインの分析

プラグインはPHPで書かれています。アプリケーションの編成方法を理解しているのであれば、プラグインの構造を理解できます。

#### プラグインのファイル構造

プラグインのディレクトリはおおよそプロジェクトのディレクトリと同じように編成されています。必要な時にsymfonyによって自動的にロードされるようにするためにプラグインファイルは正しいディレクトリに存在しなければなりません。ファイル構造の記述に関してはリスト17-23をご覧ください。

リスト17-23 - プラグインのファイル構造

    pluginName/
      config/
        *schema.yml        // データスキーマ
        *schema.xml
        config.php         // 特定のプラグインのコンフィギュレーション
      data/
        generator/
          sfPropelAdmin
            */             // administrationジェネレーターテーマ
              template/
              skeleton/
        fixtures/
          *.yml            // フィクスチャファイル
        tasks/
          *.php            // Pakeタスク
      lib/
        *.php              // クラス
        helper/
          *.php            // ヘルパー
        model/
          *.php            // モデルクラス
      modules/
        */                 // モジュール
          actions/
            actions.class.php
          config/
            module.yml
            view.yml
            security.yml
          templates/
            *.php
          validate/
            *.yml
      web/
        *                  // アセット

#### プラグインの機能

プラグインは多くのものを含みます。コマンドラインでタスクを呼び出すときに実行中のアプリケーションはこれらの内容を自動的に考慮します。しかしプラグインを適切に機能させるには、いくつかの規約を遵守しなければなりません:

  * データベースのスキーマは`propel-`タスクによって検出されます。`propel-build-model`タスクを呼び出すと、プロジェクトモデルとすべてのプラグインモデルがリビルドされます。リスト17-24で示されるように、プラグインスキーマはつねに`plugins.pluginName.lib.model`形式で`package`属性を持つことに注意してください。

リスト17-24 - スキーマ宣言の例(`myPlugin/config/schema.yml`)

    propel:
      _attributes:    { package: plugins.myPlugin.lib.model }
      my_plugin_foobar:
        _attributes:    { phpName: myPluginFoobar }
          id:
          name:           { type: varchar, size: 255, index: unique }
          ...

  * プラグインの設定はプラグインの起動スクリプト(`config.php`)に含まれています。このファイルはアプリケーションとプロジェクト設定のあとで実行されるので、symfonyはその時点ですでに起動しています。たとえば、ディレクトリをPHPのincludeパスに追加するため、もしくはミックスインで既存のクラスを拡張するためなどに、このファイルを利用できます。
  * プラグインの`data/fixtures/`ディレクトリに設置されたフィクスチャファイルは`propel-load-data`タスクで処理されます。
  * プラグインがインストールされると同時にタスクはsymfonyのコマンドラインですぐに利用できるようになります。タスクにたとえばプレフィックスとしてプラグインの名前などの意味のある名前をつけることはよい習慣です。プラグインによって追加されたタスクを含む利用可能なタスクの一覧を見るには、`symfony`コマンドを実行してください。
  * プロジェクトの`lib/`フォルダーに設置されたクラスのようにカスタムクラスはオートロードされます。
  * テンプレートのなかで`use_helper()`ヘルパーを呼び出すときにヘルパーは自動的に発見されます。これらはプラグインの`lib/`ディレクトリの1つの`helper/`サブディレクトリに存在しなければなりません。
  * `myplugin/lib/model/`ディレクトリ内のモデルクラスは(`myplugin/lib/model/om/`ディレクトリと`myplugin/lib/model/map/`ディレクトリ)内部のPropelビルダによって生成されたモデルクラスを専門に扱います。もちろんこれらもオートロードされます。独自プロジェクトのディレクトリ内で生成されたプラグインのモデルクラスはオーバーライドできません。
  * アプリケーションの`enabled_modules`設定で宣言すれば、モジュールは外部からアクセス可能な新しいアクションを提供します。
  * サーバーはWebアセット(イメージ、スクリプト、スタイルシートなど)を利用できます。コマンドライン経由でプラグインをインストールしたとき、symfonyはシステムが許可するのであればプロジェクトの`web/`ディレクトリにシンボリックリンクを作るもしくは`web/`ディレクトリの内容をプロジェクトをディレクトリにコピーします。プラグインがアーカイブもしくはバージョン管理ツールのリポジトリからインストールされた場合、手動で`web/`ディレクトリにコピーしなければなりません(プラグインに添付されている`README`に記載されています)。

#### 手動によるプラグインのセットアップ

`plugin-install`タスクが独自に処理できない要素がいくつかあります。インストール作業の間にこれらを手動でセットアップする必要があります:

  * カスタムアプリケーション設定はプラグインのコードで使われますが(たとえば、`sfConfig::get('app_myplugin_foo')`を利用する)、デフォルト値をプラグインの`config/`ディレクトリに設置された`app.yml`ファイルに設定できません。デフォルト値を処理するには、`sfConfig::get()`メソッドの2番目の引数を使います。設定はまだアプリケーションレベルでオーバーライドできます(リスト17-25で例をご覧ください)。
  * カスタムルーティングルールはアプリケーションの`routing.yml`に手動で追加しなければなりません。
  * カスタムフィルターはアプリケーションの`filters.yml`に手動で追加しなければなりません。
  * カスタムファクトリはアプリケーションの`factories.yml`に手動で追加しなければなりません。

一般的に言えば、アプリケーションの設定ファイルの1つになるすべての設定は手動で追加しなければなりません。このような手動のセットアップが必要なプラグインは`README`ファイルで詳細なインストール方法を説明しています。

#### アプリケーションのためにプラグインをカスタマイズする

プラグインをカスタマイズしたいときは、決して`plugins/`ディレクトリ内で見つかるコードを変更してはなりません。これを行うと、プラグインをアップグレードするときにすべての修正内容が失われてしまいます。必要なカスタマイズを行うために、プラグインはカスタム設定を提供し、オーバーライドをサポートします。

リスト17-25で示されるように、よく設計されたプラグインはアプリケーションの`app.yml`ファイルで変更できる設定を利用します。

リスト17-25 - アプリケーションの設定を利用するプラグインをカスタマイズする

    [php]
    // プラグインのコードの例
    $foo = sfConfig::get('app_my_plugin_foo', 'bar');

    // アプリケーションのapp.ymlで'foo'のデフォルト値('bar')を変更する
    all:
      my_plugin:
        foo:       barbar

モジュールの設定とデフォルト値はプラグインの`README`ファイルで詳しく説明されています。

独自のアプリケーション内部で同じ名前のモジュールを作成することでプラグインモジュールのデフォルトの内容を置き換えることができます。プラグイン要素の代わりにアプリケーション要素が使われているので、本当のオーバーライドではありません。プラグインの名前と同じ名前のテンプレートと設定ファイルを作ればプラグインモジュールは立派に機能します。

一方で、アクションをオーバーライドする機能を持つモジュールをプラグインに持たせたい場合、プラグインモジュール内の`actions.class.php`のメソッドがアプリケーションモジュールの`actions.class.php`によって継承できるように、`actions.class.php`は空でなければならずオートロードクラスから継承しなければなりません。お手本に関してはリスト17-26を参照してください。

リスト17-26 - プラグインのアクションをカスタマイズする

    [php]
    // myPlugin/modules/mymodule/lib/myPluginmymoduleActions.class.phpのなか
    class myPluginmymoduleActions extends sfActions
    {
      public function executeIndex()
      {
        // ここに何らかのコード
      }
    }

    // myPlugin/modules/mymodule/actions/actions.class.phpにて

    require_once dirname(__FILE__).'/../lib/myPluginmymoduleActions.class.php';

    class mymoduleActions extends myPluginmymoduleActions
    {
      // なし
    }

    // myapp/modules/mymodule/actions/actions.class.phpにて
    class mymoduleActions extends myPluginmymoduleActions
    {
      public function executeIndex()
      {
        // ここでプラグインのコードをオーバーライドする
      }
    }

### プラグインの書き方

`plugin-install`タスクではPEARパッケージ形式のプラグインのみがインストールされます。このようなプラグインは公式サイトの専用ページ、PEARチャンネル経由もしくはダウンロードできる通常のファイルとして配布されていることを覚えておいてください。プラグインを編集したい場合は、単純なアーカイブよりもPEARパッケージとして公開したほうがベターでしょう。加えて、プラグインをPEARパッケージにすればアップグレード作業が簡単になり、依存関係の宣言が可能で、自動的にアセットを`web/`ディレクトリにデプロイできます。

#### ファイルのコンフィギュレーション

新しい機能を開発し、プラグインとしてパッケージにすることを考えてみましょう。最初の段階はファイルを論理的に編成して、symfonyのロードメカニズムが必要なときにこれらのファイルを見つけることができるようにしましょう。この目的のために、リスト17-23で示されているディレクトリ構造に従う必要があります。リスト17-27は`sfSamplePlugin`プラグインのためのファイル構造の例を示しています。

リスト17-27 - プラグインとしてパッケージにするファイルの一覧の例

    sfSamplePlugin/
      README
      LICENSE
      config/
        schema.yml
      data/
        fixtures/
          fixtures.yml
        tasks/
          sfSampleTask.php
      lib/
        model/
          sfSampleFooBar.php
          sfSampleFooBarPeer.php
        validator/
          sfSampleValidator.class.php
      modules/
        sfSampleModule/
          actions/
            actions.class.php
          config/
            security.yml
          lib/
            BasesfSampleModuleActions.class.php
          templates/
            indexSuccess.php
      web/
        css/
          sfSampleStyle.css
        images/
          sfSampleImage.png

編集に関して、プラグインのディレクトリの位置(リスト17-27の`sfSamplePlugin/`)は重要ではありません。これはディスク上の任意の場所に設置できます。

>**TIP**
>既存のプラグインを練習問題として考え、初めてプラグインを作るさいには、これらの名前の規約とファイルの構造を再現してみてください。

#### package.xmlファイルを作る

プラグイン編集のつぎの段階はプラグインディレクトリのrootで`package.xml`ファイルを追加することです。`package.xml`はPEARの構文に従います。リスト17-28の典型的なsymfonyプラグインの`package.xml`をご覧ください。

リスト17-28 - symfonyプラグイン用の`package.xml`

    [xml]
    <?xml version="1.0" encoding="UTF-8"?>
    <package packagerversion="1.4.6" version="2.0" xmlns="http://pear.php.net/dtd/package-2.0" xmlns:tasks="http://pear.php.net/dtd/tasks-1.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://pear.php.net/dtd/tasks-1.0 http://pear.php.net/dtd/tasks-1.0.xsd http://pear.php.net/dtd/package-2.0 http://pear.php.net/dtd/package-2.0.xsd">
     <name>sfSamplePlugin</name>
     <channel>pear.symfony-project.com</channel>
     <summary>symfony sample plugin</summary>
     <description>Just a sample plugin to illustrate PEAR packaging</description>
     <lead>
      <name>Fabien POTENCIER</name>
      <user>fabpot</user>
      <email>fabien.potencier@symfony-project.com</email>
      <active>yes</active>
     </lead>
     <date>2006-01-18</date>
     <time>15:54:35</time>
     <version>
      <release>1.0.0</release>
      <api>1.0.0</api>
     </version>
     <stability>
      <release>stable</release>
      <api>stable</api>
     </stability>
     <license uri="http://www.symfony-project.org/license">MIT license</license>
     <notes>-</notes>
     <contents>
      <dir name="/">
       <file role="data" name="README" />
       <file role="data" name="LICENSE" />
       <dir name="config">
        <!-- model -->
        <file role="data" name="schema.yml" />
       </dir>
       <dir name="data">
        <dir name="fixtures">
         <!-- fixtures -->
         <file role="data" name="fixtures.yml" />
        </dir>
        <dir name="tasks">
         <!-- tasks -->
         <file role="data" name="sfSampleTask.php" />
        </dir>
       </dir>
       <dir name="lib">
        <dir name="model">
         <!-- model classes -->
         <file role="data" name="sfSampleFooBar.php" />
         <file role="data" name="sfSampleFooBarPeer.php" />
        </dir>
        <dir name="validator">
         <!-- validators ->>
         <file role="data" name="sfSampleValidator.class.php" />
        </dir>
       </dir>
       <dir name="modules">
        <dir name="sfSampleModule">
         <file role="data" name="actions/actions.class.php" />
         <file role="data" name="config/security.yml" />
         <file role="data" name="lib/BasesfSampleModuleActions.class.php" />
         <file role="data" name="templates/indexSuccess.php" />
        </dir>
       </dir>
       <dir name="web">
        <dir name="css">
         <!-- stylesheets -->
         <file role="data" name="sfSampleStyle.css" />
        </dir>
        <dir name="images">
         <!-- images -->
         <file role="data" name="sfSampleImage.png" />
        </dir>
       </dir>
      </dir>
     </contents>
     <dependencies>
      <required>
       <php>
        <min>5.0.0</min>
       </php>
       <pearinstaller>
        <min>1.4.1</min>
       </pearinstaller>
       <package>
        <name>symfony</name>
        <channel>pear.symfony-project.com</channel>
        <min>1.0.0</min>
        <max>1.1.0</max>
        <exclude>1.1.0</exclude>
       </package>
      </required>
     </dependencies>
     <phprelease />
     <changelog />
    </package>

ここで注目すべき部分は`<contents>`タグと`<dependencies>`タグで、つぎに説明します。残りのタグに関しては、symfony固有のものではありませんので、`package.xml`フォーマットに関する詳細な内容はPEARオンラインマニュアル([http://pear.php.net/manual/](http://pear.php.net/manual/)) を参照してください。

#### 内容

`<contents>`タグはプラグインのファイル構造を記述しなければならない場所です。このタグはコピーするファイルとその場所をPEARに伝えます。`<dir>`タグと`<file>`タグでファイル構造を記述してください。すべての`<file>`タグは`role="data"`属性を持たなければなりません。リスト17-28の`<contents>`タグの部分はリスト17-27の正しいディレクトリ構造を記載しています。

>**NOTE**
>`<dir>`タグの使用は義務ではありません。`<file>`タグ内で相対パスを`name`の値として利用できるからです。`package.xml`ファイルを読みやすくするためにお勧めです。

#### プラグインの依存関係

任意のバージョンのPHP、PEAR、symfony、PEARパッケージ、もしくはほかのプラグインの一式で動くようにプラグインは設計されています。`<dependencies>`タグでこれらの依存関係を宣言すれば必要なパッケージがすでにインストールされていることを確認してそうでなければ例外を起動するようPEARに伝えることになります。

最小要件として、少なくとも開発環境に対応したPHP、PEARとsymfonyへの依存関係をつねに宣言します。何を追加すればよいのかわからなければ、PHP 5.0、PEAR 1.4とsymfony 1.0の要件を追加してください。

それぞれのプラグインに対してsymfonyの最大のバージョン番号を追加することも推奨されます。これによって上位バージョンのsymfonyでプラグインを使うときにエラーメッセージが表示され、プラグインを再リリースするまえにこのバージョンでプラグインが正しく動作するのかを確認することをプラグインの作者に義務づけます。無言でプラグインの動作が失敗するよりも警告を発してダウンロードとアップグレードするほうがベターです。

#### プラグインをビルドする

PEARコンポーネントはパッケージの`.tgz`アーカイブを作るコマンド(`pear package`)を持ちます。リスト17-29では、`package.xml`を含むディレクトリでこのコマンドを呼び出しています。

リスト17-29 - プラグインをPEARパッケージにする

    > cd sfSamplePlugin
    > pear package

    Package sfSamplePlugin-1.0.0.tgz done

いったんプラグインのパッケージがビルドされたら、リスト17-30で示されるように、あなたの環境にこれをインストールして動作を確認してください。

リスト17-30 - プラグインをインストールする

    > cp sfSamplePlugin-1.0.0.tgz /home/production/myproject/
    > cd /home/production/myproject/
    > php symfony plugin-install sfSamplePlugin-1.0.0.tgz

`<contents>`タグにある説明にしたがって、パッケージにされたファイルは最終的にプロジェクトの異なるディレクトリに設置されます。リスト17-31はインストールのあとで`sfSamplePlugin`のファイルが設置される場所を示しています。

リスト17-31 - プラグインファイルは`plugin/`ディレクトリと`web/`ディレクトリにインストールされる

    plugins/
      sfSamplePlugin/
        README
        LICENSE
        config/
          schema.yml
        data/
          fixtures/
            fixtures.yml
          tasks/
            sfSampleTask.php
        lib/
          model/
            sfSampleFooBar.php
            sfSampleFooBarPeer.php
          validator/
            sfSampleValidator.class.php
        modules/
          sfSampleModule/
            actions/
              actions.class.php
            config/
              security.yml
            lib/
              BasesfSampleModuleActions.class.php
            templates/
              indexSuccess.php
    web/
      sfSamplePlugin/               ## システム次第で、コピーもしくはシンボリックリンク
        css/
          sfSampleStyle.css
        images/
          sfSampleImage.png

このプラグインのふるまいをアプリケーションでテストしてください。きちんと動くのであれば、プラグインを複数のプロジェクトにまたがって配布するもしくはsymfonyコミュニティに寄付する準備ができています。

#### 公式サイトでプラグインを配布する

symfonyのプラグインは以下の手順にしたがって`symfony-project.org`のWebサイトで配布されるときにもっとも幅広い利用者を得ます。独自プラグインをつぎのような方法で配布できます:

  1. `README`ファイルにプラグインのインストール方法と使いかたが、`LICENSE`ファイルにはライセンスの詳細が記述されていることを確認する。`README`はMarkdownの構文 ([http://daringfireball.net/projects/markdown/syntax](http://daringfireball.net/projects/markdown/syntax)) で記述する。
  2. 公式サイトのアカウント (http://www.symfony-project.org/user/new) を作りプラグインのページ (http://www.symfony-project.org/plugins/new) を作る。
  3. `pear package`コマンドを呼び出してプラグイン用のPEARパッケージを作りテストする。PEARパッケージの名前は`sfSamplePlugin-1.0.0.tgz` (1.0.0はプラグインのバージョン)でなければならない。
  4. PEARパッケージをアップロードする (`sfSamplePlugin-1.0.0.tgz`)。
  5. アップロードしたプラグインは一覧ページ ([http://www.symfony-project.org/plugins/](http://www.symfony-project.org/plugins/)) に表示される。

この手続きを行えば、ユーザーはプロジェクトのディレクトリでつぎのコマンドを入力するだけでプラグインをインストールできるようになります:

    > php symfony plugin-install http://plugins.symfony-project.com/sfSamplePlugin

#### 命名規約

`plugin/`ディレクトリをきれいに保つために、すべてのプラグインの名前がcamelCaseであり`Plugin`のサフィックスで終わることを確認してください(たとえば、`shoppingCartPlugin`、`feedPlugin`)。プラグインに名前をつけるまえに、同じ名前のプラグインが存在しないことを確認してください。

>**NOTE**
>Propelに依存するプラグインの名前は`Propel`を含みます。たとえば、Propelのデータアクセスオブジェクトを利用する認証プラグインは`sfPropelAuth`という名前になります。

プラグインには使用条件と選んだライセンスを説明する`LICENSE`ファイルをつねに含めなければなりません。バージョンの履歴、プラグインの目的、効果、インストールと設定の手引きなどを含めることも推奨されます。

まとめ
----

symfonyのクラスはアプリケーションレベルで修正できる機能を提供する`sfMixer`フックを含みます。ミックスイン(mixin)のメカニズムはPHPの制約が禁止している実行時のクラスの多重継承とオーバーライドを可能にします。ですのでそのためにコアクラスを修正しなければならないとしても、またファクトリ(factory)の設定がそこに存在するとしてもsymfonyの機能を簡単に拡張できます。

すでに多くの拡張機能(エクステンション)が存在し、プラグインとしてパッケージが作成されています。symfonyのコマンドラインによってインストール、アップグレード、アンインストールするのが簡単です。プラグインをPEARパッケージを作成するのと同じぐらい簡単で、複数のアプリケーションをまたがって再利用できます。

symfony公式サイトのwikiには多くのプラグインが含まれ、あなた自身のプラグインも追加できます。これであなたは方法を理解したので、私たちsymfonyの開発者はあなたが多くの便利な拡張機能でsymfonyコアを強化して下さることを望んでおります！
