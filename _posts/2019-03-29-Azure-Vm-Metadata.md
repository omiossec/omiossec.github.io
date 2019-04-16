---
layout: post
title: Azure VM Metadata
image: /img/hello_world.jpeg
author: Olivier Miossec
tags: [Azure, IaaS, DevOps, Automation]
---

Comment peut-on facilement retrouver les infos d’une instance de VM lancé sur Azure ? Comment connaitre son IP Publique (elle change à chaque démarrage de l’instance si elle est dynamique), son ressource group, sa zone ou son ID ?

En dehors de la VM c’est assez simple. Quelque commande PowerShell et vous pouvez trouver ces informations. Mais depuis l’intérieur de la VM, comment faire ?

Comme tous les providers Cloud, du moins les plus important d’entre eux, Azure dispose d’un service de MetaData. Il permet de facilement retrouver ce type de données depuis une VM qui a été créer sur Azure (mais uniquement si la vm a été créée via ARM).

Il se base sur un service web accessible depuis une IP non routable, 169.254.169.254, et disponible uniquement depuis une VM Azure.

http://169.254.169.254/metadata/

Pour accéder aux différents services, il est nécessaire d’ajouter une en-tête http supplémentaire Metadata:true à la requête http. Cette configuration permet à Azure de s’assurer de la légitimité de la requête. 

L’API Metadata d’Azure dispose de trois services : 

 * Instance, qui permet de récupérer des informations sur l’instance, comme le Resource group, la taille de la vm mais aussi la configuration réseau des ou de la carte réseau.

* ScheduledEvent, qui donne la liste des évènements de maintenance, comme les reboot prévus pour l’instance. Plusieurs évènements sont pris en compte par l’API ScheduledEvent, Freeze, Reboot, Redeploy. Ces evenements sont planifiés pour au minimum 15 minutes dans le futur. Enfin un dernier service.
Il est n’est pas activé par défaut contrairement à instance, il est nécessaire d’effectuer une première requête.  
Contrairement à instance, qui n’accepte que la méthode GET, il est possible d’envoyer un POST pour valider un évènement apres en avoir pris connaissance et avoir fait les maintenances nécessaires.
Enfin dernier point important il faut ajouter dans la chaine de requête, la version de l’API à utiliser. La version de l’API est différente entre Instance et ScheduledEvent

* Attested, qui permet de renvoyer une version signée des données pour s’assurer que celles-ci viennent bien d’Azure et non d’un autre système. Attested n’est disponible qu’avec la version 2018-10-01 de l’API.

La version courante pour Instance est 2018-10-01 et pour ScheduledEvent, 2017-08-01, pour Attested il n’y a pour le moment qu’une seule version de l’API, 2018-10-01.

http://169.254.169.254/metadata/[Instance|ScheduledEvent|Attested]?api-version=[2018-10-01|2017-08-01]


Comment utiliser ces trois services avec PowerShell ?

Nous avons vus qu’il est nécessaire dans tous les cas de rajouter l’entête http metadata :True pour que le call http soit valide.
Une entête http en PowerShell se matérialise sous la forme d’un hastable.


```powershell
$MetaDataHeaders = @{"Metadata"="true"}
```

Pour lancer la requête en PowerShell Core et Desktop on peut utiliser Invoke-RestMethod ou Invoke-WebRequest. 

```powershell
$MetaDataServiceResult = Invoke-RestMethod -Method GET -uri "http://169.254.169.254/metadata/instance?api-version=2018-10-01" -Headers $MetaDataHeaders
```

L’API renvois un JSON dont le schéma est divisé en 2, Compute pour donner des informations sur l’instance et Network pour ce qui est attaché aux cartes réseau de la VM. 

Le schéma de la partie compute est assez simple, il ne comprend pratiquement que des objets String. On y retrouve le nom et l’Id de la VM. Le schéma peut changer avec la version de l’API.

Il est possible de ne récupérer que les informations de Compute en ajoutant compute dans l’URI de l’API

"http://169.254.169.254/metadata/instance/compute?api-version=2018-10-01"


```json
"compute": {
    "azEnvironment": "AZUREPUBLICCLOUD ou AzureGouvernemant",
    "location": "Location de la VM",
    "name": "Nom de la Vm",
    "offer": "",
    "osType": "",
    "placementGroupId": "",
    "plan": {
        "name": "",
        "product": "",
        "publisher": ""
    },
    "platformFaultDomain": "",
    "platformUpdateDomain": "",
    "provider": "Microsoft.Compute",
    "publicKeys": [],
    "publisher": "MicrosoftWindowsDesktop",
    "resourceGroupName": "Resource Group",
    "sku": " ",
    "subscriptionId": "xxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx",
    "tags": "xxx:yyy;eee:tt",
    "version": "17134.345.59",
    "vmId": "13f56399-bd52-4150-9748-7190aae1ff21",
    "vmScaleSetName": "",
    "vmSize": "Standard_D1",
    "zone": "1"
  } 
```
Avec Invoke-RestMethod, l’objet JSON est retranscrit dans un PSCustomObject. 

Les données réseaux renvoyés par la partie Network l’API dépend du nombre de cartes réseau attaché à la VM. C’est un tableau JSON.

```json
"network":{
    "interface":
        [
            {
                "ipv4":
                    {
                        "ipAddress":
                        [   
                            {
                                "privateIpAddress":"172.24.20.4",
                                "publicIpAddress":"23.97.148.85"
                            }
                        ],
                    "subnet":
                        [
                            {
                                "address":"172.24.20.0",
                                "prefix":"29"
                            }
                        ]
                    },
                    "ipv6":
                        {
                            "ipAddress":[]
                    },
                    
                "macAddress":"000D3A46897A"
            }         
        ]
    }
```

On y retrouve pour chaque interface, le subnet (mais pas son nom ni le nom du Vnet), l’adresse MAC, l’ip privée et si elle existe l’IP publique. 

Comme pour Compute il est possible de changer l’URI pour n’avoir que les informations sur le réseau.

"http://169.254.169.254/metadata/instance/network?api-version=2018-10-01"


Les concepts derrière ScheduledEvent, sont un peu plus compliqué à saisir. Le but n’est pas seulement de récupérer des informations, mais aussi de les acquitter.

Un Scheduled Event permet de disposer du temps nécessaire pour préparer un système en cas de maintenance d’Azure. Cela peut être le déplacement d’un nœud du cluster ou l’enregistrement de données. 

Les évènements pris en compte sont les suivant : 

* Maintenance sur l’hyperviseur (update ou hardware)
* Redéploiement
* Scale in dans un scale set

Pour vérifier si un évènement a été planifié à partir d’une VM Azure:

```powershell
$MetaDataHeaders = @{"Metadata"="true"}
$MetaDataServiceResult = Invoke-RestMethod -Method GET -uri "http://169.254.169.254/metadata/scheduledevents?api-version=2017-11-01" -Headers $MetaDataHeaders
```

L’API renvois un json (et donc un PSCustomObject) : 

```json
{
    "DocumentIncarnation": "IncarnationID",
    "Events": [
        {
            "EventId": "eventID",
            "EventType": "Reboot" | "Redeploy" | "Freeze" | "Preempt",
            "ResourceType": "VirtualMachine",
            "Resources": "Nom de la ressource",
            "EventStatus": "Scheduled" | "Started",
            "NotBefore": "Heure au format UTC",
        }
    ]
}
```

Les éléments à prendre en compte sont surtout l’EventID, qui sert à envoyer un acquittement à l’API, EnventType qui permet de savoir ce qu’il va se passer sur la VM et NotBefore qui permet d’avoir le nombre de minute minimum avant que l’opération ne soit déclenchée. Ce temps est de 10 minutes dans le cas d’un Redeploy, 30 secondes pour un Preempt (suppression d’un scale set) et de 15 minutes pour les autres événements.

Autre point important, La propriété Resource indique l’ensemble des VM qui peuvent être impacté par la maintenance (dans le cas d’un Update Domain).

Une fois la logique applicative en place, il est possible d’envoyer un acquittement. Attention un acquittement ne bloque ou ne retarde pas l’évènement, il indique juste à la fabrique Azure qu’il peut procéder à l’action.

Pour envoyer un acquittement, il faut effectuer un POST http vers l’API avec comme donnée l’ID (ou les ID) de l’Event. 

Il est nécessaire de pouvoir construire le JSON à envoyer à l’API. 

```powershell
$JsonResponse = @{"StartRequests" = @(@{"EventId" = "xxxxxxx"})} | ConvertTo-Json 

Invoke-RestMethod -Method Post -Uri http://169.254.169.254/metadata/scheduledevents?api-version=2017-11-01 -Headers $MetaDataHeaders -Body $JsonResponse
```

Encore une fois, ce n’est pas une méthode pour annuler une maintenance planifiée, C’est la possibilité d’indiquer à Azure que le nécessaire a été fait. 

Le dernier service Metadata d’Azure, Attested, permet de s’assurer de la valider des données reçues. Il renvoi une signature.

```powershell
$MetaDataHeaders = @{"Metadata"="true"}
$MetaDataServiceResult = Invoke-RestMethod -Method GET -uri "http://169.254.169.254/metadata/attested/document?api-version=2018-10-01" -Headers $MetaDataHeaders
```

Le résultat retourné est un objet contenant une signature 

encoding  : pkcs7
signature : MIILtgYJKoZIhvcNAQcCoIILpzCCC6MCAQExDzANBgkqhkiG9w0BAQsFADCB4wYJKoZIhvcNAQcBoIHVBIHSeyJub25jZSI6IjIwMTkwMzI5LTA2NTMxMyIsInBsYW4iOnsibmFtZSI6IiIsInByb2R1Y3QiOiIiLCJwdWJsaXNoZXIiOiIifSwidGltZVN0YW….


La signature peut être vérifiée avec les assemblies System.Security.Cryptography. Cela permet à ceux qui veulent publier un service sur la Marketplace Azure de vérifier que la VM tourne bien sur Azure et non sur un cloud privé.