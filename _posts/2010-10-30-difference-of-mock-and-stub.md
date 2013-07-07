--- 
layout: post
title: "MockとStubの違い"
tags: 
- Programming
- BDD
- mock
- rspec
- stub
- TDD
status: publish
type: post
published: true
meta: 
  _edit_lock: "1288522828"
  _edit_last: "1"
---
iPhone用のUnit Testライブラリを調べていて、MockとStubの違いがあやふやになってきたので、Railsに立ち返ろうと調べてみると分かりやすいエントリがあった。

RSpec の Mock と Stub が最初分からなかったけど、理解できたら すごい！ という気持ちになった
<a href="http://d.hatena.ne.jp/takihiro/20081023/1224762895">http://d.hatena.ne.jp/takihiro/20081023/1224762895</a>

自分なりのまとめ


<strong>モック</strong>
<blockquote>
オブジェクトの呼び出し方(使用方法)を検証する。テスト対象のコードが、ある外部クラスを正しく呼び出しているかどうかかを検証するために使用する。テスト対象のコードが使用している外部クラスにexpectation(RSpecでは.should_receive)を設定する。
</blockquote>


<strong>スタブ</strong>
<blockquote>
テスト対象のコードの中で外部クラスを参照する場合に、外部クラスが単純なデータを返すことを設定する。
</blockquote>




