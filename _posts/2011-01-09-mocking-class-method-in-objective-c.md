--- 
layout: post
title: "Objective-CでクラスメソッドをMockする"
tags: 
- Programming
- GTM
- Objective-C
- OCMock
status: publish
type: post
published: true
meta: 
  _edit_lock: "1294632033"
  _edit_last: "1"
---
OCMockは素晴らしいライブラリだと思いますが、クラスメソッドにMock/Stub設定できなくて(※)困る時があります。

※OCMockの使用例(<a href="http://svn.mulle-kybernetik.com/OCMock/trunk/Source/OCMockObjectTests.m">http://svn.mulle-kybernetik.com/OCMock/trunk/Source/OCMockObjectTests.m</a>)にないのでおそらくOCMockでは無理、もしくはまだ機能として組み込んでないんだろうと思います。

ただ、Objective-Cはセレクタ(メソッドの名前のようなもの)とメソッドの実装のマッピングを、ランタイムAPIを使って実行時に変更できるので、これを使えばやりたい事はだいたい実現できます。(Method Swizzlingというらしいです)

<script src="https://gist.github.com/772337.js?file=mockOrStubClassMethodsObjc.m"></script>

メソッド交換すると、おそらく実行されるコンテキスト(ローカル/インスタンス変数、self etc)が変わると思うので、モックが実行されたかをチェックするフラグは、テストクラスのクラスメソッドにしてます。
