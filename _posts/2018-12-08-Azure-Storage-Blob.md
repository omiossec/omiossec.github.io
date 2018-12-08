---
layout: post
title: Azure Storage, uploader des fichiers vers un Blob avec PowerShell
image: /img/hello_world.jpeg
author: Olivier Miossec
tags: [PowerShell, Azure, Storage]
---


Les options de stockage de données dans le Cloud sont multiples. Sur Azure nous avons à disposition plusieurs services pour stocker des données suivant ses propres besoins. Parmi ces services existe le Compte de Stockage ou Storage Account.  

Un Storage Account permet d’avoir accès à un Namespace et des EndPoint pour stocker des données en mode objets, fichier, table ou sous forme de queue. 

Le stockage objet, ou Blob, est un moyen pratique de stocker des données pouvant être disponible partout et non seulement d’une station de travail. Les données sont accessibles depuis en Endpoint en HTTPS (il est possible de l’avoir en http) et permet de stocker un large volume de données par un prix raisonnable. Il permet aussi de déclencher des actions avec les services serverless comme Azure Fonction. 

Pour créer un stockage Blob en PowerShell, il faut dans un premier temps créer un Storage Account.

```PowerShell
$AzRessourceGroup = "DemoStorage"
$AzstorageAccount = "testomarchive001"
$AzLocation = "FranceCentral"
$AzStorageType = "Standard_LRS"
$AzBlobContainerName = "imagesarchive"

$AzStorage = New-AzureRmStorageAccount -ResourceGroupName $AzRessourceGroup -Name $AzstorageAccount -Type $AzStorageType -Location $AzLocation -Kind StorageV2 -EnableHttpsTrafficOnly $true
````

L’utilisation d’une variable pour stocker le retour de la commande permet d’avoir plus facilement accès par la suite au Context qui nous permet de gérer le stockage.

Il est nécessaire d’indiquer le type d’account par le paramêtre Kind avec StorageV2 sans cela la version par défaut est la V1 ce qui limite les possibilités, le paramètre Type permet d’indiquer le type de redondance et enfin EnableHttpsTrafficOnly permet de forcer le trafic en TLS.

Attention enfin, le nom du Storage Account doit être unique et être composé uniquement de lettre en minuscule ou de chiffre.

La gestion du stockage objet se fait au moyen de containers blobs qu’il faut créer avec un nom unique dans le Storage Account

```PowerShell
New-AzureStorageContainer -Name $AzBlobContainerName -Context $AzStorage.Context -Permission off | out-null 
````

Mettre les permissions en Off permet de faire en sorte que l’accès ne soit pas public. 

Pour le moment nous avons fait nos opérations de création à partir d’une connexion Azure. Mais si l’on veut utiliser un container blob dans un environnement moins sécurisé il n’est pas possible d’utiliser un compte Azure. Cela peut être le cas lorsque la machine qui souhaite manipuler les fichiers n’est pas totalement contrôlée ou que l’on doit automatiser certaines tâches. 

Azure dispose d’un moyen simple et sécurisé de donner accès aux ressources à partir d’une URI et pour un temps limité. Le SAS Token. 

La création du SAS Token ce fait à partir du Context du Storage Account et du nom du container. Il faut aussi donner une date d’expiration du token, cela peut être quelques minutes ou plusieurs mois.

```PowerShell
$azSasToken = New-AzureStorageContainerSASToken -container $AzBlobContainerName -Permission rwdl -ExpiryTime (get-date).AddMonths(1) -Context $AzStorage.Context

$azSasStorageContext =  New-AzureStorageContext $AzstorageAccount -SasToken $azSasToken 
````

Avec cette clé et le nom du compte de Storage il est maintenant possible d’uploader des données depuis n’importe quelle machine. 

Avec cette clé et le nom du Storage Account il est maintenant possible d’uploader des données depuis n’importe quelle machine. 

Il faut dans un premier temps créer le Context à partir de la clé SAS et du nom du Storage Account. 

```PowerShell
$azSasStorageContext =  New-AzureStorageContext $AzstorageAccount -SasToken $azSasToken 
```` 

Le Storage Account indique la liste des endpoints et les chaînes de connexion, le SAS Token permet d’avoir l’autorisation d’utilisation. Ce token représente la partie Query de l’URI qui sera utilisé pour manipuler les fichiers. 

```PowerShell
Set-AzureStorageBlobContent -File "locaux.jpg" -Container $AzBlobContainerName -Context $azSasStorageContext -Blob "team/locaux.jpg" -Force
````
Le cmdlet utilise le nom du container Blob et le token pour construire l’URI servant à envoyer le fichier.

A noter que même si en mode objet il n’y a pas de hiérarchie dans le même sens qu’un serveur de fichier Il est possible d’utiliser des dossiers en indiquant un chemin lors de l’upload.

