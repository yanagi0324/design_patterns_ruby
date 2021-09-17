# Rubyによるデザインパターン

**Rubyによるデザインパターン (著: ラス・オルセン）** という本のサンプル集です。

この本は、Gofの23のデザインパターンの14個をカバーしています。

* [テンプレートメソッド](#template-method)
* [ストラテジ](#strategy)
* [オブサーバー](#observer)
* [コンポジット](#composite)
* [イテレータ](#iterator)
* [コマンド](#commands)
* [アダプター](#adapter)
* [プロキシ](#proxy)
* [デコレーター](#decorator)
* [シングルトン](#singleton)
* [ファクトリー](#factory)
* [ビルダー](#builder)
* [インタープリタ](#interpreter)
* [DSL:Domain Specific Language](#dsl)
* [設定より規約](#convention-over-configuration)


----------------------------------------------------------------

## 基本原則

  * 変わるものを変わらないものから分離する
  * インターフェイスに対してプログラミングし、実装に対して行わない
  * 継承より集約
  * 委譲、委譲、委譲(delegation)
  * YAGNI(You Ain't Gonna Need It) - 必要になるまで作るな


## よりよいユーザーインターフェース

  * ユーザーのニーズを予測する - 何をしたいかというニーズを予測して、タスクの優先度をつける。そして、ありふれたタスクを難なくこなせるようにする。その中で最もよく出てくるケースをデフォルトにする。あまり出てこないタスクや高度なタスクは少し労力がかかってもよい。
  例えば、メールアプリで受信トレイや送信などはサイドバーに表示し、メールアプリの設定などは、設定画面を開き、タブで設定を探し、適切な設定を行うというようにタスクに優先度をつける。

  * ユーザーに何度も同じことを行わせない - 何度も同じことを聞かれたら苛立ってしまう。

  * すぐに始められるテンプレートを提供する - ユーザーがすぐに作業を始めれるように例とるなるものを提供することで、タスクの実施のとっかかりを見つけることができる。
  例えば、ワードは、真っ白なページ以外に、履歴書用や手紙用などのテンプレートを適切な余白と段落スタイルを設定して文書作成することができる。


## デザインパターン

### テンプレートメソッド<a name="template-method"></a>

目的/概要:
  * アルゴリズムに多様性をもたせたい場合に便利
  * 基底クラスに不変の部分を記述し、変わる部分はサブクラスに定義するメソッドにカプセル化する

やり方:
  1. アルゴリズム間で骨格となるメソッド(**テンプレートメソッド**)を定義した**基底クラス**を作成する
     基底クラスのテンプレートメソッドは、サブクラスのために標準実装をしておくか、raiseなどでサブクラスにオーバーライドを促すようにする。
  2. それぞれのアルゴリズムごとに**テンプレートメソッド**をオーバーライドした**サブクラス**を作成する

実例:
  * WEBrick
  * initializeメソッド - 初期化の最後にinitializeを呼び出すようにしている



### ストラテジ<a name="strategy"></a>

目的/概要:

  * アルゴリズムに多様性をもたせたい場合に便利(テンプレートメソッドパターンと同様の問題に対する委譲ベースのアプローチ)
  * アルゴリズムのパターン（出力形式など）ごとにばらばらのオブジェクトとしてシンプルに実装する。
    (テンプレートメソッドは、アルゴリズム中の変わる部分を抜き出してサブクラスに押し込む)
  * 委譲と集約を使っているので、テンプレートメソッドパターンの継承に比べて、実行時にアルゴリズム(ストラテジ)を変更することが容易。

やり方:
  1. すべて同じことを実行するクラス群（ストラテジー）を定義する（例：フォーマット出力、図を作成など）
  2. クラス群に対して、同じインターフェイスを共有し、あるクラスからそのインターフェイスを呼び出す（委譲）ことで、ストラテジを取り替え可能になる

注意点:
  * コンテキストとストラテジオブジェクト間に誤ったインターフェイスを設計しないようにする。
  * コンテキストと1つめのストラテジの依存関係を強くしすぎて、2,3つめのストラテジを盛り込めないといった状況にならないようにする。

実例:
  * rdocユーティリティ - Ruby, C, FORTRANのソースからドキュメントを抜き出せるので、パーサがストラテジになっている。またriコマンドでHTML形式やXML形式など出力形式を選べる。これもストラテジ。



### オブサーバー<a name="observer"></a>

目的/概要:

  * あるコンポーネントの動きを監視するコンポーネントを作ることができる。
    Observableオブジェクト(発信元)で発生したイベントをObserverオブジェクト(受信先)が受け取り処理をする。
  * ニュースの配信元（Observableオブジェクト）とニュースの消費者側（Observerオブジェクト）の間にインターフェイスをつくることで、Observerパターンはニュースをスムーズに渡すことができます。

やり方:
  1. Observableクラス(発信元)に対して、`include Observable`を宣言し、通知したい箇所で`notify_observers`メソッドを呼び出す。
  2. Observerクラス(受信先)に`update`メソッドを実装する
  3. どこかの処理で、Observableインスタンスの`add_observer`メソッドを使い、Observerインスタンスを登録する。

注意点:
  * Observableインスタンスの更新の頻度とタイミングを見極めないのと、莫大な通知イベントが走ってしまう。

実例:
  * ActiveRecord::Observer - レコードの作成、読み込み、書き込み、削除がされるたびに通知をうけたい場合、after_create, after_updateなどオブザーバーを定義できます。



### コンポジット<a name="composite"></a>

目的/概要:
  * フォルダ階層、組織階層、GUIのレイアウトなど**階層構造**をなす構造を表すときに使える
    (大きなオブジェクトが小さな子オブジェクトから構成されていて、その子オブジェクトも更に小さな孫オブジェクトでできているというパターン)
  * 子を持たない単一のコンポーネントの「リーフ」、複数の子(コンポーネント)を持つ複合的なコンポーネントの「コンポジット」、そして、リーフとコンポジットで共通する箇所を持つベースとなるコンポーネントの「コンポーネント」という３つの要素が必要です


やり方:
  1. すべてのオブジェクトの共通のインターフェースの「コンポーネント」を作成する
  2. 「コンポーネント」を継承し、単純な構成要素の「リーフ」を作成する(子を持たないレベル)
  3. 「コンポーネント」を継承し、複数の子要素(リーフ/コンポジット)を持つ「コンポジット」を作成する(親子関係を管理する責務を持つ)

注意点:
  * ツリーの深さが1段しかない(コンポジットオブジェクトの子コンポーネントがすべてリーフオブジェクト)と想定しない。CompositeがCompositeを保持している可能性が十分にあるので、再帰的に呼び出すようにする

実例:
  * GUIライブラリ(FXRuby)


### イテレータ<a name="iterator"></a>

目的/概要:
  * コレクションの各要素にアクセスする
  * 外部イテレータ(コレクションのメンバーを指し示すオブジェクト)と内部イテレータ(子オブジェクトを扱うためのコードを渡す)がある。

やり方:
  1. 基本的にはeach_xxxメソッドとしてRubyに定義されている
  2. 自前で作ったクラスにイテレータを実装したい場合、Enumerableモジュールをincludeし、eachメソッドを定義する。すると、さまざまなメソッドが使えるようになる。(参照:[Enumerable Module](http://docs.ruby-lang.org/ja/2.0.0/class/Enumerable.html))

注意点:
  * 集約オブジェクトを繰り返し走査しているときに、集約オブジェクトが変更(追加/削除など)が発生した場合に、うまく対処できない
  ```Ruby
  array = %w(red green blue purple)
  array.each do |color|
    puts color
    # 集約オブジェクトの操作中に、集約オブジェクトを削除している
    array.delete color if color == 'green'
  end
  ```

  blue が表示されない。greenを消したことでイテレータのインデックスがずれてしまったため。
  ```Ruby
  red
  green
  purple
  ```


実例:
  * Rubyの配列、ハッシュなどさまざまな箇所で見られる。
  * IOオブジェクトでは、「外部イテレータ」と「内部イテレータ」の両方がみれる
  ```Ruby
  # 外部イテレータ
  f = File.open('file.txt')
  until f.eof?
    puts f.readline
  end
  f.close


  # 内部イテレータ
  f = File.open('file.txt')
  f.each { |line| puts line }
  f.close
  ```


### コマンド<a name="commands"></a>

目的/概要:
  * ある特定の動作を実行するクラス(コマンド)を作成する。
  * これから行うことのリスト(ペンディングのコマンド群)や、完了したことのリスト(完了したコマンド群)を記録する場合にCommandパターンは役に立つ。

やり方:
  1. 単純な振る舞いを実行するだけの簡易なコマンドの場合は、RubyのコードブロックのProcオブジェクトを使いシンプルに実装する。
  2. 状態を保ち回ったり、いくつかのメソッドに分解したりする複雑なコマンドの場合は、コマンドクラスを作る。さまざまなことができるが、複雑になってしまう。

注意点:
  * 多くのCommandの操作は破壊的なので、戻す必要がある場合は、それも気にしながら実装する必要がある。

実例:
  * GUIフレームワークのボタンと動作内容(Command)
  * ActiveRecordのマイグレーション - upメソッドでマイグレーションを進め、downメソッドでマイグレーションを戻す


### アダプター<a name="adapter"></a>

目的/概要:
  * ソフトウェアインターフェイス間に存在する不整合のギャップを埋めるためにアダプターパターンを使う。
  * アダプタは、必要なインターフェイスと既存のオブジェクトの間の違いを吸収するためにある。
  * 不適切なインターフェイスを持つオブジェクトに困っていて、不適切なインターフェイスを扱う痛みがシステム中に広がることを防ぎたい場合に限り、アダプタを選ぶことが推奨されている。
  * アダプタクラスの作成(アダプターパターン)、クラスの変更(Rubyのオープンクラスを使う)、インスタンスの変更(Rubyの特異メソッドを使う)の3つの方法がある


アダプタパターン、クラスやインスタンスの変更の選定基準:
  * アダプタで複雑さと引き換えにカプセル化を守るか、クラス/インスタンスの変更で単純化と引き換えに、内部をいじるコストがかかる。
  * アダプタを使う
    - インターフェイスの不整合が広範囲に及んでいて複雑(文字列をFixnumとして動くようにするなど)
    - そのクラスがどのように動くかわからない
  * クラスやインスタンスの変更を行う
    - 変更内容がシンプル(メソッドに別名をつける程度)
    - 変更するクラスとその使用方法をよく理解している(思わぬ依存関係がありトラブルがおきるかも)


やり方:
  * (アダプタパターンの場合) クライアントとAdapteeの間にAdapterクラスを作り、クライアントからの操作(メソッド呼び出し)に対応するメソッド(インターフェイス)を用意し、その中でAdapeeの操作をラッピング(カプセル化)する。
  * (クラスやインスタンスの変更の場合) Adaptee自体をオープンクラスや特異メソッドを使い修正することで、クライアントからの操作(メソッドの呼び出し)に対応するメソッド(インターフェイス)を、Adapteeに用意する。

注意点:
  * Adapterでクライアントに対して提供したインターフェイス(メソッド)以外のメソッドが呼ばれた場合は対応できない。

実例:
  * ActiveRecord - ActiveRecordはMySQL,Oracle,Postgresなどあらゆるデータベースシステムと対話しなければならない。これらのDBはRuby APIを提供しているがそれぞれのDB毎にインターフェイスが異なっている(例えば、MySQLの場合 query, Oracleの場合 execute)。これらの違いすべてをカプセル化するAbstractAdapterという標準のインターフェイスを定義し、各DBに対応するAbstractAdapterのサブクラス内でDBのRuby APIに合わせた処理をすることで、同じインターフェイスでさまざまなDBを扱えるようにカプセル化しています。


### プロキシ<a name="proxy"></a>

目的/概要:
  * 「サブジェクトへのアクセス制限」、「サブジェクトの場所(リモート/ローカル)」「サブジェクトの生成を遅延」といったサブジェクトへのアクセスを制御できる
  * サブジェクトとアクセス元との間にプロキシクラスを作成し、上記のような制御を差し込むこと場所を作ることができる
  * アダプターとの違いは、アダプターはインターフェイスを変換するために使い、proxyはサブジェクトへのアクセスを制御するために使います。

やり方:
  * (防御プロキシ)サブジェクトを保持するプロキシクラスを作成し、アクセス制限を行う
  * (リモートプロキシ)サブジェクトへのアクセスをネットワークを介して行うプロキシクラスを作成する
  * (仮装プロキシ)サブジェクトを保持するプロキシクラスを作成し、サブジェクトの生成メソッドが呼ばれるまで遅延させる

注意点:
  * method_missingを使ったプロキシの場合、プロキシでラップしているオブジェクトをsendメソッドで呼び出す前にObjectクラスなどのプロキシクラスの親クラスで定義されているメソッドが呼ばれてしまう可能性があることを注意しておく必要がある。
  ```Ruby
  # もし、BankAccountのto_sメソッドを呼ぶことを意図していた場合でも、VirtualProxyのto_sメソッドが呼ばれてしまう。
  acount = VirtualProxy.new { BankAccount.new }
  puts account #=> #<VirtualProxy:0x40293b48>
  ```
  * method_missingを使ったプロキシの場合、実行速度が遅くなる。
  * method_missingを使ったプロキシの場合、継承の使いすぎと同様で、コードを分かりづらくしてしまう。

実例:
  * リモートプロキシ - RubyのSOAPクライアントやDistributed Ruby(drb)


### デコレーター<a name="decorator"></a>

目的/概要:
  * 基本的なオブジェクトにレイヤ状に機能を追加できるようにするためにデコレーターパターンが使われる。
  * 基本的な機能をカバーする１つのクラスと、それと組になる一式のデコレーターを作る。それぞれのデコレーターは核となる同じインターフェイスを持ち、インターフェイスに対してそれぞれのデコレーター特有の機能を実装する。
  * デコレーターはレイヤー状であり、あるメソッド呼び出しを受け、独自の処理を行い、次のデコレーターに処理結果を渡して、各デコレーターに機能を分解することができる。

やり方:
  1. 基本的な機能をカバーするデコレーターを作成する
  2. それを継承し、単一の機能のみをもったデコレーターを作成する
  3. デコレーターオブジェクトにデコレーターオブジェクトを保持させ、レイヤー状にさせることで複数のデコレーターの機能を使えるようにする

注意点:
  * あらゆる機能を構築する上で、様々な関心事をきちんと分離したいときに役に立つ。しかし、使うときに、機能全体として使うために、これらのクラスをかきあつめないといけない。
  * デコレーターの長い連鎖によるパフォーマンスのオーバーヘッドがある。

実例:
  * ActiveSupport - すべてのオブジェクトに alias_method_chain メソッドを追加している。これを使うと、自作のメソッドにたくさんの機能をデコレートすることができる。


### シングルトン<a name="singleton"></a>

目的/概要:
  * シングルトンとは、「正確に１つだけのインスタンスしかなく」かつ「そのインスタンスへのアクセスはどこからでも行える」という特徴を持つパターン。
  * ロガーや設定などのように、プログラム内で「たった１つだけの何かが存在するという状況を扱いやすくする」ために使う。
  * 例えば、ロガーのようなオブジェクトをそこら中で引き回さないですむことができる。

やり方:
  1. シングルトンにしたいクラスに`Singleton`モジュールを追加する。
     `Singleton`モジュールは、インスタンスを保持するクラス変数を定義し、それをシングルトンのインスタンスで初期化し、`instance`というシングルトンを取得するクラスメソッドを作成してくれ、`new`メソッドをプライベート化してくれる。

注意点:
  * シングルトンは特性上グローバルスコープに作られる。そのため、様々な各部分でシングルトンを通信チャネルとして使う事が簡単で、そうすることで各部分が密結合してしまう。
    そのため、シングルトンを部門間をつなぐコミュニケーションのパイプとしては使ってはいけない。
  * インスタンスへのアクセスはどこからでも行えるという特性のみに惑わされず、対象が実際にいくつ存在するかを考える。本当に唯一存在するものの場合のみシングルトンで実装する。
  * シングルトンはグローバルなのでテストケース間の独立性がなくなって単体テストがしずらい。1つの解決策は、次のように非シングルトンのクラスをつくり、それをテストするようにする。シングルトンは非シングルトンクラスを継承するようにする。
  ```Ruby
  require 'singleton'

  class SimpleLogger
    # ロガーのすべての機能はこのクラスがもつ
  end

  class SingletonLogger < SimpleLOgger
    include Singleton
  end
  ```

実例:
  * ActiveSupport(Railsで使われているユーティリティクラスのライブラリ)の命名規則 - Inflectionsクラスはシングルトンで作られており、どこでも同じ変換ルールで単数形や複数形を変換できるようにしている。
  * Rake(rubyでよく使われるビルドツール) - rakeプログラム全体から利用できるシングルトンとして、Rake::Applicationオブジェクトに格納する


### ファクトリ<a name="factory"></a>

目的/概要:
  * Factoryパターンとは、「どのクラスを選ぶのか？」という問いに対して答えるためのパターン
  * 「クラスの選択」の決定をサブクラスを実装することで決めるテクニックをFactory Methodパターンという。
    Template Methodパターンを、新しいオブジェクトの作成の問題に適用しただけ。
    欠点は、このパターンが作成するオブジェクトのクラスごとに別々のサブクラスを必要とするということ。
  * 矛盾のないオブジェクトの組み合わせを作るためにAbstract Factoryパターンを使う。
    矛盾のないオブジェクトの組み合わせをつくるために、Strategyパターンを適用した。

注意点:
  * よく失敗する理由は、Factoryを使うべきでないところで使ってしまうこと。複数の異なる関連したクラスがあり、その中から選ばなければいけない場合にのみ使用する。


### ビルダー<a name="builder"></a>

目的/概要:
  * Builderパターンは、複雑なオブジェクトを構築することに焦点を当てたパターン
  * 様々な構成パターンがあるオブジェクトの作成やオブジェクトが妥当かを検証することができる
  * Builderはnewメソッドを複数のメソッドに分けたようなもの

やり方:
  1. Director(オブジェクトの作成元)とProduct(作成するオブジェクト)の間にBuilderを作成する
  2. DirectorはBuilderを使うことで、複雑なProductを構築し、Productを取得するため、
     BuilderにProductを構築するメソッド群と構築されたresultメソッド(Productを取得するメソッド)を定義する
  3. (おまけ) Builderを使って作成したオブジェクトの妥当性を検証するために、resultメソッドに検証のコードを定義することもできる。


注意点:
  * Builderパターンを必要ないときに使ってしまう。基本的にはnewメソッドを使い、増大する要求が手に負えなくなったときに使用する。

実例:
  * MailFactory(Factoryという名前にもかかわらずBuilder)
  * ActiveRecordのfind系


### インタープリタ<a name="interpreter"></a>

目的/概要:
  * ある用途に専用の新しい言語を作ることで問題を解決するというアイデア
  * クエリや設定言語などの対象範囲が明確な問題を解決するのに都合がよく、既存の機能性の塊を１つに結合するためのよい選択肢です

やり方:
  1. 一連の式として新しい小さな言語を考え、そららの式を木構造に分解して抽象構文木を作成する
  2. 作成した構文木を評価するメソッドを追加する

注意点:
  * 十分に活用されていない傾向がある。適切にInterpreterパターンを適用すれば、システムに柔軟性を与えることができる。
  * 構文木を作成するパーサーの構築に手間がかかる。内部DSLを使えば比較的簡易にできる。
  * 複雑さやパフォーマンスの部分が問題がある。

実例:
  * Rubyそのもの
  * 正規表現


### DSL:Domain Specific Language<a name="dsl"></a>

目的/概要:
  * ある用途に専用の新しい言語(DSL)を作ることで問題を解決するというアイデア

やり方(内部DSL):
  1. Rubyの構文ルールに合うようにDSLを定義する
  2. DSLで書かれた、やるべきことが記述されているプログラムに必要な基盤も定義する
     ポイントは、DSLコードをRubyコードとして実行するために、`eval`を使ってDSLコードを読み込むこと。

注意点:
  * エラーメッセージがRubyのものになるので、Rubyに詳しくないDSLを使うユーザーは理解しづらい。
  * セキュリティが問題になる場合は、内部DSLの使用を避ける必要がある。

実例:
  * rake


### 設定より規約<a name="convention-over-configuration"></a>

目的/概要:
  * 設定ではなく、規約(アダプタはこのディレクトリに入れる、許可メソッドはこの形式で命名するなど)を設けて、それを運用する
  * 規約を使うことで、シンプルになり、適切な名前のファイルやクラス、メソッドを単に追加するだけでシステムを拡張することができ（クラスの登録などの設定がいらない）、比較的容易に拡張可能なプログラムを作成することができる。

やり方:
  1. 規約を設けて、それを運用する
     例: クラス名やファイル名、配置先の規則を作り、クラスのロード、クラス作成を自動で行えるようにし、クラスを作るだけで拡張できるようにする。

注意点:
  * 規約の出来が不完全な場合、その結果としてシステムにできることの幅を制限してしまう可能性がある。
    必要な場合、規約を上書きするクラスを定義できるようにすることで大抵の問題は解決出来る。
  * 規約をたくさん使っているシステムは新規ユーザーにとっては魔法のようで理解が困難。
    よくできた規約ベースのシステムは、運用のための地図を文書の形で提供する必要がある。

実例:
  * rails - リクエストされたurlより規約によりコントローラークラスとアクション名を呼び出す。そして、コントローラー名とアクション名から規約により表示するビューを呼び出す。

------

### Todo

  * 各デザインパターンのクラス図とそのときの挙動の流れを細くする。
  * わかりやすいように、「やり方」の各ステップの下にソースコードを記載し、最後に出力を記載したほうが良いかも。
