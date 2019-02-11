---
title: インタラクター
order: 20
---

## 概要

Hanamiはあなたのコードを組織化するための**オプションの**ツールを提供します。

これらは*インタラクター*で、*サービスオブジェクト*、*ユースケース*、*オペレーション*とも呼ばれます。

私たちはそれらが素晴らしくて複雑さを管理するのを助けると思いますが、あなたがそれらなしでHanamiのアプリを作るのは自由です。

このガイドでは、Hanamiのインタラクターが既存のアプリケーションに小さな機能を追加することによってどのように機能するかを説明します。

これから作業する既存のアプリケーションは、 [入門ガイド](/guides/getting-started)の`bookshelf`アプリケーションです。

## 新機能: Eメール通知

私たちの新機能のストーリー:
> 管理者として、書籍が追加されたときにEメール通知を受け取りたい

アプリケーションには認証がないため、だれでも新しい書籍を追加できます。
環境変数で管理者のメールアドレスを提供します。

これはインタラクターをいつ使うべきか、そして特に`Hanami::Interactor`をどのように使うことができるかを示すほんの一例です。

この例は、新しい書籍本が投稿される前に管理者による承認を追加したり、
ユーザーが電子メールアドレスを入力してから特別なリンクを介してその書籍を編集することを許可するなど、
他の機能の基礎を提供します。

実際には、インタラクターを使用してWebから離れて抽象化された*任意のビジネスロジック*を実装できます。
コードベースの複雑さを管理するために、一度に複数のことを行いたい場合に特に役立ちます。

これは重要なビジネスロジックを分離するために使用されています。
これは[単一責任原則](https://en.wikipedia.org/wiki/Single_responsibility_principle)に従います。

Webアプリケーションでは、一般的にコントローラのアクションから呼び出されます。
これにより、懸念を分けることができます。
あなたのビジネスロジックオブジェクト、インタラクターは、ウェブについてまったく知りません。

## コールバック？ それは必要ありません！

Eメール通知を実装する簡単な方法は、コールバックを追加することです。

つまり: データベースに新しい`Book`レコードが作成された後、Eメールが送信されます。

設計上、Hanamiはそのようなメカニズムを提供していません。
これは、永続化コールバックを**アンチパターン**と見なしているためです。
それは単一責任の原則に違反しています。
この場合、それは永続化とEメール通知を不適切に混ぜ合わせます。

テスト中(そしておそらく他でも)、あなたはそのコールバックをスキップしたくなるでしょう。
これはすぐに混乱します。同じイベントに対する複数のコールバックが特定の順序で発生するためです。
また、ある時点でいくつかのコールバックをスキップしたいと思うかもしれません。
それらはコードを理解しにくく、もろくします。

代わりに、**暗黙的よりも明示的**であることをお勧めします。

インタラクターは特定の*ユースケース*を表すオブジェクトです。

それは各クラスに一つの責任を持たせます。
インタラクターの唯一の責任は、特定の結果を達成するためにオブジェクトとメソッド呼び出しを組み合わせることです。

私たちは`Hanami::Interactor`をモジュールとして提供しているので、
あなたは生の古いRubyオブジェクトから始め、
あなたがその機能のいくつかを必要とするとき`include Hanami::Interactor`することができます。

## コンセプト

インタラクターの背後にある中心的な考え方は、機能の分離した部品を新しいクラスに抽出することです。

2つのパブリックメソッドを書くだけです: `#initialize`と`#call`。

これは、オブジェクトは簡単に推論できることを意味します。
オブジェクトが作成された後に呼び出すことができるメソッドが1つだけだからです。

動作を単一のオブジェクトにカプセル化することで、テストが簡単になります。
暗黙のうちに表現されるだけで、複雑さを隠すのではなく、コードベースを理解しやすくします。

## 準備

[Getting Started](/getting-started)の`bookshelf`アプリケーションがあり、
「追加した書籍のEメール通知」機能を追加したいとしましょう。

## インタラクターの作成

インタラクター用のフォルダーとスペック用のフォルダーを作成しましょう:

```shell
$ mkdir lib/bookshelf/interactors
$ mkdir spec/bookshelf/interactors
```

それらはWebアプリケーションから切り離されているので、`lib/bookshelf`に入れました。
後で、管理者ポータル、API、あるいはコマンドラインユーティリティを使って書籍を追加したいと思うかもしれません。

私たちのインタラクターを`AddBook`と呼び、
新しいスペック`spec/bookshelf/interactors/add_book_spec.rb`を書きましょう:

```ruby
# spec/bookshelf/interactors/add_book_spec.rb

RSpec.describe AddBook do
  let(:interactor) { AddBook.new }
  let(:attributes) { Hash.new }

  it "succeeds" do
    result = interactor.call(attributes)
    expect(result.successful?).to be(true)
  end
end
```

`AddBook`クラスがないため、テストスイートを実行するとNameErrorが発生します。
そのクラスを`lib/bookshelf/interactors/add_book.rb`ファイルに作成しましょう:

```ruby
require 'hanami/interactor'

class AddBook
  include Hanami::Interactor

  def initialize
    # set up the object
  end

  def call(book_attributes)
    # get it done
  end
end
```

これらは、このクラスが持つべきたった2つのパブリックメソッドです:
`#initialize`(データをセットアップする)と、
`#call`(実際にユースケースを満たす)です。

これらのメソッド、特に`#call`はあなたが書くプライベートメソッドを呼び出すべきです。

デフォルトでは、成功したと見なされます。
明示的に失敗したとは言っていないためです。

このテストを実行しましょう:

```shell
$ bundle exec rake
```

すべてのテストに合格するはずです！

それでは、`AddBook`インタラクターに実際に何かをさせましょう！

## 書籍を作る

`spec/bookshelf/interactors/add_book_spec.rb` を編集:

```ruby
# spec/bookshelf/interactors/add_book_spec.rb

RSpec.describe AddBook do
  let(:interactor) { AddBook.new }
  let(:attributes) { Hash[author: "James Baldwin", title: "The Fire Next Time"] }

  context "good input" do
    let(:result) { interactor.call(attributes) }

    it "succeeds" do
      expect(result.successful?).to be(true)
    end

    it "creates a Book with correct title and author" do
      expect(result.book.title).to eq("The Fire Next Time")
      expect(result.book.author).to eq("James Baldwin")
    end
  end
end
```

`bundle exec rake`してテストを実行すると、次のエラーが表示されます:

```ruby
NoMethodError: undefined method `book' for #<Hanami::Interactor::Result:0x007f94498c1718>
```

インタラクターに記入してから、何をしたかを説明しましょう:

```ruby
require 'hanami/interactor'

class AddBook
  include Hanami::Interactor

  expose :book

  def initialize
    # set up the object
  end

  def call(book_attributes)
    @book = Book.new(book_attributes)
  end
end
```

ここで注意することが2つあります:

`expose :book`行は、返される結果のメソッドとして`@book`インスタンス変数を公開します。

`call`メソッドは`@book`変数に新しいBookエンティティを割り当てます。これは結果に公開されます。

テストは成功するはずです。

新しいBookエンティティを初期化しましたが、データベースに永続化されていません。

## 書籍を永続化する

タイトルと作者から渡された新しい`Book`が渡されましたが、データベースにはまだ存在しません。

それを持続させるためには私たちの`BookRepository`を使う必要があります。

```ruby
# spec/bookshelf/interactors/add_book_spec.rb

RSpec.describe AddBook do
  let(:interactor) { AddBook.new }
  let(:attributes) { Hash[author: "James Baldwin", title: "The Fire Next Time"] }

  context "good input" do
    let(:result) { interactor.call(attributes) }

    it "succeeds" do
      expect(result.successful?).to be(true)
    end

    it "creates a Book with correct title and author" do
      expect(result.book.title).to eq("The Fire Next Time")
      expect(result.book.author).to eq("James Baldwin")
    end

    it "persists the Book" do
      expect(result.book.id).to_not be_nil
    end
  end
end
```

テストを実行すると、新しい期待値が`Expected nil to not be nil.`で失敗するのがわかるでしょう。

これは、私たちが作った書籍が`id`を持っていないからです。
永続化されている場合はいつでも1つしか取得されないためです。

このテストに合格するには、代わりに_永続化された_ `Book`を作成する必要があります。
(もう1つの、同様に有効な選択肢は、私たちがすでに持っているBookを永続化させることです。)

私たちの`lib/bookshelf/interactors/add_book.rb`インタラクターの`call`メソッドを編集します:

```ruby
def call
  @book = BookRepository.new.create(book_attributes)
end
```

`Book.new`を呼び出す代わりに、
新しい`BookRepository`を作成し、それに属性を付けて`create`を送信します。

これでも`Book`が返されますが、このレコードもデータベースに保持されます。

今すぐテストを実行すると、すべてのテストに成功するはずです。

## 依存性注入

[依存性注入](https://martinfowler.com/articles/injection.html)を利用するために、実装をリファクタリングしましょう。

依存性注入を使用することをお勧めしますが、必須ではありません。
これは`Hanami::Interactor`の完全にオプションの機能です。

specは機能しますが、リポジトリの動作に依存しています(永続性が成功した後に`id`メソッドが定義されるようになります)。
これがリポジトリの動作の詳細です。
たとえば、永続化する*前に*UUIDを作成し、その永続化が`id`列を設定する以外の方法で成功したことを示す場合は、このspecを変更する必要があります。

specとインタラクターをより堅牢にするために変更することができます:
ファイルの外部での変更のために壊れる可能性は低くなります。

インタラクターで依存性注入を使用する方法は次のとおりです:

```ruby
require 'hanami/interactor'

class AddBook
  include Hanami::Interactor

  expose :book

  def initialize(repository: BookRepository.new)
    @repository = repository
  end

  def call(book_attributes)
    @book = @repository.create(book_attributes)
  end
end
```

`@repository`インスタンス変数を作成することは、少しコードがあるだけで、基本的に同じことです。

今のところ、私たちのspecは、`id`が移入されていることを確認することによって、リポジトリのふるまいをテストします
(`expect(result.book.id).to_not be(nil)`)。

これは実装の詳細です。

代わりに、リポジトリが`create`メッセージを確実に受信するようにspecを変更し、
リポジトリがそれを永続化することを信頼することができます(それがその責務です)。

変更して`it "persists the Book"`という期待を取り除き、
`context "persistence"`ブロックを作成しましょう:

```ruby
# spec/bookshelf/interactors/add_book_spec.rb

RSpec.describe AddBook do
  let(:interactor) { AddBook.new }
  let(:attributes) { Hash[author: "James Baldwin", title: "The Fire Next Time"] }

  context "good input" do
    let(:result) { interactor.call(attributes) }

    it "succeeds" do
      expect(result.successful?).to be(true)
    end

    it "creates a Book with correct title and author" do
      expect(result.book.title).to eq("The Fire Next Time")
      expect(result.book.author).to eq("James Baldwin")
    end
  end

  context "persistence" do
    let(:repository) { instance_double("BookRepository") }

    it "persists the Book" do
      expect(repository).to receive(:create)
      AddBook.new(repository: repository).call(attributes)
    end
  end
end
```

今私達のテストは懸念の境界に違反していません。

ここで行ったことは、インタラクターのリポジトリへの依存性を**注入**することです。
注: テストしていないコードでは、何も変更する必要はありません。
`repository:`キーワード引数のデフォルト値は、新しいリポジトリオブジェクトが渡されなかった場合にそれを提供します。

## Eメール通知

Eメール通知を追加しましょう！

別のライブラリを使うこともできますが、
私たちは`Hanami::Mailer`を使います。
(SMSの送信、チャットメッセージの送信、Webフックの呼び出しなど、何でもできます。)

```shell
$ bundle exec hanami generate mailer book_added_notification
      create  lib/bookshelf/mailers/book_added_notification.rb
      create  spec/bookshelf/mailers/book_added_notification_spec.rb
      create  lib/bookshelf/mailers/templates/book_added_notification.txt.erb
      create  lib/bookshelf/mailers/templates/book_added_notification.html.erb
```

[メーラの動作の詳細](/mailers/overview)については説明しませんが、
非常に簡単です: `Hanami::Mailer`クラス、関連するspec、および2つのテンプレート(プレーンテキスト用とhtml用)があります。

テンプレートは空のままにするので、
Eメールは空になり、
件名が「Book added!」となります。

メーラーspec`spec/bookshelf/mailers/book_added_notification_spec.rb`を編集します:

```ruby
# spec/bookshelf/mailers/book_added_notification_spec.rb

RSpec.describe Mailers::BookAddedNotification, type: :mailer do
  subject { Mailers::BookAddedNotification }

  before { Hanami::Mailer.deliveries.clear }

  it 'has correct `from` email address' do
    expect(subject.from).to eq("no-reply@example.com")
  end

  it 'has correct `to` email address' do
    expect(subject.to).to eq("admin@example.com")
  end

  it 'has correct `subject`' do
    expect(subject.subject).to eq("Book added!")
  end

  it 'delivers mail' do
    expect {
      subject.deliver
    }.to change { Hanami::Mailer.deliveries.length }.by(1)
  end
end
```

そしてメーラー`lib/bookshelf/mailers/book_added_notification.rb`を編集します:

```ruby
# lib/bookshelf/mailers/book_added_notification.rb

class Mailers::BookAddedNotification
  include Hanami::Mailer

  from    'no-reply@example.com'
  to      'admin@example.com'
  subject 'Book added!'
end
```

これですべてのテストは成功するはずです！


しかし、このメーラーはどこからも呼び出されません。
`AddBook`インタラクターからこのメーラーを呼び出す必要があります。

私たちのメーラが呼ばれるように、`AddBook`specを編集しましょう:

```ruby
  ...
  context "sending email" do
    let(:mailer) { instance_double("Mailers::BookAddedNotification") }

    it "send :deliver to the mailer" do
      expect(mailer).to receive(:deliver)
      AddBook.new(mailer: mailer).call(attributes)
    end
  end
  ...
```

テストスイートを実行するとエラーが表示されます: `ArgumentError: unknown keyword: mailer`。
これは理にかなっています、なぜなら私たちのインタラクターはただ一つのキーワード引数`repository`を持っているだけだからです。

初期化に新しい`mailer`キーワード引数を追加することによって、今私たちのメーラーを統合しましょう。

また、新しい`@mailer`インスタンス変数で`deliver`を呼び出します。

```ruby
require 'hanami/interactor'

class AddBook
  include Hanami::Interactor

  expose :book

  def initialize(repository: BookRepository.new, mailer: Mailers::BookAddedNotification.new)
    @repository = repository
    @mailer = mailer
  end

  def call(book_attributes)
    @book = @repository.create(book_attributes)
    @mailer.deliver
  end
end
```

これで私たちのインタラクターは、書籍が追加されたことを通知するEメールを配信します。

## コントローラとの統合

最後に、アクションからこのインタラクターを呼び出す必要があります。

アクションファイル`apps/web/controllers/books/create.rb`を編集します:

```ruby
  def call(params)
    if params.valid?
      @book = AddBook.new.call(params[:book])

      redirect_to routes.books_path
    else
      self.status = 422
    end
  end
```

私たちのスペックはまだ合格しますが、小さな問題があります。

書籍作成コードを**2回**テストしています。

これは一般的に悪い習慣であり、インタラクターの別の利点を説明することで修正できます。

再び依存性注入を使用します。
今回は、私たちのアクションの中で。

`interactor`用のキーワード引数で、`initialize`メソッドを追加します。

しかし、最初にspec`spec/web/controllers/books/create_spec.rb`を編集しましょう。

`BookRepository`への参照を削除し、
`AddBook`インタラクター用のdoubleを利用します:

```ruby
# spec/web/controllers/books/create_spec.rb

RSpec.describe Web::Controllers::Books::Create do
  let(:interactor) { instance_double('AddBook', call: nil) }
  let(:action) { described_class.new(interactor: interactor) }

  context 'with valid params' do
    let(:params) { Hash[book: { title: '1984', author: 'George Orwell' }] }

    it 'calls interactor' do
      expect(interactor).to receive(:call)
      response = action.call(params)
    end

    it 'redirects the user to the books listing' do
      response = action.call(params)

      expect(response[0]).to eq(302)
      expect(response[1]['Location']).to eq('/books')
    end
  end

  context 'with invalid params' do
    let(:params) { Hash[book: {}] }

    it 'calls interactor' do
      expect(interactor).to receive(:call)
      response = action.call(params)
    end

    it 're-renders the books#new view' do
      response = action.call(params)
      expect(response[0]).to eq(422)
    end

    it 'sets errors attribute accordingly' do
      response = action.call(params)

      expect(action.params.errors[:book][:title]).to eq(['is missing'])
      expect(action.params.errors[:book][:author]).to eq(['is missing'])
    end
  end
end
```

initializeをオーバーライドしていないので、テストはエラーを引き起こします。
それでは、`#call`メソッドで新しいインスタンス変数を活用しましょう:

```ruby
  ...
  def initialize(interactor: AddBook.new)
    @interactor = interactor
  end

  def call(params)
    if params.valid?
      @book = @interactor.call(params[:book])

      redirect_to routes.books_path
    else
      self.status = 422
    end
  end
  ...
```

今、私たちのspecは成功しており、それらははるかに堅牢です！

私たちのアクションは今や責任が少なくなっています。
それはその実際の振る舞いを私たちのインタラクターに委譲します。

アクションは(パラメータから)入力を受け取り、
実際にその仕事をするために私たちのインタラクターを呼び出します。
唯一の責任はWebを扱うことです。
私たちのインタラクターは現在、私たちの実際のビジネスロジックを扱います。

これは私たちのアクションとそのspecにとって大きな安心です。

私たちのアクションは主に私たちのビジネスロジックから解放されています。

インタラクターを変更するときに、
アクションまたはそのspecを変更する必要は**ありません**。

(実際のアプリケーションでは、結果が成功することを確認するなど、上記のロジック以上のことをしたいと思うでしょう。
そうでなければ、失敗した場合はインタラクターからのエラーを渡したいと思うでしょう。)

## インタラクター部

### インタフェース

上記のように、インターフェースはかなり単純です。
(オプションで)実装できるもう1つの方法もあります。
それは`#valid?`という名前のプライベートメソッドです。

デフォルトでは`#valid?`はtrueを返します。
`#valid?`を定義してfalseを返した場合、
`#call`は実行されません。

代わりに、結果はすぐに返されます。
これも結果を(成功ではなく)失敗にします。

それについては[APIドキュメント](http://www.rubydoc.info/gems/hanami-utils/Hanami/Interactor/Interface)で読むことができます。

### 結果

`Hanami::Interactor#call`の結果は`Hanami::Interactor::Result`オブジェクトです。

それはあなたが`expose`したどんなインスタンス変数に対しても定義されたアクセサメソッドを持ちます。

エラーを追跡する機能もあります。

あなたのインタラクターでは、エラーを追加するためにメッセージとともに`error`を呼び出すことができます。
これは自動的に結果のオブジェクトを失敗にします。

(`error!`メソッドもあり、
これは同じことをし、*そして*フローを中断し、
インタラクターがそれ以上コードを実行するのを止めます)。

結果のオブジェクトのエラーにアクセスするには、`.errors`を呼び出します。

[APIのドキュメント](http://www.rubydoc.info/gems/hanami-utils/Hanami/Interactor/Result)でResultオブジェクトについてもっと読むことができます。
