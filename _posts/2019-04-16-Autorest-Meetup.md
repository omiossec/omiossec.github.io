---
layout: post
title: AutoRest.PowerShell retour sur le Meetup du FRPSUG
image: /img/autorest/logo.png
author: Olivier Miossec
tags: [PowerShell, API, DevOps]
---

Ce poste fait suite à ma micro-session lors des PowerShell Lightning Demo du [French PowerShell User Group](https://frpsug.com/), et je voulais faire un retour plus complet. 

La présentation est disponible sur [Youtube](https://www.youtube.com/watch?v=xA4B4m_sZ7U).



Pour bien l’utilité d’[AutoRest](https://github.com/Azure/AutoRest), il faut bien comprendre le concept d’API. On pourait résumer une API à un contrat entre deux systèmes qui, sans cela, ne pourraient pas communiquer entre eux. Un contrat, donc, je donne A pour que tu exécute B et finalement pour me renvoyer C. Comme pour tout contrat, la qualité d’une API se mesure dans la définition du A, du C et du B. A, dans notre cas, est ce que l’on envois, les données et leur forme, mais aussi la façons dont cela est envoyé y compris l’authentification nécessaire, B est ce que l’on cherche à faire et enfin C c’est le résultat que l’on attend, avec un format précis et détaillé.

Dans le monde des API Rest, ce contrat peut se matérialiser sous la forme d’un fichier de spécification.  Il prend la forme d’un fichier yaml ou json et répond à la norme OpenAPI Specification (anciennement Swagger Specification). C’est un langage déclaratif qui permet de décrire de façon détaillé et non équivoque toutes les actions disponibles et les résultats attendus. 

C’est donc un document précieux qui permet qui permet de pouvoir écrire un client pour une API. 
Mais il n’en reste pas moins que le travail peut s’avérer compliqué, pour chaque action il faut aussi bien implémenter une fonction reprenant le format d’entrée (verbe http, format des données …) et écrire un parser suivant le résultat que l’on peut attendre.

C’est le but d’Autorest, de pouvoir s’affranchir de cette étape longue et génératrice d’erreur. Autorest est un générateur de code. Il est capable d’analyser un fichier de spécification et de créer un module (pour AutoRest.PowerShell). Cela évite de consacrer de l’énergie sur des dizaines de fonctions et permet de se concentrer sur la logique métier. 

Car c’est là que se trouve notre valeur ajoutée. Prenons le cas d’un système qui gère une application de réservation de table de café en terrasse à Paris. Lorsqu’il pleut ou qu’il neige le lendemain, pas la peine d’ajouter des instances à l’application, il faut en retirer, en revanche dès que les premiers rayons de soleil font leur apparition il faut démultiplier les instances pour absorber le choc du nombre de parisiens à la recherche d’un verre au soleil. 
Ce n’est pas la capacité à interroger les API de la météo qui sont le cœur du système, mais bien la capacité du système à scaller automatiquement suivant des paramètres qui importe.

Pour la présentation du module pour le French PowerShell User Group, j’ai fait le choix de ne pas utiliser les exemples fournis sur le repos d’Autorest.PowerShell, mais de partir de pratiquement 0 avec une des API de la Nasa. Celle du [CNEOS-JPL](https://cneos.jpl.nasa.gov/).

Première chose à noter, le module généré n’est pas au format psm1, il est écrit en CSharp. Cela peut être déroutant pour un non initié, mais cela permet aussi de pouvoir la mécanique interne. Et pour chaque objet présent dans le fichier de spécification, une classe est créée pour sérialiser les données. 

Autre point, le module n’est pas construit. AutoRest.PowerShell génère uniquement le code nécessaire pour construire le module. Il y ajoute les répertoires nécessaires pour étendre le module, un dossier test et des outils pour builder le module et pour la packager. 

L’ajout d’une fonction en PowerShell est assez simple. Il faut la placer dans le dossier custom avant le build et surtout il faut qu’il respecte les paramètres obligatoires que voici :

```powershell
[ValidateNotNull()]
    [frpsug.Runtime.SendAsyncStep[]]
    # SendAsync Pipeline Steps to be appended to the front of the pipeline
    ${HttpPipelineAppend},

    [Parameter(DontShow)]
    [ValidateNotNull()]
    [frpsug.Runtime.SendAsyncStep[]]
    # SendAsync Pipeline Steps to be prepended to the front of the pipeline
    ${HttpPipelinePrepend},

    [Parameter(DontShow)]
    [System.Uri]
    # The URI for the proxy server to use
    ${Proxy},

    [Parameter(DontShow)]
    [ValidateNotNull()]
    [System.Management.Automation.PSCredential]
    # Credentials for a proxy server to use for the remote call
    ${ProxyCredential},

    [Parameter(DontShow)]
    [System.Management.Automation.SwitchParameter]
    # Use the default credentials for the proxy
    ${ProxyUseDefaultCredentials}
```

L’un des problèmes que j’ai rencontré lors de mes tests avec les API de la Nasa était le manque de fichier de spécification. J’ai construit le mien petit à petit. Mais comme je l’ai dit plus haut, un fichier de spécification est un contrat, moins il est précis et moins les données renvoyées seront exploitables. Le module généré ne renvois alors que des objets génériques ou des erreurs.

```json

  EstimatedDiameterNumericSystem:
    properties:
      estimated_diameter_max:
        type: string
      estimated_diameter_min:
        type: string
    type: object

  EstimatedDiameter:
    properties:
      feet:
        $ref: '#/definitions/EstimatedDiameterNumericSystem'
      kilometers:
        $ref: '#/definitions/EstimatedDiameterNumericSystem'
      meters:
        $ref: '#/definitions/EstimatedDiameterNumericSystem'
      miles:
        $ref: '#/definitions/EstimatedDiameterNumericSystem'
    type: object



  Neoobject:
    properties:
      links:
        type: object
      neo_reference_id:
        type: string
      name:
        type: string
      designation:
        type: string
      nasa_jpl_url:
        type: string
      absolute_magnitude_h:
        type: string  
      estimated_diameter:
        $ref: "#/definitions/EstimatedDiameter"
      is_potentially_hazardous_asteroid: 
        type: boolean
      orbital_data: 
        $ref: "#/definitions/orbitaldata"
      is_sentry_object:
        type: boolean
    type: object
```

De plus, Autorest.PowerShell certaines sérialisation sont parfois en échec, il faut ouvrir une [Issue](https://github.com/azure/autorest.powershell/issues) et tenter une mise à jour. 

Enfin autre problème, l’authentification, comme les méthodes des API sont souvent pluriels (clé, token, header, …), il est compliqué pour un outil de pouvoir toutes les gérer. Autorest laisse à votre charge cette partie. Cela peut se faire en étendant la classe Module, dans module.cs. 

Vous pouvez consulter le code source [ici](https://github.com/omiossec/Presentations/tree/master/presentations/20190416-LightningDemos/autorest-OlivierMiossec)
Et revoir le meetup [ici](https://www.youtube.com/watch?v=xA4B4m_sZ7U)
