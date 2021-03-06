---
categories: 'sdk'
date: 2015-09-27T23:47:00+09:00
description: 'Growthbeat iOS の導入方法について説明します'
draft: false
title: Growthbeat iOS Gudeliene
---

Version 2.0.4

# 共通初期設定

推奨環境

iOS 8.0以上

## SDK導入

Growthbeat SDKで、Growthbeat 全てのサービスの機能が利用できます。

Objective-C での導入方法について記載しております。

### CocoaPodsを使用して導入する場合

Podfile に下記を記述し `pod install` を実行してください:

```
pod 'Growthbeat'
```

### 手動で SDK を配置して導入する場合

<a href="/sdk">最新版iOS SDK ダウンロードページ</a>

ダウンロードしたファイルを解凍し、そのフォルダの中の **Growthbeat.framework** をプロジェクトへ組み込みます。
任意のXcodeプロジェクトを開き Growthbeat.framework をインポートしてください。

Growthbeat.framework のインポートの方法は以下の２つです:

1. Xcodeプロジェクトに Growthbeat.framework をドラッグアンドドロップする
2. Bulid Phases -> Link Binary With Libraries の + ボタンを押し、Add Other...からGrowthbeat.frameworkを選択する

### import

Growthbeat の import 文を記述します。

```objc
#import <Growthbeat/Growthbeat.h>
```

### 依存について

Growthbeat.framework は、下記 Framework が必須となります:

- Foundation.framework
- UIKit.framework
- CoreGraphics.framework
- SystemConfiguration.framework
- AdSupport.framework
- CFNetwork.framework
- SafariServices.framework

## Growthbeat の初期化

Growthbeat および Growth Push の初期化を行います。初期化では、デバイス登録、認証、および端末の基本情報の送信が行われます。

```objc
[[GrowthPush sharedInstance] initializeWithApplicationId:@"YOUR_APLICATION_ID" credentialId:@"YOUR_CREDENTIAL_ID" environment:kGrowthPushEnvironment];
```

Growth Push SDKからの乗り換えの場合は、[こちら](#growth-push-sdkからの乗り換え方法について)も参照してください。

# プッシュ通知

## プッシュ通知用の証明書の作成

Growth Push 管理画面にて、各 OS ごとに証明書の設定を行ってください。詳しくは、[iOS プッシュ通知証明書作成方法](http://growthhack.sirok.co.jp/growthpush/ios-p12/)をご参照ください。

## デバイストークンを取得・送信をする

Growthbeat の初期化後に下記を呼び出して、デバイストークンの取得を行います。

```objc
[[GrowthPush sharedInstance] requestDeviceToken];
```

`- (void)application:(UIApplication *)application
didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken` にて下記のように実装して、デバイストークンを送信します。

```objc
- (void)application:(UIApplication *)application
didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {
    [[GrowthPush sharedInstance] setDeviceToken:deviceToken];
}
```

登録されたデバイスは管理画面のデバイスページにて確認することができます。下記のように、デバイスのステータスがアクティブ（Active）で登録されていれば正常です。

<img src="/img/push/push_device_list.png" alt="push_device_list" title="push-device-list" width="100%"/>

## タグ・イベントを送信する。

セグメント配信を利用する際に、実装が必要となります。

[配信したいセグメント](/manual/growthpush/#セグメントの作成)に沿って、タグやイベントの紐付けを行ってください。

### タグ送信

```objc
[[GrowthPush sharedInstance] setTag:@"TagName" value:@"TagValue"];
```

[setTagメソッドについて](/sdk/ios/reference/#タグの送信)

### イベント送信

```objc
[[GrowthPush sharedInstance] trackEvent:@"EventName"];
```

[trackEventメソッドについて](/sdk/ios/reference/#イベントの送信)

# アプリ内メッセージ

## メッセージを作成する。

ここではアプリの起動時にメッセージを出す方法を説明します（共通初期設定でアプリの起動イベントを送信している必要があります）。

まず、管理画面にてアプリ起動時に配信されるメッセージを作成します。メッセージの作成方法は[こちら](/manual/growthmessage/#配信作成)を参考にしてください。

アプリ起動以外にも、カスタムイベントをメッセージ配信のトリガーにすることにより、アプリの任意の場所でメッセージを配信することができます。詳しくは、[こちら](/sdk/ios/reference/#カスタムイベント送信)をご参照ください。

## メッセージを表示する

Growth Pushのイベント送信と連動して、メッセージを受信します。

イベント名に紐付いたメッセージを作成するだけで、メッセージ表示することはできます。デフォルトでは、メッセージ受信時に即時に表示します。

`ShowMessageHandler` を利用することで、表示準備が完了したときに、メッセージ表示をすることができます。

例.) 起動時に、メッセージを表示する場合

```objc
[[GrowthPush sharedInstance] trackEvent:@"Launch" value:nil showMessage:^(void(^renderMessage)()){
    renderMessage();
} failure:^(NSString * detail) {

}];
```

# ディープリンク

## 初期設定

GrowthLinkのimport文を記述します。

```objc
#import <Growthbeat/GrowthLink.h>
```

## 初期化処理

Growthbeatの初期化処理の後に、Growth Linkの初期化処理を呼び出してください。 **APPLICATION_ID** と **CREDENTIAL_ID** は
Growthbeatの初期化時と同じものです。

```objc
[[GrowthLink sharedInstance] initializeWithApplicationId:@"APPLICATION_ID" credentialId:@"CREDENTIAL_ID"];
```

URL起動の処理で、handleOpenUrl:urlメソッドを呼び出す

```objc
- (BOOL) application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(NSString *)sourceApplication annotation:(id)annotation {
    [[GrowthLink sharedInstance] handleOpenUrl:url];
    return YES;
}
```

## プロジェクト設定

### アプリを開くスキームの設定
ディープリンクからアプリを起動できるように、info.plistの編集、もしくは Xcode上で Info -> URL Typesからディープリンクからアプリを開くスキームの設定をします。
URL Schemesにはスキームを、IdentifierにはBundle Identifierなどアプリごとに一意になる値を入力してください。
<img src="/img/link/link-guide-scheme.png" alt="link-guide-scheme" title="link-guide-scheme" width="100%"/>

### バージョンの設定

`General -> Identity -> Version`　が空欄であると正常に動作しません。
正しいバージョンを指定してください。

### Universal Links用の設定 (iOS9.x系)

iOS9 からカスタムスキームでの遷移に関する仕様が大幅に変更されました。なので iOS9.x 系に対応するには Universal Links の設定が必要になります。

#### apple.developer.com での設定

apple.developer.com にアクセスし “Certificate, Identifiers & Profiles” を選択。
その後"Identifers"をクリック。

<img src="/img/link/guide-universal-01.png" alt="guide-universal-01" title="guide-universal-01" width="70%"/>

<img src="/img/link/guide-universal-02.png" alt="guide-universal-02" title="guide-universal-02" width="30%"/>

Identiferを登録済みの時は"Edit"から編集を、未登録のときは"+"ボタンから新たに登録をしていきます。

NameやBundle IDは通常と同じ要領で記入してください。
<img src="/img/link/guide-universal-03.png" alt="guide-universal-03" title="guide-universal-03" width="70%"/>

Bundle ID は Xcode 上のGeneralのタブを選択することで確認できます。

<img src="/img/link/guide-xcode-bundle.png" alt="guide-xcode-bundle" title="gguide-xcode-bundle" width="70%"/>

App Servicesの欄で、Associated Domainsにチェックをてください。これを忘れるとこのあとのXcode上の操作でエラーがでてしまいます。

<img src="/img/link/guide-universal-04.png" alt="guide-universal-04" title="guide-universal-04" width="70%"/>

apple.developer.com での設定は以上です。Saveボタンをおして保存してください。

#### Xcode上 での設定

先ほどONにしたAssociated Domainsを使ってGrowthLinkのドメインを登録していきます。
登録の前に、先ほど登録したApp Identifierと同じTeamが選択されていることを確認してください。TeamはGeneralタブにあるIdentityセクションから選択できます。

CapabilitiesタブのAssociated Domainsをクリックすると展開されドメインの編集ができます。
ここで、GrowthLinkのドメインとなるgbt.ioを登録します。
＋ボタンをクリックし、"applinks:gbt.io"を追加してください。“applinks:”というのはprefixで登録ドメインの前につける必要があります。

<img src="/img/link/guide-universal-05.png" alt="guide-universal-05" title="guide-universal-05" width="70%"/>

現在、なんらかの原因でentitlementsファイルが生成されないことがあるようです。
プロジェクトブラウザ上でentitlementsファイルが生成されていることを確認してください。
<img src="/img/link/guide-universal-06.png" alt="guide-universal-06" title="guide-universal-06" width="70%"/>

また、entitlementsファイルがビルドに含まれている必要があります。含まれない場合はentitlementsファイルをクリックし、Targetにチェックが入っているか確認してください。

**ハンドリング処理の実装**
AppDelegate.mにUniversal Linksのハンドリング処理を実装します。

[Universal Links](http://faq.growthbeat.com/article/134-universallinks)専用リンクへの対応のため、以下のように実装してください。

#### UniversalLinksの取得実装について

```objc
#import <Growthbeat/GrowthLink.h> //インポートしておく


- (BOOL) application:(UIApplication *)application continueUserActivity:(NSUserActivity *)userActivity restorationHandler:(void (^)(NSArray * _Nullable))restorationHandler{
    if ([userActivity.activityType isEqualToString:NSUserActivityTypeBrowsingWeb]) {
        NSURL * webpageURL = userActivity.webpageURL;
        [[GrowthLink sharedInstance] handleUniversalLinks:webpageURL];
    }
    return true;
}

```

Xcode上での設定は以上になります。

## ディープリンクアクションの実装

SDKには、GBIntentHandlerというプロトコルが定義されており、この実装でディープリンク時のアクションを実装することができます。

たとえば下記のような形で実装できます。

```objc
#import <Growthbeat/GBCustomIntentHandler.h>  //インポート文に追記
```

```objc
 [[Growthbeat sharedInstance] addIntentHandler:[[GBCustomIntentHandler alloc] initWithBlock:^BOOL(GBCustomIntent *customIntent) {
        NSDictionary * extra = customIntent.extra;
        NSLog(@"extra: %@", extra);
        return YES;
}]];
```

### GrowthLink管理画面上 での設定

「基本設定」タブ -> リンク基本設定セクションから Universal Linksの設定ができます。
「Universal Linkに対応させる」をチェックし、
apple.developer.comに登録してあるBundle IdentifierとApple TeamIDを記入してください。どちらもapple.developer.comApp Identifiersから確認できます。
記入後「保存」ボタンを押して設定を保存してください。

<img src="/img/link/guide-universal-07.png" alt="guide-universal-07" title="guide-universal-07" width="70%"/>

### 検証の際の注意点
* 検証の際はアプリを一度アンインストールし、インストールしなおしてください。この手順を踏まない場合古い設定のままになります。
* GrowthLinkの仕様上、Universal  Links用の設定については10分ごとに反映されます。検証をする際は設定を保存後10分以上経過した後に行ってください。

### ランディングページを挟む場合の注意点

以下の記事を参考にしてください。

[【UniversalLinks】ランディングページにリンクを埋め込む際の注意点](http://faq.growthbeat.com/article/114-universallink)

# Growthbeat SDK 1.xからの変更点

## 機能削除

- インターフェスの変更があります。
 - 次の実装変更点でご確認ください。

- GrowthAnalyticsクラスがなくなりました。
 - 2.x以降は、GrowthPush#setTag, trackEventをご利用ください。

- GrowthbeatCoreクラスが、Growthbeatクラスに統合されました。
 - start, stop, initializeは削除されました。

## 実装変更点

### 初期化

- Growthbeat 1.x

```objc
- (BOOL) application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {

    [[Growthbeat sharedInstance] initializeWithApplicationId:@"YOUR_APPLICATION_ID" credentialId:@"YOUR_CREDENTIAL_ID"];
    [[GrowthPush sharedInstance] requestDeviceTokenWithEnvironment:kGrowthPushEnvironment];
    [[Growthbeat sharedInstance] getClient:^(GBClient* client) {
        NSLog(@"clientId is %@",client.id);
    }];

}

- (void) applicationDidBecomeActive:(UIApplication *)application {
    [[Growthbeat sharedInstance] start];
}

- (void) applicationWillResignActive:(UIApplication *)application {
    [[Growthbeat sharedInstance] stop];
}
```

- Growthbeat 2.x

```objc
- (BOOL) application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {

    [[GrowthPush sharedInstance] initializeWithApplicationId:@"YOUR_APPLICATION_ID" credentialId:@"YOUR_CREDENTIAL_ID" environment:kGrowthPushEnvironment];
	[[GrowthPush sharedInstance] requestDeviceToken];
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        GBClient* client = [[Growthbeat sharedInstance] waitClient];
        NSLog(@"clientId is %@", client.id);
    });

}

- (void) applicationDidBecomeActive:(UIApplication *)application {

}

- (void) applicationWillResignActive:(UIApplication *)application {

}
```

# Growth Push SDKからの乗り換え方法について

## 前準備

GrowthPushのApplicationIdから、GrowthbeatのApplicationIdに移行されるた
め、[Growthbeat](https://growthbeat.com/)にアクセスして、ApplicationId、SDKキー（CredentialID）を確認します。

ApplicationIdについては、Growth　Pushの左メニュー、シークレットキーのgrowthbeatApplicationIdという項目の左の文字列をご利用ください。

SDKキーに関しては、Growthbeatマイページにてご確認ください。

## 注意点

これまでGrowth Pushでご利用いただいた、ApplicationIdは数値型、シークレットキーは文字列になっています。

|項目|型|
|---|---|
|applicationId|数値型|
|secret|文字列型/32文字|

Growthbeat SDKで利用するものは、applicationId、credentialIdともに文字列型になっています。

|項目|型|
|---|---|
|applicationId|文字列型/16文字|
|credentailId|文字列型/32文字|

Growthbeat SDK乗り換え時に、これまでGrowth Pushで利用していたシークレットキーを設定しても、正しく動作しませんのでご注意くださいませ。

必ず、SDKキーをご利用ください。

## 実装方法

### SDKの初期化

- GrowthPush SDK

 * EasyGrowthPushクラスをご利用の場合

```objc
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    [EasyGrowthPush setApplicationId:kYourApplicationId secret:@"YOU_APP_SECRET" environment:kGrowthPushEnvironment debug:YES];
}
```

* GrowthPushクラスをご利用の場合

```objc
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    [GrowthPush setApplicationId:kYourApplicationId secret:@"YOU_APP_SECRET" environment:kGrowthPushEnvironment debug:YES];
    [GrowthPush requestDeviceToken];
    [GrowthPush setDeviceTags];
}
```

- Growthbeat SDK

```objc
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
	// Growthbeat SDKの初期化
	[[GrowthPush sharedInstance] initializeWithApplicationId:@"YOUR_APPLICATION_ID" credentialId:@"YOUR_CREDENTIAL_ID" environment:kGrowthPushEnvironment];
	// デバイストークンを明示的に要求
	[[GrowthPush sharedInstance] requestDeviceToken];

	// deviceTagの取得
	[[GrowthPush sharedInstance] setDeviceTags];
}

- (void) application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {
	// デバイストークンをGrowhPushに送信
	[[GrowthPush sharedInstance] setDeviceToken:deviceToken];
}
```

Growth Push SDKに存在したEasyGrowthPushクラスは、Growthbeat SDKでは廃止となっており、 `didRegisterForRemoteNotificationsWithDeviceToken` のデリゲートで、デバイストークンをGrowth Pushへ送信する実装を行う必要がございます。

### アプリ起動時

- Growthbeat SDK

```objc
- (void)applicationDidBecomeActive:(UIApplication *)application {
	// Restart any tasks that were paused (or not yet started) while the application was inactive. If the application was previously in the background, optionally refresh the user interface.

	// バッチの削除
	[[GrowthPush sharedInstance] clearBadge];

	// Launchイベントの取得
	[[GrowthPush sharedInstance] trackEvent:@"Launch"];
}
```

### タグ・イベントの取得

- GrowthPush SDK

```objc
// タグの取得
[GrowthPush setTag:@"TAG_NAME"];
[GrowthPush setTag:@"TAG_NAME" value:@"TAG_VALUE"];
// イベントの取得
[GrowthPush trackEvent:@"EVENT_NAME"];
[GrowthPush trackEvent:@"EVENT_NAME" value:@"EVENT_VALUE"];
```

- Growthbeat SDK

```objc
// タグの取得
[[GrowthPush sharedInstance] setTag:@"TAG_NAME"];
[[GrowthPush sharedInstance] setTag:@"TAG_NAME" value:@"TAG_VALUE"];
// イベントの取得
[[GrowthPush sharedInstance] trackEvent:@"EVENT_NAME"];
[[GrowthPush sharedInstance] trackEvent:@"EVENT_NAME" value:@"EVENT_VALUE"];
```

Growthbeat SDKは、シングルトンインスタンスの設計となっているため、実装の変更が必要となります。

## 移行確認方法

ログに下記のようなものが発生していれば、Growthbeat SDK移行が正しく行われています。

Growth Pushへの管理画面で、該当のGrowthPushClientIdのステータスが `Active` になっていれば、正しくプッシュ通知が行えます。

```
2016-03-16 20:31:52.743 GrowthbeatSample[1527:2124584] [GrowthbeatCore:INFO] convert client... (GrowthPushClientId:286049252, GrowthbeatClientId:PfbulyL0PsWOnCHj)
2016-03-16 20:31:52.876 GrowthbeatSample[1527:2124584] [GrowthbeatCore:INFO] Client converted. (id:PfbulyL0PsWOnCHj)
2016-03-16 20:31:52.969 GrowthbeatSample[1527:2124609] [GrowthPush:INFO] Create client... (growthbeatClientId: PfbulyL0PsWOnCHj, token: 0b466cb0529f435e80882ad87f5384ea8f44539307312cfc5301b0e1561b909f, environment: development)
2016-03-16 20:31:53.199 GrowthbeatSample[1527:2124609] [GrowthPush:INFO] Create client success. (clientId: PfbulyL0PsWOnCHj)
```

# 備考

ご不明な点などございます場合は、[ヘルプページ](http://growthbeat.helpscoutdocs.com/)を閲覧してください。
