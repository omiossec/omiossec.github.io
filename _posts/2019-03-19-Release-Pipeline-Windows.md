---
layout: post
title: Release Pipeline Model et Windows
image: /img/hello_world.jpeg
author: Olivier Miossec
tags: [DevOps]
---

Je suis souvent confronté au manque d’enthousiasme des équipes IT traitant des serveurs et des services sous Microsoft aux concepts apportées par la Culture DevOps.

Les objections vont de c’est un truc pour Linux à ce n’est pas ITIL et donc c’est risqué voir même c’est un truc de Geek, mais le plus souvent, l’objection est ; c’est compliqué. 

C’est un truc pour Linux. DevOps est totalement agnostique, c’est avant tout un état d’esprit, une culture avant d’être quelque chose transposé dans un outil. Donc que l’on soit sous Linux ou sous Windows ne change rien. Il devrait même être possible de le faire avec Windows NT 4 (mais là, OK, c’est extrême).



Ce n’est pas ITIL et c’est donc risqué. ITIL, prend tout changement comme un risque sur la stabilité du système. Mais DevOps gère aussi les risques, en effectuant des changements par petits lots, rapidement et de façon contrôlée. C’est le binôme deploy/learn qui est important et qui permet de revenir en arrière tres rapidement si une défaillance est détectée. DevOps prend pour adage, *failure is always an option*.

D’ailleurs, la version 4 du framework ITSM incorpore plusieurs principes de la culture DevOps.

C’est un truc de Geek. C’est sans doute une remarque des plus amusante. DevOps ne serait qu’un truc pour adolescents attardés jouant au Nerf. Le fait que des entreprises comme KPMG, et une large partie des banques (surtout en UK) ont adopté DevOps devrait nous faire croire le contraire. Un banquier en costume gris et austère n’a rien d’un Geek. On ne joue pas au Nerf à la City.

C’est compliqué, la le point est marqué. DevOps, surtout sous Windows, est compliqué. Non seulement cela implique un changement de culture et de mindset dans les équipes, toutes les équipes, pas seulement IT, mais cela demande aussi un effort d’apprentissage.
Car cela oblige à apprendre de nouvelles techniques, de nouveaux concepts et surtout changer la façon d’interagir avec les autres. Il faut sortir bien loin de sa zone de confort. 

Sous Linux, un changement de configuration se fait par la modification d’un ou plusieurs fichiers. Pour renseigner le DN racine sur OpenLdap, il suffit de modifier le fichier /etc/openldap/slapd.conf et de lancer le service. 
Dès lors il est assez simple d’avoir ce fichier dans un repos Git et de pouvoir le déployer sur chaque nouvelle machine dans un modèle de Release Pipeline. 

Sous Windows, avec Active Directory, rien de semblable. Impossible de n’avoir qu’un fichier de config pour changer la configuration d’un système. C’est l’une des différences fondamentales entre Linux et Windows. La configuration sous Windows est le plus souvent basée sur des API. 
Ces API sont surtout utilisées pour le traitement des interfaces graphiques. Un modèle de Release Pipeline est donc forcément plus complexe.


C’est la seconde difficulté sous Windows, l’omniprésence de l’interface graphique fait que nous manageons les serveurs sous Windows de la même manière que nous traitons nos postes de travail. 

Les changements de configuration sont pratiqués de façon plus ou moins manuel et les tests suivent les mêmes schémas. 
Cela est souvent source d’erreur et pour mitiger les risques on a tendance à gerer les changements par large lot. 


Les tests sont généralement manuels et trop souvent la recette est réalisée avec l’aide des outils de monitoring, c’est-à-dire une fois que toutes les opérations sont terminées.

Cela implique, aussi, que les opérateurs disposent d’un accès administrateur sur une longue période voir plus généralement de façon permanente.   

Nous sommes loin d’un modèle de Release Pipeline. Est-il possible ? Oui mais pas sans effort. 
Et cela est plus que souhaitable. Le modèle de Delivry du Cloud nous l’impose. Comment gérer des ressources dans le Cloud de manière scalable et répétable en effectuant ce travail manuellement ?

Il y a un besoin de changer le modèle, ce modèle est Release Pipeline Model
On pourait le résumer à : 

**Gestion du code source**

Dans un repos Git. Cela permet d’identifier qui, quand et pourquoi un changement a été fait.
C’est aussi la source unique de vérité de la plateforme.

**Gestion du Build**

Permet la construction et la gestion des artefacts du déploiement

**Gestion des tests**

Gestion des tests unitaires et des tests d’acceptance pour le déploiement. 

**Gestion des releases**

Le déploiement en lui-même, mais aussi les tests automatiques de validation


Quels sont les outils à la disposition d’une équipe Windows pour mettre en œuvre cela. 

Pour la gestion des sources, nous avons GitHub, Azure DevOps ou BitBucket et tout autre repos basé sur Git.
Pour la gestion des Pipelines de Build, Test et Release, Azure DevOps, AppVeyor.
Il faut aussi prendre en compte le module PowerShell PSake pour construire les scripts de build.

Pour la gestion des déploiements : 
Coté Azure nous avons ARM Templates ou Cloud Formation coté Aws. 
Pour le déploiement sous Windows, nous avons PowerShell mais aussi Chef.

Pour le déploiement des applications, PowerShell peut être utile tout comme Chef Habitat ou Octopus Deploy.

Pour la gestion de la compliance et des tests, Pester est sans doute l’outil le plus important, mais l’on peut aussi associer Operation Validation Framework, Test Kitchen et Chef InSpec.

Ce panorama n’est pas exhaustif, mais c’est sans doute sous Windows le package essentiel pour débuter le Release Pipeline Model.

