---
title: HTTP/2 Early Hints
order: 70
---

Webページは、スタイルシートや画像(アセット)などの外部リソースにリンクすることがあります。
HTTP/1.1では、ブラウザはHTMLを解析し、リンクごとにアセットをダウンロードし、最終的にそれに対してアクションを実行します: 画像のレンダリングまたはJavaScriptコードの評価。
HTTP/2では機能強化が導入されました: サーバーはHTMLペイロード**と**一部のアセットの両方を並行して事前にブラウザーに「プッシュ」することができます。このワークフローは、HTTP/2 TCP接続が多重化されているため許可されています。それは多くのコミュニケーションが同時に起こることができることを意味します。

残念ながらHTTP/2の採用はまだ遅いので、IETFはHTTPステータス[`103 Early Hints`](https://datatracker.ietf.org/doc/rfc8297/)を導入することで、このワークフローをHTTP/1.1にも「バックポート」しました。
この場合、サーバーは**単一の要求に対して1つ以上のHTTP応答**を送信します。最後のものは、ページのHTMLを返す伝統的な`200 OK`でなければなりませんが、最初の`n`は、ブラウザにアセットを事前に取得するように指示する特別なヘッダ`Link`を含めることができます。

## セットアップ

まず最初に、Early-Hintsを有効にした[Puma](http://puma.io/) `3.11.0+` が必要です:

```ruby
# Gemfile
gem "puma"
```

```ruby
# config/puma.rb
early_hints true
```

その後、プロジェクト構成から、この機能を簡単に有効にすることができます:

```ruby
# config/environment.rb
Hanami.configure do
  # ...
  early_hints true
end
```

最後のステップとして、HTTP/2と[h2o](https://h2o.examp1e.net/)のようなEarly HintsをサポートするWebサーバーが必要です。
サーバーを起動してページにアクセスすると、JavaScriptとスタイルシートがプッシュされます([アセットヘルパー](#assets-helpers)セクションを参照)。

### 他のWebサーバー

現在では、PumaだけがEarly Hintsをサポートしています。

## アセットヘルパー

あなたの資産を自動的にプッシュするために、あなたは私たちの[アセットヘルパー](/helpers/assets)を使わなければなりません。
しかし、ブラウザの制限(最大100アセットまでしかプッシュできません)により、デフォルトではHanamiはスタイルシートとJavaScriptのみを送信します。

<table class="table table-bordered">
  <thead>
    <tr>
      <th>Helper</th>
      <th>Early Hints asset type</th>
      <th>Pushed by default</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>javascript</code></td>
      <td><code>:script</code></td>
      <td>yes</td>
    </tr>
    <tr>
      <td><code>stylesheet</code></td>
      <td><code>:style</code></td>
      <td>yes</td>
    </tr>
    <tr>
      <td><code>favicon</code></td>
      <td><code>:image</code></td>
      <td>no</td>
    </tr>
    <tr>
      <td><code>image</code></td>
      <td><code>:image</code></td>
      <td>no</td>
    </tr>
    <tr>
      <td><code>video</code></td>
      <td><code>:video</code></td>
      <td>no</td>
    </tr>
    <tr>
      <td><code>audio</code></td>
      <td><code>:audio</code></td>
      <td>no</td>
    </tr>
    <tr>
      <td><code>asset_path</code></td>
      <td>N/A</td>
      <td>no</td>
    </tr>
    <tr>
      <td><code>asset_url</code></td>
      <td>N/A</td>
      <td>no</td>
    </tr>
  </tbody>
</table>

次の種類を**オプトインまたはオプトアウト**できます:

### JavaScript

デフォルトでプッシュされます:

```erb
<%= javascript "application" %>
<%= javascript "https://somecdn.test/framework.js", "dashboard" %>
```

オプトアウト:

```erb
<%= javascript "application", push: false %>
<%= javascript "https://somecdn.test/framework.css", "dashboard", push: false %>
```

### スタイルシート

デフォルトでプッシュされます:

```erb
<%= stylesheet "application" %>
<%= stylesheet "https://somecdn.test/framework.css", "dashboard" %>
```

オプトアウト:

```erb
<%= stylesheet "application", push: false %>
<%= stylesheet "https://somecdn.test/framework.css", "dashboard", push: false %>
```

### Favicon

オプトイン:

```erb
<%= favicon "favicon.ico", push: :image %>
```

### 画像

オプトイン:

```erb
<%= image "avatar.png", push: :image %>
```

### オーディオ

オプトイン:

```erb
<%= audio "song.ogg", push: true %>
```

ブロック構文(`song.ogg` のみプッシュ):

```erb
<%=
  audio do
    text "Your browser does not support the audio tag"
    source src: asset_path("song.ogg", push: :audio), type: "audio/ogg"
    source src: asset_path("song.wav"), type: "audio/wav"
  end
%>
```

### ビデオ

オプトイン:

```erb
<%= video "movie.mp4", push: true %>
```

ブロック構文(`movie.mp4` のみプッシュ):

```erb
<%=
  video do
    text "Your browser does not support the video tag"
    source src: asset_path("movie.mp4", push: :video), type: "video/mp4"
    source src: asset_path("movie.ogg"), type: "video/ogg"
  end
%>
```

### アセットパス

```erb
<%= asset_path "application.js", push: :script %>
```

### アセットURL

```erb
<%= asset_url "https://somecdn.test/framework.js", push: :script %>
```

## デモプロジェクト

完全に機能する例を探しているなら、この[デモプロジェクト](https://github.com/jodosha/hall_of_fame)をチェックしてください。
