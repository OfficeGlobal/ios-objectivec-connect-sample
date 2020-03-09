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
# Exemple de connexion d’Office 365 pour iOS avec le kit de développement logiciel (SDK) Microsoft Graph

Microsoft Graph est un point de terminaison unifié pour accéder aux données, aux relations et aux informations fournies à partir du cloud Microsoft. Cet exemple présente comment s'y connecter et s’authentifier, puis appeler les API de messagerie et utilisateur via le [Kit de développement logiciel Microsoft Graph (SDK) pour iOS](https://github.com/microsoftgraph/msgraph-sdk-ios).

> Remarque : Consultez la page relative au [Portail d’inscription de l’application Microsoft Graph](https://apps.dev.microsoft.com) pour enregistrer plus facilement votre application et exécuter plus rapidement cet exemple.

## Conditions préalables
* Téléchargement de [Xcode d’Apple](https://developer.apple.com/xcode/downloads/).

* Installation de [CocoaPods](https://guides.cocoapods.org/using/using-cocoapods.html) comme gestionnaire de dépendances.
* Un compte de messagerie professionnel ou personnel Microsoft comme Office 365 ou outlook.com, hotmail.com, etc. Vous pouvez vous inscrire à [Office 365 Developer](https://aka.ms/devprogramsignup) pour accéder aux ressources dont vous avez besoin pour commencer à créer des applications Office 365.

     > Remarque : Si vous avez déjà un abonnement, le lien précédent vous renvoie vers une page avec le message suivant : *Désolé, vous ne pouvez pas ajouter ceci à votre compte existant*. Dans ce cas, utilisez un compte lié à votre abonnement Office 365 existant.    
* Un ID client de l’application enregistrée auprès du [Portail d’inscription de l’application Microsoft Graph](https://apps.dev.microsoft.com)
* Pour effectuer des requêtes, vous devez fournir un élément **MSAuthenticationProvider** capable d’authentifier les requêtes HTTPS avec un jeton de support OAuth 2.0 approprié. Nous allons utiliser [msgraph-sdk-ios-nxoauth2-adapter](https://github.com/microsoftgraph/msgraph-sdk-ios-nxoauth2-adapter) pour un exemple d’implémentation de MSAuthenticationProvider qui peut être utilisé pour commencer rapidement votre projet. Voir la section **Code d’intérêt** ci-dessous pour plus d’informations.


## Exécution de cet exemple dans Xcode

1. Cloner ce référentiel
2. S’il n’est pas déjà installé, exécutez les commandes suivantes à partir de l’application **Terminal** à installer et configurez le gestionnaire de dépendances CocoaPods.

		sudo gem install cocoapods
	
		pod setup

2. Utilisez CocoaPods pour importer les dépendances d’authentification et le kit de développement logiciel Microsoft Graph :

		pod 'MSGraphSDK'
		pod 'MSGraphSDK-NXOAuth2Adapter'


 Cet exemple d’application contient déjà un podfile qui recevra les pods dans le projet. Accédez à la racine du projet où se trouve le podfile et, à partir de **Terminal**, exécutez la commande suivante :

        pod install

   Pour plus d’informations, consultez **Utilisation de CocoaPods** dans [Ressources supplémentaires](#AdditionalResources)

3. Ouvrez **ios-objectivec-sample.xcworkspace**
4. Ouvrez **AuthenticationConstants.m**. Vous constatez que l’**ClientID** du processus d’inscription peut être ajouté à la partie supérieure du fichier :

   ```objectivec
        // You will set your application's clientId
        NSString * const kClientId    = @"ENTER_YOUR_CLIENT_ID";
   ```


    Notez que les étendues d’autorisation suivantes ont été configurées pour ce projet : 

```@"https://graph.microsoft.com/User.Read, https://graph.microsoft.com/Mail.ReadWrite, https://graph.microsoft.com/Mail.Send, https://graph.microsoft.com/Files.ReadWrite"```
    

    
>Remarque : Les appels de service utilisés dans ce projet, l’envoi d’un courrier électronique à votre compte de messagerie, le chargement d’une image vers OneDrive et la récupération des informations de profil (nom d’affichage, adresse e-mail, photo de profil) ont besoin de ces autorisations pour que l’application s’exécute correctement.

5. Exécutez l’exemple. Vous êtes invité à vous connecter/authentifier à un compte de messagerie personnel ou professionnel, puis vous pouvez envoyer un message à ce compte ou à un autre compte de messagerie sélectionné.


## Code d’intérêt

Tout code d’authentification peut être affiché dans le fichier **AuthenticationProvider.m**. Nous utilisons un exemple d’implémentation de MSAuthenticationProvider étendu de [NXOAuth2Client](https://github.com/nxtbgthng/OAuth2Client) pour prendre en charge la connexion des applications natives inscrites, l’actualisation automatique des jetons d’accès et la fonctionnalité de déconnexion :

```objectivec

		[[NXOAuth2AuthenticationProvider sharedAuthProvider] loginWithViewController:nil completion:^(NSError *error) {
    		if (!error) {
        	[MSGraphClient setAuthenticationProvider:[NXOAuth2AuthenticationProvider sharedAuthProvider]];
        	self.client = [MSGraphClient client];
   			 }
		}];
```

Une fois le fournisseur d’authentification défini, nous pouvons créer et initialiser un objet client (MSGraphClient) qui sert à effectuer des appels par rapport au point de terminaison du service Microsoft Graph (courrier et utilisateurs). Dans **SendMailViewcontroller.m** nous pouvons obtenir l’image de profil de l’utilisateur, la télécharger sur OneDrive, regrouper une demande de message électronique avec une image en pièce jointe et l’envoyer à l’aide du code suivant :

### Obtention de l’image de profil de l’utilisateur

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
### Chargement de l’image vers OneDrive

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
### Ajouter une image en pièce jointe à un nouveau message électronique

```objectivec
   MSGraphFileAttachment *fileAttachment= [[MSGraphFileAttachment alloc]init];
    fileAttachment.oDataType = @"#microsoft.graph.fileAttachment";
    fileAttachment.contentType = @"image/png";
    
    NSString *decodedString = [UIImagePNGRepresentation(self.userPicture) base64EncodedStringWithOptions:NSDataBase64EncodingEndLineWithCarriageReturn];
    
    fileAttachment.contentBytes = decodedString;
    fileAttachment.name = @"me.png";
    message.attachments = [message.attachments arrayByAddingObject:(fileAttachment)];
```

### Envoi du message électronique

```objectivec
    MSGraphUserSendMailRequestBuilder *requestBuilder = [[self.client me]sendMailWithMessage:message saveToSentItems:true];    
    MSGraphUserSendMailRequest *mailRequest = [requestBuilder request];   
    [mailRequest executeWithCompletion:^(NSDictionary *response, NSError *error) {      
    }];
```

Pour plus d’informations, y compris sur le code d’appel à d’autres services, tels que OneDrive, reportez-vous à la section [Kit de développement logiciel Microsoft Graph pour iOS](https://github.com/microsoftgraph/msgraph-sdk-ios).

## Questions et commentaires

Nous serions ravis de connaître votre opinion sur le projet de connexion d’iOS à Office 365 avec Microsoft Graph. Vous pouvez nous faire part de vos questions et suggestions dans la rubrique [Problèmes](https://github.com/microsoftgraph/iOS-objectivec-connect-sample/issues) de ce référentiel.

Si vous avez des questions sur le développement d’Office 365, envoyez-les sur [Stack Overflow](http://stackoverflow.com/questions/tagged/Office365+API). Assurez-vous de poser vos questions ou de rédiger vos commentaires avec les tags \[MicrosoftGraph] et \[Office 365].

## Contribution
Vous devrez signer un [Contrat de licence de contributeur](https://cla.microsoft.com/) avant d’envoyer votre requête de tirage. Pour compléter le Contrat de licence de contributeur (CLA), vous devez envoyer une requête en remplissant le formulaire, puis signer électroniquement le CLA lorsque vous recevez le courrier électronique contenant le lien vers le document.

Ce projet a adopté le [Code de conduite Open Source de Microsoft](https://opensource.microsoft.com/codeofconduct/). Pour en savoir plus, reportez-vous à la [FAQ relative au code de conduite](https://opensource.microsoft.com/codeofconduct/faq/) ou contactez [opencode@microsoft.com](mailto:opencode@microsoft.com) pour toute question ou tout commentaire.

## Ressources supplémentaires

* [Centre de développement Office](http://dev.office.com/)
* [Page de présentation de Microsoft Graph](https://graph.microsoft.io)
* [Utilisation de CocoaPods](https://guides.cocoapods.org/using/using-cocoapods.html)

## Copyright
Copyright (c) 2016 Microsoft. Tous droits réservés.
