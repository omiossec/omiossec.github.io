---
layout: post
title: Azure Blueprint
image: /img/hello_world.jpeg
author: Olivier Miossec
tags: [Azure, Gouvernance, PowerShell, DevOps]
---

Lorsque j’étais au lycée, je devais faire du dessin technique. C’était des dessins détaillés avec l’ensemble des cotations et des usinages à faire. Ce dessin, était l’étape essentielle qui décrivait la pièce que l’on devait faire et qui déterminait la façon dont on aller l’usiner. Cette étape, les anglosaxons la nomme blueprints (leurs dessins techniques sont sur fond bleu).

C’est un peu comme cela que doit être compris le nouveau service Azure Blueprints. Permettre aux architectes de créer une ossature de gouvernance de manière déclarative. C’est la possibilité de pouvoir apporter des ressources comme des Rôles, des politiques, des templates ARM ou des groupes de ressource de manière répétable sur un environnement.   

Surtout, Blueprint permet d’être versionner et d’être inclus dans un pipeline CI/CD.

Pour pouvoir commencer à utiliser Azure Blueprints il faut avant tout un compte capable de manager la souscription, il faut que ce compte puisse avoir le privilège de gérer les ressources Azure (Access Management for Azure Ressources). 
Un autre point important est qu’il est nécessaire de disposer d’au moins un groupe d’administration Azure (management group https://docs.microsoft.com/en-us/azure/governance/management-groups/index) 


Pour démarrer avec Azure Blueprint deux options sont possibles, l’utilisation du portail ou par Rest API (il n'y a pas encore de Module PowerShell ou de Commande Az).

Pour créer un premier modèle blueprint depuis le portail, il faut aller dans « All Services » et rechercher BluePrints.

![image-center](/img/blueprint/blueprints1.PNG)

Pour commencer, cliquer sur "create" 

Il faut donner un nom (unique pour le tennant) et sélectionner le groupe d’administration.

![image-center](/img/blueprint/blueprints2.PNG)

Il faut ensuite rajouter les artefacts, à savoir : 
* Les ressources groupes (appliqué uniquement aux souscriptions) 
* La définition de roles (appliqué aux ressources groupes et au souscriptions)
* La définition des Template ARM (appliqué aux ressources groupes et au souscriptions)
* La définition des policies (appliqué aux ressources groupes et au souscriptions)


![image-center](/img/blueprint/blueprints3.PNG)

Ici je définis un canevas de règles qui indique que pour les groupes de ressource en France Central 
je dois appliquer une policy qui audit si les VM sont bien configurées en disaster recover et que les ressources disposent bien d’un tag.

Par défaut la définition est sauvegardée en mode brouillons, on peut y revenir sans devoir effectuer une nouvelle version. 

Pour passer le blueprint en production, il est nécessaire de le publier. Le processus est assez simple, il suffit de donner un numéro de version (sous la forme X.X.X ou vX) avec une note de publication.

Une fois publié, le blueprint peut être modifié, mais le numéro de version doit être incrémenté.


Une fois publié, le blueprint peut être assigné à une souscription. Il est nécessaire de choisir un nom pour l’opération, le lieu où seront stocker les données du blueprint (ces données sont stockées dans une base CosmoDB, il est possible de stocker les données n’importe où et enfin la version que l’on souhaite appliquer. Il faut aussi choisir si l’on souhaiter locker les ressources, si oui, les ressources ne pourront pas être supprimées ou modifiées.

Le portail présente alors la liste des données à moddifier.


![image-center](/img/blueprint/blueprints4.PNG)

Le portail n’est pas la seule possibilité pour créer ou gérer un blueprint. Il est possible de passer directement par les API Azure. 
La tache est plus complexe car il n’y a pas, encore, d’outil en PowerShell.

Pour commencer il faut gerer la connexion au portail 

```PowerShell
$azContext = Get-AzContext
$azLocalProfile = [Microsoft.Azure.Commands.Common.Authentication.Abstractions.AzureRmProfileProvider]::Instance.Profile

$AzClientProfile= New-Object -TypeName Microsoft.Azure.Commands.ResourceManager.Common.RMProfileClient -ArgumentList ($azLocalProfile)

$AzToken = $profileClient.AcquireAccessToken($azContext.Subscription.TenantId)

$authHeader = @{
    'Content-Type'='application/json'
    'Authorization'='Bearer ' + $ AzToken.AccessToken
 }
``` 

A partir de là il est possible de créer une URI pour lister les Blueprint qui sont déjà en place 

```PowerShell

$azManagementGroupName = 'myMgtGrp'
$BluePrintRestApiUri = "https://management.azure.com/providers/Microsoft.Management/managementGroups/$($ azManagementGroupName)/providers/Microsoft.Blueprint/blueprints?api-version=2017-11-11-preview"


$JsonBluePrintResult = Invoke-RestMethod -Uri $BluePrintRestApiUri -Method Get -Headers $authHeader

```
L'opération retourne un JSON listant les blueprints avec leur nom (celui que l'on voit dans le portail) et leur ID (en fait une URI : )

```PowerShell
@{properties=; id=/providers/Microsoft.Management/managementGroups/rootomiossec/providers/Microsoft.Blueprint/blueprints/testblueprint01; type=Microsoft.Blueprint/blueprints;name=testblueprint01}
```

Pour la plupart des actions il faut pouvoir maitriser la documentation de l’API qui est encore en preview.
La construction est complexe, mais l’utilisation direct de l’API est rapide et surtout cela permet d’aller plus loin. 
Cela fera l’objet d’un nouveau billet. 

