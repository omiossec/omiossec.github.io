---
layout: post
title: DORA, State Of DevOps 2018
image: /img/hello_world.jpeg
author: Olivier Miossec
tags: [ DevOps, DORA]
---

DevOps Research and Assessment (DORA) édite chaque année, depuis 5 ans, une étude sur le l’état du monde DevOps. 

Cette étude compile plus de 30000 sondages sur près de 1900 professionnels dans le monde. Il propose un index, **Software Delivery Performance (ou SDP)** basé sur les fréquences de déploiement, les temps pour appliquer des changements, le temps de restauration d’un service, le taux d’erreur lors des changements et la disponibilité. Le rapport y ajoute aussi la disponibilité.

Cet index est alors comparé aux performances des entreprises non seulement en terme IT mais aussi en terme business. 
L’index SDP et la mesure de la disponibilité permet de prédire les performances des organisations.
Le rapport établit une échelle des organisations :

* Elite Performer
* High Performer
* Medium Performer 
* Low Performer 


Cette échelle prend en compte aussi bien les performances lors des déploiements que les performances business. 

Le rapport est disponible [ici](https://cloudplatformonline.com/2018-state-of-devops-typ.html) 

Que peut-on y apprendre ?

L’adoption de la culture DevOps et l’efficacité dans les opérations dans la mise en production des logiciels permet de meilleures performances en termes de profit et de satisfaction client. 

Le rapport trace quelques pratiques influant sur les performances

L’adoption du Cloud Computing est un facteur de succès à condition que cela respecte les 5 caractéristiques essentielles du Cloud (Self-Service, disponible partout, élastique, mesuré et multi tenant).

Cette définition convient aussi bien au Cloud Public qu’à certain service de cloud privée (s’ils respectent ces conditions).

Quel sont les conditions nécessaires du Cloud Computing selon le rapport DORA ?

* Self-Services, les ressources doivent être directement disponible sans interaction humaine
* Réseau public largement disponible depuis partout
* Mutualisation des ressources et système multi tenant 
* Ressource élastique rapidement. Possibilité pour les utilisateurs de pouvoir élargir ou restreindre leurs ressources à la demande et rapidement
* Métriques complètes des ressources


La catégorie Top Performers ont un taux d’adoption du cloud 23 plus important que le groupe des Low Performer. 

Mais au-delà, les pratiques annexes au Cloud comme l’infrastructure as code, le PaaS, le Loosely coupled architecture ou les services managés et les containers ont un impact sur les performances.

Les utilisateurs du l’IaC ont 1.8 fois plus de chances d’appartenir au groupe Elite performer.
De même les utilisateurs du PaaS sont 1.5 fois plus nombreux dans ce groupe. Enfin les utilisateurs des containers sont 1.3 fois plus nombreux dans le groupe Elite Performer. 

Dans un registre similaire, les utilisateurs de l’Open Source (comme composant, librairie ou plateforme) ont 1.75 fois plus de chance de se trouver dans le groupe Elite Performer. 
On comprend mieux le virage dans l’Open Source de Satya Nadella à la lumière de ces résultats.

Dans un autre sens, l’externalisation est aussi un marqueur de la performance, le groupe Low Performer ont plus tendance à avoir recours à l’externalisation des fonctions IT (Ops, Dev, Test …) que les tops performer. Ils sont 3.2 fois susceptible d’utiliser l’outsourcing.

L’externalisation des différente fonction IT oblige à la mise en place de silos qui sont autant d’obstacle au déploiement rapide de solution. Elle diminue la vitesse des déploiements mais aussi la qualité des déploiements ce qui peut avoir des conséquences financières. 

Cependant cela ne s’applique pas forcément à tous les modèles, comme l’utilisation de partenaires pour le développement d’un système complet. La clé est la disparition des silos. 

L’important pour atteindre un niveau élevé de SDP n’est pas forcement l’utilisation d’outils mais le changement culturel. 
Dans un premier temps le concept d’equipe multifonction. Un tel type d’équipe permet de réaliser un produit en ayant peu d’interaction extérieur.   

Le Lean Management est aussi un point important dans l’évaluation des performances. 

Le lean Management se caractérise par un découpage des features et des produits en petits lots pouvant être traiter en une semaine et déployer rapidement. Un besoin de feedback permanent sur les produits déployés et la possibilité pour une équipe de pouvoir changer ou créer des spécifications sans devoir passer par un processus d’approbation.

Plusieurs autres techniques sont capables d’influer sur l’index SDP. C’est le cas de l’utilisation d’un gestionnaire de code sources, du déploiements automatiques, de l’intégration continue. Ce sont les constituant du Continuous Delivry et c’est un marqueur de succès dans l’index SDP. 
Le Continuous Delivry est mesuré par la capacité des équipes à déployer en production dans le cadre de leur cycle de production logiciel et par la capacité du système à renvoyer des informations sur la qualité du déploiement. 

Il faut ajouter deux composants, le monitoring (qui permet de voir et comprendre l’état d’un système) et l’observabilité (qui permet au equipe de débugger et de voir des propriétés d’un système non définies à l’avance). Les équipes ou les organisations utilisant ces techniques on 1.3 fois plus de chance de se trouver le groupe Elite Performer.

Un autre facteur impactant le SDP est le Continuous Testing. Cette pratique diffère des tests automatiques par l’ajout : 

* De Relecture et d’amélioration des tests pendant toute la durée de vie du projet
* Le travail de concert des testeurs avec les équipes de dev
* D’étapes de test d’acceptance et des tests exploratoires
* L’emploie du Test Driven Developpement
* La mise en place en place d’outil de feedback des tests automatiques depuis le serveur de CI et sur le poste des dév.


Un autre point important dans l’établissement de l’indice de performance est l’intégration et la prise en compte de la sécurité dans le process de construction. Le groupe des Low Performer mettent plusieurs semaines à mettre en place les changements nécessaires comparativement au High Performer. 

La notion de Culture fait partie intégrante du mouvement DevOps. Le rapport se base sur les travaux de sociologue Ron Westrum sur les la culture des organisations. La culture est une clé dans les performances IT et Business. 

Le rapport montre bien que la culture est la cheville ouvrière d’une transformation technique et que le type d’organisation influx sur les performances et la sécurité. 

La culture DevOps (Coopération, Gestion des problèmes, la suppression des silos, le blameless et l’expérimentation continue) s’aligne sur les organisations orientées vers la performance. 

Le rapport DORA s’interroge sur la manière d’influencer la culture. En premier lieu le rôle du Leader.

* Enonce les objectifs mais laisse la team les choix pour l’atteindre
* Met en place de règles simples 
* Permet aux équipes de changer les règles si elles sont un obstacle pour atteindre l’objectif
* Permet aux équipes d’aller contre les règles si cela permet de mettre la priorité sur la satisfaction des clients

Le rapport s’intéresse, enfin, à l’impact de l’apprentissage sur la performance. 

Le premier élément concerne les rétrospectives et la possibilité de pouvoir apprendre à travers ces réunions. Cela a un impact important sur la culture de l’entreprise lorsque les équipes s’en saisissent pour apprendre des erreurs et y voient une opportunité pour innover. 

Les équipes faisant ce type de rétrospectives ont 1.5 de chances de se trouver dans le groupe Elite Performer. 

La culture de l’éducation est aussi importante pour les performances. Les entreprises où l’apprentissage est vu comme une opportunité de croissance et non une simple nécessité sont parmi les plus performante. 

L’un des points importants est où et à quel moment l’apprentissage doit avoir lieu, sur le lieu de travail pendant les heures de travail ou en dehors. C’est la première option qui est la plus importante car elle permet d’inclure tous ceux qui ne peuvent se permettre de consacrer du temps en dehors des heures de travail. 

Cela peut être des hack days, des meetups internes ou la présence a des salons et des conférences. 
