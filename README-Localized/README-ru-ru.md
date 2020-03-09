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
# Пример приложения для iOS, подключающегося к Office 365 и использующего пакет SDK Microsoft Graph

Microsoft Graph — единая конечная точка для доступа к данным, отношениям и аналитике из Microsoft Cloud. В этом примере показано, как подключиться к ней и пройти проверку подлинности, а затем вызывать почтовые и пользовательские API через[пакет SDK Microsoft Graph для iOS](https://github.com/microsoftgraph/msgraph-sdk-ios).

> Примечание. Воспользуйтесь упрощенной регистрацией на [портале регистрации приложений Microsoft Graph](https://apps.dev.microsoft.com), чтобы ускорить запуск этого примера.

## Необходимые компоненты
* Скачивание [Xcode](https://developer.apple.com/xcode/downloads/) от Apple.

* Установка [CocoaPods](https://guides.cocoapods.org/using/using-cocoapods.html) в качестве диспетчера зависимостей.
* Рабочая или личная учетная запись Майкрософт, например Office 365, outlook.com или hotmail.com. Вы можете [подписаться на план Office 365 для разработчиков](https://aka.ms/devprogramsignup), который включает ресурсы, необходимые для создания приложений Office 365.

     > Примечание. Если у вас уже есть подписка, при выборе приведенной выше ссылки откроется страница с сообщением *К сожалению, вы не можете добавить это к своей учетной записи*. В этом случае используйте учетную запись, связанную с текущей подпиской на Office 365.    
* Идентификатор клиента из приложения, зарегистрированного на [портале регистрации приложений Microsoft Graph](https://apps.dev.microsoft.com)
* Чтобы отправлять запросы, необходимо указать протокол **MSAuthenticationProvider**, который способен проверять подлинность HTTPS-запросов с помощью соответствующего маркера носителя OAuth 2.0. Для реализации протокола MSAuthenticationProvider и быстрого запуска проекта мы будем использовать [msgraph-sdk-ios-nxoauth2-adapter](https://github.com/microsoftgraph/msgraph-sdk-ios-nxoauth2-adapter). Дополнительные сведения см. в разделе **Полезный код** ниже.


## Запуск примера в Xcode

1. Клонируйте этот репозиторий
2. Если диспетчер зависимостей CocoaPods еще не установлен, запустите указанные ниже команды из приложения **Терминал**, чтобы установить и настроить его.

		sudo gem install cocoapods
	
		pod setup

2. Используйте CocoaPods, чтобы импортировать пакет SDK Microsoft Graph и зависимости проверки подлинности:

		pod 'MSGraphSDK'
		pod 'MSGraphSDK-NXOAuth2Adapter'


 Этот пример приложения уже содержит podfile, который добавит компоненты pod в проект. Просто перейдите к корню проекта, где находится podfile, и в приложении **Терминал** запустите следующий код:

        pod install

   Для получения дополнительных сведений выберите ссылку **Использование CocoaPods** в разделе [Дополнительные ресурсы](#AdditionalResources).

3. Откройте **ios-objectivec-sample.xcworkspace**
4. Откройте файл **AuthenticationConstants.m**. Вы увидите, что в верхнюю часть файла можно добавить **идентификатор клиента**, скопированный в ходе регистрации.

   ```objectivec
        // You will set your application's clientId
        NSString * const kClientId    = @"ENTER_YOUR_CLIENT_ID";
   ```


    Вы увидите, что для этого проекта настроены следующие разрешения: 

```@"https://graph.microsoft.com/User.Read, https://graph.microsoft.com/Mail.ReadWrite, https://graph.microsoft.com/Mail.Send, https://graph.microsoft.com/Files.ReadWrite"```
    

    
>Примечание. Эти разрешения необходимы для правильной работы приложения, в частности отправки сообщения в учетную запись почты, загрузки изображения в OneDrive и получения информации из профиля (отображаемое имя, адрес электронной почты, аватар).

5. Запустите пример. Вам будет предложено подключить рабочую или личную учетную запись почты и войти в нее, после чего вы сможете отправить сообщение в эту или другую учетную запись.


## Полезный код

Весь код для проверки подлинности можно найти в файле **AuthenticationProvider.m**. Мы используем протокол MSAuthenticationProvider из файла [NXOAuth2Client](https://github.com/nxtbgthng/OAuth2Client) для поддержки входа в зарегистрированных собственных приложениях, автоматического обновления токенов доступа и выхода:

```objectivec

		[[NXOAuth2AuthenticationProvider sharedAuthProvider] loginWithViewController:nil completion:^(NSError *error) {
    		if (!error) {
        	[MSGraphClient setAuthenticationProvider:[NXOAuth2AuthenticationProvider sharedAuthProvider]];
        	self.client = [MSGraphClient client];
   			 }
		}];
```

Если поставщик проверки подлинности настроен, мы можем создать и инициализировать объект клиента (MSGraphClient), который будет использоваться для вызова службы Microsoft Graph (почта и пользователи). С помощью **SendMailViewcontroller.m** можно получить аватар пользователя, добавить его в OneDrive, собрать почтовый запрос с вложенным изображением и отправить его, воспользовавшись приведенным ниже кодом.

### Получение аватара пользователя

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
### Добавление изображения в OneDrive

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
### Добавление вложенного изображения к новому сообщению

```objectivec
   MSGraphFileAttachment *fileAttachment= [[MSGraphFileAttachment alloc]init];
    fileAttachment.oDataType = @"#microsoft.graph.fileAttachment";
    fileAttachment.contentType = @"image/png";
    
    NSString *decodedString = [UIImagePNGRepresentation(self.userPicture) base64EncodedStringWithOptions:NSDataBase64EncodingEndLineWithCarriageReturn];
    
    fileAttachment.contentBytes = decodedString;
    fileAttachment.name = @"me.png";
    message.attachments = [message.attachments arrayByAddingObject:(fileAttachment)];
```

### Отправка сообщения

```objectivec
    MSGraphUserSendMailRequestBuilder *requestBuilder = [[self.client me]sendMailWithMessage:message saveToSentItems:true];    
    MSGraphUserSendMailRequest *mailRequest = [requestBuilder request];   
    [mailRequest executeWithCompletion:^(NSDictionary *response, NSError *error) {      
    }];
```

Дополнительные сведения, в том числе код для вызова других служб, например OneDrive, см. в статье [Пакет SDK Microsoft Graph для iOS](https://github.com/microsoftgraph/msgraph-sdk-ios).

## Вопросы и комментарии

Мы будем рады получить от вас отзывы о проекте приложения iOS, подключающегося к Office 365 и использующего Microsoft Graph. Отправляйте нам свои вопросы и предложения в раздел этого репозитория, посвященный [проблемам](https://github.com/microsoftgraph/iOS-objectivec-connect-sample/issues).

Общие вопросы о разработке решений для Office 365 следует публиковать на сайте [Stack Overflow](http://stackoverflow.com/questions/tagged/Office365+API). Обязательно помечайте свои вопросы и комментарии тегами \[Office365] и \[MicrosoftGraph].

## Участие
Прежде чем отправить запрос на включение внесенных изменений, необходимо подписать [лицензионное соглашение с участником](https://cla.microsoft.com/). Чтобы заполнить лицензионное соглашение с участником (CLA), вам потребуется отправить запрос с помощью формы, а затем после получения электронного сообщения со ссылкой на этот документ подписать CLA с помощью электронной подписи.

Этот проект соответствует [Правилам поведения разработчиков открытого кода Майкрософт](https://opensource.microsoft.com/codeofconduct/). Дополнительные сведения см. в разделе [вопросов и ответов о правилах поведения](https://opensource.microsoft.com/codeofconduct/faq/). Если у вас возникли вопросы или замечания, напишите нам по адресу [opencode@microsoft.com](mailto:opencode@microsoft.com).

## Дополнительные ресурсы

* [Центр разработки для Office](http://dev.office.com/)
* [Страница с общими сведениями о Microsoft Graph](https://graph.microsoft.io)
* [Использование CocoaPods](https://guides.cocoapods.org/using/using-cocoapods.html)

## Авторские права
(c) Корпорация Майкрософт (Microsoft Corporation), 2016. Все права защищены.
