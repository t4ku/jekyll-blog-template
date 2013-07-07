--- 
layout: post
title: "iPhoneでラジオボタン"
tags: 
- Programming
- iOS
- iPad
- iphone
- Objective-C
- tips
- UIKit
status: publish
type: post
published: true
meta: 
  _edit_lock: "1300957423"
  _edit_last: "1"
---
iPhone/iPadのネイティブアプリでフォームっぽいUIを使いたくなることが多いのですが、UIKitにはなぜかラジオボタンがなくていっつも困るので簡単なラジオボタンを実装してみました。  

<a href="/img/uploads/2011/03/RadioButton.png">
  <img src="/img/uploads/2011/03/RadioButton-159x300.png" alt="RadioButton" title="RadioButton" width="159" height="300" class="aligncenter size-medium wp-image-530" /></a>  

サンプルのプロジェクトをgithubにあげときました。<br>
<ul>
	<li>Githubページ : <a href="https://github.com/t4ku/RadioButtonWithUIKit">t4ku / RadioButtonWithUIKit</a></li>
	<li>Zipダウンロード : <a href="https://github.com/t4ku/RadioButtonWithUIKit/zipball/master">Download</a></li>
</ul>

<a href="">RadioButtonViewController.m</a>
グループIDとグループ内での番号を指定してラジオボックスのUIを初期化します。
{% prism clike %}
RadioButton *rb1 = [[RadioButton alloc] initWithGroupId:@"first group" index:0];
RadioButton *rb2 = [[RadioButton alloc] initWithGroupId:@"first group" index:1];
RadioButton *rb3 = [[RadioButton alloc] initWithGroupId:@"first group" index:2];

rb1.frame = CGRectMake(10,30,22,22);
rb2.frame = CGRectMake(10,60,22,22);
rb3.frame = CGRectMake(10,90,22,22);

[self.view addSubview:rb1];
[self.view addSubview:rb2];
[self.view addSubview:rb3];

// 選択値が変わったときに、delegateメソッドが呼ばれるようにする 
[RadioButton addObserverForGroupId:@"first group" observer:self];
{% endprism %}

<a href="https://github.com/t4ku/RadioButtonWithUIKit/blob/master/RadioButton/RadioButton.h">RadioButton.h</a>
ラジオボタンの選択状態が変わったときに呼ばれるdelegateメソッドの宣言は下記。
{% prism clike %}
@protocol RadioButtonDelegate <NSObject>
-(void)radioButtonSelectedAtIndex:(NSUInteger)index inGroup:(NSString*)groupId;
@end
{% endprism %}

<a href="">RadioButtonViewController.m</a>
実装例。単にグループIDとグループ内の何番目のラジオボタンが押されたかを表示

{% prism clike %}
@protocol RadioButtonDelegate <NSObject>
-(void)radioButtonSelectedAtIndex:(NSUInteger)index inGroup:(NSString *)groupId{
    NSLog(@"changed to %d in %@",index,groupId);
}
{% endprism %}


実装してみて思ったのは、今回はRadioButtonというUIViewのサブクラスに全部いれこんだんですが、ホントはラジオボタン間のやりとりを別クラス(RadioButtonGroupとか)にしといたほうがよいかもとか、ラベルとかも一緒に(textViewとかプロパティつくって)管理したほうがよいかもとか。。。まあ、Quick&Dirtyで行きましょう。
