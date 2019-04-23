---
title: react-native下使用原生SDK-iOS
date: 2019-04-23 15:15:06
categories: 
- React-Native
tags:
- React-Native
---
项目中需要接入国外的一款客服系统（Zendesk Chat），使用 Web 版打开在线客服体验不好，等待时间需要10秒左右，难以接受，所以周末试着接入原生 SDK。接完安卓版的，继续接入 iOS 版的，SDK 文档在[这里](https://developer.zendesk.com/embeddables/docs/ios-chat-sdk/introduction)。

## iOS SDK

可以使用 CocoaPods 或者使用 Xcode 导入 SDK，详见[文档](https://developer.zendesk.com/embeddables/docs/ios-chat-sdk/setup)。

1、使用 Xcode 在 ios/ 目录下新建 OpenChat.h 文件，内容如下：

```Objective-C
#import <Foundation/Foundation.h>
#import <ZDCChat/ZDCChat.h>
#import <React/RCTBridgeModule.h>
NS_ASSUME_NONNULL_BEGIN

@interface OpenChat : NSObject <RCTBridgeModule>

@end

NS_ASSUME_NONNULL_END
```

2、继续新建 OpenChat.m 文件

```Objective-C
#import "OpenChat.h"

@implementation OpenChat
RCT_EXPORT_MODULE();

RCT_EXPORT_METHOD(openService:(NSString *)name phone:(NSString *)phone email:(NSString *)email)
{
  dispatch_async(dispatch_get_main_queue(), ^{
    [ZDCChat startChat:^(ZDCConfig *config) {
      // 打开会话时需要用户状态（ZDCPreChatDataNotRequired：不是必须）
      config.preChatDataRequirements.email = ZDCPreChatDataNotRequired;
      config.preChatDataRequirements.name = ZDCPreChatDataNotRequired;
      config.preChatDataRequirements.phone = ZDCPreChatDataNotRequired;
      // 关闭会话时不提示发送邮件提醒
      config.emailTranscriptAction = ZDCEmailTranscriptActionNeverSend;
    }];
    // 打开会话时设置用户信息
    [ZDCChat updateVisitor:^(ZDCVisitorInfo *user) {
      user.phone = phone;
      user.name = name;
      user.email = email;
    }];
  });

}
```

3、AppDelegate.h 中添加

```Objective-C
#import <ZDCChat/ZDCChat.h>

// @interface AppDelegate ...
@property (nonatomic, strong) UINavigationController * nav;
```

4、AppDelegate.m 中添加

```Objective-C
@implementation AppDelegate
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
  // ...
  //初始化聊天配置AccountKey
  [ZDCChat  initializeWithAccountKey:@"xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"];
  //侦听聊天连接事件
  [[ZDCChatAPI instance] addObserver:self forConnectionEvents:@selector(timeoutAction)];
}

-(void)timeoutAction {
  
  ZDCLoadingErrorView  * view = [ZDCLoadingErrorView appearance];
  if ([ZDCChatAPI instance].chatStatus == ZDCChatSessionStatusConnected && [ZDCChatAPI instance].connectionStatus == ZDCConnectionStatusConnected && ![ZDCChatAPI instance].isAccountOnline) {
    view.hidden = YES;
    [[ZDCChat instance].chatViewController.chatUI activate];
    [[ZDCChat instance].chatViewController.chatUI showOfflineForm];
    [[ZDCChatAPI instance] removeObserverForConnectionEvents:self];
  }
}
// ...
```

5、在 react-native 代码中使用

```js
import { NativeModules } from "react-native";

if (Platform.OS === 'ios') {
  NativeModules.OpenChat.openService('name', '18888888888', 'email@gmail.com');
}
```

6、定制样式

a. 新建 ChatStyling.h 文件：

```Objective-C
#import <Foundation/Foundation.h>
#import <ZDCChat/ZDCChat.h>


@interface ChatStyling : NSObject


/**
 * Apply the chat styling.
 */
+ (void) applyStyling;

@end
```

b. 新建 ChatStyling.m 文件：（样式在此文件修改）

```Objective-C
#import "ChatStyling.h"

@implementation ChatStyling

+ (void) applyStyling
{
  UIEdgeInsets insets;
  
  ////////////////////////////////////////////////////////////////////////////////////////////////
  // Style the chat cells
  ////////////////////////////////////////////////////////////////////////////////////////////////
  
  insets = UIEdgeInsetsMake(10.0f, 70.0f , 10.0f, 20.0f);
  [[ZDCJoinLeaveCell appearance] setTextInsets:[NSValue valueWithUIEdgeInsets:insets]];
  [[ZDCJoinLeaveCell appearance] setTextColor:[UIColor colorWithWhite:0.26f alpha:1.0f]];
  [[ZDCJoinLeaveCell appearance] setTextFont:[UIFont boldSystemFontOfSize:14]];
  
  insets = UIEdgeInsetsMake(8.0f, 75.0f , 7.0f, 15.0f);
  [[ZDCVisitorChatCell appearance] setBubbleInsets:[NSValue valueWithUIEdgeInsets:insets]];
  insets = UIEdgeInsetsMake(12.0f, 15.0f, 12.0f, 15.0f);
  [[ZDCVisitorChatCell appearance] setTextInsets:[NSValue valueWithUIEdgeInsets:insets]];
//  [[ZDCVisitorChatCell appearance] setBubbleBorderColor:[ChatStyling darkerColorForColor:[ChatStyling navBarTintColor] by:0.2f]];
  [[ZDCVisitorChatCell appearance] setBubbleBorderColor:[UIColor colorWithRed:70.0f/255.0f green:158.0f/255.0f blue:253.0f/255.0f alpha:0.5]];

  // 聊天气泡颜色
//  [[ZDCVisitorChatCell appearance] setBubbleColor:[UIColor colorWithRed:92.0f/255.0f green:201.0f/255.0f blue:245.0f/255.0f alpha:1.0]];
  [[ZDCVisitorChatCell appearance] setBubbleColor:[ChatStyling navBarTintColor]];
  [[ZDCVisitorChatCell appearance] setBubbleCornerRadius:@(3.0f)];
  [[ZDCVisitorChatCell appearance] setTextAlignment:@(NSTextAlignmentLeft)];
//  [[ZDCVisitorChatCell appearance] setTextColor:[UIColor colorWithWhite:1.0f alpha:1.0f]];
//  [[ZDCVisitorChatCell appearance] setTextColor:[ChatStyling navTintColor]]
  [[ZDCVisitorChatCell appearance] setTextColor:[UIColor colorWithRed:70.0f/255.0f green:158.0f/255.0f blue:253.0f/255.0f alpha:1.0]];
  [[ZDCVisitorChatCell appearance] setTextFont:[UIFont systemFontOfSize:14.0f]];
  [[ZDCVisitorChatCell appearance] setUnsentTextColor:[UIColor colorWithWhite:0.26f alpha:1.0f]];
  [[ZDCVisitorChatCell appearance] setUnsentTextFont:[UIFont systemFontOfSize:12.0f]];
  [[ZDCVisitorChatCell appearance] setUnsentMessageTopMargin:@(5.0f)];
  [[ZDCVisitorChatCell appearance] setUnsentIconLeftMargin:@(10.0f)];
  
  insets = UIEdgeInsetsMake(8.0f, 55.0f, 7.0f, 30.0f);
  [[ZDCAgentChatCell appearance] setBubbleInsets:[NSValue valueWithUIEdgeInsets:insets]];
  insets = UIEdgeInsetsMake(12.0f, 15.0f, 12.0f, 15.0f);
  [[ZDCAgentChatCell appearance] setTextInsets:[NSValue valueWithUIEdgeInsets:insets]];
  [[ZDCAgentChatCell appearance] setBubbleBorderColor:[UIColor colorWithWhite:0.90f alpha:1.0f]];
  [[ZDCAgentChatCell appearance] setBubbleColor:[UIColor colorWithWhite:0.95f alpha:1.0f]];
  [[ZDCAgentChatCell appearance] setBubbleCornerRadius:@(3.0f)];
  [[ZDCAgentChatCell appearance] setTextAlignment:@(NSTextAlignmentLeft)];
  [[ZDCAgentChatCell appearance] setTextColor:[UIColor colorWithWhite:0.26f alpha:1.0f]];
  [[ZDCAgentChatCell appearance] setTextFont:[UIFont systemFontOfSize:14.0f]];
  [[ZDCAgentChatCell appearance] setAvatarHeight:@(30.0f)];
  [[ZDCAgentChatCell appearance] setAvatarLeftInset:@(14.0f)];
  [[ZDCAgentChatCell appearance] setAuthorColor:[UIColor colorWithWhite:0.60f alpha:1.0f]];
  [[ZDCAgentChatCell appearance] setAuthorFont:[UIFont systemFontOfSize:12]];
  [[ZDCAgentChatCell appearance] setAuthorHeight:@(25.0f)];
  
  insets = UIEdgeInsetsMake(10.0f, 20.0f, 10.0f, 20.0f);
  [[ZDCSystemTriggerCell appearance] setTextInsets:[NSValue valueWithUIEdgeInsets:insets]];
  [[ZDCSystemTriggerCell appearance] setTextColor:[UIColor colorWithWhite:0.26f alpha:1.0f]];
  [[ZDCSystemTriggerCell appearance] setTextFont:[UIFont boldSystemFontOfSize:14.0f]];
  
  insets = UIEdgeInsetsMake(10.0f, 20.0f, 10.0f, 20.0f);
  [[ZDCChatTimedOutCell appearance] setTextInsets:[NSValue valueWithUIEdgeInsets:insets]];
  [[ZDCChatTimedOutCell appearance] setTextColor:[UIColor colorWithWhite:0.26f alpha:1.0f]];
  [[ZDCChatTimedOutCell appearance] setTextFont:[UIFont boldSystemFontOfSize:14.0f]];
  
  [[ZDCRatingCell appearance] setTitleColor:[UIColor colorWithWhite:0.26f alpha:1.0f]];
  [[ZDCRatingCell appearance] setTitleFont:[UIFont boldSystemFontOfSize:14]];
  [[ZDCRatingCell appearance] setCellToTitleMargin:@(20.0f)];
  [[ZDCRatingCell appearance] setTitleToButtonsMargin:@(10.0f)];
  [[ZDCRatingCell appearance] setRatingButtonToCommentMargin:@(20.0f)];
  [[ZDCRatingCell appearance] setEditCommentButtonHeight:@(44.0f)];
  [[ZDCRatingCell appearance] setRatingButtonSize:@(40.0f)];
  
  [[ZDCAgentAttachmentCell appearance] setActivityIndicatorViewStyle:@(UIActivityIndicatorViewStyleGray)];
  
  [[ZDCVisitorAttachmentCell appearance] setActivityIndicatorViewStyle:@(UIActivityIndicatorViewStyleGray)];
  
}

+ (UIColor *)darkerColorForColor:(UIColor *)color by:(float)diff
{
  CGFloat r, g, b, a;
  if ([color getRed:&r green:&g blue:&b alpha:&a])
    return [UIColor colorWithRed:MAX(r - diff, 0.0)
                           green:MAX(g - diff, 0.0)
                            blue:MAX(b - diff, 0.0)
                           alpha:a];
  return nil;
}

+ (UIColor*) navBarTintColor
{
  return [UINavigationBar appearance].barTintColor;
}

+ (UIColor*) navTintColor
{
  return [UINavigationBar appearance].tintColor;
}

@end
```

c. AppDelegate.m 中添加

```Objective-C
#import "ChatStyling.h"

@implementation AppDelegate
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
  // ...
  //初始化聊天配置AccountKey
  [ZDCChat  initializeWithAccountKey:@"xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"];
  //侦听聊天连接事件
  [[ZDCChatAPI instance] addObserver:self forConnectionEvents:@selector(timeoutAction)];

  // 自定义 Zendesk Chat 聊天 UI
  [ChatStyling applyStyling];
}
```

7、修改会话文字等
文档中原文：The provided strings can be changed easily with XCode. In your project, right-click the strings file in the bundle and select Open As > ASCII Property List.

即使用 Xcode 打开导入的 SDK 文件夹，进入 ZDCChatStrings.bundle 目录，选择需要修改的语言，右键 Localizable.strings，选择使用 ASCII Property List 方式打开，然后修改文字即可。