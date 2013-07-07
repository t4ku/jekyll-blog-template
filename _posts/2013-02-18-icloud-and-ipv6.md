--- 
layout: post
title: "iCloudとIPv6"
tags: 
- Technology
- IPv6
- Apple
- OSX
- iCloud
published: false
---

iCloudの知られざる機能に、「どこでもMy Mac」という外出先から自分のMacにVNC(Macでは画面共有といってる)で入れるできる機能がある。
これを使うと何が便利かというと、下記みたいなのが簡単にできてしまう。。

* [iCloud を利用して自宅の Mac へ SSH する](http://www.talkabout.jp/2012/10/icloud-mac-ssh.html)
  * Githubに上げるのもめんどくさい書きかけのコードとか、あれどんな風に書いてたっけ？ぬおおっ!てならない
  * ディスク共有もOnにしとけば、容量10倍のDropboxみたいに使える
* [iCloud + SSH + dns-sd で自宅の iTunes へ接続](http://www.talkabout.jp/2012/10/icloud-ssh-dns-sd-itunes.html)
  * 職場のAirPlayに、一日中自分の好きな曲でBGMを流せる(といいつつ大体イヤホンで聞く人)

大体自宅サーバーって言うと、ISPが固定のグローバルIP割り当ててくれなければ、
ダイナミックDNS登録しなきゃいけないとか、NAT設定する必要があったりとか
面倒くさいことが多い。

iCloudの場合は基本的には設定パネルの[iCould] - [どこでもMy Mac]をオンにするだけという手軽さ。
(あとはディスク共有、画面共有など必要なものをOnにしとけばよい)

SSHでログインする場合も、dns-sd (dns service discovery)コマンドでホスト名を
確認しておいて、

{% prism bash %}
 $ dns-sd -E
Looking for recommended registration domains:
Timestamp     Recommended Registration domain
 1:39:00.260  Added     (More)               local
 1:39:00.260  Added                          icloud.com
                                             - > btmm
                                             - - > members
                                             - - - > 123456789
{% endprism %}

リモートからは、下記のsshコマンドでログインするだけ。

{% prism bash %}
ssh -2 -6 username@computer-name.[account number].members.btmm.icloud.com
{% endprism %}

 sshの-6オプション?
----------

sshで-6オプションってなんだっけ?と調べてみると、どうやらIPv6のアドレスを使う指定のようで、
これを指定しないとSSHで接続できなかった。

dns-sdコマンドで問い合わせを行なっても、IPv4指定だとレコードが見つからないと言われるが、
IPv6指定だとちゃんと帰ってくる。

{% prism bash %}
 $ dns-sd -G v4 mba.123456789.members.btmm.icloud.com
Timestamp     A/R Flags if Hostname                  Address                                      TTL
 1:50:58.398  Add     2  0 mba.1382993828.members.btmm.icloud.com. 0.0.0.0                                      77   No Such Record
{% endprism %}

{% prism bash %}
 $ dns-sd -G v6 mba.123456789.members.btmm.icloud.com
Timestamp     A/R Flags if Hostname                  Address                                      TTL
 1:51:18.216  Add     2  0 mba.1382993828.members.btmm.icloud.com. FD4A:481A:3F21:BC2D:7256:81FF:FEBB:EB7F%<0>  152
{% endprism %}

どうやら、「どこでもMy Mac」というやつはIPv6に依存しているらしい。


