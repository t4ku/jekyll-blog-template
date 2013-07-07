--- 
layout: post
title: "gitignore集"
tags: 
- Programming
- Android
- git
status: publish
type: post
published: true
meta: 
  _edit_last: "1"
  _edit_lock: "1295676281"
---
Android+Eclipse用のgit設定を探していたら、便利なサイトを見つけたのでメモ。

<a href="http://gitignore.com/">.gitignore</a>
様々なタイプのプロジェクトに必要な.gitignoreを集めているサイト
<a href="https://github.com/github/gitignore">github/gitignore - GitHub  A collection of useful .gitignore templates</a>
gitignoreを集めたリポジトリ

とりあえず<a href="https://github.com/github/gitignore/blob/master/Android.gitignore">これ</a>を使うことにしました。
{% prism clike %}
# built application files
*.apk
*.ap_

# files for the dex VM
*.dex

# Java class files
*.class

# generated GUI files
*R.java
{% endprism %}
