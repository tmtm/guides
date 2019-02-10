---
title: "はじめに"
order: 10
aliases: [
  "getting-started"
]
---

<p id="getting-started-lead" class="lead">
  こんにちは。あなたがこのページを見ているということはおそらくHanamiについてもっと学びたいということでしょう。
  すばらしい。おめでとう！ メンテ可能でセキュアで高速でテスト可能なWebアプリケーションを構築する新しい方法を探しているのであれば大丈夫です。
  <br><br>
  <strong>Hanamiはあなたのような人のために作られています。</strong>
  <br><br>
  あなたが完全な初心者であろうと経験豊富な開発者であろうと<strong>この学習プロセスは難しい</strong>ことを警告します。
  時間が経つにつれて、私たちは物事がどうあるべきかについての期待を築き、それを変えるのは苦痛になる可能性があります。 しかし、<strong>変化がなければ、挑戦はありません</strong>。そして挑戦がなければ、成長はありません。
  <br><br>
  機能が正しく見えないことがあっても、それはあなたが正しくないということではありません。
  それは、形成された習慣、設計上の誤り、またはバグである可能性があります。
  <br><br>
  私とコミュニティは、Hanamiを良くするために毎日最善の努力を払っています。
  <br><br>
   このガイドでは、最初のHanamiプロジェクトをセットアップし、簡単な本棚Webアプリケーションを作成します。
   テストによって導かれる、Hanamiフレームワークのすべての主要コンポーネントに触れます。
  <br><br>
  <strong>あなたが孤独を感じている、または欲求不満を感じているなら、あきらめないで、私たちの<a href="http://chat.hanamirb.org">チャット</a>に飛び乗って助けを求めてください。</strong>
  あなたと話をすることでより幸せになる人がいるでしょう。
  <br><br>
  エンジョイ<br>
  Luca Guidi<br>
  <em>Hanami作者</em>
</p>

<br>
<hr>

## 前提条件

始める前に、離れていくつかの前提条件を見てみましょう。
まず、Webアプリケーション開発の基本的な知識を想定しています。

また[Bundler](http://bundler.io), [Rake](http://rake.rubyforge.org), 端末の操作, および [Model, View, Controller](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller)パラダイムを使用したアプリケーションの作成にも精通している必要があります。

最後に、このガイドでは[SQLite](https://sqlite.org/)データベースを使います。
先に進む場合は、Ruby 2.3以降とSQLite 3以降がシステムにインストールされていることを確認してください。

## 新しいHanamiプロジェクトの作成

新しいHanamiプロジェクトを作成するには、RubygemsからHanami gemをインストールする必要があります。
それから、新しい`hanami`実行ファイルを使って新しいプロジェクトを生成することができます:

```shell
$ gem install hanami
$ hanami new bookshelf
```

<p class="notice">
 デフォルトでは、プロジェクトはSQLiteデータベースを使用するように設定されます。実際のプロジェクトでは、エンジンを指定できます:
  <br>
  <code>
    $ hanami new bookshelf --database=postgres
  </code>
</p>

これにより、現在の場所に新しいディレクトリ`bookshelf`が作成されます。
内容を見てみましょう:

```shell
$ cd bookshelf
$ tree -L 1
.
├── Gemfile
├── README.md
├── Rakefile
├── apps
├── config
├── config.ru
├── db
├── lib
├── public
└── spec

6 directories, 4 files
```

ここで私たちが知る必要があるもの:

* `Gemfile` 私たちのRubygemsの依存関係を定義しています(Bundlerを使って)。
* `README.md` プロジェクトの設定方法と使い方を教えてくれます。
* `Rakefile` 私たちのRakeタスクを記述しています。
* `apps` Rackと互換性のある1つ以上のWebアプリケーションが含まれていて、`web`という最初に生成されたHanamiアプリケーションを見つけることができます。
  そこには controllers, views, routes, templates が置かれています。
* `config` 設定ファイルを含んでいます。
* `config.ru` Rackサーバー用です。
* `db` データベーススキーマとマイグレーションを含んでいます。
* `lib` エンティティやリポジトリを含む私たちのビジネスロジックとドメインモデルが含まれています。
* `public` コンパイルされた静的アセットを含んでいます。
* `spec` テストを含んでいます。

Bundlerを使ってgemの依存関係をインストールしてください。それから開発用サーバーを立ち上げます:

```shell
$ bundle install
$ bundle exec hanami server
```

そして… [http://localhost:2300](http://localhost:2300)であなたの最初のHanamiプロジェクトの栄光を浴びてください！ 次のような画面が表示されます:

<p><img src="welcome-page.png" alt="Hanami welcome page" class="img-responsive"></p>

## Hanamiのアーキテクチャ

Hanamiのアーキテクチャは、多くの`apps`を含むプロジェクトを中心に解決します。
これらはすべて同じコードベースに存在し、同じRubyプロセスに存在します。

これらは `apps/` 配下にあります。

デフォルトでは、`web`アプリがあります。これは、標準のユーザー向けWebインターフェイスと見なすことができます。
これは最もポピュラーなので、おそらくあなたの将来のHanamiプロジェクトでそれを保ちたいと思うでしょう。
しかし、このアプリに特有のものは何もありません。Hanamiがそれを生成するのはごく普通のことです。

後で(実際のプロジェクトで)`admin`パネル、JSON `api`、または分析`dashboard`などの他のアプリを追加します。
また、`web`アプリを小さな部品に分割して、分離された機能の一部を抽出することもできます。
Hanamiもそれをフルサポートします！

さまざまな`apps`が__配信メカニズム__を表します。
つまり、それらはあなたのプロジェクトの中核、つまり「ビジネスロジック」と対話するためのさまざまな方法であるということです。

Hanamiは[自分自身を繰り返す](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)ことを望んでいないので、「ビジネスロジック」が共有されています。
Webアプリケーションは、ほとんどの場合、データベースに格納されているデータに作用します。
私たちの「ビジネスロジック」と永続性はどちらも`lib/`にあります。

(Hanamiのアーキテクチャは[Clean Architecture](https://blog.8thlight.com/uncle-bob/2012/08/13/the-clean-architecture.html)に強く触発されています)

## 最初のテストを書く

ブラウザをアプリに向けた時に目にするスクリーンは、ルートが定義されていない時に表示されるデフォルトページです

HanamiはWebアプリケーションを書く方法として[振る舞い駆動開発](https://en.wikipedia.org/wiki/Behavior-driven_development)(BDD)を奨励します。
最初のカスタムページを表示させるために、高レベルの機能テストを書きます:

```ruby
# spec/web/features/visit_home_spec.rb
require 'features_helper'

RSpec.describe 'Visit home' do
  it 'is successful' do
    visit '/'

    expect(page).to have_content('Bookshelf')
  end
end
```

Hanamiは、革新的な振る舞い駆動開発(BDD)ワークフローの準備ができていますが、**特定のテストフレームワークに制限されることは決してありません**。
特別なテスト統合やライブラリは付属していませんので、RSpec(またはMinitest)を知っていれば、そこで学ぶことは何も新しいことではありません。

次を実行して、テストデータベース内のスキーマをマイグレーションする必要があります:

```shell
$ HANAMI_ENV=test bundle exec hanami db prepare
```

最初の`HANAMI_ENV`は環境変数です。これは、Hanamiに`test`環境を使用するように指示します。
デフォルトは`HANAMI_ENV=development`であるため、これはここで必要です。

<p class="notice">
問題がおこった場合は、<tt>DATABASE_URL</tt>が<tt>.env.test</tt>に定義されています。
</p>


### リクエストに従う

これでテストが終わりました。失敗するのがわかります:

```shell
$ bundle exec rake
F.

Failures:

  1) Visit home is successful
     Failure/Error: expect(page).to have_content('Bookshelf')
       expected to find text "Bookshelf" in "404 - Not Found"
     # ./spec/web/features/visit_home_spec.rb:7:in `block (2 levels) in <top (required)>'

Finished in 0.02604 seconds (files took 1.14 seconds to load)
2 examples, 1 failure

Failed examples:

rspec ./spec/web/features/visit_home_spec.rb:4 # Visit home is successful
```

それでは成功させましょう。
このテストに成功するために必要なコードを段階的に追加します。

最初に追加する必要があるものはルートです：

```ruby
# apps/web/config/routes.rb
root to: 'home#index'
```

アプリのroot URLを`home`コントローラの`index`アクションに向けます(詳細は[ルーティングガイド](/routing/overview)を見てください)。

テストを実行すると、エンドポイントが見つからないというエラーが発生します。

`home#index`アクションを作成する必要があるので、それは合理的です。

```ruby
# apps/web/controllers/home/index.rb
module Web
  module Controllers
    module Home
      class Index
        include Web::Action

        def call(params)
        end
      end
    end
  end
end
```

これは何も特別なことをしない空のアクションです。
Hanamiの各アクションは[単一のクラス](https://en.wikipedia.org/wiki/Single_responsibility_principle)によって定義されているため、テストが簡単になります。
さらに、各アクションには対応するビューがあり、これもそのクラスによって定義されています。
これを要求を完了するために追加する必要があります。

```ruby
# apps/web/views/home/index.rb
module Web
  module Views
    module Home
      class Index
        include Web::View
      end
    end
  end
end
```

これも空で特別なことはしません。
その唯一の責任はテンプレートをレンダリングすることであり、これはビューがデフォルトで行うことです。

このテンプレートは、テストに合格するために追加する必要があるものです。
必要なのは本棚の見出しを追加するだけです。

```erb
# apps/web/templates/home/index.html.erb
<h1>Bookshelf</h1>
```

それではもう一度テストスイートを実行しましょう。

```shell
$ bundle exec rake

Finished in 0.01394 seconds (files took 1.03 seconds to load)
2 examples, 0 failures
```

これはすべてのテストが成功したことを意味します！

## 新しいアクションを生成する

Hanamiの __routes__, __actions__, __views__, __templates__ についての新しい知識を活用しましょう。

サンプルのBookshelfプロジェクトの目的は書籍を管理することです。

書籍をデータベースに保存し、ユーザーが私たちのプロジェクトを使ってユーザーがそれを管理できるようにします。

私たちの最初のステップは私たちが知ってるすべての書籍をリストすることです。

達成したいことを説明する新しい機能テストを書きましょう:

```ruby
# spec/web/features/list_books_spec.rb
require 'features_helper'

RSpec.describe 'List books' do
  it 'displays each book on the page' do
    visit '/books'

    within '#books' do
      expect(page).to have_css('.book', count: 2)
    end
  end
end
```

このテストは、[/books](http://localhost:2300/books)に行くと、
`book`クラスを持つ2つのHTML要素があり、
どちらも`books`というIDを持つHTML要素の中にあることを意味します。

私たちのテストスイートは`"#books" を見つけられません`。

その要素が欠けているだけでなく、
その要素を載せるページもありません！

それを修正するための新しいアクションを作成しましょう。

### Hanamiジェネレータ

Hanamiには、いくつかの**ジェネレータ**が付属しています。これらは、あなたのためにコードを作成するためのツールです。

ターミナルで実行しましょう:

```shell
$ bundle exec hanami generate action web books#index
```

<p class="notice">
  ZSHを使用していてそれが機能しない場合
  (<tt>zsh: no matches found: books#index</tt>のようなエラーで)、
  Hanamiは代わりにこのように書けます: <tt>hanami generate action web books/index</tt>
</p>

これは私たちのために多くのことを行います:

- `apps/web/controllers/books/index.rb`アクション(とそのspec)を作成し、
- `apps/web/views/books/index.rb`ビュー(とそのspec)を作成し、
- `apps/web/templates/books/index.html.erb`テンプレートを作成します。

('action' と 'controller' に混乱しているなら: Hanamiには`action`クラスしかないので、コントローラは関連するいくつかのアクションをまとめたモジュールです。)

これらのファイルはすべてほとんど空です。
そこにいくつかの基本的なコードを持っているので、Hanamiはクラスの使い方を知っています。
ありがたいことに、これらの5つのファイルを手動で作成する必要はありません。
その中にその特定のコードがあります。

ジェネレータはまた、`web`アプリケーションのルートファイル(`apps/web/config/routes.rb`)に新しいルートを追加します。

```ruby
get '/books', to: 'books#index'
```

テストを成功させるには、新しく生成されたテンプレートファイル `apps/web/templates/books/index.html.erb` を編集する必要があります:

```html
<h1>Bookshelf</h1>
<h2>All books</h2>

<div id="books">
  <div class="book">
    <h3>Patterns of Enterprise Application Architecture</h3>
    <p>by <strong>Martin Fowler</strong></p>
  </div>

  <div class="book">
    <h3>Test Driven Development</h3>
    <p>by <strong>Kent Beck</strong></p>
  </div>
</div>
```

これでテストは成功しました！

アプリケーションで新しいエンドポイント(ページ)を作成するためにジェネレータを使用しました。

しかし、私たちは自分自身を繰り返し始めました。

`books/index.html.erb`テンプレートと上記の`homes/index.html.erb`テンプレートの両方に`<h1>Bookshelf</h1>`を持っています。

これは大したことではありませんが、実際のアプリケーションでは、
`app`内のすべてのページでロゴまたは共通のナビゲーションが共有される可能性があります。

それがどのように機能するかを示すために、その繰り返しを修正しましょう。

### レイアウト

ひとつひとつのテンプレートで自分自身を繰り返さないようにするために、レイアウトテンプレートを修正することができます。
`apps/web/templates/application.html.erb`を次のように編集しましょう:

```rhtml
<!DOCTYPE html>
<html>
  <head>
    <title>Web</title>
    <%= favicon %>
  </head>
  <body>
    <h1>Bookshelf</h1>
    <%= yield %>
  </body>
</html>
```

また、重複している行は他のテンプレートから削除します。それらは今重複されたためです。

**レイアウトテンプレート**は他のテンプレートと同じですが、通常のテンプレートをラップするために使用されます。
`yield`行は、通常のテンプレートの内容に置き換えられます。
繰り返しのヘッダーとフッターを配置するのに最適な場所です。

## エンティティでデータをモデル化する

私たちのテンプレートでハードコーディングされた書籍は、明らかにインチキです。
動的データをアプリケーションに追加しましょう！

書籍をデータベースに保存して、ページに表示します。
そのためには、データベースを読み書きする方法が必要です。
これに使用するオブジェクトは2種類あります。

* **エンティティ**は、その識別子によって一意に識別されるドメインオブジェクト(`Book`)です。
* **リポジトリ**は、エンティティのデータを永続化、取得、削除するために、永続性レイヤーで使用します。

エンティティはデータベースをまったく認識していません。
これにより、**軽量**で**テストが容易**になります。

エンティティはデータベースから完全に分離されているので、
`Book`背後にあるデータを永続化するためにリポジトリを使用します。

(データベースからのデータを`Book`に戻したり、そのデータを削除するためにもリポジトリを使用します。)

[モデルガイド](/models/overview)でエンティティとリポジトリの詳細を読んでください 。

Hanamiはモデル用のジェネレータを同梱しているので、`Book`エンティティとそれに対応するリポジトリを作成するためにそれを使用しましょう:

```shell
$ bundle exec hanami generate model book
create  lib/bookshelf/entities/book.rb
create  lib/bookshelf/repositories/book_repository.rb
create  db/migrations/20181024110038_create_books.rb
create  spec/bookshelf/entities/book_spec.rb
create  spec/bookshelf/repositories/book_repository_spec.rb
```

ジェネレータはエンティティ、リポジトリ、そしてそれらに関連するテストファイルを提供してくれます。

データベースのマイグレーションも可能です。

### データベーススキーマを変更するためのマイグレーション

生成されたマイグレーションを変更して、`title`と`author`フィールドを含めます:

```ruby
# db/migrations/20181024110038_create_books.rb

Hanami::Model.migration do
  change do
    create_table :books do
      primary_key :id

      column :title,  String, null: false
      column :author, String, null: false

      column :created_at, DateTime, null: false
      column :updated_at, DateTime, null: false
    end
  end
end
```

Hanamiは私達のデータベーススキーマへの変更を記述するためのDSLを提供します。
マイグレーションがどのように機能するかについて詳しくは、[マイグレーションのガイド](/migrations/overview)を参照してください 。

今回は、エンティティの属性ごとに列を持つ新しいテーブルを定義します。
開発環境とテスト環境用にデータベースを準備しましょう:

```shell
$ bundle exec hanami db prepare
$ HANAMI_ENV=test bundle exec hanami db prepare
```

### エンティティとの連携

エンティティは、普通のRubyオブジェクトに非常に近いものです。
私たちはそれから欲しい振る舞いとそれを保存する方法に焦点を当てるべきです。

今のところ、簡単なエンティティクラスを作成する必要があります:

```ruby
# lib/bookshelf/entities/book.rb
class Book < Hanami::Entity
end
```

このクラスは、パラメータを初期化するために渡す各属性に対してゲッターとセッターを生成します。
ユニットテストで、すべて正常に機能することを確認できます:

```ruby
# spec/bookshelf/entities/book_spec.rb

RSpec.describe Book do
  it 'can be initialized with attributes' do
    book = Book.new(title: 'Refactoring')
    expect(book.title).to eq('Refactoring')
  end
end
```

### リポジトリの使用

これで私たちのリポジトリで遊ぶ準備が整いました。
Hanamiの`console`コマンドを使用して、アプリケーションをプリロードした状態で`irb`を起動することができるので、オブジェクトを使用できます:

```shell
$ bundle exec hanami console
>> repository = BookRepository.new
=> #<BookRepository relations=[:books]>
>> repository.all
=> []
>> book = repository.create(title: 'TDD', author: 'Kent Beck')
=> #<Book:0x007f9ab61c23b8 @attributes={:id=>1, :title=>"TDD", :author=>"Kent Beck", :created_at=>2018-10-24 11:11:38 UTC, :updated_at=>2018-10-24 11:11:38 UTC}>
>> repository.find(book.id)
=> #<Book:0x007f9ab6181610 @attributes={:id=>1, :title=>"TDD", :author=>"Kent Beck", :created_at=>2018-10-24 11:11:38 UTC, :updated_at=>2018-10-24 11:11:38 UTC}>
```

Hanamiリポジトリには、データベースから1つ以上のエンティティをロードし、既存のレコードを作成および更新するためのメソッドがあります。
リポジトリは、カスタムクエリを実装するための新しいメソッドを定義する場所でもあります。

要約すると、Hanamiがデータをモデル化するためにエンティティとリポジトリをどのように使用するかを説明しました。
エンティティは私たちの振る舞いを表しますが、リポジトリはマッピングを使って私たちのエンティティを私たちのデータストアに変換します。
データベーススキーマに変更を適用するためにマイグレーションを使用できます。

### 動的データの表示

データをモデル化した新しい体験により、書籍リストページに動的データを表示して作業を始めることができます。
先ほど作成した機能テストを調整しましょう:

```ruby
# spec/web/features/list_books_spec.rb
require 'features_helper'

RSpec.describe 'List books' do
  let(:repository) { BookRepository.new }
  before do
    repository.clear

    repository.create(title: 'PoEAA', author: 'Martin Fowler')
    repository.create(title: 'TDD',   author: 'Kent Beck')
  end

  it 'displays each book on the page' do
    visit '/books'

    within '#books' do
      expect(page).to have_selector('.book', count: 2), 'Expected to find 2 books'
    end
  end
end
```

テストで必要なレコードを作成してから、ページに正しい数の書籍クラスをアサートします。
このテストを実行すると、成功するはずです。 成功しない場合は、テストデータベースがマイグレーションされなかったことが考えられます。

これでテンプレートを変更して静的HTMLを削除できます。
私たちのビューは、すべての有効なレコードをループしてレンダリングする必要があります。
私たちのビューでこの変更を強制するためのテストを書きましょう:

```ruby
# spec/web/views/books/index_spec.rb

RSpec.describe Web::Views::Books::Index do
  let(:exposures) { Hash[books: []] }
  let(:template)  { Hanami::View::Template.new('apps/web/templates/books/index.html.erb') }
  let(:view)      { described_class.new(template, exposures) }
  let(:rendered)  { view.render }

  it 'exposes #books' do
    expect(view.books).to eq(exposures.fetch(:books))
  end

  context 'when there are no books' do
    it 'shows a placeholder message' do
      expect(rendered).to include('<p class="placeholder">There are no books yet.</p>')
    end
  end

  context 'when there are books' do
    let(:book1)     { Book.new(title: 'Refactoring', author: 'Martin Fowler') }
    let(:book2)     { Book.new(title: 'Domain Driven Design', author: 'Eric Evans') }
    let(:exposures) { Hash[books: [book1, book2]] }

    it 'lists them all' do
      expect(rendered.scan(/class="book"/).length).to eq(2)
      expect(rendered).to include('Refactoring')
      expect(rendered).to include('Domain Driven Design')
    end

    it 'hides the placeholder message' do
      expect(rendered).to_not include('<p class="placeholder">There are no books yet.</p>')
    end
  end
end
```

表示する書籍がない場合は、インデックスページに簡単なプレースホルダメッセージが表示されるように指定します。
書籍がある場合は、それらすべてを一覧表示します。
いくつかのデータを使ってビューをレンダリングするのは比較的簡単です。
Hanamiは、独立してテストするのは簡単で、それでもうまく機能する、最小限のインターフェースを持つ単純なオブジェクトを中心に設計されています。

これらの要件を実装するためにテンプレートを書き換えましょう:

```erb
# apps/web/templates/books/index.html.erb
<h2>All books</h2>

<% if books.any? %>
  <div id="books">
    <% books.each do |book| %>
      <div class="book">
        <h2><%= book.title %></h2>
        <p><%= book.author %></p>
      </div>
    <% end %>
  </div>
<% else %>
  <p class="placeholder">There are no books yet.</p>
<% end %>
```

今すぐ機能テストを実行すると、失敗することになります。
コントローラのアクションによって書籍がビューに[_表示_](/actions/exposures)されないためです。
その変更に対するテストを書くことができます。:

```ruby
# spec/web/controllers/books/index_spec.rb

RSpec.describe Web::Controllers::Books::Index do
  let(:action) { described_class.new }
  let(:params) { Hash[] }
  let(:repository) { BookRepository.new }

  before do
    repository.clear

    @book = repository.create(title: 'TDD', author: 'Kent Beck')
  end

  it 'is successful' do
    response = action.call(params)
    expect(response[0]).to eq(200)
  end

  it 'exposes all books' do
    action.call(params)
    expect(action.exposures[:books]).to eq([@book])
  end
end
```

コントローラアクションのテストを書くことは基本的に2つの部分から成ります: レスポンスのオブジェクト(Rack互換のステータス、ヘッダ、コンテンツの配列)に対するアサート、そしてアクションそれ自身(これには呼び出し後のexposuresが含まれます)。
これで、アクションが`:books`をexposeするように記述したので、アクションを実装できます:

```ruby
# apps/web/controllers/books/index.rb
module Web
  module Controllers
    module Books
      class Index
        include Web::Action

        expose :books

        def call(params)
          @books = BookRepository.new.all
        end
      end
    end
  end
end
```

アクションクラスで`expose`メソッドを使用することで、`@books`インスタンス変数の内容を外部に公開し、Hanamiがそれをビューに渡すことができるようにします。
すべてのテストに再び合格するのに十分です！

```shell
$ bundle exec rake
Run options: --seed 59133

# Running:

.........

Finished in 0.042065s, 213.9543 runs/s, 380.3633 assertions/s.

6 runs, 7 assertions, 0 failures, 0 errors, 0 skips
```

## レコードを作成するためのフォームの構築

最後の残りのステップの1つはシステムに新しい書籍を追加することを可能にすることです。
計画は簡単です: 詳細を入力するフォームを使ってページを作成します。

ユーザーがフォームを送信すると、新しいエンティティを作成して保存し、ユーザーを書籍の一覧にリダイレクトします。
これがテストで表現されたストーリーです:

```ruby
# spec/web/features/add_book_spec.rb
require 'features_helper'

RSpec.describe 'Add a book' do
  after do
    BookRepository.new.clear
  end

  it 'can create a new book' do
    visit '/books/new'

    within 'form#book-form' do
      fill_in 'Title',  with: 'New book'
      fill_in 'Author', with: 'Some author'

      click_button 'Create'
    end

    expect(page).to have_current_path('/books')
    expect(page).to have_content('New book')
  end
end
```

### フォームの基礎を築く

ここまでで、アクション、ビュー、およびテンプレートの動作に精通しているはずです。

私たちは物事を少しスピードアップするので、すぐに良い部分にたどり着くことができます。
まず、「New Book」ページに新しいアクションを作成します:

```
$ bundle exec hanami generate action web books#new
```

これは我々のアプリに新しいルートを追加します:

```ruby
# apps/web/config/routes.rb
get '/books/new', to: 'books#new'
```

興味深いのは新しいテンプレートです。私たちの`Book`エンティティの周りにHTMLフォームを構築するためにHanamiのフォームビルダーを使用するからです。

### フォームヘルパーを使う

このフォームを`apps/web/templates/books/new.html.erb`に作成するために、[フォームヘルパー](/helpers/forms)を使用しましょう:

```erb
# apps/web/templates/books/new.html.erb
<h2>Add book</h2>

<%=
  form_for :book, '/books' do
    div class: 'input' do
      label      :title
      text_field :title
    end

    div class: 'input' do
      label      :author
      text_field :author
    end

    div class: 'controls' do
      submit 'Create Book'
    end
  end
%>
```

フォームフィールドに`<label>`タグを追加し、Hanamiの[HTMLビルダーヘルパー](/helpers/html5)を使用して各フィールドをコンテナー`<div>`にラップしました。

### フォームを送信する

フォームを送信するには、さらに別のアクションが必要です。
`Books::Create`アクションを作成しましょう:

```
$ bundle exec hanami generate action web books#create
```

これは我々のアプリに新しいルートを追加します:

```ruby
# apps/web/config/routes.rb
post '/books', to: 'books#create'
```

### Createアクションの実装

私たちの`books#create`アクションは、2つのことをする必要があります。
それらをユニットテストとして表現しましょう:

```ruby
# spec/web/controllers/books/create_spec.rb

RSpec.describe Web::Controllers::Books::Create do
  let(:action) { described_class.new }
  let(:params) { Hash[book: { title: 'Confident Ruby', author: 'Avdi Grimm' }] }
  let(:repository) { BookRepository.new }

  before do
    repository.clear
  end

  it 'creates a new book' do
    action.call(params)
    book = repository.last

    expect(book.id).to_not be_nil
  end

  it 'redirects the user to the books listing' do
    response = action.call(params)

    expect(response[0]).to eq(302)
    expect(response[1]['Location']).to eq('/books')
  end
end
```

これらのテストに成功することは十分に簡単です。
データベースにエンティティを書き込む方法をすでに説明したので、`redirect_to`を使用してリダイレクションを実装できます:

```ruby
# apps/web/controllers/books/create.rb
module Web
  module Controllers
    module Books
      class Create
        include Web::Action

        def call(params)
          BookRepository.new.create(params[:book])

          redirect_to '/books'
        end
      end
    end
  end
end
```

この最小限の実装で、テストに合格することができます。

```shell
$ bundle exec rake
Run options: --seed 63592

# Running:

...............

Finished in 0.081961s, 183.0142 runs/s, 305.0236 assertions/s.

12 runs, 14 assertions, 0 failures, 0 errors, 2 skips
```

おめでとう！

### バリデーションによるフォームの保護

ちょっと待って！ 本当に堅牢なフォームを構築するためには、いくつかの追加対策が必要です。
ユーザーが値を入力せずにフォームを送信するとどうなるでしょうか？

データベースに不正なデータを埋め込むか、データ整合性違反の例外が発生する可能性があります。
システムから不正なデータを排除する方法が明らかに必要です！

テスト内でバリデーションを表現するために、疑問に思うべきです: 検証に失敗した場合どうなるのでしょうか？
1つ目の選択肢は`books#new`フォームを再レンダリングすることです。そのため、正しく完成させたときに、もう一度ショットを表示させることができます。
この動作を単体テストとして指定しましょう。

```ruby
# spec/web/controllers/books/create_spec.rb

RSpec.describe Web::Controllers::Books::Create do
  let(:action) { described_class.new }
  let(:repository) { BookRepository.new }

  before do
    repository.clear
  end

  context 'with valid params' do
    let(:params) { Hash[book: { title: 'Confident Ruby', author: 'Avdi Grimm' }] }

    it 'creates a new book' do
      action.call(params)
      book = repository.last

      expect(book.id).to_not be_nil
      expect(book.title).to eq(params.dig(:book, :title))
    end

    it 'redirects the user to the books listing' do
      response = action.call(params)

      expect(response[0]).to eq(302)
      expect(response[1]['Location']).to eq('/books')
    end
  end

  context 'with invalid params' do
    let(:params) { Hash[book: {}] }

    it 'returns HTTP client error' do
      response = action.call(params)
      expect(response[0]).to eq(422)
    end

    it 'dumps errors in params' do
      action.call(params)
      errors = action.params.errors

      expect(errors.dig(:book, :title)).to eq(['is missing'])
      expect(errors.dig(:book, :author)).to eq(['is missing'])
    end
  end
end
```

今回のテストでは、2つの代替シナリオ、つまり元のハッピーパスとバリデーションが失敗する新しいシナリオを指定します。
テストに成功するためには、バリデーションを実装する必要があります。

バリデーション規則をエンティティに追加することはできますが、Hanamiでは入力規則のソース、つまりアクションにできるだけ近いバリデーション規則を定義することもできます。
Hanamiコントローラのアクションは、許容可能な入力パラメータを定義するために`params`クラスメソッドを使用できます。

このアプローチはどのパラメータが使用されるかをホワイトリスト化し(信頼できないユーザ入力による大量割り当ての脆弱性を防ぐために他のものは破棄されます)、許容できる値を定義するルールを追加します。今回の場合は書籍のタイトルと著者がネストされた属性が存在するように指定しました。

バリデーションが整ったら、エンティティの作成とリダイレクトを、入力パラメータが有効な場合に限定することができます:

```ruby
# apps/web/controllers/books/create.rb
module Web
  module Controllers
    module Books
      class Create
        include Web::Action

        expose :book

        params do
          required(:book).schema do
            required(:title).filled(:str?)
            required(:author).filled(:str?)
          end
        end

        def call(params)
          if params.valid?
            @book = BookRepository.new.create(params[:book])

            redirect_to '/books'
          else
            self.status = 422
          end
        end
      end
    end
  end
end
```

パラメータが有効になると、Bookが作成され、アクションによって別のURLにリダイレクトされます。
しかし、パラメータが無効な場合はどうなりますか？

まず、HTTPステータスコードが[422 (Unprocessable Entity)](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes#422)に設定されます。
その後、コントロールは対応するビューに移動します。ビューはどのテンプレートをレンダリングするかを知る必要があります。
この場合、`apps/web/templates/books/new.html.erb`を使用してフォームを再度レンダリングします。

```ruby
# apps/web/views/books/create.rb
module Web
  module Views
    module Books
      class Create
        include Web::View
        template 'books/new'
      end
    end
  end
end
```

このアプローチはうまく機能します。Hanamiのフォームビルダーは、このアクションで`params`を調べて、パラメーターに見つかった値をフォームフィールドに入力するのに十分スマートだからです。
送信する前にユーザーが1つのフィールドにのみ入力すると、元の入力が表示されるので、もう一度入力する手間が省けます。

テストをもう一度実行して、テストがすべて成功していることを確認してください！

### バリデーションエラーの表示

何かがうまくいかなかったときにただユーザーをフォームに押し込むのではなく、私たちは彼らに彼らに何が期待されているかのヒントを与えるべきです。不正なフィールドに関する通知を表示するようにフォームを調整しましょう。

まず、`params`にエラーが含まれている場合、エラーのリストがページに含まれるようにします:

```ruby
# spec/web/views/books/new_spec.rb

RSpec.describe Web::Views::Books::New do
  let(:params)    { OpenStruct.new(valid?: false, error_messages: ['Title must be filled', 'Author must be filled']) }
  let(:exposures) { Hash[params: params] }
  let(:template)  { Hanami::View::Template.new('apps/web/templates/books/new.html.erb') }
  let(:view)      { described_class.new(template, exposures) }
  let(:rendered)  { view.render }

  it 'displays list of errors when params contains errors' do
    expect(rendered).to include('There was a problem with your submission')
    expect(rendered).to include('Title must be filled')
    expect(rendered).to include('Author must be filled')
  end
end
```

また、この新しい動作を反映するように機能仕様を更新する必要があります:

```ruby
# spec/web/features/add_book_spec.rb
require 'features_helper'

RSpec.describe 'Add a book' do
  # Spec written earlier omitted for brevity

  it 'displays list of errors when params contains errors' do
    visit '/books/new'

    within 'form#book-form' do
      click_button 'Create'
    end

    expect(current_path).to eq('/books')

    expect(page).to have_content('There was a problem with your submission')
    expect(page).to have_content('Title must be filled')
    expect(page).to have_content('Author must be filled')
  end
end
```

私たちのテンプレートでは、`params.errors`をループし(もしあれば)、友好的なメッセージを表示することができます。
`apps/web/templates/books/new.html.erb`を開きます:

```erb
<% unless params.valid? %>
  <div class="errors">
    <h3>There was a problem with your submission</h3>
    <ul>
      <% params.error_messages.each do |message| %>
        <li><%= message %></li>
      <% end %>
    </ul>
  </div>
<% end %>
```

テストをもう一度実行して、テストがすべて成功していることを確認してください！

```shell
$ bundle exec rake
Run options: --seed 59940

# Running:

..................

Finished in 0.078112s, 230.4372 runs/s, 473.6765 assertions/s.

15 runs, 27 assertions, 0 failures, 0 errors, 1 skips
```

### ルーターの使用を改善する

私達がしようとしている最後の改善は私達のルーターの使用です。
「web」アプリケーションのroutesファイルを開きます:

```ruby
# apps/web/config/routes.rb
post '/books',    to: 'books#create'
get '/books/new', to: 'books#new'
get '/books',     to: 'books#index'
root              to: 'home#index'
```

Hanamiは、これらのRESTスタイルのルートを構築するための便利なヘルパーメソッドを提供しています。これを使用すると、ルータを少し簡単にすることができます:

```ruby
root to: 'home#index'
resources :books, only: [:index, :new, :create]
```

どのルートが定義されているかを理解するために、この変更を加えました。特別なコマンドラインタスク`routes`を使用して最終結果を調べることができます:

```shell
$ bundle exec hanami routes
     Name Method     Path                           Action

     root GET, HEAD  /                              Web::Controllers::Home::Index
    books GET, HEAD  /books                         Web::Controllers::Books::Index
 new_book GET, HEAD  /books/new                     Web::Controllers::Books::New
    books POST       /books                         Web::Controllers::Books::Create
```

`hanami routes`の出力は、定義されたヘルパーメソッドの名前(あなたは`_path`または`_url`で接尾辞を付けて`routes`ヘルパーでそれを呼び出すことができます)、許可されたHTTPメソッド、パスそして最後にリクエストを処理するために使用されるコントローラアクションを示します。

これで、`resources`ヘルパーメソッドを適用しました。名前付きルートメソッドの利点を得ることができます。
`form_for`を使用してフォームを作成した方法を覚えていますか？

```erb
# apps/web/templates/books/new.html.erb
<h2>Add book</h2>

<%=
  form_for :book, '/books' do
    # ...
  end
%>
```

私たちのルーターがすでにどのルートを指し示すべきかを完全に認識しているときは、ハードコーディングされたパスを私たちのテンプレートに含めるのは愚かです。
ビューとアクションで利用可能な`routes`ヘルパーメソッドを使用して、ルート固有のヘルパーメソッドにアクセスできます:

```erb
# apps/web/templates/books/new.html.erb
<h2>Add book</h2>

<%=
  form_for :book, routes.books_path do
    # ...
  end
%>
```

`apps/web/controllers/books/create.rb` にも同様の変更を加えることができます:

```ruby
redirect_to routes.books_path
```

## まとめ

**最初のHanamiプロジェクトの完成おめでとう！**

私たちが行ったことを見てみましょう: 私たちはHanamiの主要なフレームワークを通してそれらがどのように関連しているかを理解するためにリクエストをトレースしました。エンティティとリポジトリを使用してドメインをモデル化する方法を説明しました。フォームを構築し、データベーススキーマを維持し、ユーザー入力を検証するためのソリューションを見てきました。

私たちは長い道のりを歩んできましたが、探求するべきことがまだたくさんあります。
[他のガイド](/)、[Hanami APIドキュメント](https://docs.hanamirb.org)を調べ、[ソースコード](https://github.com/hanami)を読み、[ブログ](http://hanamirb.org/blog)をたどってください。

**何よりも、素晴らしいものを作ることを楽しもう！**
