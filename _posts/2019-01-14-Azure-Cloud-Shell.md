---
layout: post
title: Azure Cloud Shell
image: /img/hello_world.jpeg
author: Olivier Miossec
tags: [Azure, Cloud, PowerShell]
---

Azure Cloud Shell est un nouveau moyen pour travailler sur Azure depuis un navigateur web. Il est disponible depuis le portail Azure, mais le mieux est de passer directement par l’URI https://shell.azure.com 
Azure vous donne, ainsi,  accès à l’interface web d’un shell, en PowerShell ou en BASH depuis une machine linux. 
Comme tout est déjà pré-installer vous n’avez plus besoin de configurer ou d'installer quoi que ce soit.  

Pour résumé vous avez une machine de travail sous Ubuntu 16.04 LTS pensée pour Azure. Cette machine est en faite un container tournant dans un cluster Kubernetes (vous pouvez vous en rendre compte en utilisant cat /proc/1/cgroup). 
Ce container est temporaire, il est désalloué à chaque fin de session ou 20 minutes apres la dernière interaction dans le navigateur. 
C’est donc un environnement temporaire. Il est recréé à neuf à chaque session. Il est donc impossible d’installer durablement quoi ce soit. 
Mais si le container est recréé à chaque nouvelle session, il reste possible de garder des données entre les sessions. Pour cela Azure vous propose de créer ou d’utilise un compte de stockage lors de la préparation de votre première session de Cloud Shell. A patir de ce compte de stockage un partage de fichiers Azure est monté sur chaque nouvelle instance du container. 

Par défaut Azure Cloud Shell demande la création d’un ressource groupe, d’un compte de stockage et d’un partage de fichiers à la première connexion, vous pouvez utiliser celui proposé ou utiliser des ressources déjà en place. Il faut disposer du rôle contributeur à minima pour mener à bien l’opération. 

Le partage contient l’image de votre container en .img, mais vous pouvez aussi l’utiliser pour stocker les données d’un ou plusieurs projets. 

Il est d’ailleurs possible de monter ce partage sur une machine de travail pour y transférer ses propres scripts et outils. Le partage est monté et non synchronisé donc il est possible d’ouvrir le partage de plusieurs points en même temps. 

Dans le container, le point de montage est dans votre répertoire home ($HOME), mais en PowerShell vous pouvez aussi utiliser le cmdlet Get-CloudDrive

````Powershell 
set-location (get-clouddrive).MountPoint 
````

Permet de se déplacer sur le point de montage.

Mais vous pouvez aussi remarquer, lorsque vous démarrer Azure Cloud Share, vous vous trouvé dans "PS Azure:\\>"

Ce n’est pas un effet décoratif, c’est un outil qui permet de naviguer parmis les ressources azure comme si c’était un système de fichier, un PS Drive. 
Vous pouvez faire CD le nom d’une souscription et vous pouvez naviguer dans les ressources comme sur votre disque local. 

![image-center](/img/azshell/AzureDrive2.PNG)


![image-center](/img/azshell/AzureDrive.PNG)

Le pseudo répertoire du Ps Drive, AllResources, permet de lister l’ensemble des ressources de la souscription, cela peut vitre être compliqué à lire, il vaut mieux passer par le pseudo répertoire ResourceGroups pour effectuer ce listing.

Le signe + à coté d’un non de ressource signifie qu’il est possible de naviguer à l’intérieur de la ressource.

L’on peut aussi filtrer la sortie pour trouver ce que l’on cherche, par example, pour lister toutes les vm désallouées dans une souscription : 

````
dir | grep deallocated
````

Mais attention, les ressources ne sont pas automatiquement mises à jour, donc cela ne donne pas forcement une image exacte de la situation de la souscription. 

Attention seul la commande DIR fonctionne, la commande ls est, elle, réservée à la navigation dans le système de fichiers du container.

Si vous avez navigué en dehors de ce PsDrive, vous pouvez toujours revenir en tapant : 

````
cd Azure:
````

Ce container dispose d’une série d’outils utiles pour travailler sur Azure, outre PowerShell Code en version 6.1, l’on dispose aussi du module AZ pour travailler avec Azure en PowerShell Core. Le container est fourni avec un nombre important d’outils.

Azure CLI
AzCopy
Service Frabic tools 
Blobxfer (un outil de transfert pour le storage Azure)
batch-shipyard (pour gerer le batch HPC)
Git
Des outils de build (Maven, Make, npm,…)
Des clients de bases de données (Sql Server, Mysql, PostgreSql)
Des outils de gestion de containers (Docker, Kubernetes, DC/OS …)
Des outils d’Infrastructure as Code (Terraform, Ansible, Chef, …)

Le container dispose également des outils pour les langages .net, Go, Java, Python et node.js

Autre point important pour comprendre le fonctionnement Azure Cloud Shell, l’authentification. On pourrait penser que lorsque l’on ouvre une session dans Cloud Shell, l’identité utilisée est celle ayant à se loguer à l’application elle-même. 

Mais si l’on regarde à travers Get-AzContext on comprend que ce n’est pas le cas 

![image-center](/img/azshell/Msi.PNG)

A la place nous avons un Managed Service Identity. Cela peut poser quelques soucis dans certaines conditions.

Je vous invites à lire le poste de Benoit Sautière ici pour mieux me comprendre
http://danstoncloud.com/simplebydesign/2019/01/06/retrouver-son-identit-dans-azure-cloud-shell/



Bref Azure Cloud Shell est un produit complet, il peut permettre de faciliter le travail en ayant tout sous la main.
Cependant il y a de fortes limitations qui rend le confort d’utilisation difficile à apprécier.
On peut noter le mauvais support du copier/coller par raccourcie clavier, impossible de copier quelque chose d’un document sur votre PC et de le coller par Ctrl-V dans le shell, il faut passer par un clic droit. 
Surtout si vous perdez trop longtemps le focus du shell pour vous concentrer sur un script dans vscode ou une page web ou tout autre chose, vous êtes bon pour relancer le container.
Autre chose, si vous avez des souscriptions portant le même nom, vous pouvez avoir une warning lors de la première commande, mais cela ce corrige rapidement en changeant le nom d’une des souscriptions. 
