---
title: プロジェクトセキュリティ
order: 60
---

現代のWeb開発は多くの課題を抱えており、それらのセキュリティは非常に重要であり、またしばしば強調されすぎていません。

Hanamiは脆弱性から保護する最も一般的な方法を提供します。セキュリティオプションは<code>application.rb</code>で設定できます。

## X-Frame-Options

`X-Frame-Options`は最近のブラウザでサポートされているHTTPヘッダです。信頼できないドメインによって`<frame>`および`<iframe>`タグを介してWebページを含めることができるかどうかを決定します。

Clickjacking攻撃を防ぐために、Webアプリケーションはこのヘッダーを送信できます:

```ruby
# Denies all untrusted domains (default)
security.x_frame_options 'DENY'
```

```ruby
# Allows iframes on example.com
security.x_frame_options 'ALLOW-FROM https://example.com/'
```

## X-Content-Type-Options

`X-Content-Type-Options`はブラウザがHTTPヘッダのContent-Typeによって宣言されたもの以外のものとしてファイルを解釈するのを防ぎます。

```ruby
# Will prevent the browser from MIME-sniffing a response away from the declared content-type (default)
security.x_content_type_options 'nosniff'
```

## X-XSS-Protection

`X-XSS-Protection`は、XSS攻撃が検出された場合のブラウザの動作を決定するためのHTTPヘッダーです。

```ruby
# Filter enabled. Rather than sanitize the page, when a XSS attack is detected,
# the browser will prevent rendering of the page (default)
security.x_xss_protection '1; mode=block'
```

```ruby
# Filter disabled
security.x_xss_protection '0'
```

```ruby
# Filter enabled. If a cross-site scripting attack is detected, in order to stop the attack,
# the browser will sanitize the page
security.x_xss_protection '1'
```

```ruby
# The browser will sanitize the page and report the violation.
# This is a Chromium function utilizing CSP violation reports to send details to a URI of your choice
security.x_xss_protection '1; report=http://[YOURDOMAIN]/your_report_URI'
```

## Content-Security-Policy

Content-Security-Policy (CSP)は、最近のブラウザでサポートされているHTTPヘッダーです。動的コンテンツ(JavaScript)またはその他のWeb関連アセット(スタイルシート、画像、フォント、プラグインなど)の信頼できるソースを特定します。

Webアプリケーションはこのヘッダーを送信してクロスサイトスクリプティング(XSS)攻撃を軽減することができます。

デフォルト値では、同じoriginの画像、スクリプト、AJAX、フォント、およびCSSが許可され、他のリソース(オブジェクト、フレーム、メディアなど)のロードは許可されません。

インラインJavaScriptは許可されていません。有効にするには、<code>script-src 'unsafe-inline'</code>を使用してください。

デフォルト値:

```ruby
security.content_security_policy %{
  form-action 'self';
  frame-ancestors 'self';
  base-uri 'self';
  default-src 'none';
  script-src 'self';
  connect-src 'self';
  img-src 'self' https: data:;
  style-src 'self' 'unsafe-inline' https:;
  font-src 'self';
  object-src 'none';
  plugin-types application/pdf;
  child-src 'self';
  frame-src 'self';
  media-src 'self'
}
```
