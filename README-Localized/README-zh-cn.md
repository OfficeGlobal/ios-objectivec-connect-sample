---
page_type: sample
products:
- office-365 
- ms-graph
languages:
- objc
extensions:
  contentType: samples
  technologies:
  - Microsoft Graph
  - Microsoft identity platform
  services:
  - Office 365
  - Microsoft identity platform
  - Users
  platforms:
  - iOS
  createdDate: 5/12/2016 8:22:39 AM
---
# 适用于 iOS 的 Office 365 连接示例（使用 Microsoft Graph SDK）

Microsoft Graph 是访问来自 Microsoft 云的数据、关系和见解的统一终结点。此示例介绍如何连接并对其进行身份验证，然后通过[适用于 iOS 的 Microsoft Graph SDK](https://github.com/microsoftgraph/msgraph-sdk-ios) 调用邮件和用户 API。

> 注意：尝试 [Microsoft Graph 应用注册门户](https://apps.dev.microsoft.com)页，该页简化了注册，因此你可以更快地运行该示例。

## 先决条件
* 从 Apple 下载 [Xcode](https://developer.apple.com/xcode/downloads/)。

* 安装 [CocoaPods](https://guides.cocoapods.org/using/using-cocoapods.html) 成为依存关系管理器。
* Microsoft 工作或个人电子邮件帐户，例如 Office 365 或 outlook.com、hotmail.com 等。你可以注册 [Office 365 开发人员订阅](https://aka.ms/devprogramsignup)，其中包含你开始构建 Office 365 应用所需的资源。

     > 注意：如果您已经订阅，之前的链接会将您转至包含以下信息的页面：*抱歉，您无法将其添加到当前帐户*。在这种情况下，请使用当前 Office 365 订阅中的帐户。    
* [Microsoft Graph 应用注册门户](https://apps.dev.microsoft.com)中已注册应用的客户端 ID
* 若要生成请求，必须提供 **MSAuthenticationProvider**（它能够使用适当的 OAuth 2.0 持有者令牌对 HTTPS 请求进行身份验证）。我们将使用 [msgraph-sdk-ios-nxoauth2-adapter](https://github.com/microsoftgraph/msgraph-sdk-ios-nxoauth2-adapter) 作为 MSAuthenticationProvider 的示例实现，它可用于快速启动你的项目。有关详细信息，请参阅下面的“**相关代码**”部分。


## 在 Xcode 中运行此示例

1. 克隆该存储库
2. 如果尚未安装，从**终端**应用运行以下命令来安装和设置 CocoaPods 依存关系管理器。

		sudo gem install cocoapods
	
		pod setup

2. 使用 CocoaPods 导入 Microsoft Graph SDK 和身份验证依赖项：

		pod 'MSGraphSDK'
		pod "MSGraphSDK-NXOAuth2Adapter"


 该示例应用已包含可将 pod 导入项目中的 Podfile。仅定位到 pod 文件所在的项目根并从**终端**运行：

        pod install

   更多详细信息，请参阅[其他资源](#AdditionalResources)中的**使用 CocoaPods**

3. 打开 **ios-objectivec-sample.xcworkspace**
4. 打开 **AuthenticationConstants.m**。你会发现，注册过程的 **ClientID** 可以添加到文件顶部：

   ```objectivec
        // You will set your application's clientId
        NSString * const kClientId    = @"ENTER_YOUR_CLIENT_ID";
   ```


    你会发现，已为此项目配置了以下权限范围： 

```@"https://graph.microsoft.com/User.Read, https://graph.microsoft.com/Mail.ReadWrite, https://graph.microsoft.com/Mail.Send, https://graph.microsoft.com/Files.ReadWrite"```
    

    
>注意：此项目向邮件帐户发送邮件，将图片上传到 OneDrive，并检索一些个人资料信息（显示名称、电子邮件地址和个人资料图片）。其中使用的服务调用需要这些权限，这样应用才能正常运行。

5. 运行示例。系统将要求你连接至工作或个人邮件帐户或对其进行身份验证，然后你可以向该帐户或其他所选电子邮件帐户发送邮件。


## 相关代码

可以在 **AuthenticationProvider.m** 文件中查看所有身份验证代码。我们使用从 [NXOAuth2Client](https://github.com/nxtbgthng/OAuth2Client) 扩展的 MSAuthenticationProvider 示例实现来提供对已注册的本机应用的登录支持、访问令牌的自动刷新和注销功能：

```objectivec

		[[NXOAuth2AuthenticationProvider sharedAuthProvider] loginWithViewController:nil completion:^(NSError *error) {
    		if (!error) {
        	[MSGraphClient setAuthenticationProvider:[NXOAuth2AuthenticationProvider sharedAuthProvider]];
        	self.client = [MSGraphClient client];
   			 }
		}];
```

对身份验证提供程序进行设置后，我们可以创建并初始化客户端对象 (MSGraphClient)，它将被用于对 Microsoft Graph 服务终结点（邮件和用户）进行调用。在 **SendMailViewcontroller.m** 中可以获取用户配置文件图片，将其上传到 OneDrive，使用图片附件汇编邮件请求，然后使用以下代码发送：

### 获取用户个人资料图片

```objectivec
[[[self.graphClient me] photoValue] downloadWithCompletion:^(NSURL *location, NSURLResponse *response, NSError *error) {
        //code
        if (!error) {
            NSData *data = [NSData dataWithContentsOfURL:location];
            UIImage *img = [[UIImage alloc] initWithData:data];
                            completionBlock(img, error);
        } 
    }];
```
### 将图片上传到 OneDrive

```objectivec
    NSData *data = UIImagePNGRepresentation(image);
    [[[[[[[self.graphClient me]
          drive]
         root]
        children]
       driveItem:(@"me.png")]
      contentRequest]
     uploadFromData:(data) completion:^(MSGraphDriveItem *response, NSError *error) {
         
         if (!error) {
             NSString *webUrl = response.webUrl;
             completionBlock(webUrl, error);
         } 
    }];

```
### 将图片附件添加到新电子邮件中

```objectivec
   MSGraphFileAttachment *fileAttachment= [[MSGraphFileAttachment alloc]init];
    fileAttachment.oDataType = @"#microsoft.graph.fileAttachment";
    fileAttachment.contentType = @"image/png";
    
    NSString *decodedString = [UIImagePNGRepresentation(self.userPicture) base64EncodedStringWithOptions:NSDataBase64EncodingEndLineWithCarriageReturn];
    
    fileAttachment.contentBytes = decodedString;
    fileAttachment.name = @"me.png";
    message.attachments = [message.attachments arrayByAddingObject:(fileAttachment)];
```

### 发送邮件

```objectivec
    MSGraphUserSendMailRequestBuilder *requestBuilder = [[self.client me]sendMailWithMessage:message saveToSentItems:true];    
    MSGraphUserSendMailRequest *mailRequest = [requestBuilder request];   
    [mailRequest executeWithCompletion:^(NSDictionary *response, NSError *error) {      
    }];
```

有关详细信息（包括用于调用 OneDrive 等其他服务的代码），请参阅[适用于 iOS 的 Microsoft Graph SDK](https://github.com/microsoftgraph/msgraph-sdk-ios)

## 问题和意见

我们乐意倾听您有关 Office 365 iOS Microsoft Graph Connect 项目的反馈。您可以在该存储库中的[问题](https://github.com/microsoftgraph/iOS-objectivec-connect-sample/issues)部分将问题和建议发送给我们。

与 Office 365 开发相关的问题一般应发布在[堆栈溢出](http://stackoverflow.com/questions/tagged/Office365+API)中。确保您的问题或意见使用了 \[Office365] 和 \[MicrosoftGraph] 标记。

## 贡献
您需要在提交拉取请求之前签署[参与者许可协议](https://cla.microsoft.com/)。要完成参与者许可协议 (CLA)，你需要通过表格提交请求，并在收到包含文件链接的电子邮件时在 CLA 上提交电子签名。

此项目已采用 [Microsoft 开放源代码行为准则](https://opensource.microsoft.com/codeofconduct/)。有关详细信息，请参阅[行为准则常见问题解答](https://opensource.microsoft.com/codeofconduct/faq/)。如有其他任何问题或意见，也可联系 [opencode@microsoft.com](mailto:opencode@microsoft.com)。

## 其他资源

* [Office 开发人员中心](http://dev.office.com/)
* [Microsoft Graph 概述页](https://graph.microsoft.io)
* [使用 CocoaPods](https://guides.cocoapods.org/using/using-cocoapods.html)

## 版权信息
版权所有 (c) 2016 Microsoft。保留所有权利。
