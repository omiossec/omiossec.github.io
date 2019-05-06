---
layout: post
title: PowerShell Core dans Azure Function V2, une intro
image: /img/azure-function-powershell.png
author: Olivier Miossec
tags: [PowerShell, Azure, DevOps, Azure Function]
---

Azure vient d’annoncer le support de PowerShell Core dans Azure Function v2. Ce n’est plus, comme en version 1, un langage expérimental, mais bien un mode « Preview », donc avec une forte probabilité d’avoir un GA et en support complet à plus ou moins brève échéance.

Azure Function est l’offre Serverless ou Function as a Service (FaaS), d’Azure. Serverless est un design patern qui permet à une équipe de développement de déployer une architecture faiblement interdépendante en se basant uniquement sur des composants de code. 
C’est l’une des solutions qui existe, avec AWS Lambda et Cloud Function. Le fonctionnement est simple. Un évènement déclenche l’exécution du code. Cet événement peut être un call http, mais aussi lors de l’ajout d’un élément dans un container blob, une queue ou une base de données, ou tout autre chose.


Le serverless computing est souvent à la base des nouveaux services sur Internet et est tres populaire parmi les développeurs. C’est sa facilité de mise en œuvre et de sa modularité qui veut cela. 

Est-ce à dire que PowerShell devient un langage de programmation Web ? En fait non, ou du moins pas vraiment. Faire une architecture web à base de PowerShell est possible, mais il manque à PowerShell les librairies nécessaires à la gestion d’un projet de ce type. D’autres langage seront sans doute mieux adaptés. Le vrai cas d’usage de PowerShell est l’**automatisation Serverless**.  Ou plus exactement **automatisation événementielle serverless**. 
Elle permet d’élaborer des scénarios complexe d’automatisation qui serait compliqué à mettre en œuvre sans cela.

Imaginons les scénarios suivants : 

* Une application qui doit déployer une base de données et une nouvelle vm à la suite d’une requête d’un client 
* Lancer une VM pour traiter un fichier qui arrive de façon aléatoire 
* Effectuer un classement de fichier venant d’être uploader sur un storage blob
* Relancer un service lorsqu’un évènement est reçut sur EventGrid

Voila des cas d’usage pour PowerShell Core dans Azure Function. On pourait parler de gestion évènementielle des ressources.

En quoi consiste Azure Function. C’est, avant tout, une App Azure. Elle gère les ressources mis à disposition pour exécuter du code. C’est le container qui permet d’héberger les fonctions. C’est aussi un runtime qui permet à du code écrit dans un langage donné de pouvoir s’exécuter dans cette App. Enfin ce sont des triggers (Queue, http, Blob, CosmoDb, …) qui déclenchent l’exécution du code et des liaisons qui permettent d’accéder aux données du déclencher et gerer l’output. 

Pour résumé, Azure fonction est une App Azure qui permet l’exécution d’une ou plusieurs fonctions. Chacune de ces fonctions est déclenchée par un évènement et des liaisons avec des points d’entrées et de sorties sont fournis. 

En termes de code, Azure fonction se présente comme une hiérarchie de fichiers. 

A la racine se trouve le fichier host.json, il définit la fonction, un fichier profile.ps1 est aussi présent, il est exécuter à chaque démarrage de l’App, il permet par exemple de gérer une connexion Azure ou de définir des variables ou des fonctions qui seront disponible pour l’ensemble des Azure Functions de l’App.  Il est nécessaire aussi d’avoir un fichier local.settings.json, il définit le runtime, ou langage,  de l’App dans « Functions_Worker_Runtime ».

Il est aussi possible d’avoir requirements.psd1, il permet d’importer les modules qui sont nécessaire. Il est utilisé par défaut pour importer les modules Azure AZ.*. 

Mais pour cela il faut que le fichier host.json active les dépendances avec le mot clé ManagedDependency. 

C’est aussi à ce niveau que l’on peut ajouter un dossier modules. Ce dossier est ainsi disponible pour l’ensemble des fonctions de l’App. Comme il n’est pas possible d’utiliser install-module pendant l’exécution de la fonction (et sans doute que cela prendrait un peu trop temps sur la durée de vie de la fonction). Il est possible d’utiliser save-module pour les modules de la Powershell Gallery. Cependant il faudra gérer d’une autre manière les mises à jour de ces modules. C’est aussi là que l’on peut déposer ses propres modules. 
Les modules dans ce dossier sont automatiquement importés lors de l’exécution du code. 


Un troisième fichier est optionnel, sample.dat, il permet de donner un exemple, pour le panneau test de l’UI. 

Function.json et le fichier qui définit le comportement de la fonction. En premier lieu il gère les liaisons (bindings) avec les points d’entrée (http, storage, queue, …) et les points de sortie (http, storage, …) 

Cette définition a un schéma précis 

Name : Cela correspond au nom du paramètre que l’on retrouve dans notre script

Type : le type de données (http, Queue, Blob …) et si c’est un trigger ou non (BlobTrigger, httpTrigger, …)

Direction : In ou Out, données en entrées ou en sorties

Ensuite des éléments qui sont propre à chaque type de données (Connexion, nom du Container pour le storage Blob, méthode pour les données http, …).

Les données définie dans functions.json sont disponible lors de l’exécution de run.ps1. Pour les données, dont la direction est in dans function.json, elles sont disponible en tant que paramètres du fichier run.ps1. Leur nom correspond au nom (name) de la données dans function.json. Pour les données, dont la direction est out, elles doivent être ajouter au paramettre name du cmdlet de sortie, correspondant au nom de ou des liasons du fichier function.json.

De plus à chaque call du run.ps1, le paramètre TriggerMetadata est ajouté. Il n’est pas obligatoire mais il permet d’obtenir des informations sur le déclencheur. Les données dépendent du type de trigger. 

Ce paramètre dispose aussi d’une propriété sys avec plusieurs valeurs : 

* utcNow : un objet datetime représentant la date et l’heure du déclanchement de la fonction 
* MethodName : le nom de la fonction déclenchée
* RandGuid : l’identifiant unique de l’exécution 


fichier function.json 

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
  ],
  "disabled": false
}
```

Fichier run.ps1

```powershell
param($Request, $TriggerMetadata)
```

Le Bindings dont la direction est out, permet de gerer la sortie de la fonction, que cela soit une réponse http, une écriture sur une queue ou un storage, ou une autre sortie disponible avec Azure Function.

Là encore le mot clé name dans function.json mais cette fois avec la direction out permet d’identifier vers où les données doivent être envoyées. 

Il est possible d’avoir plusieurs outputs et donc plusieurs liaisons. 

Pour gerer l’output, il faut utiliser le cmdlet **Push-OutputBinding**. Il prend deux paramètres, name, un string représentant le nom de la liason, value, de type object, pour les données devant être transmise  

```powershell
Push-OutputBinding -Name Response -Value ([HttpResponseContext]@{
    StatusCode = [HttpStatusCode]::OK
    Body = "Bonjour From PowerShell Core in Azure Function V2"
})
```
Il est possible de voir les données qui ont été renvoyées en utilisant, **Get-OutputBinding**.

_Cet article est une introduction à Azure Function V2 avec PowerShell Core. Il y a bien d’autres chose à voir. Il fait partie d’une série sur PowerShell Core dans Azure Function V2. D’autres sont en préparation avec des exemples plus proches de la réalité et de la production._
* _un exemple complet_
* _CI/CD avec Azure Function_
* _..._