---
layout: post
title: Azure Functions V2 PowerShell, trigger et bindings
image: /img/azure-function-powershell.png
author: Olivier Miossec
tags: [PowerShell, Azure, DevOps, Azure Function]
---

Le support de PowerShell Core, la possibilité d’utiliser des identités managées Azure et l’ajout du module PowerShell AZ permettent d’utiliser Azure Functions V2 comme système d’automatisation évènementiel ou en anglais Event Based Serverless Automation.  
Pourquoi évènementiel ? car l’exécution du code est déclenchée par un évènement. Cela peut être un call http, l’ajout d’un objet dans un blob, d’un message dans une queue, ... c’est le déclencheur ou trigger. C’est ce qui va lancer l’exécution du code indépendamment de la nature et du contenue de ce déclencheur.  

Les déclencheurs possibles sont :  

* Blob 
* Queue 
* HTTP 
* Webhook 
* Cosmo DB 
* Event Grid 
* Event Hub 
* MS Graph Event 
* Servive Bus 
* Timer (ou scheduler) 

Un évènement sur l’un de ces composants déclenche l’exécution du code de la fonction. Il ne peut y avoir qu’un seul déclencheur par fonction. Si le code que l’on met en place doit pouvoir être déclencher par plus d’un type d’événement, il faut autant de fonction qu’il y a d’évènement. 

En plus des déclencheurs, il est possible d’utiliser une liaison ou binding pour gérer une connexion d’une ressource Azure à la fonction, que cela soit en entrée ou en sortie. Cela permet facilement d’accéder à cette ressource.  

|   Bindings	|   Direction	
|---	|---	
|  Blob 	|   	En entrée, en Sortie 
|  Queue 	|   	En Sortie  
|  Table 	|   	En entrée, en Sortie  
|  CosmoDb  	|   	En entrée, en Sortie   
|  Event Hubs  	|   	En Sortie  
|  Http 	|   	En Sortie  
|  Graph Excel  	|   	En entrée, en Sortie   
|  Graph Onedrive  	|   	En entrée, en Sortie  
|  Graph Email  	|   	En Sortie  
|  Graph Event  	|   	En entrée, en Sortie 
|  Notification Hubs  	|   	En Sortie  
|  SendGrid  	|   	En Sortie  
|  Service bus  	|   	En Sortie  
|  SignalR 	|   	En entrée, en Sortie  

Ces liaisons sont disponibles en paramètre lorsqu’elles sont en entrée et disponible en tant qu’objet de sortie pour le cmdlet Push-OutputBinding. 
La définition de ces paramètres s’effectue dans le fichier de configuration de la fonction ; fonction.json 

```json
{ 
  "bindings": [ 
    { 
      "authLevel": "function", 
     "type": "httpTrigger", 
      "direction": "in", 
      "name": "Request", 
      "methods": [ 
        "get", 
        "post" 
      ] 
    }, 
    { 
      "type": "http", 
      "direction": "out", 
      "name": "Response" 
    } 
  ] 
} 
```

Chaque trigger et bindings peuvent avoir leur propres spécificités. Suivant la famille du déclencheur il faut installer une extension (Microsoft.Azure.WebJobs.Extensions.xxx). Cela est fait automatiquement lors de la création du trigger. 

Passons-en revus les principaux, HTTP, Blob, Queue, table, Service Bus et EventGrid.

## HTTP

C’est le trigger sans doute le plus associé à l’idée que l’on peut se faire d’une Fonction. Ce trigger permet la création d’API serverless et de répondre à des GET ou des POST et autres verbes HTTP. 
Il dépend du package Microsoft.Azure.WebJobs.Extensions.Http 

Il prend la forme suivante dans function.json  

```json
{ 
    "authLevel": "function", 
    "name": "req",  
    "route": "", 
    "type": "httpTrigger", 
    "direction": "in", 
    "methods": [
         "get",
          "post" 
          ] 
} 
```

**AuthLevel** permet d’indiquer le niveau d’autorisation que l’on souhaite pour la fonction. Il faut comprendre qu’une AppFunction est une WebApp, elle est donc disponible sur Internet par défaut (il est possible tout de même de changer cela via Plateform Features et Access Restrictions au niveau de la FunctionApp. Il est aussi possible de changer le plan de la FunctionApp ce qui permet d’avoir plus de maitrise sur la partie Réseau. 

Les niveaux d’Authentification sont : 

* **Anonymous** : Aucune Authentification

* **Function** : Une clé est nécessaire pour accéder à la fonction, cette clé est liée à la fonction. Pour accéder à la fonction il faut donc rajouter le paramètre code avec cette clé. Il est aussi possible d’ajouter la clé dans les en-têtes http avec le champ x-function-key

* **Admin** : utilise la clé master. Cette clé permet l’accès à l’ensemble des fonctions de la FunctionApp

A noter cependant que pour gerer des API en production il est préférable d’utiliser Azure API management ou l’authentification via App Service Authentication (avec Azure AD, Twitter, Facebook, …)  

**route** : définie le template de l'url et la façon dont la fonction répond. Par exemple "products/{category:alpha}/{id:int?} qui définie le parametre category et id

**methods** : permet de limiter les verb http qui sont autorisés au niveau de la fonction, par défaut tous les verbes http sont permis. 

**direction** : c'est un trigger, donc in 

**name** : le nom du paramètre que l'on retrouve dans la fonction

Ce dernier est un objet de type HttpRequestContext 

Il dispose des propriétés  

* Body, un objet représentant le corp de la requête HTTP, permet la récupération des données provenant d’un POST http
* Headers Dictionnaire des entêtes de la requête 
* Method un string représentant le verbe Http utilisé (Get, Post, Put, …) 
* Params, Dictionnaire des paramètres de la requêtes  
* Query, un dictionnaire des paramètres de l’url lors d’un GET 
* Url, String représentant l’url utilisée lors de la requête 

Le trigger http n’a pas besoin d’avoir d’output Binding, mais dans ce cas la fonction renvois un code un code http 204 no content. 

Pour renvoyer un contenu il est nécessaire d’ajouter un output Binding.  

```json
{ 
"type": "http", 
"direction": "out", 
"name": "Response" 
} 
```

Name permet d’indiquer le nom du paramètre que l’on doit utiliser avec Push-OutoutBinding.  

L’objet devant être renvoyer doit être un objet .net HttpResponseContext provenant de System.web (à importer avec  _using system.web_).  

C’est un hastable qui peut contenir StatusCode (par default [HttpStatusCode]::OK),  on peut aussi ajouter Body mais aussi le type contenu renvoyer par la fonction ContentType (ex text/json) et enfin Header pour renvoyer un Hashtable représentant les en-tête que l’on souhaite rajouter. 

La valeur de _StatusCode_ est celle de la constance provenant de **WinHTPP constants**  

Pour le construire  
```powershell 
[HttpStatusCode]::OK  
```

Les constantes que l’on peut utiliser sont, lister [ici](https://docs.microsoft.com/en-us/windows/desktop/WinHttp/http-status-codes)

L’objet peut se présenter comme cela 

```powershell 
$HttpResponse = [HttpResponseContext]@{ 
StatusCode = $HttpStatus 
Body = $RestultReponse | ConvertTo-Json 
ContentType = "text/json" 
} 

Push-OutputBinding -Name Response -Value $HttpResponse 
```

## Trigger et Bindings liés à un compte de storage.  

Les éléments des comptes de storage (Storage Account) sont normalement associés à la fonction. Cela peut être celui de la functionApp ou d'une autre connexion associée. Le nom de la connexion doit être renseigné dans le champs connection de chaque Binding ou trigger dans function.json. 
Le fonctionnement dans Azure functions est assuré par  Microsoft.Azure.WebJobs.Extensions.Storage

## Queue 

Queue est simple à manipuler. L’élément queue n’est disponible qu’en trigger ou en sortie. 

```json 
"bindings": [ 
    { 
        "name": "QueueItem", 
        "type": "queueTrigger", 
        "direction": "in", 
        "queueName": "alertearmagedon", 
        "connection": "AzureWebJobsStorage" 
    }, 
    { 
        "type": "queue", 
        "name": "outputQueueItem", 
        "queueName": "vmalertequeue", 
        "connection": "AzureWebJobsStorage", 
        "direction": "out" 
    } 
] 
```

Le trigger déclenche l’exécution du code dès qu’un nouveau message arrive dans la queue et le message est transmis à la function par paramètre. Le nom du paramètre reprend le champ name de function.json. C’est un objet de type String.  

```powershell
param([string] $QueueItem, $TriggerMetadata) 
```

Il n’y a qu’une direction disponible lors d’un binding, out. Là aussi l’objet attendue par le cmdlet Push-OutputBinding est un String.  

```powershell
Push-OutputBinding -Name outputAlerteArmagedon -Value "Test" 
```

Si pour une raison ou pour une autre, la fonction n’arrive pas à traiter le message, la fonction est exécutée encore 4 fois. À la dernière tentative, le message est marqué avec –poison. 

## Table

Il n’existe pas de trigger pour la gestion des évènements sur une table d’un compte de Storage. Il n’est possible que d’utiliser en tant que sortie ou en entrée. 

En Sortie

```json
{ 
    "type": "table", 
    "name": "outputTable", 
    "tableName": "outTable", 
    "connection": "AzureWebJobsStorage", 
    "direction": "out" 
} 
```

La particularité d’une table est que l’objet devant être envoyer à Push-OutputBinding doit être un Hastable ou un tableau de Hastable.   

```powershell
$entity1 = @{ 
    partitionKey = 'testtable' 
    rowKey = (new-guid).guid 
    name = "Valeur 1" 
} 

$entity2 = @{ 
    partitionKey = 'testtable' 
    rowKey = (new-guid).guid 
    name = "Valeur 2" 
} 

$arrayOut = @($entity1,$entity2) 

Push-OutputBinding -Name outputTable -Value $arrayOut 
```

En entrée  

```json
  { 
      "type": "table", 
      "name": "inputTable", 
      "tableName": "inTable", 
      "take": 50, 
      "connection": "AzureWebJobsStorage", 
      "direction": "in", 
      "partitionKey": "tablein", 
      "rowKey": "1" 
    } 
```
Lors du déclanchement de la fonction, le runtime va effectuer une requête sur la table indiquée dans le champ tableName de storage associé à la fonction. Le champ take indique le nombre d’enregistrement à retourner, partitionKey le nom de la partition a interroger et RowKey la clé de la ligne à retourner. Seul le champ tablename est obligatoire. 

Le paramètre $InputTable dans cet exemple retourne un Tableau de Hastable correspondant aux lignes renvoyées.

## blob

Les blobs peuvent servir à la fois comme trigger et comme liaison en entrée ou en sortie.  

Pour un trigger blob, le déclanchement se fait à chaque ajout d’objet dans le container associé au trigger. Le contenu de l’objet est renvoyé à la fonction en paramètre.  

Comme trigger  
```json
    { 
      "name": "InputBlob", 
      "type": "blobTrigger", 
      "direction": "in", 
      "path": "triggerblob/{name}", 
      "connection": "AzureWebJobsStorage" 
    } 
```

Le champ name nous indique le nom du paramètre à utiliser dans run.ps1. Par défaut c’est un tableau d’objet, mais suivant le contenue nous pouvons aussi utiliser un string (cas d’un fichier json ou text), un Stream ou un objet textreader. Cela dépend des objets qui sont uploader sur le storage.  

Path représente le chemin où l’on doit chercher le fichier. Il est possible de filtrer les fichier par extension en utilisant {name}.json  

C’est le runtime qui s’assure que l’ajout d’un objet dans un container ne déclenche qu’une seule fois la fonction.  
Comme c’est le contenue du blog qui est renvoyé en paramètre, pour pouvoir avoir des informations sur le blob, il faut utiliser triggermetadata qui doit être passé en paramètre à la fonction.

Cela permet d’obtenir : 

* **BlobTrigger**, le chemin de l’objet  
* **Uri**, l’url de l’objet 
* **Properties**, un objet [Blobproperties](https://docs.microsoft.com/dotnet/api/microsoft.azure.storage.blob.blobproperties)  

En input la configuration est un peu la même sauf que la fonction n’est pas lancée lors de l’ajout d’un objet. Donc dans fonction.json il n’est pas possible d’utiliser le champ Path comme pour le trigger.  

Par contre il est possible de donner soit le chemin complet de l’objet, soit un pattern, soit encore le type de trigger de la fonction (par exemple, le message de la queue contient le nom du blob, il est possible d’utiliser le type dans Path, _"path": "samples-workitems/{queueTrigger}"_)  

En sortie  

```json
{ 
      "name": "InputBlob", 
      "type": "blobTrigger", 
      "direction": "in", 
      "path": "outputblob/{rand-guid}.json", 
      "connection": "AzureWebJobsStorage" 
} 
```
Il est possible de donner un nom random ou comme en entrée le type de données du trigger.  

Attention cependant, si la fonction est sensible au cold start ou si le nombre de fichier est important. Le cold Start peut rajouter un délai de plusieurs minutes avant que la fonction soit disponible et cela peut être tres frustrant (même en test!). De même si l’on doit traiter plusieurs dizaines d’ajouts par seconde, Il est préférable de passer par EventGrid pour gérer par Azure Functions l’ajout de Blob.   

## EventGrid 

EventGrid n’est disponible qu’en trigger. La configuration dans function.json  

```json
{ 
      "type": "eventGridTrigger", 
      "name": "eventAction", 
      "direction": "in" 
} 
```

L’important est de pouvoir créer une souscription   

![Event Grid Subscription](/img/bindings/gridsubscription.PNG)

Ici pour gérer les évènements au niveau d’un ressource group 

![Event Grid Subscription](/img/bindings/subscription.PNG)

Dans run.ps1, à partir de l’objet de type EventGridEvent $EventAction passé en paramètre il est possible de récupérer les propriétés de l’évènement.  

* **Subject**   le sujet de l'évenement
* **EventType**  le type d'evenement suivant la source
* **EventTime**  le datetime de l'evenement en UTC

```powershell
param($eventGridEvent, $TriggerMetadata)

$resourceId = $eventGridEvent.subject
$eventType = $eventGridEvent.eventType
$eventdate = $eventGridEvent.eventTime
```

Il y a plusieurs autres types de trigger et de binding possible. Mais il y a plusieurs à se rappeler avec PowerShell dans Azure Function v2. C’est pour le moment une preview et donc il est possible d’avoir des bug et des fonctionnalités limitées. La programmation est différente de ce qu’elle pourrait être dans d’autres langage comme C# et la documentation est assez limitée. 

Un point important aussi est que la durée de fonctionnement d’une fonction n’est que de 2 minutes 30. Même s'il est possible d’entendre cette limite cela n’est pas forcément la meilleure solution. Azure Functions est une architecture serverless. Il faut donc pouvoir architecturer le code pour qu’il soit faiblement dépendant et qu’une fonction ne fait qu’une chose précise et qu’elle passe l’action à une autre fonction. Il faut résister à l’idée de faire une fonction qui ferait tout. 