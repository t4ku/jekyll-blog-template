--- 
layout: post
title: "ObjectiveResourceでネストしたリソースを取得する"
tags: 
- Programming
- iphone
- Objective-C
- ObjectiveResource
status: publish
type: post
published: true
---
CodeZineにObjectiveResourceの解説記事がありました。ObjectiveResourceは日本語情報はまだまだ少ないですが、一通り使用方法が書かれているので使い方を知りたい人にはわかりやすいのではないでしょうか？
[iPhoneとRuby on Railsを超簡単に連携する ObjectiveResource- iPhone編](http://codezine.jp/article/detail/5126)

ただRailsを使用していると、単純に一つのモデルを操作するだけのコントローラーだけではないし、パラメータを含んだだリクエストもあるし、標準的なモデルに対するCRUD以外のURLを利用する場面は多いと思います。

ObjectiveResourceの[Getting Started](http://iphoneonrails.com/getting-started)に解説がありますが、当然ながらObjectiveResourceでもCRUD以外のURLも使用できます。

例えば、BookモデルとReviewモデルに対して一対多の関連を設定し、本に対して投稿されたレビューを下記のようなURLで取得できるとします。
http://example.com/books/3/reviews.json

この場合iphone側のReviewモデルで、Bookのidをキーにその本に対して投稿されたReviewの配列を返すメソッド(findAllForBookWithId)を実装するというやり方になります。

Review.h
{% prism clike %}
#import <Foundation/Foundation.h>

@interface Review : NSObject {
    NSString *title;
    NSString *body;
    NSString *postedBy;
    NSString *userId;
}
@property(nonatomic,retain) NSString *title;
@property(nonatomic,retain) NSString *body;
@property(nonatomic,retain) NSString *postedBy;
@property(nonatomic,retain) NSString *userId;

// bookIdを引数に本に対して投稿されたレビューの配列を取得するメソッド
+(NSArray *)findAllForBookWithId:(NSString*)bookId;

@end
{% endprism %}

メソッドの実装ですが、カスタムのURL文字列を生成して
Connectionクラスの +get:withUser:andPassword: メソッドを使用してリクエストを飛ばす形になります。

Review.m
{% prism clike %}
#import "Review.h"
#import "Book.h"
#import "Response.h"
#import "Connection.h"
#import "ObjectiveResourceConfig.h"

@implementation Review
@synthesize title,postedBy,userId,body;

+(NSArray *) findAllForBookWithId:(NSString *)bookId{
	// URLを設定
	// books/:id/reviews.json
	NSString *bookReviewPath = [NSString stringWithFormat:@"%@%@/%@/%@%@",
										[self getRemoteSite],
										[Book getRemoteCollectionName],
										bookId,
										[self getRemoteCollectionName],
										[self getRemoteProtocolExtension]];
        // URLを引数にgetリクエストの結果を取得										
	Response *res = [Connection get:bookReviewPath
						   withUser:[ObjectiveResourceConfig getUser]
						andPassword:[ObjectiveResourceConfig getPassword]];
						
	return [self fromJSONData:res.body];
}

@end
{% endprism %}

後は、必要な箇所でReviewインスタンスを取得するコードを使用すればよいです。

{% prism clike %}
- (void)viewDidLoad {
    [super viewDidLoad];

    self.navigationItem.title = self.book.title;
    NSArray *reviews = [Review findAllForBookWithId:book.bookId];

}
{% endprism %}

パラメーターつきのURL(http://example.com/book.json?isbn=123456789とか)なども、URLの設定を変えれば使用できるようになります。
