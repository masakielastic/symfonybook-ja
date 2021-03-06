第1章 - symfonyの紹介
=====================

symfonyはあなたのために何ができるのでしょうか？使うために何が必要なのでしょうか？この章はこれらの質問にお答えします。

symfonyとは
-----------

特定の目的のために採用される多くのパターンを自動化することで、フレームワークはアプリケーションの開発を合理化します。またフレームワークはコードに構造を加えることで、より読みやすく維持しやすいコードを書くことを開発者に奨励します。フレームワークは複雑な作業を簡単なステートメントにまとめるので、最終的に、フレームワークはプログラミング作業をより簡単にします。

symfonyは、Webアプリケーションの開発を最適化するためにいくつかの重要な特徴を通して設計された完全なフレームワークです。初心者のために、symfonyはWebアプリケーションのビジネスルール、サーバーのロジック、そしてプレゼンテーションのビューを分離します。symfonyは複雑なWebアプリケーションの開発期間を短くすることを目的とした多くのツールとクラスを搭載しています。加えて、開発者がアプリケーションだけに集中できるようにsymfonyは共通のタスクを自動化します。これらの利点の最終結果は新しいアプリケーションが作られるたびに車輪を再発明する(reinvent the wheel)必要がないことを意味します！

symfonyは完全なPHP 5で書かれました。symfonyは現実の世界のさまざまなプロジェクトで徹底的にテストされ、高度な要件が求められる電子商取引のWebサイトでも実際に利用されています。symfonyはMySQL、PostgreSQL、Oracle、Microsoft SQL Serverを含む利用可能なほとんどのデータベースエンジンと互換性があります。symfonyはUnix系やWindowsプラットフォームで動作します。機能に関してより詳しく見ることにします。

### symfonyの機能

symfonyは以下の要件を満たすために開発されました:

  * 多くのプラットフォームでのインストールと設定作業が簡単である(そして標準的なUnix系とWindowsプラットフォームでの動作が保証されている) 
  * データベースエンジンに依存していない
  * 多くの案件で使いやすい一方で、複雑な案件に適合する柔軟性がある
  * 設定よりも規約(convention over configuration)の前提に基づく--開発者は型にはまらないものだけ設定する必要がある
  * Web開発のベストプラクティスとデザインパターンの大半に準拠する
  * 企業の既存の情報技術(IT)の方針とアーキテクチャに適合する用意ができていて、長期間のプロジェクトにおいて十分に安定している
  * 維持しやすく、phpDocumentorコメントを含む、とても読みやすいコード
  * 拡張するのが簡単で、ほかのベンダーライブラリとの統合が可能である

#### 自動化されたWebプロジェクト機能

Webプロジェクトの大部分の共通機能は、つぎのようにsymfonyの範囲内で自動化されます:

  * 内容のローカライゼーション(現地化)と同様に、組み込みの国際化レイヤーによってデータとインターフェイスの両方の翻訳を可能にする。
  * プレゼンテーションはsymfonyの知識を持たないHTMLデザイナーによって開発できるテンプレートとレイアウトを利用する。大部分のコードを単純なコードの呼び出しにカプセル化することでヘルパーは書く必要のあるプレゼンテーションのコード量を減らす。
  * フォームは自動化されたバリデーション(検証)と再投入をサポートする。このことによって質のよいデータベースのデータと優れたユーザーエクスペリエンスが保証される。
  * 出力エスケーピングによって不正なデータ経由の攻撃からアプリケーションを保護する。
  * キャッシュの管理機能は回線の帯域幅の使用量とサーバーの読み込み回数を減らす。
  * 認証とクレデンシャル機能は制限つきのセクションとユーザーのセキュリティ管理機能の作成を円滑にする。
  * ルーティング(routing)とスマートURL(smart URL)はページのアドレスをインターフェイスの一部にして検索エンジンに対してユーザーフレンドリーにする。
  * 組み込みのEメールとAPIの管理機能はWebアプリケーションが古典的なブラウザーのインタラクションを越えることを可能にする
  * 自動化されたページ分割(パジネーション - pagination)、ソート、そしてフィルタリングのおかげでリストはよりユーザーフレンドリーになる。
  * ファクトリ、プラグイン、そしてミックスインは高いレベルの拡張性を提供する。
  * 複数のブラウザーで互換性のあるJavaScriptのイフェクトをカプセル化する一行ヘルパーのおかげでAjaxインタラクションの実装が簡単である。

#### 開発環境とツール

独自のコーディングのガイドラインとプロジェクトの管理ルールを持つ企業の要件を満たすために、symfonyを完全にカスタマイズできます。symfonyは標準でいくつかの開発環境を提供し、一般的なソフトウェアエンジニアリングのタスクを自動化する複数のツールを搭載しています:

  * コード生成ツールはプロトタイプ作成とワンクリックで実行できるバックエンドの管理画面のために大いに役立つ。
  * 組み込みのユニットテスト(単体テスト)と機能テスト(ファンクショナルテスト)のフレームワークはテスト駆動開発を可能にする完璧なツールを提供する。
  * デバッグパネルは開発者がとり組んでいるページで必要なすべての情報を表示するのでデバッグ作業を迅速に行うことができる。
  * コマンドラインインターフェイス(CLI)は2つのサーバー間のアプリケーションの配置を自動化する。
  * 稼働しているアプリケーションの設定を効率的に変更できる
  * ロギング機能はアプリケーションの活動に関する完全な詳細内容をサイト管理者に提供する。

### symfonyを作ったのは誰？なぜ？

プロジェクトの創設者であり、この本の共著者でもあるFabien Potencier(ファビアン・プゥトンシェ)によってsymfonyの最初のバージョンは2005年10月に公開されました(訳注：そのほかの開発メンバーの名前の読み方は[「Sensio(symfony)訪問レポート」](http://akimoto.jp/presentation/sensio-symfony-visit-report.html)を参照)。FabienはSensio ([http://www.sensio.com/](http://www.sensio.com/))のCEOです。SensioはフランスのWeb制作会社でWeb開発に関して革新的な見解を持つことでよく知られています。

2003年の頃に、FabienはWebアプリケーションのためにPHP製の既存のオープンソース開発ツールの調査に時間を費やしましたが、まえに説明した要件を満たすものが存在しないことが判明しました。PHP 5が公開されたとき、彼は利用できるツールをフル機能のフレームワークに統合するのに十分に成熟した段階に到達したと判断しました。その後彼はsymfonyコアの開発に1年費やしました。symfonyコアはモデル・ビュー・コントローラー(MVC)フレームワークであるMojavi、オブジェクトリレーショナルマッピング(ORM)であるPropel、そしてRuby on Railsのテンプレートヘルパーなどがベースとなっています。

Fabienは元々はSensioのプロジェクトのためにsymfonyを開発しました。思いどおりに使える効率的なフレームワークを保有することはアプリケーションをより速くて効率的な開発を行う理想的な方法を示すからです。これによってWeb開発がより直感的なものになり、アプリケーションは堅牢で維持が簡単になりました。ランジェリー小売店用の電子商取引のWebサイトを開発するためにsymfonyを採用したときにその性能は確かめられました。そしてその後、ほかのプロジェクトにも適用されました。

symfonyを利用していくつかのプロジェクトを成功させた後に、Fabienはsymfonyをオープンソースのライセンスのもとで公開しました。公開したのは、作品をコミュニティに寄贈するため、ユーザーのフィードバックの恩恵を受けるため、Sensioの経験を展示するため、そして面白いからです。

>**NOTE**
>"symfony"が"FooBarFramework"ではないのはなぜでしょうか？覚えやすくほかの開発ツールを連想しないようにするために、FabienはSensioを表すsとフレームワークを表すfを含む省略語が欲しかったからです。また、彼は大文字も好みませんでした。symfonyは完全な英語ではないとしても十分に英語に近い名前であり、プロジェクトの名前としても利用可能でした。ほかの代替案は"baguette"(フランスパン)でした。

symfonyをオープンソースプロジェクトとして成功させるために、採用率を上げるために、大規模な英語のドキュメントが必要でした。Fabienは仲間のSensio従業員で、この本のもう一人の著者であるFrançois Zaninotto (フランソワ・ザニノット)にコードを徹底的に研究してオンラインの教科書を書くように依頼しました。プロジェクトが公開されてからしばらくかかりましたが、数多くの開発者に対してアピールをするのに十分なドキュメントが作成されました。その後の話はご存じのとおりです。

### symfonyのコミュニティ

symfonyのWebサイト([http://www.symfony-project.org/](http://www.symfony-project.org/))が開始されるとすぐに、世界中からたくさんの開発者がフレームワークをダウンロートしてインストールし、オンラインのドキュメントを読み、symfonyで最初のアプリケーションを構築し、興奮のざわめきが始まりました。

当時、Webアプリケーションのフレームワークの人気が出つつあり、PHP製のフル機能のフレームワークは高い需要がありました。symfonyは、すばらしい品質のコードと特筆すべき量のドキュメント、フレームワークのカテゴリにおいてほかのプレイヤーを越える2つの長所、のおかげで説得力のあるソリューションを提案しました。貢献者がすぐに名乗り出て、パッチと機能の強化を提案し、ドキュメントの校正を行い、そのほかの必要とされた役割を果たしました。

公開ソースリポジトリ(svn)とチケットシステム(trac)は貢献するためのさまざまな方法を提供し、ボランティアはみな歓迎されます。Fabienは今もなおソースコードリポジトリのtrunkの主要なコミッターで、コードの品質を保証します。

今日において、symfonyのフォーラム、メーリングリスト、インターネットリレーチャットチャンネル(IRC)はざっと見て一つの質問に対して平均4つの回答を得られる理想的なサポート施設を提供します。毎日新人がsymfonyをインストールし、wikiとコードスニペットのセクションでは多くのユーザーが投稿したドキュメントがホストされています。symfony製の既知のアプリケーションの数は一週間ごとに平均で合計5個の割合で増えています。

symfonyのコミュニティはこのフレームワークの3番目の強みで、この本を読んだ後にあなたも参加して下さることを我々執筆者は望んでいます。

### symfonyは私の用途に合っていますか？

あなたがPHP 5の専門家であっても、Webアプリケーションのプログラミングの新人であってもsymfonyを使えるようになります。symfonyで開発をするかどうか決める主な要因はあなたのプロジェクトの規模です。

データベースへのアクセスが制限されており、5ページから10ページ程度の簡単なWebサイトを開発したい場合で、パフォーマンスを保証するもしくはドキュメントを提供する義務がない場合、素のPHPコードにこだわるべきでしょう。Webアプリケーションのフレームワークから多くのことを得られず、おそらくはオブジェクト指向もしくはMVCモデルが開発プロセスを遅らせることになるでしょう。追記として、symfonyはPHPスクリプトがCGI(コモンゲートウェイインターフェイス)の共用サーバー上で効率的に動作するように最適化されていません。

一方で、重いビジネスロジックをかかえる、より複雑なWebアプリケーションを開発する場合、素のPHPでは十分ではありません。将来アプリケーションを維持するもしくは拡張することを計画する場合、コードを軽量に、読みやすく、効果的なものにすることが必要になります。直感的な方法で(Ajaxのような)ユーザーインタラクションの最新機能を使いたい場合、JavaScriptを数百行書くだけではすまされません。楽しく早く開発したい場合、素のPHPだけではおそらく失望するでしょう。これらすべての場合において、symfonyはあなたを支援します。

そして、もちろん、あなたがプロのWeb開発者であるなら、すでにWebアプリケーションのすべての利点を理解しており、成熟し、充実したドキュメントがあり、大きなコミュニティを持つフレームワークが必要です。もう検索しなくても、symfonyがあなたのソリューションです。

>**TIP**
>視覚的なデモンストレーションがお好みなら、symfonyのWebサイトからスクリーンキャストをご覧ください。symfonyでアプリケーションを開発することがどんなに速くて楽しいことなのかわかるでしょう。

基本概念
--------

symfonyで始めるまえに、基本概念を学んでおきます。OOP、ORM、RAD、DRY、KISS、TDD、YAMLとPEARの意味をすでにご存じでしたら、読み飛ばしてください。

### PHP 5

symfonyはPHP 5([http://www.php.net](http://www.php.net/))で開発され、同言語でWebアプリケーションを開発することを専門としてしています。そのため、フレームワークを最大限活用するにはPHP 5をしっかりと理解していることが必要です。symfonyを動かすために必要なPHPの最小バージョンは5.1です。

すでにPHP 4を知っているがPHP 5を知らない開発者はこの言語の新しいオブジェクト指向のモデルに主な焦点を当てます。

### オブジェクト指向プログラミング(OOP)

オブジェクト指向プログラミング(OOP - Object-Oriented Programming)はこの章では説明しません。1冊の本が必要だからです! symfonyはPHP 5で利用できるオブジェクト指向のメカニズムを広範囲で活用しているので、OOPはsymfonyを学ぶための必須条件です。

WikipediaにおいてOOPはつぎのように説明されています:

オブジェクト指向プログラミングの背景にある考え方は、プログラムを関数の集まりもしくはコンピュータへの単なる命令の一覧とみなす伝統的な見方とは対照的に、コンピュータプログラムはお互いに影響を与える個別のユニット、もしくはオブジェクトの集まりをコンフィギュレーションするものとしてみなすことです。

PHP 5はクラス、オブジェクト、メソッド、継承やそのほかのオブジェクト指向のパラダイムを実装します。これらの概念になじみがない方は[http://www.php.net/manual/language.oop5.basic.php](http://www.php.net/manual/language.oop5.basic.php)などの関連ドキュメントを読むことをお勧めします。

### マジックメソッド

PHPのオブジェクト機能の強みの1つはマジックメソッド(magic method)を利用できることです。これらのメソッドは外部コードの修正を行わずにクラスのデフォルトのふるまいをオーバーライドできます。これらによってPHPの構文はより簡潔で拡張性のあるものになります。マジックメソッドの名前は2つのアンダースコア(`__`)で始まるので、これらを見分けることは簡単です。

たとえば、オブジェクトを表示するとき、カスタム表示フォーマットが開発者によって定義されたのかを確認するために暗黙でPHPはこのオブジェクト用の`__toString()`メソッドを探します:

    [php]
    $myObject = new myClass();
    echo $myObject;

    // マジックメソッドを探す
    echo $myObject->__toString();

symfonyはマジックメソッドを利用するので、これらをよく理解すべきです。これらはPHPの公式マニュアル([http://www.php.net/manual/language.oop5.magic.php](http://www.php.net/manual/language.oop5.magic.php))で説明されています。

### PHPの拡張とアプリケーションリポジトリ (PEAR)

PEAR(PHP Extension and Application Repository)は"再利用可能なPHPコンポーネントのためのフレームワークかつ配布システム"です。PEARによってPHPスクリプトのダウンロード、インストール、アップグレード、アンインストールが可能になります。PEARパッケージを利用するとき、どこにスクリプトを設置するのか、どのようにして利用可能にするのか、もしくはどのようにしてコマンドラインインターフェイス(CLI)を拡張するかなどについて悩む必要はありません。

PEARはPHPで書かれたコミュニティ駆動のプロジェクトで、PHPの標準ディストリビューションに搭載されています。

>**TIP**
>PEARのWebサイト、[http://pear.php.net/](http://pear.php.net/)、はドキュメントとカテゴリによって分類されたパッケージを提供します。

PEARはPHPベンダーのライブラリをインストールするためのもっともプロフェッショナルな方法です。symfonyの開発において複数のプロジェクトにまたがってライブラリを集中管理するためにPEARを利用することが推奨されます。symfonyのプラグインは特別な設定を格納するPEARパッケージです。symfonyフレームワーク自身がPEARパッケージとして利用できます。

symfonyを利用するためにPEAR構文のすべてを知る必要はありません。symfonyが何を行い何をインストールしたのかだけを理解しておく必要があります。CLI(コマンドラインインターフェイス)でつぎのコマンドを入力すればコンピュータにインストールされたPEARを確認できます:

    > pear info pear

このコマンドはインストールされたPEARのバージョン番号を返します。

symfonyのプロジェクトは独自のPEARリポジトリもしくはチャンネルを持ちます。チャンネルはバージョン1.4.0以降のPEARで利用可能なので、バージョンが古い場合はアップグレードすべきです。PEARのバージョンをアップグレードするには、つぎのコマンドを入力します:

    > pear upgrade PEAR

### オブジェクトリレーショナルマッピング(ORM)

(symfonyで利用する)データベースはリレーショナル(relational)です。PHP 5とsymfonyはオブジェクト指向です。オブジェクト指向の方法でデータベースにアクセスするには、オブジェクトのロジックをリレーショナルデータベースのロジックに翻訳するインターフェイスが必要です。このインターフェイスはオブジェクトリレーショナルマッピング(Object-Relational Mapping)もしくはORMと呼ばれます。

ORMはデータにアクセス可能でビジネスロジックをオブジェクト自身の範囲に留めるオブジェクトで構成されます。

オブジェクト/リレーショナル抽象レイヤーの利点の1つは特定のデータベース固有の構文を使わずにすむことです。これによってモデルオブジェクトの呼び出しは現在のデータベースに最適化されたSQLクエリに自動的に変換されます。

このことはプロジェクトの途中で別のデータベースシステムに簡単に切り替えできることを意味します。素早くプロトタイプを書かなければならないが、顧客がどのデータベースシステムが自分のニーズに最適なのかをまだ決めていない状況を想像してください。たとえば、SQLiteでアプリケーションの開発を始め、顧客が決断する準備ができたときに、設定ファイルの一行を変更するだけで、MySQL、PostgreSQL、もしくはOracleに切り替えられます。

抽象化レイヤーはデータロジックをカプセル化します。アプリケーションの残りの部分はSQLクエリについて知る必要はないので、データベースにアクセスするSQLを見つけることは簡単です。データベースプログラミングを専門にする開発者はどこを見ればよいのかを理解しています。

レコードの代わりにオブジェクト、テーブルの代わりにクラスを使うことには別の利点があります。新しいアクセサーをテーブルに追加できることです。たとえば、`FirstName`と`LastName`の2つのフィールドを持つ`Client`という名前のテーブルが存在する場合、単に`Name`を求められるようにしたい場合があります。オブジェクト指向の世界において、これは、つぎのように`Client`クラスに新しいアクセサーメソッドを追加することで簡単に実現できます:

    [php]
    public function getName()
    {
      return $this->getFirstName().' '.$this->getLastName();
    }

繰り返されるすべてのデータアクセス関数とデータのビジネスロジックはこのようなオブジェクトの範囲に保つことができます。たとえば、(オブジェクトである)アイテムを保有する`ShoppingCart`クラスを考えてください。会計用のショッピングカートの総数を読みとるために、つぎのように`getTotal()`メソッドを追加できます:

    [php]
    public function getTotal()
    {
      $total = 0;
      foreach ($this->getItems() as $item)
      {
        $total += $item->getPrice() * $item->getQuantity();
      }
      return $total;
    }

これでお終いです。同じことを行うSQLクエリを書くためにどれだけの時間がかかるのか想像してください！

別のオープンソースプロジェクトであるPropelは現時点でPHP 5のためのベストなオブジェクト/リレーショナル抽象化レイヤーの一つです。PropelはデフォルトのORMとしてシームレスにsymfonyに統合されるので、この本で説明されるたいていのデータの操作方法はPropelの構文に従います。この本はPropelオブジェクトの使いかたを説明しますが、もっと完全なリファレンスについては、PropelのWebサイト([http://propel.phpdb.org/trac/](http://propel.phpdb.org/trac/))を訪問することをお勧めします。

>**Note**
>symfony 1.1では、Propelはプラグインとしてsymfonyに搭載されており、簡単にほかのORMに切り替えることができます。実際のところ、もし[Doctrine](http://www.phpdoctrine.org/)を主要なORMツールとして使いたい場合、`sfDoctrinePlugin`のインストールすることだけが必要です。

### ラピッドアプリケーション開発(RAD)

長い間Webアプリケーションのプログラミングは退屈で遅い作業でした。(たとえばラショナル統一プロセス(Rational Unified Process)のような)通常のソフトウェアエンジニアリングのライフサイクルに従えば、Webアプリケーションの開発は、要件のドキュメントの完全な一式が揃うまで始めることが不可能で、多くのUML(Unified Modeling Language)のダイアグラムが描かれ、膨大な下準備のドキュメントが生産されました。これは、一般的な開発速度、プログラミング言語が器用ではないこと(ビルド、コンパイル、再起動しなければならず、実際にプログラムを動かさないとわからないということ)、そして、とりわけ、顧客が聞き分けがよく考えをしょっちゅう変えないことを前提としていたためです。

今日において、ビジネスはより早く変化するので、開発の最中に顧客は気が変わりがちです。もちろん、顧客は開発チームにも顧客自身のニーズに適合してアプリケーションの構造を素早く修正することを求めます。幸いにして、PerlやPHPなどのスクリプト言語を利用することで、ラピッドアプリケーション開発(RAD - Rapid Application Development)もしくはアジャイルソフトウェア開発(agile software development)などの別のプログラミング戦略を簡単に適用できます。

方法論の理想の1つは開発を始めると同時にできるかぎり早く顧客が動くプロトタイプを再検討して、指示を追加することです。アプリケーションは反復プロセスに組み込まれ、短い開発サイクルのなかで、しだいに機能がリッチなバージョンがリリースされます。

開発者の結末は無数にあります。開発者は機能を実装しているときに将来を考える必要はありません。使われるメソッドはできるかぎりシンプルで単刀直入であるべきです。この原則はKISS(Keep It Simple, Stupid)の格言で十分に説明されます。

要件が発展するときもしくは機能が追加されるとき、通常では既存のコードの一部を書き換えなければなりません。このプロセスはリファクタリング(refactoring)と呼ばれ、Webアプリケーションの開発プロセスにおいて多く行われます。コードは種類に合わせてほかの場所に移動させます。コードの重複部分を一カ所にまとめるようにリファクタリングが行われます。これはDRY(Don't Repeat Yourself)の原則の適用です。

そしてアプリケーションが定期的に変更されるときにアプリケーションがいつでも動作することを確認するには、自動化できるユニットテストのフルセットが必要です。よく書かれているのであれば、ユニットテストはコードの追加もしくはリファクタリングによって何も壊れていないことを保証する堅実な方法です。中にはコードを書くまえにテストを書くことをとり決める開発の方法論があります。これはテスト駆動開発(TDD - Test-Driven Development)と呼ばれます。

>**NOTE**
>アジャイル開発に関連する別の原則やよい習慣はたくさんあります。もっとも効果的なアジャイル開発の方法論の1つはエクストリームプログラミング(XP - Extreme Programming)と呼ばれ、XPの文献はアプリケーションを早くて効率的な方法で開発する方法をたくさん教えてくれます。よい出発点はKent Beck(ケント・ベック)によって執筆されたXPシリーズ(Addison-Wesley)です。

symfonyはRADを実践するための完璧なツールです。当然のことながら、symfonyフレームワークは独自プロジェクトのためにRADを適用するWeb制作会社によって開発されました。このことはsymfonyを利用することは新しい言語を学ぶことではなく、より効率的な方法でアプリケーションを開発するための正しい反射神経と最良の判断を身につけることを意味します。

### YAML

YAMLの公式Webサイト([http://www.yaml.org/](http://www.yaml.org/))によれば、YAMLとは"人間が読みやすくスクリプト言語とのインタラクションのために設計され機械が単刀直入に解析できるデータシリアライゼーションフォーマット"です。言い換えると、YAMLはXMLのような方法で記述されますがはるかにシンプルな構文を使います。このフォーマットはとりわけ配列とハッシュに変換されるデータを記入するために便利です。コードの例はつぎのとおりです:

    [php]
    $house = array(
      'family' => array(
        'name'     => 'Doe',
        'parents'  => array('John', 'Jane'),
        'children' => array('Paul', 'Mark', 'Simone')
      ),
      'address' => array(
        'number'   => 34,
        'street'   => 'Main Street',
        'city'     => 'Nowheretown',
        'zipcode'  => '12345'
      )
    );

つぎのYAMLの文字列を解析することで上記のPHP配列が自動的に作成されます:

    house:
      family:
        name:     Doe
        parents:
          - John
          - Jane
        children:
          - Paul
          - Mark
          - Simone
      address:
        number: 34
        street: Main Street
        city: Nowheretown
        zipcode: "12345"

YAMLにおいて、構造はインデント(字下げ)を通して示され、連続する項目はダッシュ(-)記号によって表現され、マップ内のキーと値の組はコロン(:)によって分割されます。YAMLには同じ構造をより少ない行で記述する省略記法があります。その省略記法では、配列は`[]`によってハッシュは`{}`によって明確に示されます。そのため、つぎのように前述のYAMLのデータはより短い表記で書けます:

    house:
      family: { name: Doe, parents: [John, Jane], children: [Paul, Mark, Simone] }
      address: { number: 34, street: Main Street, city: Nowheretown, zipcode: "12345" }

YAMLは"YAML Ain't Markup Language"の頭文字語で"yamel"(ヤメル)と発音します。このフォーマットは2001年ごろから存在しており、多種多様なプログラミング言語でYAMLパーサーが実装されています。

>**TIP**
>YAMLフォーマットの仕様ドキュメントは[http://www.yaml.org/](http://www.yaml.org/)で読むことができます。

YAMLはXMLよりもはるかに素早く記入することが可能で(閉じタグ、明示的なクォートが不要)、`.ini`ファイルよりもずっと強力です(階層をサポートしない)。設定を保存するためにsymfonyがYAMLを望ましい言語として利用する理由はそういうわけです。この本で多くのYAMLファイルを見ることになりますが、とても分かりやすいのでさらに学ぶ必要はないでしょう。

まとめ
-------

symfonyはPHP 5で動くWebアプリケーションフレームワークです。複雑なWebアプリケーションの開発を加速するツールを提供することで、 PHPの上に新しいレイヤーを追加します。この本でsymfonyのすべてをお教えします。必要なのは、現代プログラミングの基本概念、すわなち、オブジェクト指向プログラミング(OOP)、O/Rマッピング(ORM)、ラピッドアプリケーション開発(RAD)などに慣れていることです。求められる技術的な背景知識はPHP 5のみです。
