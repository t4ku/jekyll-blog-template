--- 
layout: post
title: "redirect_toによる条件分岐"
tags: 
- Programming
- ActionController
- Rails
- ruby
status: publish
type: post
published: true
meta: 
  _edit_lock: "1268315437"
  _edit_last: "1"
---
RailsでURLに渡されたパラメーターによって、他のコントローラーにリダイレクトするような処理を書いているとこんなエラーに出くわしました。

<blockquote>Can only render or redirect once per action redirect  render</blockquote>

{% prism ruby %}
  def new
    if(params[:url])
      @existing_page = Page.find_by_url(params[:url])
      unless @existing_page.nil? 
        redirect_to :controller => "pages",
                  :action => "edit",
                  :params => {:id => @existing_page.id } 
      end
    end
    
    @page = Page.new
    @sentence = @page.sentences.build
    @sentence.translations.build
    
    respond_to do |format|
      format.html 
    end
  end
{% endprism %}


ブラウザに返すHTTPレスポンスは1回だけなので、至極当然っぽい仕様ですが、メソッド全体を条件分岐させて(if ~ else ~ end)、renderもしくはredirect_toがどのパターンでも一回しか実行されないようにするというのは、あんまり気に入らなかったので、redirect_to のあとにreturn false を入れてしのいでました。

まあ、期待どおりの動きをしてくれるのでよかったんですが、ダーティーハックな感じがしたので、ActionControllerのAPIを見てみると、redirect_to ~ and returnというのを使えと書いてました。

[Class::ActionController::Base](http://api.rubyonrails.org/classes/ActionController/Base.html)
{% prism ruby %}
  def do_something
    redirect_to(:action => "elsewhere") and return if monkeys.nil?
    render :action => "overthere" # won't be called if monkeys is nil
  end
{% endprism %}

書き直し後のコード。

{% prism ruby %}
  def new
    if(params[:url])
      @existing_page = Page.find_by_url(params[:url])
      unless @existing_page.nil? 
        # redirect_to ~ and return
        redirect_to :controller => "pages",
                  :action => "edit",
                  :params => {:id => @existing_page.id }  and return
      end
    end
    
    @page = Page.new
    @sentence = @page.sentences.build
    @sentence.translations.build
    
    respond_to do |format|
      format.html 
    end
  end
{% endprism %}
