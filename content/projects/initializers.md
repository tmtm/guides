---
title: イニシャライザ
order: 40
---

プロジェクトは**オプションで** 1つ以上のカスタムイニシャライザを持つことができます。

<p class="notice">
  イニシャライザはオプションです
</p>

イニシャライザは、サードパーティのライブラリやその他のコードの側面を設定するために使用されるRubyファイルです。

これらは、依存関係、フレームワーク、プロジェクトコードがロードされた後 、サーバーまたはコンソールが起動される**前に** **最後に**実行されます。

たとえば、[Bugsnag](https://bugsnag.com)を自分のプロジェクト用に設定したい場合は、次のようにします:

```ruby
# config/initializers/bugsnag.rb
require 'bugsnag'

Bugsnag.configure do |config|
  config.api_key = ENV['BUGSNAG_API_KEY']
end
```

<p class="convention">
  プロジェクトイニシャライザは<code>config/initializers</code>下に追加されなければなりません。
</p>

<p class="warning">
  イニシャライザはアルファベット順に実行されます。
</p>
