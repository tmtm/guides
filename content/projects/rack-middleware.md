---
title: Rackミドルウェア
order: 50
---

Hanamiはプロジェクトレベルの[Rackミドルウェアスタック](http://www.rubydoc.info/github/rack/rack/master/file/SPEC)を次のように設定します:

```ruby
# config/environment.rb
Hanami.configure do
  middleware.use MyRackMiddleware
end
```

これは`config.ru`ファイルにミドルウェアを追加するのと同じことは注目に値します。
唯一の違いは、サードパーティのプラグインが`Hanami.configure`にフックして独自のミドルウェアを注入できることです。
