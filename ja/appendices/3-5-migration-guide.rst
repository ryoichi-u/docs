3.5 移行ガイド
##############

CakePHP 3.5 は、3.4 の API の完全上位互換です。
このページでは、3.5 の変更と改善についてのアウトラインを紹介します。

3.5.xにアップグレードするには、次のComposerコマンドを実行してください。

.. code-block:: bash

    php composer.phar require --update-with-dependencies "cakephp/cakephp:3.5.*"

非推奨
======

以下は、非推奨のメソッド、プロパティーと動作の一覧です。
これらの機能は、4.0.0 以後に削除されるまで機能し続けます。

* ``Cake\Http\Client\CookieCollection`` は非推奨です。代わりに ``Cake\Http\Cookie\CookieCollection`` を使用してください。
* ``Cake\View\Helper\RssHelper`` は非推奨です。まれ利用されることがあるためRssHelperは非推奨となっています。
* ``Cake\Controller\Component\CsrfComponent`` は非推奨です。代わりに :ref:`csrf-middleware` を使用してください。
* ``Cake\Datasource\TableSchemaInterface`` は非推奨です。 ``Cake\Database\TableSchemaAwareInterface`` を使用してください。
* ``Cake\Console\ShellDispatcher`` は非推奨です。アプリケーションでは代わりに ``Cake\Console\CommandRunner`` 使用するように更新してください。

非推奨の複合 get / set メソッド
-------------------------------

過去には、CakePHP は get / set モードの両方を提供する 'モーダル' メソッドを
利用していました。これらのメソッドにより、IDE の自動補完や、将来的に厳格な戻り値の型を追加する機能が
複雑になります。これらの理由から、複合 get / set メソッドは、
個別の get および set メソッドに分割されています。

推奨されなくなり、 ``getX()`` と ``setX()`` メソッドに置き換えられたメソッドのリストを次に示します。

``Cake\Cache\Cache``
    * ``config()``
    * ``registry()``
``Cake\Console\Shell``
    * ``io()``
``Cake\Console\ConsoleIo``
    * ``outputAs()``
``Cake\Console\ConsoleOutput``
    * ``outputAs()``
``Cake\Database\Connection``
    * ``logger()``
``Cake\Datasource\TypedResultTrait``
    * ``returnType()``
``Cake\Database\Log\LoggingStatement``
    * ``logger()``
``Cake\Datasource\ModelAwareTrait``
    * ``modelType()``
``Cake\Database\Query``
    * ``valueBinder()`` (今は ``getValueBinder()``)
``Cake\Datasource\QueryTrait``
    * ``eagerLoaded()`` (今は ``isEagerLoaded()``)
``Cake\Event\EventDispatcherTrait``
    * ``eventManager()``
``Cake\Error\Debugger``
    * ``outputAs()`` (今は ``getOutputFormat()`` / ``setOutputFormat()``)
``Cake\Http\ServerRequest``
    * ``env()`` (今は ``getEnv()`` / ``withEnv()``)
    * ``charset()`` (今は ``getCharset()`` / ``withCharset()``)
``Cake\I18n\I18n``
    * ``locale()``
    * ``translator()``
``Cake\ORM\LocatorAwareTrait``
    * ``tableLocator()``
``Cake\ORM\EntityTrait``
    * ``invalid()`` (今は ``getInvalid()``, ``setInvalid()``,
      ``setInvalidField()``, そして ``getInvalidField()``)
``Cake\ORM\Table``
    * ``validator()``
``Cake\Routing\RouteBuilder``
    * ``extensions()``
    * ``routeClass()``
``Cake\Routing\RouteCollection``
    * ``extensions()``
``Cake\TestSuite\TestFixture``
    * ``schema()``
``Cake\Utility\Security``
    * ``salt()``
``Cake\View\View``
    * ``template()``
    * ``layout()``
    * ``theme()``
    * ``templatePath()``
    * ``layoutPath()``
    * ``autoLayout()`` (今は ``isAutoLayoutEnabled()`` / ``enableAutoLayout()``)

振る舞いの変更
==============

以下の変更は、API 互換性はありますが、あなたのアプリケーションに影響を及ぼし得る
振る舞いのわずかな差異があります。

* ``BehaviorRegistry``, ``HelperRegistry`` 及び ``ComponentRegistry`` 未知のオブジェクト名で
  ``unload()`` が呼び出されたときに例外を発生させるようになりました。 
  この変更はタイポをより目立たせることでエラーを見つけやすくしています。
* ``HasMany`` は ``BelongsToMany`` と同様にアソシエーションのプロパティにからの値が
  設定されても正常に処理します。つまり空の配列と同じ方法で
  ``false`` 、 ``null`` 及び空の文字列を処理します。
  ``HasMany`` の場合アソシエーションの保存方法として ``replace`` は使用されているとき、
  関連するすべてのレコードが削除/リンク解除されます。
  その結果、フォームを使用して空の文字列を渡すことによって、関連するレコードをすべて
  削除/リンク解除することができます。これまではカスタムマーシャリングを作成する必要がありました。
* ``ORM\Table::newEntity()`` は 変換された関連付けレコードが ``dirty`` の場合にのみ
  アソシエーションプロパティに ``dirty`` をつけるようになりました。
  プロパティを含まない関連エンティティが作成される場合、
  空のレコードには永続化させるためのフラグはつきません。
* ``Http\Client`` はリクエストオブジェクトを作成するときに``cookie()`` の結果を
  使用しなくなりました
  代わりに ``Cookie`` ヘッダーと ``CookieCollection`` が使用できます。
  これは、クライアントにカスタムHTTPアダプターを使用しているときにのみ影響があります。
* シェルを呼び出すときにサブコマンドに複数文字を用いる場合、名前にはキャメルケースを
  使用する必要がありました。これからはアンダースコアでサブコマンドを呼び出すことができます。
  例えば、 ``cake tool initMyDb`` は ``cake tool init_my_db`` と呼び出すことができます。
  あなたのシェルが変換規則の異なる2つのサブコマンドを用いていた場合は
  最後に関連付けた規則のコマンドだけが機能します。
* ``SecurityComponent`` はリクエストデータを持たないPOSTリクエストを捕獲します。この変更は
  データベースのデフォルト値のみを使用してレコードを作成するアクションを保護するのに役立ちます。
* ``Cake\ORM\Table::addBehavior()`` と ``removeBehavior()`` は ``$this`` すようになりました。
  テーブルオブジェクトを流れるようなインターフェイスで定義するのに便利です。
* キャッシュエンジンは失敗または誤って構成されたときに例外を発生させなくなりました。
  代わりに、操作不能な ``NullEngine`` としてフォールバックさせます。フォールバックは
  エンジンごとに定義することができます。
  詳しくは、 :ref:`configured <cache-configuration-fallback>` をご覧ください
* ``Cake\Database\Type\DateTimeType`` は以前からのフォーマットに加えて ISO-8859-1 で
  フォーマットされた日付文字列（例えば、 2017-07-09T12:33:00+00:02) を変換するようになりました。
  DateTimeTypeのサブクラスを作成している場合はコードを更新する必要があります。

新機能の追加
============

スコープ付きミドルウェア
-----------------

特定のURLスコープのルートに条件付きでミドルウェアを適用できるようになりました。
これにより、ミドルウェア内部でURLチェックコードを記述せずに、
アプリケーションのさまざまな部分に対応するミドルウェア層を構築できます。
詳しくは、 :ref:`connecting-scoped-middleware` をご覧ください。

新しいコンソールランナー
------------------

3.5.0 では ``Cake\Console\CommandRunner`` が追加されました。
このクラスは ``Cake\Console\CommandCollection`` とともに、CLI環境と
新しい ``Application`` クラスを統合します。 ``Application`` クラスは
``console()`` フックを実装できるようになりました。これは、どのCLIコマンドが公開されているか、
それらがどのように命名されているか、シェルがどのように依存関係を取得するかを完全に制御できます。
この新しいクラスを採用するには ``bin/cake.php`` のファイルの内容を `こちら <https://github.com/cakephp/app/tree/3.next/bin/cake.php>`_ のファイルに置き換える必要があります。 


アクセス権の変更
================

* ``MailerAwareTrait::getMailer()`` は protected になります。
* ``CellTrait::cell()`` は protected になります。

上記のトレイトがコントローラーで使用されている場合、その public メソッドには、
デフォルトでアクションとしてルーティングすることでアクセスできます。これらの変更は、
コントローラーの保護に役立ちます。メソッドを公開する必要がある場合は、
``use`` ステートメントを次のように更新する必要があります。 ::

    use CellTrait {
        cell as public;
    }
    use MailerAwareTrait {
        getMailer as public;
    }


Collection
==========

* ``CollectionInterface::chunkWithKeys()`` が追加されました。
  ``CollectionInterface`` のユーザーランド実装は、現在このメソッドを実装する必要があります。
* ``Collection::chunkWithKeys()`` が追加されました。

エラー
======

* ``Debugger::setOutputMask()`` と ``Debugger::outputMask()`` が追加されました。
  これらのメソッドを使用すると、Debugger によって生成された出力からマスクする
  プロパティー/配列キーを設定することができます (たとえば ``debug()`` を呼び出すとき) 。

Event
=====

* ``Event::getName()`` が追加されました。
* ``Event::getSubject()`` が追加されました。
* ``Event::getData()`` が追加されました。
* ``Event::setData()`` が追加されました。
* ``Event::getResult()`` が追加されました。
* ``Event::setResult()`` が追加されました。

I18n
====

* フォールバックメッセージローダーの動作をカスタマイズできるようになりました。
  詳しくは、 :ref:`creating-generic-translators` をご覧ください。

ルーティング
============

* ``RouteBuilder::prefix()`` は、接続された各ルートに追加するデフォルトの配列を
  受け入れるようになりました。
* ルートは、 ``_host`` オプションを使用して特定のホストだけを一致させることができます。

Email
=====

* ``Email::setPriority()``/``Email::getPriority()`` が追加されました。

HtmlHelper
==========

* ``HtmlHelper::scriptBlock()`` は、デフォルトで ``<![CDATA[]]`` タグに JavaScript コードを
  ラップすることはありません。この動作を制御する ``safe`` オプションは、デフォルトで ``false`` に
  なりました。 ``<![CDATA []]`` タグを使うことは、もはや HTML ページで使われている主要な
  doctype ではない XHTML にのみ必要でした。

BreadcrumbsHelper
=================

* ``BreadcrumbsHelper::reset()`` が追加されました。
  このメソッドでは、既存のパンくずをクリアすることができます。

PaginatorHelper
===============

* ``PaginatorHelper::numbers()`` はデフォルトのテンプレートで '...' の代わりに
  HTML 省略記号を使用するようになりました。
* ``PaginatorHelper::total()`` が追加され、現在ページングされている結果の総ページ数が
  読み取れるようになりました。
* ``PaginatorHelper::generateUrlParams()`` が下位レベルの URL 構築メソッドとして追加されました。
* ``PaginatorHelper::meta()`` は 'first'、 'last' のリンクを作成できるようになりました。

FormHelper
==========

* FormHelper が読み込むソースを設定できるようになりました。これは、単純な GET のフォームを
  作成することができます。詳しくは、 :ref:`form-values-from-query-string` をご覧ください。
* ``FormHelper::control()`` が追加されました。
* ``FormHelper::controls()`` が追加されました。
* ``FormHelper::allControls()`` が追加されました。

Validation
==========

* ``Validation::falsey()`` と ``Validation::truthy()`` が追加されました。

TranslateBehavior
=================

* ``TranslateBehavior::translationField()`` が追加されました。

PluginShell
===========

* ``cake plugin load`` と ``cake plugin unload`` は ``--cli`` をサポートします。
  これは、代わりに ``bootstrap_cli.php`` を更新します。

TestSuite
=========

* ``PHPUnit 6`` のサポートが追加されました。PHP 5.6.0 を最低限必要とする
  このフレームワークバージョンでは、PHPUnit のサポートされているバージョンは、
  現在 ``^5.7|^6.0`` です。
