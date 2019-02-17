---
title: アプリを選択的に起動する
order: 80
---

Hanamiでは、[Monolith-First](/architecture/overview/#monolith-first)原則に従ってプロジェクトを構築できます。
プロジェクトにコードを追加するにつれて、プロジェクトを複数のHanamiアプリに分割することで、それを有機的拡張できます。

実際のHanamiプロジェクトでは、**同じプロジェクト内に多数のHanamiアプリ**(たとえば、フロントエンド用の`web`、管理用の`admin`、JSON API用のAPIなど)を持つことができます。
それらはすべて同じプロジェクトの一部ですが、異なるサーバーにデプロイしたいかもしれません。
たとえば、ほとんどのサーバーは`web`アプリ(サイトの顧客用)に、いくつかは`api`(おそらくモバイルアプリを使用する顧客用)に使用し、そして、単一のサーバーでadminアプリを実行することもできます。これは他の2つよりもトラフィックが少なくなる可能性があるためです。

_選択的起動_ でこれをサポートします:

```ruby
# config/environment.rb
# ...
Hanami.configure do
  if Hanami.app?(:web)
    require_relative '../apps/web/application'
    mount Web::Application, at: '/'
  end

  if Hanami.app?(:api)
    require_relative '../apps/api/application'
    mount Api::Application, at: '/api'
  end

  if Hanami.app?(:admin)
    require_relative '../apps/admin/application'
    mount Admin::Application, at: '/admin'
  end
end
```

`HANAMI_APPS`環境変数を使用して、使用するアプリを宣言できます。
単一のアプリ、または複数のアプリ(コンマで結合)を指定できます:

```shell
$ HANAMI_APPS=web,api bundle exec hanami server
```

This would start only the `web` and `api` applications.
