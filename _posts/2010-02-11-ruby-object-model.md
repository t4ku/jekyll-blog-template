--- 
layout: post
title: "Rubyにおける特異クラス"
tags: 
- Programming
- ruby
status: publish
type: post
published: true
meta: 
  _edit_lock: "1267357181"
  _edit_last: "1"
  enclosure: |
    http://scotland-on-rails.s3.amazonaws.com/2A04_DaveThomas-SOR.mp4
    285514197
    video/mp4

---
Dave ThomasがRubyのオブジェクトモデルについて熱く語っている動画(<a href="http://scotland-on-rails.s3.amazonaws.com/2A04_DaveThomas-SOR.mp4">The Ruby Object Model</a>)を最近みたのですが、今までよくわかってなかった、特異メソッドとかclass &lt;&lt; selfの意味がわかりました。
<ul>
	<li>オブジェクトはインスタンス変数(状態)を保持している</li>
	<li>クラスはメソッド(振る舞い)を保持している</li>
</ul>
という前提とメソッドルックアップの順番を把握するとすっきり理解できた気がします。

オブジェクトに対してメソッド呼び出ししたときにのメソッドルックアップはオブジェクトが属するクラスを参照する。（親クラスやモジュールにもなければ、method_missingをクラスの継承関係の子クラスから探し、最終的にObject#method_missingが実行される）
<pre class="brush: ruby;">class Person
  def instance_method_a
      puts "instance_method_a called"
  end
end

bob = Person.new
bob.instance_method_a</pre>
特異メソッドがを追加すると、メソッドルックアップの順番の一番最初に、匿名のクラスが追加されその後にクラスを参照するように変更される。この匿名のクラスが、特異クラスということ。
<pre class="brush: ruby;">def bob.another_method
  puts "another_method called"
end

bob.another_method</pre>
<ul>
	<li>オブジェクト=&gt; オブジェクトが属するクラス(通常のメソッドルックアップ)</li>
	<li>オブジェクト =&gt; 特異クラス　=&gt; オブジェクトが属するクラス(特異メソッドが追加された場合のメソッドルックアップ)</li>
</ul>
class &lt;&lt; [オブジェクト] 〜 end というシンタックスは、[オブジェクト]の特異クラスを参照するというシンタックス。なので、上記の特異メソッドは下の様にもかける。
<pre class="brush: ruby;">class &lt;&lt; bob
  def another_method
     puts "another_method called"
  end
end
</pre>
Rubyでクラスメソッドを定義するときは、クラス自体を一つのオブジェクトとして見て、クラスオブジェクトの特異メソッドを定義しているという事だったんですね。
<pre class="brush: ruby;">
class Person
  # ここでは self =&gt; Personなので、Personオブジェクトの特異クラスにメソッドを定義できる
  class &lt;&lt; self     
    def class_method_a
       puts "class_method_a called" 
    end
   end

   def self.class_method_b
     puts "class_method_b called"
   end
end

Person.class_method_a  # =&gt;"class_method_a called"
Person.class_method_b  # =&gt;"class_method_b called"
</pre>
ちなみに、特異クラスはmetaclass/virtual class/proxy class/eigen class/singleton などいろいろな呼ばれ方をしているみたいです。もっといい(わかりやすい)日本語があればいいのに。。。
