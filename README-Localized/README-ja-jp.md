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
# Microsoft Graph SDK を使用した iOS 用 Office 365 Connect サンプル

Microsoft Graph は、Microsoft Cloud からのデータ、リレーションシップおよびインサイトにアクセスするための統合エンドポイントです。このサンプルでは、これに接続して認証し、Microsoft [Graph SDK for iOS](https://github.com/microsoftgraph/msgraph-sdk-ios) 経由でメールとユーザーの API を呼び出す方法を示します。

> 注:このサンプルをより迅速に実行するため、登録手順が簡略化された「[Microsoft Graph アプリ登録ポータル](https://apps.dev.microsoft.com)」ページをお試しください。

## 前提条件
* Apple 社の [Xcode](https://developer.apple.com/xcode/downloads/) をダウンロードする。

* 依存関係マネージャーとしての [CocoaPods](https://guides.cocoapods.org/using/using-cocoapods.html) のインストール。
* Office 365、outlook.com、hotmail.com などの、Microsoft の職場または個人用のメール アカウント。Office 365 アプリのビルドを開始するために必要なリソースを含む [Office 365 Developer サブスクリプション](https://aka.ms/devprogramsignup)にサインアップできます。

     > 注:サブスクリプションをすでにお持ちの場合、上記のリンクをクリックすると、「*申し訳ございません。現在のアカウントに追加できません*」というメッセージが表示されるページに移動します。その場合は、現在使用している Office 365 サブスクリプションのアカウントをご利用いただけます。    
* [Microsoft Graph アプリ登録ポータル](https://apps.dev.microsoft.com) で登録済みのアプリのクライアント ID
* 要求を実行するには、適切な OAuth 2.0 ベアラー トークンを使用して HTTPS 要求を認証できる **MSAuthenticationProvider** を指定する必要があります。プロジェクトをすぐに開始するために使用できる MSAuthenticationProvider をサンプル実装するために、[msgraph-sdk-ios-nxoauth2-adapter](https://github.com/microsoftgraph/msgraph-sdk-ios-nxoauth2-adapter) を使用します。詳細については、以下の「**目的のコード**」セクションを参照してください。


## Xcode でこのサンプルを実行する

1. このリポジトリの複製を作成する
2. CocoaPods の依存関係のマネージャーがまだインストールされていない場合、**ターミナル** アプリから以下のコマンドを実行してインストールし、設定を行います。

		sudo gem install cocoapods
	
		pod setup

2. CocoaPods を使用して、Microsoft Graph SDK と認証の依存関係をインポートします:

		pod 'MSGraphSDK'
		pod 'MSGraphSDK-NXOAuth2Adapter'


 このサンプル アプリには、プロジェクトに pod を取り込む podfile が既に含まれています。profile のあるプロジェクト ルートに移動し、**ターミナル**から以下を実行します：

        pod install

   詳細について、「[その他のリソース](#AdditionalResources)」の「**CocoaPods を使う**」を参照してください。

3. **ios-objectivec-sample.xcworkspace** を開きます。
4. **AuthenticationConstants.m** を開きます。登録プロセスで取得した **ClientID** がファイルの一番上に追加されていることが分かります。:

   ```objectivec
        // You will set your application's clientId
        NSString * const kClientId    = @"ENTER_YOUR_CLIENT_ID";
   ```


    このプロジェクトに対して次のアクセス許可の適用範囲が構成されていることがわかります。 

```@"https://graph.microsoft.com/User.Read、https://graph.microsoft.com/Mail.ReadWrite、https://graph.microsoft.com/Mail.Send、https://graph.microsoft.com/Files.ReadWrite"```
    

    
>注:このプロジェクトで使用されるサービス呼び出し、メール アカウントへのメールの送信、OneDrive への画像のアップロード、および一部のプロファイル情報 (表示名、メール アドレス、プロファイル画像) の取得では、アプリが適切に実行するためにこれらのアクセス許可が必要です。

5. サンプルを実行します。職場または個人用のメール アカウントに接続または認証するように求められ、そのアカウントか、別の選択したメール アカウントにメールを送信することができます。


## 目的のコード

すべての認証コードは、**AuthenticationProvider.m** ファイルで確認することができます。[NXOAuth2Client](https://github.com/nxtbgthng/OAuth2Client) から拡張された MSAuthenticationProvider のサンプル実装を使用して、登録済みのネイティブ アプリのログインのサポート、アクセス トークンの自動更新、ログアウト機能を提供します。

```objectivec

		[[NXOAuth2AuthenticationProvider sharedAuthProvider] loginWithViewController:nil completion:^(NSError *error) {
    		if (!error) {
        	[MSGraphClient setAuthenticationProvider:[NXOAuth2AuthenticationProvider sharedAuthProvider]];
        	self.client = [MSGraphClient client];
   			 }
		}];
```

認証プロバイダーを設定すると、Microsoft Graph サービス エンドポイント (メールとユーザー) に対して呼び出しを実行するために使用されるクライアント オブジェクト (MSGraphClient) の作成と初期化が行えます。**SendMailViewcontroller.m** では、ユーザー プロファイル画像を取得し、それを OneDrive にアップロードし、メール要求を画像添付ファイル付きで作成し、それを次のコードを使用して送信することができます。

### ユーザーのプロファイル画像を取得する

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
### 画像を OneDrive にアップロードする

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
### 新しいメール メッセージに画像添付ファイルを追加する

```objectivec
   MSGraphFileAttachment *fileAttachment= [[MSGraphFileAttachment alloc]init];
    fileAttachment.oDataType = @"#microsoft.graph.fileAttachment";
    fileAttachment.contentType = @"image/png";
    
    NSString *decodedString = [UIImagePNGRepresentation(self.userPicture) base64EncodedStringWithOptions:NSDataBase64EncodingEndLineWithCarriageReturn];
    
    fileAttachment.contentBytes = decodedString;
    fileAttachment.name = @"me.png";
    message.attachments = [message.attachments arrayByAddingObject:(fileAttachment)];
```

### メール メッセージを送信する

```objectivec
    MSGraphUserSendMailRequestBuilder *requestBuilder = [[self.client me]sendMailWithMessage:message saveToSentItems:true];    
    MSGraphUserSendMailRequest *mailRequest = [requestBuilder request];   
    [mailRequest executeWithCompletion:^(NSDictionary *response, NSError *error) {      
    }];
```

OneDrive などのその他のサービスへの呼び出しを行うコードなどの詳細については、「[Microsoft Graph SDK for iOS](https://github.com/microsoftgraph/msgraph-sdk-ios)」を参照してください。

## 質問とコメント

Office 365 iOS Microsoft Graph Connect プロジェクトに関するフィードバックをお寄せください。質問や提案につきましては、このリポジトリの「[問題](https://github.com/microsoftgraph/iOS-objectivec-connect-sample/issues)」セクションで送信できます。

Office 365 開発全般の質問につきましては、「[スタック オーバーフロー](http://stackoverflow.com/questions/tagged/Office365+API)」に投稿してください。質問やコメントには、必ず \[Office365] と \[MicrosoftGraph] のタグを付けてください。

## 投稿
プル要求を送信する前に、[投稿者のライセンス契約](https://cla.microsoft.com/)に署名する必要があります。投稿者のライセンス契約 (CLA) を完了するには、ドキュメントへのリンクを含むメールを受信した際に、フォームから要求を送信し、CLA に電子的に署名する必要があります。

このプロジェクトでは、[Microsoft Open Source Code of Conduct (Microsoft オープン ソース倫理規定)](https://opensource.microsoft.com/codeofconduct/) が採用されています。詳細については、「[Code of Conduct の FAQ](https://opensource.microsoft.com/codeofconduct/faq/)」を参照してください。また、その他の質問やコメントがあれば、[opencode@microsoft.com](mailto:opencode@microsoft.com) までお問い合わせください。

## その他のリソース

* [Office デベロッパー センター](http://dev.office.com/)
* [Microsoft Graph の概要ページ](https://graph.microsoft.io)
* [CocoaPods を使う](https://guides.cocoapods.org/using/using-cocoapods.html)

## 著作権
Copyright (c) 2016 Microsoft.All rights reserved.
