---
layout: post
title: Arm Template et Copy
image: /img/hello_world.jpeg
author: Olivier Miossec
tags: [Azure, Cloud, ARM]
---

Les templates ARM sont un peu la bête de tous ceux qui souhaitent passer une certification sur Azure. Mais les Templates ARM sont un des outils les plus puissant de la mise en place d’infrastructure sur Azure. Ils sont au cœur de l’Infrastructure as Code, car ces Templates sont conceptuellement déclaratifs (on décrit ce que l’on veut au lieu d’écrire ce que l’on doit faire) et sont idempotent (chaque exécution donne le même résultat, ils sont répétables). 
Pour faire un raccourcie avec le monde PowerShell, ARM pourrait être comparé à DSC, où il suffit d’indiquer ce que l’on veut à l'aide de ressources au lieu d’écrire un script complet pour le mettre en place. 

L’utilisation d’un template est généralement simple, lorsque l’on doit déployer des infrastructures simples, il y a beaucoup d’exemple sur Github. Mais cela peut être plus compliquer avec des cas plus proches de la réalité.

Prenons l’exemple du déploiement de multiple Vnet dans un même groupe de ressource (pour faire simple). Un Vnet est une structure assez simple, un nom, un préfix sous la forme CIDR et un à plusieurs sous réseaux avec leur nom avec leur propre identifiant réseau. 

Comment le faire pour écrire une template qui puisse être réutilisé pour d’autres réseau par la suite ?

L’option de tout hard coder n’est pas la meilleure, cela risquerait d’être tres vite compliqué et illisible. Mais surtout, l’esprit du Configuration Management, voudrais que l’on puisse séparer les données (les noms des réseaux et sous réseaux) de l’action (le template ARM).

Avant tout il faut se rappeler de la structure d’un template ARM. C’est un fichier JSON composé de plusieurs sections, Parameters, Variables, Version, Schema, functions et Resources. Functions, Parameters et Variables servent à fournir des données qui seront utilisées par Resources pour la mise en œuvre la solution. 

Comme on le devine, Parameters permet d’envoyer des données. Ces données sont typées, on indique si l’on souhaite un String, en entier, mais l’on peut avoir aussi des objets plus complexes comme des tableaux. 

Ce sont les tableaux qui vont nous permettre de déployer des ressources similaires plusieurs fois avec le même template ARM sans avoir à multiplier les déploiements. 

Pour déclarer un Tableau en paramètre : 

```json
"NetConf": {
            "type": "array"
        }
````

Pour le construire dans un fichier paramètre : 

```json
"NetConf": {
            "value": [
                {
                    "VirtualNetworkName": "Production",
                    "CIDR": "172.24.0.0/24",
                    "SousReseau": [
                        {
                            "Nom": "Prod",
                            "Prefix": "172.24.0.0/24"
                        }
                    ]
                },
                {
                    "VirtualNetworkName": "Dev",
                    "CIDR": "172.25.0.0/24",
                    "SousReseau": [
                        {
                            "Nom": "Dev",
                            "Prefix": "172.25.0.0/25"
                        },
                        {
                            "name": "bacasable",
                            "Prefix": "172.25.0.128/25"
                        }
                    ]
                }
````

L’on a ici un tableau imbriqué, un tableau contenant un autre tableau. Pour le traiter il faut parser le premier niveau, celui qui contient le nom des réseaux virtuelles puis pour chaque réseau parser les sous réseaux.

Le traitement d’un objet Array dans un template ARM se fait avec l’élément COPY. Copy est capable de traiter aussi bien des ressources, il peut effectuer une itération sur une ressource, que des propriétés d’une ressource. 

Copy pourait être comparer à une boucle ForEach en PowerShell. 

Pour effectuer une itération sur une ressource COPY prend deux paramètres

Name : Le nom de la boucle, il sera utilisé pour identifier les éléments par la suite
Count : le nombre d’itération, soit ici la taille du tableau 

```json
"copy": {
                "name": "Vnets",
                "count": "[length(parameters('NetConf'))]"
            }
````

Par la suite si l’on souhaite accéder au éléments du tableau, il faut ajouter une référence avec CopyIndex et le nom de l’itération données dans COPY.

```json
"name": "[parameters('NetConf')[copyIndex('Vnets')].VirtualNetworkName]",
````

Lorsque l’itération concerne une propriété et non une ressource, COPY prend un paramètre supplémentaire, INPUT

```json
"copy": [
                                    {
                                        "name": "subnets",
                                        "count": "[length(parameters('NetConf')[copyIndex('Vnets')].Subnet)]",
                                        "input": {
                                            "name": "[parameters('NetConf')[copyIndex('Vnets')].Subnet[copyIndex('subnets')].SubnetName]",
                                            "properties": {
                                                "addressPrefix": "[parameters('NetConf')[copyIndex('Vnets')].Subnet[copyIndex('subnets')].Prefix]"
                                            }
                                        }
                                    }
                                ]
````


La boucle effectue alors une itération, non pas sur le tableau de premier niveau mais sur le tableau de second niveau. 

Il prend comme paramètre un nom, qui sert à identifier la boucle par la suite et aussi un Count, qui prend comme valeur la taille du tableau de second niveau.

Il utilise enfin le paramètre input, c’est sur ce dernier que porte l’itération 

Input créer une itération sur la propriété Subnets de la ressource Virtual Network. Cela permet de créer le tableau de sous réseau définie en paramètre. 

Vous pouvez voir l’exemple complet sur Github 

[Github ARM COPY](https://github.com/omiossec/ARM-TEMPLATES/tree/master/multiple-Vnet-from-array)

Copy n’est pas le seul élément permettant de décrire une architecture complexe sur Azure. L’une des autres méthodes est de modulariser les déploiements en utilisant des templates imbriqués. Ce qui devrait être l’objet d’un prochain billet.