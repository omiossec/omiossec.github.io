---
layout: post
title: Pester, AppVeyor et Continous Integration
image: /img/Appveyor_logo.svg.png
author: Olivier Miossec
tags: [PowerShell, DevOps, CI, Pester]
---

L’intégration continue est une technique de software engineering qui vise à vérifier que chaque modification apporter au code source maitre n’apporte pas de régressions. 
En règles général un pull request ou un commit sur la branche master déclenche une série d’actions automatiques. Parmi ces taches nous pouvons trouver des taches de build (compilation) et des taches de tests (Unit testing et Integration Testing).

PowerShell dispose d’un Framework de test, Pester. Pester permet d’effectuer des tests unitaires et des tests d’intégrations. Il existe plusieurs options pour automatiser ces tests depuis un repos, GitLab CI, TeamCity, Azure DevOps (ex VSTS) et AppVeyor. 

AppVeyor est un outil d’intégration continue permettant d’automatiser les phases de build et de test depuis les repos en ligne GitLab, GitHub et Bitbucket. Les environnements misent à disposition sont des VM Windows ou Linux Ubuntu. Il n’y a pas de facturation pour les projets Open Source. 

Le principe de fonctionnement est simple. Un fichier appveyor.yml est ajouter à la racine du repos, il contient les élements essentiel pour la phase de build et la phase de test. A chaque commit, appveyor clone le repos, lit les instructions dans le fichier yml, provisionne la machine demandée pour les tests et copie le code sur la vm. Les actions de build et de test sont alors exécutées.

Pour commencer il faut un compte Github et un repos (il est possible d’utiliser aussi Gitlab ou Bitbucket, mais dans ce cas des frais peuvent être demandés). 

Il faut ensuite créer un compte sur [https://ci.appveyor.com](https://ci.appveyor.com)  avec son compte GitHub. Il faut ensuite ajouter un projet dans l’application. 

Appveyor va automatiquement ajouter un webhooks dans le repos Github permetant les interactions. 

Il faut ensuite ajouter du code PowerShell et le ou les fichiers de tests pester. 

Pour illustrer ce post j’ai mis en place un repos avec des fonctions et ses fichiers de test. 

[https://github.com/omiossec/demo-pester-ci](https://github.com/omiossec/demo-pester-ci)

Ajouter du code et des scripts pester est essentiel mais non suffisant pour automatiser les actions de test. Il faut encore dire à AppVeyor comment gerer les tests. 

Comme indiqué plus haut cela se passe dans le fichier appveyor.yml à la racine du repos. 

Un des premiers choix à faire est celui de l’environnement dans lequel faire tourner les tests.

C’est le mot clé -image.

Il existe 5 images 
3 sous Windows qui dispose toutes de WMF 5.1 :

* Visual Studio 2013, Windows 2012 R2
* Visual Studio 2015, Windows 2012 R2
* Visual Studio 2017, Windows 2016


Les deux dernières disposent aussi de PowerShell 6.1

Ubuntu, Ubuntu 16.04 LTS
Ubuntu1804, Ubuntu 18.04 LTS


Les images Windows disposant de PowerShell 5.1 par défaut, il est nécessaire de mettre à jour Pester 
Pour cela, il faut utiliser le mot clé install 


````powershell
Install-Module Pester -Force -SkipPublisherCheck -Scope CurrentUser
````

Il ne nous reste plus qu’à exécuter notre test, c’est le role du mot clé test_script 
Cela donne cela

````
version: 1.0.{build}
image:
- Visual Studio 2017
install:
- ps: Install-Module Pester -Force -SkipPublisherCheck -Scope CurrentUser
build: off
test_script:
- ps: Invoke-Pester -EnableExit
````

Il est nécessaire de faire un -EnableExit avec invoke-pester pour indique a AppVeyor, le succès ou l’échec de l’opération. 


 ![image-center](/img/appveyor/simpleAppVeyor.PNG)


Le test est automatisé à chaque commit. Mais, même si l’on reçoit une alerte par email et que l’on peut voir le résultat sur le site d’AppVeyor, cela ne permet que peu d’interaction avec l’equipe. 

Comment savoir si ce que l’on a fait et comment le dire aux autres acteurs du projet. 

Il est possible d’afficher un badge sur le repos Github dans le readme principal.

Pour cela il faut sur la page du projet de appveyor puis dans l’onglet Badges 

Il suffit de copier le code Markdown dans le readme du repos.

````
[![Build status](https://ci.appveyor.com/api/projects/status/clpi7shipiujumih/branch/master?svg=true)](https://ci.appveyor.com/project/omiossec/demo-pester-ci/branch/master)

````

Pour aller plus loin encore il est aussi possible de publier les résultats du test dans AppVeyor 

Pour cela il faut utiliser l’output du cmdlet invoke-pester (-OutputFile fichier.xml -PassThru) 

Il faut envoyer le résultat dans l’API de AppVeyor https://ci.appveyor.com/api/testresults/nunit/

Il faut utiliser la variable d’environnement APPVEYOR_JOB_ID

Il faut cependant faire attention, pour qu’un test soit pris en compte, il faut qu’il puisse générer une erreur en cas d’échec.

````
version: 1.0.{build}
image:
- Visual Studio 2017
install:
- ps: Install-Module Pester -Force -SkipPublisherCheck -Scope CurrentUser
build: off
test_script:
- ps: | 
    $pesterTestResultsFile = ".\TestsResults.xml"
        $res = Invoke-Pester  -OutputFile $pesterTestResultsFile -PassThru
        (New-Object 'System.Net.WebClient').UploadFile("https://ci.appveyor.com/api/testresults/nunit/$($env:APPVEYOR_JOB_ID)", (Resolve-Path $pesterTestResultsFile))
        
        $errorString = ''
        # All tests should pass
        if ($res.FailedCount -gt 0) {
            $errorString += "$($res.FailedCount) tests failed.`n"
            throw $errorString
        }
````

Après cela, l'accès au resultat de l'ensemble des tests de l'execution Pester est disponible 

![image-center](/img/appveyor/teststeps.PNG)