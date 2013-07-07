--- 
layout: post
title: "キーワードで検索した時のgoogleランキングを調べるrubyスクリプト"
tags: 
- Programming
- ruby tips programming google
status: publish
type: post
published: true
---
「SEO対策の有無をチェックするために、あるキーワードで検索して対象のWebサイトが何位くらいか調べる 」 という事をちょっと前仕事でやっていたのだが、数をこなさなければならないのと、対象のWebサイトが多かったのでバッチコマンドっぽく書いてみた 

## 使い方
{% prism clike %}
google_rank.rb  [-l | --limit 検索数 ] 検索文字列
{% endprism %}

例えば、"bit sized"という文字列で30位までのURLを取得しようとすると
{% prism clike %}
google_rank.rb  -l 30 bit sized
{% endprism %}
とすればよい、結果は下記のようになる
{% prism clike %}
google_rank.rb  -l 30 bit sized
{% endprism %}

## 出力例

{% prism clike %}
1,bitsized.wordpress.com/
2,bitsized.wordpress.com/.../
3,en.wikipedia.org/wiki/Drill_bit_sizes
4,www.madison.k12.wi.us/toki/teched/codrills.htm
5,www.imao.us/archives/000960.html
6,www.theoriginalbitfit.com/
7,bitesizestandards.com/
8,www.bitesizebonus.com/
9,www.japaninc.com/article.php?articleID=521
10,www.crypto.rub.de/imperia/md/content/
11,home-and-garden.become.com/
12,www.springerlink.com/index/74HV57D1UH95PHHL.pdf
13,www.hardforum.com/showthread.php?t=967973
14,www.hydracore.com/drill_bit_sizes.htm
15,dic.yahoo.co.jp/dsearch?enc=UTF-8&amp;p=bit&amp;
16,atlaspen.com/search/?item=503438&amp;pv=1
17,www.goodexperience.com/blog/archives/000633.php
18,www.wired.com/wired/archive/15.03/snackminifesto.html
19,search.luky.org/linux-kernel.2001/msg05005.html
20,www.modulaware.com/mdlt76.htm
21,www.patentstorm.us/patents/5501020-description.html
22,answers.yahoo.com/question/
23,www.hechinger.com/hardware/tools/
24,www.gfl-hand-tools.com/product-word/209/Bit-Sets.doc
25,bobmay.astronomy.net/misc/drillchart.htm
26,www.theoriginalbitfit.com/v4/go.gnf?s=bitfit&amp;
27,thatgrrlca.blogspot.com/2007/
28,www.dri.co.jp/auto/report/wf/wfmts340mb07.htm
29,www.ttora.com/forum/showthread.php?t=29091
30,docs.sun.com/app/docs/doc/
{% endprism %}

準備
HTMLのパースにhpricotというライブラリを使用しているため、あらかじめ準備しておく必要がある。”gem&nbsp; install&nbsp; hpricot”でインストールできる

## ソースコード

{% prism ruby %}
require 'rubygems'
require 'optparse'
require 'hpricot'
require 'open-uri'
require 'cgi'   
# class definition
class GoogleRank
    def initialize
      @str_query = ""
      @base_url = "http://www.google.co.jp/search?hl=ja&q="
      @suffix_url = "&lr="
      @srch_url = ""
      @srch_limit = 20
      @current_number = 0
    end

    def encode_query
      @str_query = CGI.escape(@str_query)
    end

    def cat_query
      if @current_number == 0
         @srch_url = @base_url + @str_query + @suffix_url
      else
         @srch_url = @base_url + @str_query + @suffix_url + "&start=" + @current_number.to_s + "&sa=N"
      end
    end

   def get_rank
      doc = Hpricot(open(@srch_url))
      # get the <div> that contains search result
      res = doc.search("//div[@id='res']")
      # <span class="a">hoge.com/target.html
      spans = res.search("//span[@class='a']")

      spans.each_with_index{ |span,i|
         page_info =  CGI.unescape(span.inner_html)
         #puts "page_info:" + page_info
         page_url = page_info.split(/ /)

         # trims html tags
         item_url = page_url[0].gsub(/<[\/]*[a-z]+[\/]*>/,"")
         puts (@current_number + i + 1).to_s + ","+ item_url
      }
    end
    def search_rank
      self.encode_query
      while  @current_number <  @srch_limit
        self.cat_query
        self.get_rank
        @current_number += 10
        #puts "current_number=" + @current_number.to_s
      end
    end
    attr_accessor :str_query
    attr_accessor :srch_limit
end

# bat command process

rank = GoogleRank.new
opt = OptionParser.new

opt.on('-l VAL','--limit VAL'){ |v| rank.srch_limit = v.to_i }

opt.parse!(ARGV)

ARGV.each {|i|

  if rank.str_query == ""
    rank.str_query = i.to_s
  else
    rank.str_query = rank.str_query + " " + i.to_s
  end
}

rank.search_rank
{% endprism %}
