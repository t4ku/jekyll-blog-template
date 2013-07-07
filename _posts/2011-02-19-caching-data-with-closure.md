--- 
layout: post
title: "クロージャーを使ったデータのキャッシュ"
tags: 
- Programming
- javascript
- jQuery
status: publish
type: post
published: true
meta: {}

---
<p>「jQueryで新し要素を作って、ページに追加する。」よくあるパターンですが、そのたびにHTML文字列組み立ててたら、なんかパフォーマンスを犠牲にしているような気がしていたので、最近実装したAjaxのアプリでは下記のようにクロージャーでデータをキャッシュするようにした。</p>
<p>ここではhtmlTemplateに割り当てられているfunctionオブジェクトは、(function(){....}())内部でreturnされているfunction。function(){}()というのは無名functionを生成するのと同時に実行するという意味で、大外の()はシンタックス的には不要なんですが、var hoge = function(){};のように単純に変数に関数を代入するのとは「なんか違うぞ」という事を示すコンベンションなんだとか。</p>
<p>return される関数はfunctionの外側の変数(htmlとか)を参照でき、呼出しの度にhtml(の文字列の配列)を初期化するわけではなく、初回に実行されたhtmlを使いまわせる。</p><iframe style="width: 80%; height: 600px" src="http://jsfiddle.net/t4ku/HpqCg/11/embedded/"></iframe>
