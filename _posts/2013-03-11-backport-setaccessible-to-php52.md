--- 
layout: post
title: "古いPHPでプライベートメソッドをテストする"
tags: 
- PHP
published: true
---

レガシーなコードと対決する場合、うまくメソッド分割されてなかったり
してどうしてもprivateなメソッドのテストコードを書かざるを得ない場合がある。

理想的な世界では、公開したいインターフェースに対して、確認すべき実行
パターン分のテストケースがあって、外部リソースに依存する場合にスタブ
アウトするとか気をつければいいだけで、その中で使用されるプライベート
メソッドは特に注視する必要がないハズである。

ただ、人はPHPという言語でプログラムを書く場合に、往々にしてそういうことを忘れて、
巨大(クソ)メソッドやハードコードされた依存性を埋め込まずにはいられない生物らしい。
そして、一旦実装されるとそれらをパブリックなメソッドのみでテストすることは非常に難しい。

### Enter ReflectionMethod::setAccessible()

PHP 5.3.2では、こうしたprivateメソッドのテスト目的以外にはほとんど利用価値
がないであろうリフレククションAPIが追加されており、privateなメソッドも呼び出し可能になる。

ReflectionMehtod::setAccessible()

{% prism php %}
<?php
class A{
    private function hello(){
        return "hello";
    }
}
$method = new ReflectionMethod('A','hello');
$method->setAccessible(true);
$method->invoke(new A());
?>
{% endprism %}

ただし、このAPIはPHP5.3.2で追加されているためそれ以前のバージョンで書かれた
コードでは使用できない。この場合はほとんどお手上げで、残る手段はevalするくらいしかないようだ。

参考 - [PHP5.4時代のprivateメソッドテスト手法 #php5_4](http://blog.tojiru.net/article/239067872.html)

### PHP 5.3.2以前でInvoke()できるようにする

ただ、若干ダーティハックだけどPHP 5.3.2以前でもprivateなメソッドをinvoke可能
にする方法はなくはない。PHP 5.3.2で加えられたsetAccessibleをバックポートすればいい。

Reflection APIのソース(Cで書かれてる)をみてみると、メソッドを呼び出すinvoke()/invokeArgs()
が更に低位のAPI(Zend Engine)を呼び出しているときは、特にメソッドのVisibilityを指定してはいない。
つまり、メソッドのVisibilityはZend Engineの機能でなくPHP側の機能であるようだ。

{% prism clike %}
result = zend_call_function(&fci, &fcc TSRMLS_CC);
{% endprism %}

第一、第二引数のプロトタイプはそれぞれ、zend_fcall_info/zend_fcall_info_cache
という構造体で要はphpのfunctionオブジェクトみたいなものっぽい。

TSRMLS_CCというのはPHPをスレッドセーフ有効(zts使用)でコンパイルした時に
有効になるマクロらしいがあんま関係なさそう。

実際PHPにsetAccessible()が追加された時のコミットをみてみると、このメソッド自体はinvokeする際に
methodのvisibilityを無視するかどうかの内部変数(フラグ)をセットしているだけにすぎない。

[Merge ReflectionMethod::setAccessible() to PHP 5.3.2, approved by Johannes.](https://github.com/php/php-src/commit/ceaf5905308de3244cdb213c91929c77355eea9d)

※ちなみに、最近は大体のソフトウェアがGithubにソースをミラーしてるので
確認が楽になった。PHPのソースもGithubにある。

ext/reflection/php_reflection.c
{% prism clike %}
ZEND_METHOD(reflection_method, setAccessible)
{
   reflection_object *intern;
   zend_bool visible;

   if (zend_parse_parameters(ZEND_NUM_ARGS() TSRMLS_CC, "b", &visible) == FAILURE) {
       return;
   }

   intern = (reflection_object *) zend_object_store_get_object(getThis() TSRMLS_CC);

   if (intern == NULL) {
       return;
   }

   intern->ignore_visibility = visible;
}
{% endprism %}

実際に$method->invoke()された時点では、publicでないmethodやabstractなmethodに対して
このフラグを立てずに呼び出されていると、エラー(例外)を発生させる。

ext/reflection/php_reflection.c
{% prism clike %}
    if ((!(mptr->common.fn_flags & ZEND_ACC_PUBLIC)
         || (mptr->common.fn_flags & ZEND_ACC_ABSTRACT))
         && intern->ignore_visibility == 0)
    {
        if (mptr->common.fn_flags & ZEND_ACC_ABSTRACT) {
            zend_throw_exception_ex(reflection_exception_ptr, 0 TSRMLS_CC, 
                "Trying to invoke abstract method %s::%s()", 
                mptr->common.scope->name, mptr->common.function_name);
        } else {
            zend_throw_exception_ex(reflection_exception_ptr, 0 TSRMLS_CC,
                "Trying to invoke %s method %s::%s() from scope %s", 
                mptr->common.fn_flags & ZEND_ACC_PROTECTED ? "protected" : "private",
                mptr->common.scope->name, mptr->common.function_name,
                Z_OBJCE_P(getThis())->name);
        }
        return;
    }
{% endprism %}

### 別にバックポートしなくても。。。

とここまで見るとわかるがReflection::invoke/Reflection::invokeArgsで、上記のprivate呼び出しチェック
を無効化すれば、Reflection::setAccessibleをバックポートしなくてもprivateメソッド呼び出せるようになるので
以下パッチを当てて、再コンパイルすればprivate/protectedなメソッドも外側から呼び出し可能になる。
(以下はphp-5.2.8に対してのパッチ)

php-5.2.8.patch

{% prism clike %}
diff --git a/ext/reflection/php_reflection.c b/ext/reflection/php_reflection.c
index 564e9d8..c751a65 100644
--- a/ext/reflection/php_reflection.c
+++ b/ext/reflection/php_reflection.c
@@ -2313,18 +2313,11 @@ ZEND_METHOD(reflection_method, invoke)

        GET_REFLECTION_OBJECT_PTR(mptr);

-       if (!(mptr->common.fn_flags & ZEND_ACC_PUBLIC) ||
-               (mptr->common.fn_flags & ZEND_ACC_ABSTRACT)) {
+       if ((mptr->common.fn_flags & ZEND_ACC_ABSTRACT)) {
                if (mptr->common.fn_flags & ZEND_ACC_ABSTRACT) {
                        zend_throw_exception_ex(reflection_exception_ptr, 0 TSRMLS_CC,
                                "Trying to invoke abstract method %s::%s()",
                                mptr->common.scope->name, mptr->common.function_name);
-               } else {
-                       zend_throw_exception_ex(reflection_exception_ptr, 0 TSRMLS_CC,
-                               "Trying to invoke %s method %s::%s() from scope %s",
-                               mptr->common.fn_flags & ZEND_ACC_PROTECTED ? "protected" : "private",
-                               mptr->common.scope->name, mptr->common.function_name,
-                               Z_OBJCE_P(getThis())->name);
                }
                return;
        }
@@ -2416,18 +2409,11 @@ ZEND_METHOD(reflection_method, invokeArgs)
                return;
        }

-       if (!(mptr->common.fn_flags & ZEND_ACC_PUBLIC) ||
-               (mptr->common.fn_flags & ZEND_ACC_ABSTRACT)) {
+       if ((mptr->common.fn_flags & ZEND_ACC_ABSTRACT)) {
                if (mptr->common.fn_flags & ZEND_ACC_ABSTRACT) {
                        zend_throw_exception_ex(reflection_exception_ptr, 0 TSRMLS_CC,
                                "Trying to invoke abstract method %s::%s",
                                mptr->common.scope->name, mptr->common.function_name);
-               } else {
-                       zend_throw_exception_ex(reflection_exception_ptr, 0 TSRMLS_CC,
-                               "Trying to invoke %s method %s::%s from scope %s",
-                               mptr->common.fn_flags & ZEND_ACC_PROTECTED ? "protected" : "private",
-                               mptr->common.scope->name, mptr->common.function_name,
-                               Z_OBJCE_P(getThis())->name);
                }
                return;
        }    
{% endprism %}
