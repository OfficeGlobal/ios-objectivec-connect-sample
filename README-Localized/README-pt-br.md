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
# Exemplo de Conexão com o Office 365 para iOS usando o SDK do Microsoft Graph

O Microsoft Graph é um ponto de extremidade unificado para acessar dados, relações e ideias que vêm do Microsoft Cloud. Este exemplo mostra como realizar a conexão e a autenticação no Microsoft Graph e chamar APIs de mala direta e usuário por meio do [SDK do Microsoft Graph para iOS](https://github.com/microsoftgraph/msgraph-sdk-ios).

> Observação: Experimente a página [Portal de Registro de Aplicativos do Microsoft Graph](https://apps.dev.microsoft.com) que simplifica o registro para que você possa executar este exemplo com mais rapidez.

## Pré-requisitos
* Baixe o [Xcode](https://developer.apple.com/xcode/downloads/) da Apple.

* A instalação de [CocoaPods](https://guides.cocoapods.org/using/using-cocoapods.html) como um gerenciador de dependências.
* Uma conta de email comercial ou pessoal da Microsoft como o Office 365, ou outlook.com, hotmail.com, etc. Inscreva-se para uma [Assinatura de Desenvolvedor do Office 365](https://aka.ms/devprogramsignup), que inclui os recursos necessários para começar a criação de aplicativos do Office 365.

     > Observação: Caso já tenha uma assinatura, o link anterior direcionará você para uma página com a mensagem *Não é possível adicioná-la à sua conta atual*. Nesse caso, use uma conta de sua assinatura atual do Office 365.    
* Uma ID de cliente do aplicativo registrado no [Portal de Registro de Aplicativos do Microsoft Graph](https://apps.dev.microsoft.com)
* Para realizar solicitações de autenticação, é necessário fornecer um **MSAuthenticationProvider** para autenticar solicitações HTTPS com um token de portador OAuth 2.0 apropriado. Usaremos [msgraph-sdk-ios-nxoauth2-adapter](https://github.com/microsoftgraph/msgraph-sdk-ios-nxoauth2-adapter) para uma implementação de exemplo de MSAuthenticationProvider que pode ser usado para iniciar rapidamente o projeto. Confira a seção **Código de Interesse** abaixo para obter mais informações.


## Executando este exemplo em Xcode

1. Clonar este repositório
2. Se não estiver instalado, execute os seguintes comandos do aplicativo **Terminal** para instalar e configurar o gerenciador de dependências do CocoaPods.

		sudo gem install cocoapods
	
		pod setup

2. Use o CocoaPods para importar as dependências de autenticação e o SDK do Microsoft Graph:

		pod 'MSGraphSDK'
		pod 'MSGraphSDK-NXOAuth2Adapter'


 Este aplicativo de exemplo já contém um podfile que colocará os pods no projeto. Basta acessar a raiz do projeto em que o podfile está armazenado e no **Terminal** executar:

        pod install

   Para saber mais, confira o artigo **Usando o CocoaPods** em [Recursos adicionais](#AdditionalResources)

3. Abra **ios-objectivec-sample.xcworkspace**
4. Abra **AuthenticationConstants.m**. Observe que você pode adicionar o valor de **ClientID** do processo de registro, na parte superior do arquivo.:

   ```objectivec
        // You will set your application's clientId
        NSString * const kClientId    = @"ENTER_YOUR_CLIENT_ID";
   ```


    Os escopos de permissão a seguir foram configurados para esse projeto: 

```@"https://graph.microsoft.com/User.Read, https://graph.microsoft.com/Mail.ReadWrite, https://graph.microsoft.com/Mail.Send, https://graph.microsoft.com/Files.ReadWrite"```
    

    
>Observação: As chamadas de serviço usadas neste projeto, que enviam emails para sua conta de email, carregam imagens para o OneDrive e recuperam algumas informações de perfil (nome de exibição, endereço de email, imagem do perfil) exigem essas permissões para que o aplicativo seja executado corretamente.

5. Execute o exemplo. Você será solicitado a conectar/autenticar uma conta de email comercial ou pessoal e, em seguida, poderá enviar um email a essa conta ou a outra conta de email selecionada.


## Código de Interesse

Todo código de autenticação pode ser visualizado no arquivo **AuthenticationProvider.m**. Usamos um exemplo de implementação do MSAuthenticationProvider estendida do [NXOAuth2Client](https://github.com/nxtbgthng/OAuth2Client) para oferecer suporte a logon de aplicativos nativos registrados, atualizações automáticas de tokens de acesso e funcionalidade de logout:

```objectivec

		[[NXOAuth2AuthenticationProvider sharedAuthProvider] loginWithViewController:nil completion:^(NSError *error) {
    		if (!error) {
        	[MSGraphClient setAuthenticationProvider:[NXOAuth2AuthenticationProvider sharedAuthProvider]];
        	self.client = [MSGraphClient client];
   			 }
		}];
```

Depois que o provedor de autenticação estiver definido, podemos criar e inicializar um objeto de cliente (MSGraphClient) que será usado para fazer chamadas no ponto de extremidade do serviço do Microsoft Graph (email e usuários). Em **SendMailViewcontroller.m**, é possível obter a foto do perfil do usuário, carregá-la no OneDrive, montar uma solicitação de email com anexo de imagem e enviá-la usando o seguinte código:

### Obter a imagem do perfil do usuário

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
### Carregar imagem no OneDrive

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
### Adicionar anexo de imagem a uma nova mensagem de email

```objectivec
   MSGraphFileAttachment *fileAttachment= [[MSGraphFileAttachment alloc]init];
    fileAttachment.oDataType = @"#microsoft.graph.fileAttachment";
    fileAttachment.contentType = @"image/png";
    
    NSString *decodedString = [UIImagePNGRepresentation(self.userPicture) base64EncodedStringWithOptions:NSDataBase64EncodingEndLineWithCarriageReturn];
    
    fileAttachment.contentBytes = decodedString;
    fileAttachment.name = @"me.png";
    message.attachments = [message.attachments arrayByAddingObject:(fileAttachment)];
```

### Enviar a mensagem de email

```objectivec
    MSGraphUserSendMailRequestBuilder *requestBuilder = [[self.client me]sendMailWithMessage:message saveToSentItems:true];    
    MSGraphUserSendMailRequest *mailRequest = [requestBuilder request];   
    [mailRequest executeWithCompletion:^(NSDictionary *response, NSError *error) {      
    }];
```

Para obter mais informações, incluindo código para chamar outros serviços, como o OneDrive, confira o [SDK do Microsoft Graph para iOS](https://github.com/microsoftgraph/msgraph-sdk-ios)

## Perguntas e comentários

Gostaríamos de saber sua opinião sobre o projeto de conexão com o Office 365 para iOS usando o Microsoft Graph. Você pode enviar perguntas e sugestões na seção [Problemas](https://github.com/microsoftgraph/iOS-objectivec-connect-sample/issues) deste repositório.

Faça a postagem de perguntas sobre desenvolvimento do Office 365 em geral na página do [Stack Overflow](http://stackoverflow.com/questions/tagged/Office365+API). Não deixe de marcar as perguntas ou comentários com \[Office365] e \[MicrosoftGraph].

## Colaboração
Assine o [Contributor License Agreement (Contrato de Licença de Colaborador)](https://cla.microsoft.com/) antes de enviar a solicitação pull. Para concluir o CLA (Contributor License Agreement), você deve enviar uma solicitação através do formulário e assinar eletronicamente o CLA quando receber o email com o link para o documento.

Este projeto adotou o [Código de Conduta de Código Aberto da Microsoft](https://opensource.microsoft.com/codeofconduct/).  Para saber mais, confira as [Perguntas frequentes sobre o Código de Conduta](https://opensource.microsoft.com/codeofconduct/faq/) ou entre em contato pelo [opencode@microsoft.com](mailto:opencode@microsoft.com) se tiver outras dúvidas ou comentários.

## Recursos adicionais

* [Centro de Desenvolvimento do Office](http://dev.office.com/)
* [Página de visão geral do Microsoft Graph](https://graph.microsoft.io)
* [Usando o CocoaPods](https://guides.cocoapods.org/using/using-cocoapods.html)

## Direitos autorais
Copyright © 2016 Microsoft. Todos os direitos reservados.
