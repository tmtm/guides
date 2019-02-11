---
title: "概要"
order: 10
---

Hanami は2つの原則に基づいています: [クリーンアーキテクチャ](https://blog.8thlight.com/uncle-bob/2012/08/13/the-clean-architecture.html) と [Monolith First](http://martinfowler.com/bliki/MonolithFirst.html)。

## クリーンアーキテクチャ

このアーキテクチャの主な目的は、プロダクトの**コア**と**配信メカニズム**との間で**懸念を分離**することです。
前者は私たちのプロダクトが実装する**ユースケース**のセットで表現され、後者はこれらの機能を外の世界で利用できるようにするためのインターフェースです。

新しいプロジェクトを生成すると、2つの重要なディレクトリが見つかります: `lib/`と`apps/`。
それらは上記の主要部分のホームです。

### アプリケーションコア

どのように外部の世界に公開されるのかを心配することなく、一連の機能を実装します。
これが私たちのプロダクトの**土台**であり、私たちはそれに対する依存関係をどのように管理するかに注意を払いたいのです。

`Hanami::Model`は、私たちのRubyオブジェクトを永続化するためのデフォルトの選択です。
これは_ソフトな依存関係_です。`Gemfile`から削除して他のものに置き換えることができます。

`Hanami::Model`を使用する`bookshelf`という名前の新しく作られたプロジェクトに対して`lib/`ディレクトリがどのように表示されるかを見てみましょう。

```shell
$ tree lib
lib
├── bookshelf
│   ├── entities
│   ├── mailers
│   │   └── templates
│   └── repositories
└── bookshelf.rb

5 directories, 1 file
```

そのアイデアは私たちのアプリケーションをRuby gemのように開発することです。

`lib/bookshelf.rb`ファイルが私たちのアプリケーションの入り口になっています。このファイルが必要なときは、`lib/`下にあるすべてのコードを必要とし、初期化します。

2つの重要なディレクトリがあります:

  * `lib/bookshelf/entities`
  * `lib/bookshelf/repositories`

それらは私たちのモデルドメインの中核にあるRubyオブジェクトである[エンティティ](/guides/1.2/entities/overview)を含んでいます、そしてそれらはどんな永続化メカニズムも知りません。
この目的のために私たちは別の概念、[リポジトリ](/guides/1.2/repositories/overview)を持っています。それは私たちのエンティティと基礎となるデータベースの間の仲介者です。

`Book`というエンティティごとに、`BookRepository`を持つことができます。

私たちはユースケースを実装するために`lib/bookshelf/interactors`のように望むだけ多くのディレクトリを追加することができます。

### 配送メカニズム

Hanamiは`Web`という名前のデフォルトアプリケーションを生成します。これは`apps/web`下にあります。
このアプリケーションは、エンティティ、リポジトリ、およびそこで定義されている他のすべてのオブジェクトを使用するため、製品のコアに**依存**します。

それは私たちの機能のために、Web配信メカニズムとして使用されています。

```shell
$ tree apps/web
apps/web
├── application.rb
├── assets
│   ├── favicon.ico
│   ├── images
│   ├── javascripts
│   └── stylesheets
├── config
│   └── routes.rb
├── controllers
├── templates
│   └── application.html.erb
└── views
    └── application_layout.rb

8 directories, 5 files
```

このコードを簡単に見てみましょう。

ファイル`apps/web/application.rb`は、`Web::Application`という名前のHanamiアプリケーションを含んでいます。ここで、プロジェクトのこの**コンポーネント**のすべての設定を構成できます。
`apps/web/controllers`、`views`、`templates`などのディレクトリには、[アクション](/guides/1.2/actions/overview)、[ビュー](/guides/1.2/views/overview)、[テンプレート](/guides/1.2/views/templates)が含まれます。

JavaScriptやスタイルシートなどのWebアセットは、アプリケーションによって自動的に提供されます。

## モノリスファースト

デフォルトのアプリケーション`Web`は、顧客用のUIインターフェースとして使用できます。
私たちのストーリーのある時点で、管理パネルを使ってユーザーを管理したいと思います。

私たちが導入しようとしている機能のセットは私たちのメインのUI(`Web`)に属していないことを私たちは知っています。
一方、マイクロサービスアーキテクチャを実装するのは**時期尚早**です。これは、ユーザーが自分のパスワードをリセットできるようにすることだけを目的としています。

Hanamiは私たちの問題に対する解決策を持っています: 私たちは同じRubyプロセスの中にある新しいアプリを生成することができますが、それは別々のコンポーネントです。

```shell
$ bundle exec hanami generate app admin
```

このコマンドは私たちのプロジェクトのルートから実行しなければなりません。`apps/admin`下に新しいアプリケーション(`Admin::Application`)が生成されます。

私たちのプロダクトライフサイクルの後期段階では、これをスタンドアロンコンポーネントに抽出することにしました。
`apps/admin`下にあるすべてのものを別のリポジトリーに移動して別々にデプロイする必要があるだけです。

## プロジェクトの内部構造

`lib/`と`apps/`についてはすでに検討しましたが、新たに生成されたプロジェクトには説明が必要な部分が他にもあります。

```shell
$ tree -L 1
.
├── Gemfile
├── Gemfile.lock
├── README.md
├── Rakefile
├── apps
├── config
├── config.ru
├── db
├── lib
└── spec
```

簡単に紹介しましょう:

  * `Gemfile`と`Gemfile.lock`は[Bundler](http://bundler.io)の生成物です。
  * `README.md`はプロジェクトの設定方法と使用方法を教えてくれます。
  * `Rakefile`は私たちのプロジェクトのためのRakeタスクを記述しています。
  * `config/`は重要なファイル`config/environment.rb`が含まれていて、これはプロジェクトの**エントリポイント**です。
    それを要求することによって、私たちは依存関係(Ruby gems)、Hanamiフレームワーク、そして私たちのコードをプリロードします。
  * `config.ru`はRackサーバーがどのように私たちのアプリケーションを実行しなければならないかを記述するファイルです。
  * `db/`はデータベースファイルが含まれます(ファイルシステムアダプタやSQLite用)
    私たちのプロジェクトがSQLデータベースを使用するとき、それは`db/migrations`と`db/schema.sql`も含みます。
  * `spec/`はユニットテストと受け入れテストが含まれます。
