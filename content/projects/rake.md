---
title: Rakeタスク
order: 20
---

HanamiにはデフォルトのRakeタスクが同梱されています。これは開発者が独自のタスクを作成するための _前提条件_ として使用できます。

```shell
$ bundle exec rake -T
rake environment # Load the full project
rake test        # Run tests (for Minitest)
rake spec        # Run tests (for RSpec)
```

## 環境

プロジェクトコード(エンティティ、アクション、ビューなど)にアクセスする必要がある場合は、これをRakeタスクの前提条件として使用します。

### 例

プロジェクトコード(リポジトリなど)にアクセスできるRakeタスクを作成したいとします。

```ruby
# Rakefile

task clear_users: :environment do
  UserRepository.new.clear
end
```

```shell
$ bundle exec rake clear_users
```

## テスト/スペック

これはテストスイートを実行するデフォルトのRakeタスクです。

以下のコマンドは同等です。

```shell
$ bundle exec rake
```

```shell
$ bundle exec rake test
```

<p class="convention">
  <code>:test</code> (または<code>--test=rspec</code>スイッチでアプリケーションを生成した場合は<code>:spec</code>) Rakeタスクがデフォルトです。
</p>

## Rubyサーバーホスティングエコシステムの互換性

Rubyサーバーホスティングエコシステムの多くのSoftware as a Service(SaaS)は、Ruby on Railsをモデルにしています。
たとえば、HerokuはRubyアプリケーションで次のRakeタスクを見つけることを期待しています:

  * `db:migrate`
  * `assets:precompile`

Herokuにはデプロイをカスタマイズする方法がないので、Ruby on Railsのこれらの「標準的な」Rakeタスクをサポートしています。

**あなたがあなたのデプロイを管理しているなら、これらのRakeタスクに頼らないでください。代わりに`hanami`[コマンドライン](/guides/1.2/command-line/database)を使ってください。**

## カスタムrakeタスク

カスタムrakeタスクを作成したい場合は、プロジェクトルートに`rakelib`フォルダを作成できます:

```
$ mkdir rakelib/
```

その後、`*.rake`ファイルを作成します。たとえば`export.rake`:

```ruby
# rakelib/export.rake

namespace :export do
  desc 'Export books to algolia service'
  task :books do
    ExportInteractor.new.call
  end
end
```

これで、カスタムrakeタスクがリストに表示されます:
Now you can see your custom rake task in the list:

```shell
$ bundle exec rake -T
rake export:books  # Export books to algolia service
rake environment   # Load the full project
rake spec          # Run RSpec code examples
```
