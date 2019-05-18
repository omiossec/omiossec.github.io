---
layout: post
title: Azure Functions V2 PowerShell, premier pas avec Vscode
image: /img/azure-function-powershell.png
author: Olivier Miossec
tags: [PowerShell, Azure, DevOps, Azure Function]
---

Depuis peu, Azure Functions V2 supporte PowerShell en preview. Il est donc possible de construire des applications serverless en PowerShell. Ces applications sont plus orientées Ops, on peut parler d’Event Based ServerLess Automation.

Qu’est-ce qu’Azure Functions ? Azure Fuction est composé de deux éléments, une fonction APP, similaire à une Web APP, c’est l’endroit et le contexte où s’exécutera notre code et une ou plusieurs fonctions qui sont nos applications serverless. 


Comment travailler sur Azure Functions v2 et PowerShell à partir de Vscode ? 

Pour ce poste, je propose de créer une Azure Function qui nous renvois la date et l’heure suivant une timezone donnée (Europe, UK, Pacific time, Asie, …). Elle doit être déclencher par un call http avec comme paramètre le code de la timezone.

Pour pouvoir développer et tester localement son code pour Azure Function, il est nécessaire d’installer un certain nombre d’outils. 

Pour pouvoir développer et tester localement son code pour Azure Function, il est nécessaire d’installer un certain nombre d’outils. Et pour faciliter les choses, il est préférable d’installer le package manager [Chocolatey](https://chocolatey.org/).

En premier lieu, si ce n’est pas le cas, **PowerShell Core 6.2** 

```powershell
choco install powershell-core 
````

L’intérêt d’installer PowerShell core à partir de Chocolatey est de pouvoir plus facilement faire les mises à jour. 

```powershell
choco upgrade powershell-core 
````

Les outils pour tester Azure Functions nécessite aussi la **SDK .net Core 2.2**

```powershell
choco install dotnetcore-sdk
````

Un peu étonnant, il faut aussi installer **node.js**, non pas que nous allons devoir faire du Javascript, mais pour utiliser son package manager, **npm**, les outils de Test d’Azure Functions s’installe grâce à cela.

```powershell
choco install nodejs-lts
````

La version LTS (10.15) permet d’utiliser d’autres outils gérer par Microsoft (comme AutoRest).

On peut maintenant installer les outils Azure Functions Core (à noter le @2 pour installer les outils de la V2).

```javascript
npm install -g azure-functions-core-tools@2
````
edit @TylerLeonhardt 
On peut aussi installer Azure Functions Core tools avec Chocolatey
```powershell
choco install azure-functions-core-tools
````


Azure Function v2 utilise PowerShell core et le module Azure Az, il faut installer ce dernier avec PowerShell Core 

```powershell
install-module -name AZ 
````
Pour aller plus loin il faut une souscription Azure. Si vous n’en n’avez pas, vous pouvez en avoir une [ici](https://azure.microsoft.com/en-us/free/)

Maintenant que nous avons préparé l’environnement local il faut préparer la souscription Azure. Il nous faut un groupe de ressource et un compte de storage. La localisation est importante, elle doit être la même pour l’ensemble des ressources que nous allons déployer dans Azure.

Avec PowerShell Core

```powershell
New-AzResourceGroup -Name functionlab -Location northeurope
New-AzStorageAccount -ResourceGroupName functionlab -Name functionlab01xx001 -location northeurope
````
On peut maintenant passer à la configuration de VSCode. 

Si ce n’est pas déjà fait il faut installer l’extension PowerShell. 

Pour cela il faut cliquer sur l’icône Extension puis rechercher PowerShell et cliquer sur le bouton Install

Il faut faire de même avec Azure Functions. Attention si vous avez déjà installer l’extension, il faut la mettre à jour faute de quoi pas de support de PowerShell !

Il faut configurer le comportement de l’extension Azure Functions.  Par défaut, la création d’une fonction se fait sans demander de groupe de ressource ou d’autres éléments pourtant importants.

Pour changer cela, un [Ctrl] + une , (virgule) permet  d’atteindre les Seetings de Vscode. 

Sélectionner Extensions puis Azure Functions et cocher la case « Advanced Creation »

![Seetings VsCode](/img/functionvscode/AzureFunctionsExtentions.PNG)


Il faut se loger sur Azure depuis VsCode. [Ctrl] [Shift] p pour ouvrir la palette de commande 

Taper Azure, et sélectionner "Signin with Azure Device Code" 

Vscode ouvre une boite de dialogue avec un code qu’il faut renseigner dans https://microsoft.com/devicelogin

Il faut ensuite ouvrir le panneau d’extensions Azure, [Ctrl] [Shift] a

Dans Azure Functions cliquer sur l’icône Create new Functions

![Create New Function](/img/functionvscode/newfunctionv2.png)

on selectionne un dossier sur le disque local

![Select folder](/img/functionvscode/SelectFolder.png)

Le wizzard nous demande de choisir le langage de la fonction, PowerShell

![Select runtime](/img/functionvscode/runtime.PNG)

Il faut maintenant choisir notre déclancheur, un HTTP trigger

![Select triger](/img/functionvscode/SelectTrigger.PNG)


On donne un nom à la fonction que nous alons mettre en place, ici GetPowerShellTime. 
Le nom d'une fonction ne doit être unique que dans le contexte de l'APP, contrairement au nom de la fonction APP qui doit être unique globalement.

![Function name](/img/functionvscode/functionname.PNG)

et on clique sur Enter pour valider

On sélectionne, ensuite, le mode d'autorisation de la fonction, Function c'est à dire par une clé d’API envoyer dans la requête, Admin, par la clé Master qui donne aussi accès à l’administration de la fonction (/admin). Pour notre exemple, function

![authorization levels](/img/functionvscode/auth.PNG)


Le wizzard va créer une function localement. Cela comprend l'ensemble des fichiers de la fonction (run.ps1 et function.json) et aussi la définition de la Function APP (host.json, profile.ps1, ...)

![Functions Files](/img/functionvscode/showExplorerPan.PNG)

Le fichier "run.ps1" dans le dossier GetPowerShellTime est le point d'entrée de la fonction et function.json définit le comportement de la fonction et de ses déclancheurs. 

Première chose à faire, par défault le wizzard ajoute les deux verbe HTTP GET et POST. 
On doit changer ce comportement pour qu'il n'accepte que le verbe GET. 

Il faut donc ouvrir function.json et supprimer "POST" dans oe champs methods. 

Le but de notre fonction est de pouvoir récuperer la date et l'heure suivant une timezone. Pour ce faire il faut changer le code de notre fonction qui se trouve dans run.ps1

```powershell
using namespace System.Net


param($Request, $TriggerMetadata)



$TimeZone = $Request.Query.TimeZone


$ValideTimeZone = @("CET","RET","PR","ET","GMT","WIT","UTC")


$WebServiceApi = "http://worldclockapi.com/api/json/$($TimeZone)/now"


$WorlClockJsonResult = Invoke-RestMethod -Method Get -UseBasicParsing -Uri $WebServiceApi


if ($TimeZone -in $ValideTimeZone) {
    $HttpStatus = [HttpStatusCode]::OK
    $RestultReponse = [datetime]::parseexact($WorlClockJsonResult.currentDateTime, 'yyyy-MM-ddTHH:mm+ss:ff', $null)
}
else {
    $HttpStatus = [HttpStatusCode]::BadRequest
    $RestultReponse = "Not a valide time zone name, please use CET,RET,PR,ET,GMT WIT UTC only ! "
}


$HttpResponse = [HttpResponseContext]@{
    StatusCode = $HttpStatus
    Body = $RestultReponse
}


Push-OutputBinding -Name Response -Value $HttpResponse
```

Nous pouvons maintenant debugger localement notre fonction.

Première étape, il faut vérifier que PowerShell est bien associé avec VsCode 
[Ctrl] + p et PoweShell : Show Session Menu et sélectionner swith to: PowerShell Core

Il faut aussi ajouter dans notre code un **wait-debugger** dans notre code (par exemple avant Push-OutputBinding**).

On peut maintenant lancer F5 pour lancer la fonction localement

![debug fonction](/img/functionvscode/debug.PNG)

Dès que les tests sont positifs, l’on peut publier la fonction. 
Il faut avant tout supprimer wait-debugger, sinon nous aurons un petit souci lors de la mise en production de la fonction. 

Notre fonction n’existe que sur notre poste local, il faut la publier pour cela.
[Ctlr] + A pour ouvrir l’extension Azure. 

Il faut se rendre, ensuite, sur le panneau Azure Functions  et cliquer sur Deploy Function

![Deploy fonction](/img/functionvscode/deploy.PNG)

Il faut ensuite sélectionner la souscription où l’on souhaite créer l’APP et la fonction

![Subscription](/img/functionvscode/selectsub.PNG)

On clique sur  create New Function App in Azure

![App Nom](/img/functionvscode/name.PNG)

On doit donner un nom à notre APP, attention le nom doit être globalement unique.

Le wizzard demande si vous l’on veut avoir une APP sous Linux ou Windows. Pour une fonction en PowerShell il faut choisir Windows.

Pour le hosting plan, on prend Consumption plan, car c’est le plus économique et le plus adapté, l’on ne payer que pour la consomation de ressource lorsque la fonction est utilisée. 

Enfin pour le runtime sélectionner PowerShell

![Runtime](/img/functionvscode/runtime.PNG)

Il faut ensuite sélectionner le groupe de ressources qui va héberger notre app, FunctionLab que nous avons mis en place plus tôt

![Rg](/img/functionvscode/rg.PNG)

Pour finaliser le wizzard on sélectionne le storage account créé plus tôt.

Notre app devrait prendre quelques minutes pour se déployer 

![Rg](/img/functionvscode/deploy2Azure.PNG)

Une fois notre APP et notre fonction sont déployées on peut commencer à l’utiliser. 

Mais il faut se souvenir que lors de la création de la fonction, on a sélectionné une authentification au niveau de la fonction. Donc notre fonction n’est pas disponible sans avoir une clé. 

Ce code est la Function Keys nommé défault. On peut retrouver le code à partir du portail Azure ou à partir de l’API de management d’Azure functions (ici https://functionappname.azurewebsites.net/admin/functions/GetPowerShellTime/keys).

Il est aussi possible d’obtenir l’url complète avec la keys à partir de vscode.

[Ctrl] + A pour ouvrir les extensions Azure. 

On déploie le nom de notre Function App, puis Functions (Read Only) et enfin on sélectionne le nom de notre fonction. 

![Vscode Function](/img/functionvscode/functionapp.PNG)

En faisant un clic droit sur le nom de notre fonction un menu nous propose de copier l’url de notre fonction avec en paramètre le code d’authentification.

```powershell
$AzureFunctionTimeUrl = "https://FUNCTIONAPPNAME.azurewebsites.net/api/GetPowerShellTime?code=3A1sRPFM1IDRvpJSc3j4xmXl1NPq5c%2F0e2Aox4fFKn%2FbPTVEALElcw%3D%3D&TimeZone=CET"

Invoke-RestMethod -Method get -Uri $AzureFunctionTimeUrl
```

Voilà notre premier épisode sur Azure Functions V2 et PowerShell Core. D’autres article suivront bientôt.