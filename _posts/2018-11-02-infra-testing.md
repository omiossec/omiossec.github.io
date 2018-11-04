---
layout: post
title: Infra Testing
image: /img/hello_world.jpeg
author: Olivier Miossec
tags: [PowerShell, DevOps, Pester, Gherkin, CD, OVF]
---

Déployer automatiquement plusieurs dizaines de serveurs et/ou services, que ce soit en datacenter ou sur le Cloud devient de plus en plus courant. Ce n’est pas le but de ce post d’expliquer comment faire (j’aurais sans doute le temps d’en faire d’autre sur ce point). Le but est de voir comment l’on peut valider ce que l’on vient de faire et ce  que cela soit par une chaine de CD ou manuellement. 

La réponse a souvent été un cahier de recette avec des tests manuels. Outre le fait qu’un test manuel est souvent galvaudé cela ne prend que tres rarement en compte les différentes modifications qui peuvent être appliquer lors de la vie d’un système. 

Ce n’est en rien satisfaisant. Avoir une armée mexicaine de techniciens pour vérifier que le fonctionnement est correct n’est ni fiable ni souhaitable. C’est encore plus le cas si tous les composants sont déployés automatiquement, depuis une chaine de CD ou pas.

L’homme n’aime pas tester. C’est humain, on pense toujours que ce que l’on a fait n’est pas si mal et on est retissant à rentrer dans les détails. De plus, tester est bien moins valorisé que la construction. Enfin dans les équipes d’OPS il y a souvent une pression et une culture du reproche qui fait que parfois il est préférable de minimiser voire dissimuler les disfonctionnements au lieu de les corriger. 

Pourtant l’erreur est toujours possible et c’est essentiel de pouvoir les voir au cours ou apres un déploiement pour apporter une solution. Car apres tout notre but est de mettre en place un service qui fonctionne et répond au besoin du client.

On doit assumer que l’on peu se tromper. En software engineering, les chaines d’intégration contiennent toutes des étapes de tests automatisées, comme les tests unitaires par exemple. Si une erreur est détectée, la chaine stoppe, on retrouve l’erreur et on la corrige. Cela ne provoque pas ni crise, ni drame, ni reproche, car l’erreur est dans la nature des choses. 

Il devrait en être ainsi aussi pour tout ce qui touche au déploiement et à l’exploitation.

Quel sont les options disponibles pour automatiser ces tests de validation. Il faut qu’ils puissent à la fois être scalable, passer de 1 machine ou 1 services à des centaines, qu’ils puissent être intégrer à une chaine de déploiement et qu’ils puissent être modifier et rejouer  à chaque modification, enfin il faut qu’ils puissent s’intégrer dans un repos de la  même manière que le code déploiement.



* **Pester**, Pester est le Framework de test de PowerShell, il est même directement intégré depuis la version 5.1 de Windows Management Framework. Il utilise un « Domain Specific Language » pour décrire et gerer les tests. Si à l’origine il était destiné à la gestion des tests unitaires, il permet, aujourd'hui, de réaliser des tests opérationnels et d’exploitabilité. 

* **Operation Validation Framework**, est une extension de PowerShell, il permet de gerer un bloc de tests comme un module. Les tests peuvent ainsi être versionnés et être disponible facilement à partir d’un repos privé. Un de ces avantages et qu’il oblige à standardiser les tests pester.

* **PoshSpec** est une autre solution s’appuyant sur Pester, il a été développé à l’origine par TicketMaster. Il ajouter un DSL supplémentaire pour les tests spécifiques à l’infrastructure et s’appuie sur des "fonctions" spécifiques pour réaliser les tests. 

* **Gherkin**, est lui aussi un DSL. Il se base sur langage qui se veut simple et compréhensible en Anglais pour décrire des spécifications et des scénarios (ex. GIVEN Quelque Chose WHEN un Evènement THEN un résultat attendu) puis gerer les tests en eux même. Gherkin est intégré à Pester dans ses dernières versions. 
Il se base sur Cucumber 

C’est 4 composants feront l’objets de posts dans les semaines qui viennent