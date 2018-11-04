---
layout: post
title: PowerShell et code Signing
image: /img/hello_world.jpeg
author: Olivier Miossec
tags: [PowerShell, PKI, Code Signing]
---


On la tous fait un jour, pour faire tourner script souvent télécharger depuis un site ou GitHub on a lancer PowerShell en mode ByPass ou changer la politique d’exécution. 

Tout cela, car la politique d’exécution par défaut, à partir de Windows 2012 R2, exige que les scripts télécharger depuis Internet soit signé (si vous vous voulez savoir comment Windows peut savoir si un fichier a été télécharger il faut regarder du côté des metadata du système de fichier, Zone.Identifier). 

Ce n’est pas un mécanisme de sécurité, il est très facile de passer outre (ByPass et le plus simple). Mais cela permet d’éviter les actions non intentionnelles. Un changement malicieux d’un script peut être plus facilement détecté par quelqu’un.  

Contrairement à ce que pourraient pensaient certains, signer un fichier ne l’encrypte pas. Signer un script, un module ou un logiciel, c’est indiquer et prouver son identité. 

La preuve d’identité se fait par un tiers de confiance, qui valide que la signature et le fichier n’ont pas été altéré et que l’identité et bien valide. 

Pour faire une analogie, c’est un peu comme un caissier qui prend votre carte d’identité pour vérifier la signature du chèque que vous venez de faire. Le tiers de confiance c’est la préfecture qui vous a délivré cette carte d’identité.

En informatique, un tiers de confiance passe par une PKI, Il faut une autorité de certification, elle peut être publique (mais généralement payante) ou privée (mais dons la disponibilité est limitée). 

On le voit donc, vouloir utiliser un certificat autosigné pour distribuer un script et un module n’est pas tres utile. 

En informatique, un tiers de confiance passe par une PKI, Il faut une autorité de certification, elle peut être publique (mais généralement payante) ou privée (mais dons la disponibilité est limitée). 

Pour une autorité de certification basée sur Microsoft, voici la procédure pour configurer le code signing. 

 Il faut avoir un accès au à l’autorité de certification de l’entreprise.

 * Ouvrir la console de l'autorité de certification et faire un clic droit sur "Certificate Template" 
 ![image-center](/img/cs01/2018/CertTemplate.png)
 * Cliquer sur "Manage" pour ouvrir le gestionnaire de template
 *  Faire un clic droit sur "Code Signing" puis "Duplicate template". Dans la nouvelle fenêtre sélectionnez l’onglet General
 * Entrer un nom, sélectionner une période de validé (1 an par défaut)
  ![image-center](/img\cs01\2018\templateProp.png)
  Vous pouvez modifier d’autres propriétés comme la durée de validité et la période renouvellement.
 * Dans "Security", limiter les utilisateurs (ou les groupe) qui peuvent utiliser la signaature
 ![image-center](/img\cs01\2018\TemplateRight.png)
 * De retour dans la console de l’autorité de certification, faire un clic droit sur "Certificate Template" et sélectionner New puis "Certificate Template to Issue".
 * Sélectionner le template et cliquer sur OK
 * Le template est disponible pour les utilisateurs

 Il est possible, alors, de signer des scripts. 

 ````PowerShell 
# Les certificats étant délivrés à un utilisateur, il faut se rendre en PS dans la Provider Certificats cert:\currentuser\my

Set-Location -Path cert:\CurrentUser\My
Get-Certificate -Template PowerShellSign

# On peut lister les certificats 
dir

# On peut aussi sélectionner le certificat en utilisant son thumbprint 
# et stocker un pointeur vers ce certificat dans une variable
$cert = dir Cert:\CurrentUser\My\7DEBE3605DE375F6F1051C9D6B39C4A76FBD349D

Set-Location -Path c:\

Set-AuthenticodeSignature -Certificate $cert -FilePath C:\scripts\monScript.ps1
 ````

La signature prend la forme d’un commentaire à la fin du fichier. Cette signature comprend un checksum du fichier que l’on vient de signer. Donc en cas de modification il faut résigner le fichier sous peine qu’il soit invalide. Il contient aussi la clé de l’autorité de certification pour vérification. 

Il est possible de faire un clic droit puis propriété sur le fichier pour voir un nouvel onglet signature. 
