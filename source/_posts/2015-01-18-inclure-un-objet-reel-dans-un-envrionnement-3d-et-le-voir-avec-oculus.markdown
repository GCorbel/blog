---
layout: post
title: "Inclure un objet réel dans un environnement 3d et le voir avec Oculus"
date: 2015-01-18 19:34
comments: true
categories:
---

Grâce à Unity 3d, il est possible de créer une scène, d'y intégrer le module
Oculus et d'importer n'importe quel modèle 3d en quelques minutes. Dans cet
article, nous allons voir comment insérer un objet réel dans un tel
environnement.

<!-- more -->

## Introduction

Unity 3d est un outil simple pour créer des environnements 3d,
même sans avoir de formation en modélisation. Oculus fournit un package nous
permettant de nous immerger dans un monde virtuel créé avec Unity. En moins de 10 minutes, il
est possible d'intégrer Oculus dans un tel univers et d'y inclure un objet
réel. J'ai choisi de présenter la marche à suivre de manière détaillée et
d'essayer de faire en sorte que l'article soit le plus compréhensible possible.

## Installation et téléchargements

Il existe une version d'Unity 3d pour Mac OS X et Windows. Malheureusement, il
n'existe pas de version pour Linux. Pour ma part, j'ai dû quitter mon OS préféré et aller
sur Windows. Il est possible de télécharger l'application
[ici](http://Unity3d.com/Unity/download). Le package d'intégration d'Oculus est
disponible [ici](https://developer.oculus.com/downloads/). Faites attention à
bien télécharger le fichier "Unity 4 Integration". L'installation se fait comme
n'importe quel autre logiciel.

## Création de la scène

La marche à suivre pour cette étape et la suivante est disponible [sur Youtube](https://www.youtube.com/watch?v=7kuQYcIYPvQ), en anglais.

Il est possible de créer un environnement à partir de zéro. À des fins de test,
on peut également télécharger un modèle existant grâce à l'Asset Store d'Unity
en quelques étapes faciles. Pour cela, il faut lancer l'Asset Store.

![Asset Store](http://img4.hostingpics.net/pics/46705220150118204656UnityUntitledNewUnityProject1PCMacLinuxStandalone.png)

Il existe des modèles gratuits comme "Bootcamp" que l'on peut trouver en
cliquant sur "Complete Projects" et cliquer sur "Bootcamp".

![Bootcamp](http://img4.hostingpics.net/pics/48129520150118204919.png)

Une fois la page descriptive ouverte, il est possible de télécharger le projet
et de l'importer.

![Import Bootcamp](http://img4.hostingpics.net/pics/47543920150118205034Importingpackage.png)

Après un moment, vous pouvez voir que la structure du projet à été créée.

![Asset Explorer](http://img4.hostingpics.net/pics/79352520150118205551UnityBootcampunityNewUnityProject1PCMacLinuxStandalone.png)

Vous pouvez alors cliquer sur "Bootcamp" pour le lancer. Et voilà, la scène
virtuelle est prête. Vous pouvez lancer le jeu en cliquant sur le bouton play
dans le menu du haut.

## Intégration d'Oculus

Pour ajouter le module Oculus au monde créé précédemment, il suffit de
décompresser le package téléchargé sur le site d'Oculus puis faire un Drag &
Drop du fichier "OculusUnityIntegration.Unitypackage".

![Intégration Oculus](http://img4.hostingpics.net/pics/34186720150118213256UnityBootcampunityNewUnityProject1PCMacLinuxStandalone.png)

Cet écran vous sera affiché :

![Importation Oculus](http://img4.hostingpics.net/pics/62659120150118213345Importingpackage.png)

Il suffit de cliquer sur "import" et d'attendre quelques secondes.

Le répertoire OVR sera alors créé.

![Asset Explorer](http://img4.hostingpics.net/pics/92238420150118213701UnityBootcampunityNewUnityProject1PCMacLinuxStandalone.png)

Naviguez dans "OVR/Prefabs", vous verrez alors le fichier "OVRPlayerController".
Un simple Drag & Drop du navigateur vers la scène suffit pour l'ajouter au
projet. Vous pouvez positionner la caméra comme bon vous semble. La position
choisie correspondra à votre position de départ lors du lancement du jeu.

Pour plus d'aisance, vous pouvez supprimer le soldat mis en place dans
Bootcamp en faisant un clic droit sur "Soldier_Locomation" et choisir
"Delete".

Vous pouvez d'ores et déjà lancer le jeu et utiliser Oculus!

## Import d'un objet réel

L'étape d'intégration d'un modèle 3d se fait de manière tout aussi simple. J'ai
choisi d'importer le modèle créé dans [l'article précédent](http://gcorbel.github.io/blog/blog/2015/01/03/reconstruire-un-modele-3d-a-partir-dobjets-reels/).
Il s'agit de la table basse de mon salon reconstituée grâce à des photos. Je
suis allé télécharger le modèle en format .obj sur [123D catch](http://www.123dapp.com/catch). Pour l'intégrer, il faut faire un clic
droit sur l'explorateur d'assets et cliquer sur "Show in Explorer".

![Show in Explorer](http://img11.hostingpics.net/pics/664926unnamed1.png)

Vous pouvez ensuite placer les fichiers dans le répertoire. Unity détectera les
nouveaux fichiers. Vous pouvez ensuite trouver votre fichier obj dans
l'explorateur et faire à nouveau un Drag & Drop ce qui ajoutera le modèle 3d
dans l'environnement. Vous pouvez ensuite redimensionner l'objet pour lui
donner l'aspect souhaité. Voici donc ma table basse dans l'environnement :

![Résultat de l'intégration](http://img4.hostingpics.net/pics/797068image841.png)

Finalement, vous pouvez générer l'application pour la partager.

## Conclusion

Quand j'ai acheté mes lunettes Oculus, je ne pensais pas être capable de faire
de telles créations. Heureusement, Unity nous simplifie grandement la vie et nous
donne un résultat impressionnant en quelques étapes simples.
