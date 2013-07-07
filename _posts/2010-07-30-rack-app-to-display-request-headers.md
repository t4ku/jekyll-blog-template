--- 
layout: post
title: "RackでHTTPリクエストを確認"
tags: 
- Programming
- HTTP
- iphone
- Rack
- ruby
status: publish
type: post
published: true
meta: 
  _edit_lock: "1280560569"
  _edit_last: "1"
---
全然用途が違うとおもったのですが、iPhoneのUIWebViewやmobile Safariがどういうリクエストを投げているか確認するために、リクエストHeaderを表示する簡単なRackアプリを作ってみた。

<script src="http://gist.github.com/501834.js"> </script>

<br/>
PCのブラウザ(Firefox)でアクセスすると、こんな感じです。
<br/>

<pre class="brush: plain">


GATEWAY_INTERFACE: CGI/1.1
HTTP_ACCEPT: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
HTTP_ACCEPT_CHARSET: Shift_JIS,utf-8;q=0.7,*;q=0.7
HTTP_ACCEPT_ENCODING: gzip,deflate
HTTP_ACCEPT_LANGUAGE: ja,en-us;q=0.7,en;q=0.3
HTTP_CONNECTION: keep-alive
HTTP_HOST: localhost:9923
HTTP_KEEP_ALIVE: 115
HTTP_USER_AGENT: Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10.6; ja-JP-mac; rv:1.9.2.8) Gecko/20100722 Firefox/3.6.8
HTTP_VERSION: HTTP/1.1
PATH_INFO: /
QUERY_STRING:
REMOTE_ADDR: ::1
REMOTE_HOST: localhost
REQUEST_METHOD: GET
REQUEST_PATH: /
REQUEST_URI: http://localhost:9923/
SCRIPT_NAME:
SERVER_NAME: localhost
SERVER_PORT: 9923
SERVER_PROTOCOL: HTTP/1.1
SERVER_SOFTWARE: WEBrick/1.3.1 (Ruby/1.8.7/2009-06-12)
rack.errors: #
rack.input: #
rack.multiprocess: false
rack.multithread: true
rack.run_once: false
rack.url_scheme: http
rack.version: 11 
</pre>
<br/>
下記は、シュミレータで試したときのスクリーンショット
<br/>
<a href="/img/uploads/2010/07/iphone_sim.png"><img src="/img/uploads/2010/07/iphone_sim-161x300.png" alt="iphone_sim" title="iphone_sim" width="161" height="300" class="aligncenter size-medium wp-image-405" /></a>

<br/>

<pre class="brush: plain">
GATEWAY_INTERFACE: CGI/1.1 
HTTP_ACCEPT: application/xml,application/xhtml+xml,text/html;q=0.9,text/plain;q=0.8,image/png,*/*;q=0.5 
HTTP_ACCEPT_ENCODING: gzip, deflate 
HTTP_ACCEPT_LANGUAGE: en-us 
HTTP_CACHE_CONTROL: max-age=0 
HTTP_CONNECTION: keep-alive 
HTTP_HOST: localhost:9923 
HTTP_USER_AGENT: Mozilla/5.0 (iPhone Simulator; U; CPU iPhone OS 4_0_1 like Mac OS X; en-us) AppleWebKit/532.9 (KHTML, like Gecko) Version/4.0.5 Mobile/8A306 Safari/6531.22.7 
HTTP_VERSION: HTTP/1.1 
PATH_INFO: / 
QUERY_STRING:  
REMOTE_ADDR: ::1 
REMOTE_HOST: localhost 
REQUEST_METHOD: GET 
REQUEST_PATH: / 
REQUEST_URI: http://localhost:9923/ 
SCRIPT_NAME:  
SERVER_NAME: localhost 
SERVER_PORT: 9923 
SERVER_PROTOCOL: HTTP/1.1 
SERVER_SOFTWARE: WEBrick/1.3.1 (Ruby/1.8.7/2009-06-12) 
rack.errors: #<IO:0x100183b88> 
rack.input: #<StringIO:0x1017b6d38> 
rack.multiprocess: false 
rack.multithread: true 
rack.run_once: false 
rack.url_scheme: http 
rack.version: 11 

</pre>
