---
layout: post
title: "Sbire : Dites ce que vous voulez, il le fera"
date: 2013-10-27 13:24
comments: true
categories:
  - Ruby
---

Sbire est un outil permettant d'exécuter des commandes grâce à la voix. Le principe est simple. Vous taper une commande pour l'enregistrement d'une phrase avec un micro et vous taper une seconde commande pour terminer l'enregistrement. Suite à cela, Sbire reconnait votre voix et exécute la commande associée.
<!--more-->
Sbire en action
---------------
{% youtube Fvu9tS0lsYE %}

Comment ça marche?
------------------
Il s'agit d'un programme écrit en Ruby. Celui-ci fait appel à sox pour faire un enregistrement audio. Il fait ensuite appel à Google voice en y joignant le fichier et y récupère le résultat en format JSon. Ce résultat est ensuite traité et la commande trouvée est exécutée.

Contibuer
---------
Évidemment, je suis ouvert à toute remarque et suggestion. Les contributions sont également les bienvenues. Vous pouvez les soumettes sur [Github](https://github.com/GCorbel/sbire).

Plus d'informations
------------------
Pour des informations sur l'installation, l'utilisation et la configuration, rendez-vous sur la [page Github du projet](https://github.com/GCorbel/sbire).

Encore plus d'informations
--------------------------
Si vous avez d'autres questions, contactez moi par email, à guirec.corbel@gmail.com, ou sur [twitter](https://twitter.com/GuirecCorbel).
