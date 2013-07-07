--- 
layout: post
title: "EventMachineを触ってみた"
tags: 
- Programming
- EventMachine
- ruby
status: publish
type: post
published: true
meta: 
  _edit_lock: "1273983265"
  _edit_last: "1"
---
機会があったので、以前から少し気になっていたEventMachineを触ってみました。EventMachineがどんなものかというのは下記のスライドに簡単にまとまっています。(いつもお世話になってるWebサービスで結構使用されてます)

<a title="View EventMachine: scalable non-blocking i/o in ruby on Scribd" href="http://www.scribd.com/doc/28253878/EventMachine-scalable-non-blocking-i-o-in-ruby" style="margin: 12px auto 6px auto; font-family: Helvetica,Arial,Sans-serif; font-style: normal; font-variant: normal; font-weight: normal; font-size: 14px; line-height: normal; font-size-adjust: none; font-stretch: normal; -x-system-font: none; display: block; text-decoration: underline;">EventMachine: scalable non-blocking i/o in ruby</a> <object id="doc_846290292482129" name="doc_846290292482129" height="500" width="100%" type="application/x-shockwave-flash" data="http://d1.scribdassets.com/ScribdViewer.swf" style="outline:none;" >		<param name="movie" value="http://d1.scribdassets.com/ScribdViewer.swf">		<param name="wmode" value="opaque"> 		<param name="bgcolor" value="#ffffff"> 		<param name="allowFullScreen" value="true"> 		<param name="allowScriptAccess" value="always"> 		<param name="FlashVars" value="document_id=28253878&access_key=key-1rb2iijpl7bew7i1f04i&page=1&viewMode=slideshow"> 		<embed id="doc_846290292482129" name="doc_846290292482129" src="http://d1.scribdassets.com/ScribdViewer.swf?document_id=28253878&access_key=key-1rb2iijpl7bew7i1f04i&page=1&viewMode=slideshow" type="application/x-shockwave-flash" allowscriptaccess="always" allowfullscreen="true" height="500" width="100%" wmode="opaque" bgcolor="#ffffff"></embed> 	</object>	

<a href="http://wiki.github.com/eventmachine/eventmachine/tutorials">Githubにあるeventmachineのtutorial</a>にあるリンクをテキトーに見てみました。

まずは、一番簡単な例
<a href="http://20bits.com/articles/an-eventmachine-tutorial/">An EventMachine Tutorial"</a>
{% prism ruby %}
require "rubygems"
require "eventmachine"

# From "An EventMachine Tutorial"
# http://20bits.com/articles/an-eventmachine-tutorial/

# ハンドラー
module EchoServer
  # connectionからデータを受け取った時に呼び出されるメソッド
  def receive_data(data)
    # receive_dataを呼び出したconnectionにデータを返す
    # receive_dataしたらsend_dataする
    send_data(data)
  end
end

# イベントループ開始
# EventMachine::stop_event_loopを呼び出すまで止まらない
EventMachine::run do
  host = '0.0.0.0'
  port = 8080
  
  # 3番目の引数はHandlerで、コールバック用の関数でグローバル変数を汚染しないために
  # moduleを定義することが多い。
  EventMachine::start_server host,port,EchoServer
  puts "started EchoServer on #{host}:#{port}"
end
{% endprism %}
このスクリプトを実行させておいて、"telnet  0.0.0.0  8080"で繋げれば、入力したものがそのまま返されます。

おなじく、<a href="http://20bits.com/articles/an-eventmachine-tutorial/">An EventMachine Tutorial"</a>
にあるサーバーの例
{% prism ruby %}
require "rubygems"
require "eventmachine"

# From "An EventMachine Tutorial"
# http://20bits.com/articles/an-eventmachine-tutorial/

class Server < EventMachine::Connection
  attr_accessor :status,:options
  
  def receive_data
    puts "#{@status} -- #{data}"
    send_data("hello\n")
  end
end

EventMachine::run do
  # start_serverは3番目の引数に指定されたハンドラーを引数にyeildする
  EM::start_server host,port,Server do |conn|
    conn.options = {:my => 'options'}
    conn.status = :OK
  end
end
{% endprism %}

この例では、Serverクラスをハンドラーに指定しています。EventMachine::start_serverは任意のモジュールかクラスをハンドラーに指定することができます。モジュールを指定した場合もEventMachine::Connectionクラスにそのモジュールがインクルードされた後、接続したクライアントごとにインスタンス化されます。

そこらへんはEventMachineのソースを追いつつ、絵を描いてたのですが、自分でも微妙な絵になってしまった。。。
とはいえ、一応さらしておきます。
<a href="http://blog.showqase.com/wp-content/uploads/2010/05/Untitled.png"><img src="http://blog.showqase.com/wp-content/uploads/2010/05/Untitled.png" alt="EventMachine" title="EventMachine" width="600" height="500" class="alignleft size-full wp-image-390" /></a>



今度もおなじく、<a href="http://20bits.com/articles/an-eventmachine-tutorial/">An EventMachine Tutorial"</a>ですが、httpクライアントの例

{% prism ruby %}
require "rubygems"
require "eventmachine"

# From "An EventMachine Tutorial"
# http://20bits.com/articles/an-eventmachine-tutorial/

module HttpHeaders
  # connectionがセットアップされた直後に呼び出される。
  # クライアントアプリならサーバーにつながった直後/サーバーアプリならクライアントが接続してきた直後
  def post_init
    send_data "GET /\r\n\r\n"
    @data = ""
  end
  
  def receive_data(data)
    @data << data
  end
  
  # クライアント/サーバー側のいずれかの接続が終了した時
  # このhttp_clientの場合は、サーバーがデータを送り終えたら呼び出される
  def unbind
    puts "server have sent data"
    if @data =~ /[\n][\r]*[\n]/m
      $`.each do |line|
        puts ">>> #{line}"
      end
    end
    
    # ループを終了させる
    EventMachine::stop_event_loop
  end
end

EventMachine::run do
  EventMachine::connect "google.com",80,HttpHeaders
end
{% endprism %}
ポイントとしては、

- 接続の開始時にpost_init/終了時にunbindが実行される
- TCPはストリームなので、receive_dataは何度も実行され、渡されるデータの単位は(改行区切りとか)意味のあるものではない

というところでしょうか。

今度は、<a href="http://everburning.com/news/playing-with-eventmachine/">Playing with EventMachine</a>にあるタイマーの設定例

{% prism ruby %}
require "rubygems"
require "eventmachine"

# Playing with EventMachine
# http://everburning.com/news/playing-with-eventmachine/

# EventMachineとEMは同じ
# runでイベントループが開始されるが、引数として渡されるブロックをその前に実行する
EventMachine::run do
  # 定期的に実行されるタイマーを設定
  EM.add_periodic_timer(1) { puts "Tick ..." }
  
  # 指定した秒数が経過した後に、1回実行されるタイマーを設定
  EM.add_timer(3) do
    puts "I waited 3 seconds"
    EM.stop_event_loop
  end
end

puts "All done"
{% endprism %}

一応試してみたコードはGithubにアップしてます。<a href="http://github.com/t4ku/em_snippet">em_snipett</a>
もうちょっとわかってきたら、アップデートします。

