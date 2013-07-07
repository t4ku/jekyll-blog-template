--- 
layout: post
title: "SWFからObjective-C(Core Graphics/Quartz)へ変換するActionScriptライブラリ"
tags: 
- Programming
- ActionScript
- as3swf
- Objective-C
- SWF
status: publish
type: post
published: true
meta: 
  _edit_lock: "1267943931"
  _edit_last: "1"
---
as3swfというのはswfファイルのローレベルAPIを提供するライブラリらしいですが、swfファイル内のshapeを読み込み、Objective-Cのコードを出力することが可能だそうです。

<a href="http://wiki.github.com/claus/as3swf/shape-export-to-objective-c">Shape Export to Objective-C</a>

変換後のObjective-Cファイルはこんな感じらしい。
<script src="http://gist.github.com/186719.js?file=Shape1View.h"></script>
<script src="http://gist.github.com/186722.js?file=Shape1View.m"></script>

SWFの仕様書は下記にあります(読んでませんが)が、テキストだったんですね。。。
<a href="http://www.adobe.com/devnet/swf/">SWF File Format Specification (Version 10)</a>
