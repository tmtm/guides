---
title: ロギング
order: 30
---

各プロジェクトは`Hanami.logger`で利用可能なグローバルロガーを持っています。これは次のように使用できます: `Hanami.logger.debug "Hello"`

それは`config/environment.rb`で設定することが出来ます。

```ruby
# config/environment.rb
# ...

Hanami.configure do
  # ...

  environment :development do
    logger level: :info
  end

  environment :production do
    logger level: :info, formatter: :json

    # ...
  end
end
```

ほとんどのホスティングSaaS企業が[推奨](https://devcenter.heroku.com/articles/rails4#logging-and-assets)する[ベストプラクティス](http://12factor.net/logs)であるため、デフォルトでは標準出力が使用されます。

ファイルを使いたい場合は、`stream: 'path/to/file.log'`をオプションとして渡します。

##  機密情報のフィルタ

Hanamiは自動的に非GET HTTPリクエストの本体をログに記録します。

ユーザーがフォームを送信すると、すべてのフィールドとその値がログに現れます:

```log
[bookshelf] [INFO] [2017-08-11 18:17:54 +0200] HTTP/1.1 POST 302 ::1 /signup 5 {"signup"=>{"username"=>"jodosha", "password"=>"secret", "password_confirmation"=>"secret", "bio"=>"lorem"}} 0.00593
```

機密情報がログに記録されるのを防ぐために、それらをフィルタリングすることができます:

```ruby
# config/environment.rb
# ...

Hanami.configure do
  # ...
  environment :development do
    logger level: :debug, filter: %w[password password_confirmation]
  end
end
```

出力は次のようになります:

```log
[bookshelf] [INFO] [2017-08-11 18:17:54 +0200] HTTP/1.1 POST 302 ::1 /signup 5 {"signup"=>{"username"=>"jodosha", "password"=>"[FILTERED]", "password_confirmation"=>"[FILTERED]", "bio"=>"lorem"}} 0.00593
```

同じ名前のパラメータを明確にするためのきめ細かいパターンもサポートしています。
たとえば、ストリート番号とクレジットカード番号を含む請求フォームがあり、クレジットカードのみをフィルタリングたい場合:

```ruby
# config/environment.rb
# ...

Hanami.configure do
  # ...
  environment :development do
    logger level: :debug, filter: %w[credit_card.number]
  end
end
```

```log
[bookshelf] [INFO] [2017-08-11 18:43:04 +0200] HTTP/1.1 PATCH 200 ::1 /billing 2 {"billing"=>{"name"=>"Luca", "address"=>{"street"=>"Centocelle", "number"=>"23", "city"=>"Rome"}, "credit_card"=>{"number"=>"[FILTERED]"}}} 0.009782
```

`billing => address => number` はフィルタされず、代わりに `billing => credit_card => number` がフィルタされたことに注意してください。

本体のログを完全に無効にしたい場合は、カスタムフォーマッタを使用すると簡単に実現できます:

```ruby
class NoParamsFormatter < ::Hanami::Logger::Formatter
  def _format(hash)
    hash.delete :params
    super hash
  end
end
```

そして、loggerに新しいフォーマッタを使ってロギングするように伝えるだけです

```ruby
logger level: :debug, formatter: NoParamsFormatter.new
```

## 任意の引数

Rubyの`Logger`と互換性のある[任意の引数](https://ruby-doc.org/stdlib/libdoc/logger/rdoc/Logger.html#class-Logger-label-How+to+create+a+logger)を指定できます。

毎日のログローテーションを設定する方法は次のとおりです:

```ruby
# config/environment.rb
# ...

Hanami.configure do
  # ...

  environment :production do
    logger 'daily', level: :info, formatter: :json, stream: 'log/production.log'

    # ...
  end
end
```

あるいは、ファイル数(`10`としましょう)と各ファイルのサイズ(例: `1,024,000`バイト、`1`メガバイト)に制限を設けることもできます:

```ruby
# config/environment.rb
# ...

Hanami.configure do
  # ...

  environment :production do
    logger 10, 1_024_000, level: :info, formatter: :json, stream: 'log/production.log'

    # ...
  end
end
```

## 自動ロギング

すべてのHTTP要求、SQL照会、およびデータベース操作は自動的にロギングされます。

プロジェクトが開発モードで使用されている場合、ロギング形式は人間が判読可能です:

```ruby
[bookshelf] [INFO] [2017-02-11 15:42:48 +0100] HTTP/1.1 GET 200 127.0.0.1 /books/1  451 0.018576
[bookshelf] [INFO] [2017-02-11 15:42:48 +0100] (0.000381s) SELECT "id", "title", "created_at", "updated_at" FROM "books" WHERE ("book"."id" = '1') ORDER BY "books"."id"
```

プロダクション環境の場合、デフォルトのフォーマットはJSONです。
JSONは解析可能で、よりマシン指向です。ログ収集やSaaSロギング製品に最適です。

```json
{"app":"bookshelf","severity":"INFO","time":"2017-02-10T22:31:51Z","http":"HTTP/1.1","verb":"GET","status":"200","ip":"127.0.0.1","path":"/books/1","query":"","length":"451","elapsed":0.000391478}
```

## カスタムロガー

異なるロギング動作が必要な場合は、カスタムロガーを指定できます。たとえば、[Timberロガー](https://github.com/timberio/timber-ruby):

```ruby
# config/environment.rb
# ...

Hanami.configure do
  # ...

  environment :production do
    logger Timber::Logger.new(STDOUT)

    # ...
  end
end
```

`Hanami.logger`経由で通常通りこのロガーを使用してください。選択されたどのロガーもデフォルトの`::Logger`インターフェースに準拠しなければならないことに注意することが重要です。

## 色付け

### 色付けを無効にする

色付けを無効にするには:

```ruby
# config/environment.rb
# ...

Hanami.configure do
  # ...

  environment :development do
    logger level: :info, colorizer: false
  end
end
```

### カスタム色付け

あなたはあなた自身の色付け戦略を構築することができます

```ruby
# config/environment.rb
# ...
require_relative "./logger_colorizer"

Hanami.configure do
  # ...

  environment :development do
    logger level: :info, colorizer: LoggerColorizer.new
  end
end
```

```ruby
# config/logger_colorizer.rb
require "hanami/logger"
require "paint" # gem install paint

class LoggerColorizer < Hanami::Logger::Colorizer
  def initialize(colors: { app: [:red, :bright], severity: [:red, :blue], datetime: [:italic, :yellow] })
    super
  end

  private

  def colorize(message, color:)
    Paint[message, *color]
  end
end
```
