--- 
layout: post
title: "ObjectiveResourceのConnectionタイムアウト設定"
tags: []

status: publish
type: post
published: true
meta: 
  _edit_lock: "1288524187"
  _edit_last: "1"
---
先日、開発中のiphoneアプリを実機で走らせてみました。最初のビューは、RailsからJSONで取得したデータを表示する画面だったのですが、何回かアプリを起動していると時々うまく表示がされず、JSONがうまく取得できてないみたいでした。

表示される場合とされない場合があるので、シュミレーターではMacの回線、実機では3Gという違いがある、Railsの方では、商品の情報をAWSから取得するため（二回目のリクエストからはキャッシュを利用）初回リクエストは時間がかかるということから、まあ接続がらみの問題なのかなあと思ったので、ObjectiveResourceのタイムアウト値を下記のページを参考に変更。とりあえず常にデータは表示されるようになった。

<a href="http://groups.google.com/group/objectiveresource/browse_thread/thread/12b3c406f20bcae2">Response timeout in development</a>

{% prism clike %}
- (void)applicationDidFinishLaunching:(UIApplication *)application {
	NSString *prodUrl = @"http://prod.example.com/";
	NSString *devUrl  = @"http://localhost:3000/";
	
	[ObjectiveResourceConfig setSite:prodUrl];
	
	[ObjectiveResourceConfig setUser:@"test_user"];
	[ObjectiveResourceConfig setPassword:@"test_user"];

        // timout値を10秒に設定
	[Connection setTimeout:10];
	
	[ObjectiveResourceConfig setResponseType:JSONResponse];
    
    // Add the tab bar controller's current view as a subview of the window
    [window addSubview:tabBarController.view];
}
{% endprism %}

