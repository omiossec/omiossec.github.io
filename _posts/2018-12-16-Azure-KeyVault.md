---
layout: post
title: Comment utiliser Azure KeyVault avec PowerShell
image: /img/hello_world.jpeg
author: Olivier Miossec
tags: [PowerShell, Azure, Security]
---

Lors d’un déploiement par ARM Template se pose toujours la question ; comment gerer les mots de passe des services et des VM à déployer. Souvent l’option est de laisser les différents mots de passe dans les fichiers paramètres avec le risque d’être compromis. Il y a plusieurs stratégies comme celle qui consiste à n’ajouter les mots de passe qu’au moment du déploiement et de changer les fichiers paramètres apres le déploiement, ou celle qui consiste à utiliser un mot de passe générique qui sera changer par la suite.  
Aucune de ces méthodes n’est la bonne. D’ailleurs on trouve facilement des outils capables de parcourir les repos github à la recherche de ces mots de passe dans les fichiers json.

Posons les termes du problème, nous avons besoin d’un mot de passe pour une déployer une VM, ce mot de passe peut être aussi utilisé pour déployer le Workload depuis Azure Automation et/ou Azure DevOps. Finalement l’opérateur n’a besoin du mot de passe que dans deux cas, lors d’une opération de débug ou lorsque une opération totalement manuelle est nécessaire. 

Azure dispose d’un service pour cela, KeyVault. Ce service permet de sécuriser les différents secrets que l’on peut avoir à utiliser lors d’un build sur Azure, Mot de passe, compte, clé RSA, certificat …
Le principe est simple, Azure Key Vault est un container permettant de stocker des données sensibles. Ces données peuvent être limité dans le temps et sont accessible depuis un endpoint suivant un policy à définir.

Comment gerer cela depuis PowerShell ?

Dans un premier temps il faut créer un KeyVault.

```PowerShell
param (
    [String]
    $ProjectName,

    [String]
    $AdminAccount
)

$AzVaultName = $ProjectName + "-vault2"
$AzRessourceGroupe = $ProjectName + "-rg"

New-AzureRmResourceGroup -Name $AzRessourceGroupe -Location francecentral

[Reflection.Assembly]::LoadWithPartialName("System.Web")
$StrSecret = [system.web.security.membership]::GeneratePassword(19,0)
$Secret = ConvertTo-SecureString -String  $StrSecret -AsPlainText -Force

New-AzureRmKeyVault -Name $AzVaultName  -ResourceGroupName $AzRessourceGroupe -Location francecentral -EnabledForTemplateDeployment

Set-AzureKeyVaultSecret -VaultName $AzVaultName -Name $AdminAccount -SecretValue $Secret
````

L’on peut maintenant associer des droits sur ce KeyVault soit aux personne qui vont déployer les solutions soit à un Service Principal Name (SPN). 

Pour donner les droits de lecture à ce compte : 

```PowerShell
Set-AzureRmKeyVaultAccessPolicy -VaultName $AzVaultName -UserPrincipalName "MyDep@something" -PermissionsToSecrets get,list
````

Comment utiliser ce KeyVault pour les déploiements ?  

Il suffit de remplacer les éléments suivants : 

```json
     "adminPassword": {
          "value": "CHANGEME"
      }
````

Par un appel vers notre container KeyVault. 

Pour commencer il faut retrouver la chaine de connexion au KeyVault. Elle se compose de l’ID de la Souscription, du nom du groupe de ressources et du nom du KeyVault.

/subscriptions/xxxx-xx-xxx-xx-xx/resourceGroups/mydeploi-rg/providers/Microsoft.KeyVault/vaults/mydeploi-vault2

```json
        "adminPassword": {
            "reference": {
              "keyVault": {
                "id": "/subscriptions/xxxx-xx-xxx-xx-xx/resourceGroups/mydeploi-rg/providers/Microsoft.KeyVault/vaults/mydeploi-vault2"
              },
              "secretName": "omadmin"
            }
          }
````

Keyvault peut aussi être disponible à partir d’Azure DevOps. Il y a ici plusieurs méthodes, mais si vous devez utiliser un des secrets dans un pipeline, il faut créer un Groupe de Variable dans Library. 


Il est nécessaire de sélectionner « Link secrets from an Azure KeyVault as variable » 
Il faut sélectionner le SPN, le ressource group et le KeyVault. Il est possible, alors d’utiliser les secrets comme variable.

![image-center](/img/keyvault/azdevopskeyvault.PNG)