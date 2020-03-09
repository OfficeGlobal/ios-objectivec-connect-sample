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
# Muestra de conexión de Office 365 para iOS con el SDK de Microsoft Graph

Microsoft Graph es un punto de conexión unificado para tener acceso a los datos, relaciones e información procedente de la nube de Microsoft. En esta muestra le enseña cómo conectarse, autenticar y llamar a las API de usuario y correo a través del [SDK de Microsoft Graph para iOS](https://github.com/microsoftgraph/msgraph-sdk-ios).

> Nota: Consulte la página del [Portal de registro de la aplicación de Microsoft Graph](https://apps.dev.microsoft.com) que simplifica el registro para poder conseguir que esta muestra se ejecute más rápidamente.

## Requisitos previos
* Descargue el [Xcode](https://developer.apple.com/xcode/downloads/) de Apple.

* Instalar [CocoaPods](https://guides.cocoapods.org/using/using-cocoapods.html) como administrador de dependencias.
* Una cuenta de correo electrónico personal o profesional de Microsoft como Office 365, outlook.com, hotmail.com, etc. Puede registrarse para obtener [una suscripción a Office 365 Developer](https://aka.ms/devprogramsignup) que incluye los recursos que necesita para comenzar a crear aplicaciones de Office 365.

     > Nota: Si ya tiene una suscripción, el vínculo anterior lo dirigirá a una página con el mensaje *No se puede agregar a su cuenta actual*. En ese caso, use una cuenta con su suscripción actual a Office 365.    
* La Id. de cliente de la aplicación registrada en el [Portal de registro de la aplicación de Microsoft Graph](https://apps.dev.microsoft.com)
* Para realizar solicitudes, se debe proporcionar un **MSAuthenticationProvider** capaz de autenticar solicitudes de HTTPS con un token de portador OAuth 2.0 adecuado. Usaremos [msgraph-sdk-ios-nxoauth2-adapter](https://github.com/microsoftgraph/msgraph-sdk-ios-nxoauth2-adapter) para una implementación de muestra de MSAuthenticationProvider con la cual puede usarse para poner en marcha el proyecto. Consulte la sección **Código de interés** para ver más información.


## Ejecutar esta muestra en Xcode

1. Clonar este repositorio
2. Si no está instalado ya, ejecute los comandos siguientes de la aplicación **terminal** para instalar y configurar el administrador de dependencias de CocoaPods.

		instalar cocoapods con sudo gem
	
		configuración de Pod

2. Use CocoaPods para importar el SDK de Microsoft Graph y las dependencias de autenticación:

		Pod de 'MSGraphSDK'
		Pod de 'MSGraphSDK-NXOAuth2Adapter'


 Esta aplicación de muestra ya contiene un podfile que recibirá los pods en el proyecto. Solo tiene que ir a la raíz del proyecto donde esté el podfile y ejecutar desde la **terminal**:

        instalación de pod

   Para más información, vea **usar CocoaPods** en [recursos adicionales](#AdditionalResources)

3. Crear **ios-objectivec-sample.xcworkspace**
4. Crear **AuthenticationConstants.m**. Verá que el **ClientID** del proceso de registro puede ser agregado a la parte superior del archivo:

   ```objectivec
        // You will set your application's clientId
        NSString * const kClientId    = @"ENTER_YOUR_CLIENT_ID";
   ```


    Verá que los siguientes ámbitos de permiso han sido configurados para este proyecto: 

```@"https://graph.microsoft.com/User.Read, https://graph.microsoft.com/Mail.ReadWrite, https://graph.microsoft.com/Mail.Send, https://graph.microsoft.com/Files.ReadWrite"```
    

    
>Nota: Las llamadas de servicio usadas en este proyecto, el envío de un mensaje a su cuenta de correo, la carga de una imagen a OneDrive y la recuperación de parte de la información de perfil (nombre para mostrar, dirección de correo electrónico, imagen de perfil) requieren estos permisos para que la aplicación se ejecute correctamente.

5. Ejecutar la muestra. Deberá conectarse a una cuenta de correo personal o profesional, o autenticarlas, y, después, puede enviar un correo a esa cuenta, o a otra cuenta de correo electrónico seleccionada.


## Código de interés

Todos los códigos de autenticación pueden ser visualizados en el archivo **AuthenticationProvider.m**. Usamos una implementación de muestra de MSAuthenticationProvider procedente de [NXOAuth2Client](https://github.com/nxtbgthng/OAuth2Client) para proporcionar compatibilidad de inicio de sesión a aplicaciones nativas registradas, actualización automática de tokens de acceso y funcionalidad de cierre de sesión:

```objectivec

		[[NXOAuth2AuthenticationProvider sharedAuthProvider] loginWithViewController:nil completion:^(NSError *error) {
    		if (!error) {
        	[MSGraphClient setAuthenticationProvider:[NXOAuth2AuthenticationProvider sharedAuthProvider]];
        	self.client = [MSGraphClient client];
   			 }
		}];
```

Una vez se defina el proveedor de autenticación, podemos crear e inicializar un objeto de cliente (MSGraphClient) que se usará para realizar llamadas en el punto de conexión del servicio de Microsoft Graph (correo y usuarios). En **SendMailViewcontroller. m** podemos obtener la imagen de perfil del usuario, cargarla a OneDrive, montar una solicitud de correo electrónico con datos adjuntos de la imagen y enviarla con el código siguiente:

### Obtener la imagen de perfil del usuario

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
### Cargar la imagen a OneDrive

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
### Agregar una imagen adjunta a un nuevo mensaje de correo electrónico

```objectivec
   MSGraphFileAttachment *fileAttachment= [[MSGraphFileAttachment alloc]init];
    fileAttachment.oDataType = @"#microsoft.graph.fileAttachment";
    fileAttachment.contentType = @"image/png";
    
    NSString *decodedString = [UIImagePNGRepresentation(self.userPicture) base64EncodedStringWithOptions:NSDataBase64EncodingEndLineWithCarriageReturn];
    
    fileAttachment.contentBytes = decodedString;
    fileAttachment.name = @"me.png";
    message.attachments = [message.attachments arrayByAddingObject:(fileAttachment)];
```

### Enviar el mensaje de correo electrónico

```objectivec
    MSGraphUserSendMailRequestBuilder *requestBuilder = [[self.client me]sendMailWithMessage:message saveToSentItems:true];    
    MSGraphUserSendMailRequest *mailRequest = [requestBuilder request];   
    [mailRequest executeWithCompletion:^(NSDictionary *response, NSError *error) {      
    }];
```

Para más información, incluyendo el código para llamar a otros servicios, como OneDrive, vea el [GDK de Microsoft Graph para iOS](https://github.com/microsoftgraph/msgraph-sdk-ios)

## Preguntas y comentarios

Nos encantaría recibir sus comentarios sobre el proyecto Connect de Office 365 para iOS con Microsoft Graph. Puede enviarnos sus preguntas y sugerencias a través de la sección [Problemas](https://github.com/microsoftgraph/iOS-objectivec-connect-sample/issues) de este repositorio.

Las preguntas generales sobre desarrollo en Office 365 deben publicarse en [Stack Overflow](http://stackoverflow.com/questions/tagged/Office365+API). Asegúrese de que sus preguntas o comentarios se etiquetan con \[Office365] y \[MicrosoftGraph].

## Colaboradores
Deberá firmar un [Contrato de licencia de colaborador](https://cla.microsoft.com/) antes de enviar la solicitud de incorporación de cambios. Para completar el Contrato de licencia de colaborador (CLA), deberá enviar una solicitud mediante un formulario y, después, firmar electrónicamente el CLA cuando reciba el correo electrónico que contiene el vínculo al documento.

Este proyecto ha adoptado el [Código de conducta de código abierto de Microsoft](https://opensource.microsoft.com/codeofconduct/). Para obtener más información, vea [Preguntas frecuentes sobre el código de conducta](https://opensource.microsoft.com/codeofconduct/faq/) o póngase en contacto con [opencode@microsoft.com](mailto:opencode@microsoft.com) si tiene otras preguntas o comentarios.

## Recursos adicionales

* [Centro para desarrolladores de Office](http://dev.office.com/)
* [Página de información general de Microsoft Graph](https://graph.microsoft.io)
* [Usar CocoaPods](https://guides.cocoapods.org/using/using-cocoapods.html)

## Copyright
Copyright (c) 2016 Microsoft. Todos los derechos reservados.
