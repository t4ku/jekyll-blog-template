--- 
layout: post
title: "TextmateでちょっとしたObjective-Cのコードを実行する"
tags: 
- Programming
- Objective-C
- tips
- Xcode
status: publish
type: post
published: true
meta: 
  _edit_last: "1"
  _edit_lock: "1294646903"
---
TextMateでコンパイル&実行までできることに気づいたのでメモ。

test.mというファイル名でファイルを作成した後、"Select Bundle Item"ダイアログをひらいてみる(ショートカットは^⌘T)。Cのメイン関数を挿入するスニペットがあるので、
それを使うとだいたい出来上がり。

<a href="/img/uploads/2011/01/textnte-objc-cmd.png"><img src="/img/uploads/2011/01/textnte-objc-cmd.png" alt="textnte-objc-cmd" title="textnte-objc-cmd" width="350" height="258" class="alignnone size-full wp-image-505" /></a>

<a href="/img/uploads/2011/01/textmate-objc.png"><img src="/img/uploads/2011/01/textmate-objc.png" alt="textmate-objc" title="textmate-objc" width="586" height="456" class="alignnone size-full wp-image-502" /></a>

⌘Rでコンパイル&実行できる。

<a href="/img/uploads/2011/01/textmate-objc-run.png"><img src="/img/uploads/2011/01/textmate-objc-run.png" alt="textmate-objc-run" title="textmate-objc-run" width="500" height="600" class="alignnone size-full wp-image-504" /></a>

ちなみに、上記の手順の場合、TextMateのバンドルはCのものを使っているみたいで、コンパイルは下記のようなオプションで実行されているみたいです。


<blockquote>
gcc test.m -Wall -include stdio.h -framework Cocoa
</blockquote>


