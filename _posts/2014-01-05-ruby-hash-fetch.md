--- 
layout: post
title: "Hash#fetch()でデフォルト値を設定する"
tags: 
- ruby,tips,idiom
published: true
---

Rubyではオプショナルな引数をハッシュとして集めて、「||」演算子を使用して
デフォルト値を設定する場面は結構多いと思います。

{% prism ruby %}
class ImageResizer

  def initialize(path,options={})
    @path             = path
    @width            = options[:width]  || 600
    @height           = options[:height] || 400
    @remove_original  = options[:remove_original] || true
  end
  
  def resize
    ....
  end
  
end
{% endprism %}

ただ、@remove_originalのように引数の値がfalsy(=偽、rubyの場合はnilとかfalseですね)の場合、若干扱いが面倒になります。

例えば、このImageResizerというクラスを下記のような形で初期化すると、@remove_originalが意図せず、
trueになってしまいます。

{% prism ruby %}
resizer = ImageResizer.new File.join('tmp','abc.png'), :remove_original => false
{% endprism %}

これは「||」演算子によるチェックでは、直前の式がfalseyであるかのみで判断して(falseyであれば)デフォルト値を採用するため、
「キーが存在しない」場合に加えて、「falseyな値を持つキーが存在する」にもデフォルト値で上書かれるためです。

{% prism ruby %}
# keyがない場合
{}[:key]               || true  
=> true

# 値がfalseの場合
{:key => false }[:key] || true  
=> true
{% endprism %}

そのため、下記のようにキー自体があるかどうかまずチェックする必要があります。

{% prism ruby %}
@remove_original  = options.has_key?(:remove_original) ? options[:remove_original] : true
{% endprism %}

# Hash#fetch to rescue

これでやりたいことはできますが、若干ややこしさというか、これでいいのか感が否めません。

このように「Hash内のキーが存在すればその値を、なければ指定したデフォルト値を」というロジックであれば、
Hash#fetch(key [,default])のほうが、シンプルに表現できます。

{% prism ruby %}
@remove_original  = options.fetch(:remove_original,true)
{% endprism %}

Hash#fetch(key ,default)は、keyが存在しなかった場合にdefaultを返します。
上記のコードを書き直すと下記のようになります。

{% prism ruby %}
@width            = options.fetch :width, 600 
@height           = options.fetch :height, 400 
@remove_original  = options.fetch :remove_original, true 
{% endprism %}

引数がどのような値であっても「キー名 → 指定されなかった場合のデフォルト値」という流れでそのまま書けるため、スッキリ読めると思います。

また、default値の部分についてブロックも使用できるので、動的に判断する時にも使えます。

{% prism ruby %}
@width            = options.fetch :width,  { calculate_width  }
@height           = options.fetch :height, { calculate_height }
@remove_original  = options.fetch :remove_original, true
{% endprism %}

細かいことを言うと、コストの大きい処理などはブロックで指定してデフォルト値が必要になった時のみ評価するようにするのが
良いかもしれません。
