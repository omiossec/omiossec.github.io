---
layout: post
title: Comment builder le nouveau Windows Terminal
image: /img/hello_world.jpeg
author: Olivier Miossec
tags: [Microsoft]
---

Lundi 6 mai, à la conférence MS Build, Microsoft a annoncé la sortie d’un tout nouveau terminal pour Windows 10. Et il dispose de toutes nouvelles fonctionnalités, comme les tab, le support des emoji, la transparence et bien d’autres. 

Vous pouvez voir ici la [vidéo de présentation](https://www.youtube.com/watch?v=KMudkRcwjCw)

Cependant la version finale ne sera pas disponible avant l’hivers 2019 et les premières beta versions ne devront être disponible qu’à partir de l’été. 

Mais si comme moi, vous êtes impatient de pouvoir utiliser ce nouvel outil, il reste une solution. Microsoft ayant pris la décision de faire de mettre ce terminal en Open Source, vous pouvez le compiler vous-même. 

Pour cela, il y a quelques prérequis : 

- [x] Une machine sous Windows 10 (et avec la version 1903 si vous voulez vous les nouvelles fonctionnalités)
- [x] Un client git 
- [x] Nuget, le gestionnaire de paquets du monde .Net 
- [x] Visual Studio 17 ou 19

Si vous n’avez ni Git ni Nuget, vous pouvez les installer à la main, mais il y a une meilleur méthode. Vous pouvez, vous aussi, utiliser un package manager, un peu comme on fait un apt-get ou un yum install sur les distro Linux. 

Le package manager le plus abouti sur Windows est dans doute [Chocolatey](https://chocolatey.org/). 

Il est simple à installer et son utilisation est basique. 


```
Choco install git
Choco install NuGet.CommandLine
```

Maintenant la partie ardue, l’installation de Visual Studio. 

En premier lieu, il n’est pas la peine de sortir la carte bleue, le build fonctionne parfaitement avec la version Visual Studio Community. Cette version contient tous les outils nécessaires pour la tâche et est totalement gratuite. 

Vous pouvez la télécharger [ici](https://visualstudio.microsoft.com/vs/community/)

Pendant le setup, vous aurez à faire le choix des composant à installer, vous pouvez regarder les différents composants et même en choisir quelqu’un, comme Azure, mais pour ce qui concerne le terminal, il y a des packages spécifiques à choisir, et vous devez prévoir de l’espace disque. 

Si vous avez déjà installé Visual Studio 2019, vous pouvez lancer Visual studio installer puis cliquer sur « Plus » ou « More » et modifier pour ajouter les éléments nécessaires. 

- [x] Desktop Developpement C++
- [x] Universal Windows Platform development 

![Visual Studio Installer](https://thepracticaldev.s3.amazonaws.com/i/bo85dm0h5846ko1i72k4.PNG)



Il faut ensuite passer sur Composants individuels et ajouter Windows SDK 10.0.18362.0

![Individual Components](https://thepracticaldev.s3.amazonaws.com/i/dmgzwtwhzy4x2dgdnst1.PNG)

Une fois le tout installé, on peut commencer à builder le terminal

Il faut ouvrir "Developer command prompt for VS 2019 tool" et se déplacer dans un dossier où l’on souhaite télécharger le code source de MS Terminal. 

```
git clone https://github.com/microsoft/Terminal.git
```

Cela nous permet de télécharger le code source. Mais ce n’est pas totalement suffisant. Le projet contient deux sous modules qu’il faut aussi télécharger 

```
cd Terminal

git submodule init

git submodule update
```

Maintenant, c’est au tour de Nuget de rentrer dans la boucle
```
nuget restore OpenConsole.sln
```

Cela permet de télécharger toutes les dépendances liées à la solution. Cela évite de trop faire grossir les repos. 

On peut maintenant lancer la compilation 

```
msbuild /p:Configuration=Release /p:Platform=x64 /p:PlatformToolset=v142 /p:TargetPlatformVersion=10.0.18362.0 /p:PreferredToolArchitecture=x64 OpenConsole.sln
```

Cela peut prendre un certain temps. 

Après plusieurs minutes, la compilation se termine, et l’on peut se déplacer dans le dossier \bin\x64\Release et ouvrir OpenConsole.exe. Mais attention si vous n’avez la version 1903 ou plus de Windows 10, vous n’aurais pratiquement rien de neuf.

Il faut se rappeler que c’est une version alpha, elle est limitée et comporte certainement des bugs et des limitations. C’est peut-être l’occasion de participer à un projet Open Source en signalant un bug par la création d’une issue sur Githib ?