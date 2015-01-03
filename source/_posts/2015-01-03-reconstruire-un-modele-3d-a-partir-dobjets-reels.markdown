---
layout: post
title: "Reconstruire un modèle 3D à partir d'objets réels"
date: 2015-01-03 07:59
comments: true
categories:
---

La reconstitution d'objets 3d basée sur des images permet de créer des modèles
3d d'objets réels. Ce processus qui peut paraître complexe est en fait rendu
très simple grâce aux outils que nous allons voir dans cet article.

<!-- more -->

## Introduction

À la base, mon objectif était de faire la cartographie 3d avec mon drone en
suivant les étapes indiquées
[ici](http://copter.ardupilot.com/wiki/common-3d-mapping/). La cartographie 3D
utilise la technique de reconstruction 3d à partir d'images (image based 3d
reconstruction). C'est cette technique que je vous propose d'approfondir dans
cet article.

## Reconstitution 3d à partir d'image vs Scanner 3d

Il existe [plusieurs types de scanneurs 3d](http://fr.wikipedia.org/wiki/Scanner_tridimensionnel). Ceux-ci produisent
généralement des modèles de meilleure qualité que ceux obtenus en utilisant des images. Ceci
est dû au fait qu'ils utilisent différents types de données comme des lasers ou
des sonars en plus des images. La raison pour laquelle j'ai tout de même
préféré utiliser la reconstitution 3d à partir d'image est tout simplement le
prix. En effet, un scanneur 3d coûte entre 300$ et 100000$ ou plus. Bien qu'il
soit possible d'en avoir à prix réduit si l'on est prêt à faire [un peu de bricolage](http://3dprintingindustry.com/2014/08/05/make-3dollar-3d-scanner/),
j'ai choisi de rester dans la simplicité pour le moment.

## Le principe

Le principe est à la fois simple et compliqué. Un algorithme complexe va
trouver des points similaires entre les différentes images et ainsi pouvoir
les replacer dans un environnement. Par triangulation, l'algorithme est capable
de reconstituer un nuage de point représentant l'objet. Il va ensuite
reconstituer la surface puis appliquer une texture avec les images.

![Visual SFM](http://www.ismailsirma.com/wp-content/uploads/2014/12/3-sparse-reconstruction.jpg)

## Logiciels de reconstitution 3d à partir d'images

Il existe plusieurs logiciels de ce type et j'en ai essayé quelques-uns :

 - [VisualSFM](http://ccwu.me/vsfm/) et [CMVS](http://www.di.ens.fr/cmvs/) : Deux produits open source. Je n'ai malheureusement pas
 eu le rendu souhaité;
 - [Agisoft Photoscan](http://www.agisoft.com/) : Logiciel payant très simple d'utilisation donnant un
 résultat satisfaisant;
 - [Trnio](http://www.trnio.com/) : Application iPhone très simple, mais un rendu limité;
 - [Autodesk 123d catch](http://www.123dapp.com/catch) : Application web et mobile (iPhone et Androïd);

Selon mon expérience, j'ai eu les meilleurs résultats avec 123d catch.
De plus, tous les calculs se font sur leurs
serveurs, il n'y a donc pas besoin de bloquer un PC pendant une heure, car le
taux d'utilisation du CPU est élevé, contrairement à VisualSFM ou Photoscan.
Enfin, Autodesk possède une gamme d'outils très complète dans le domaine. 123d
catch est un outil à la fois simple et puissant.

Toutefois, je n'exclus pas encore Trnio. Jan-Michael Tressler, le PDG de Trnio,
m'assure que le rendu va être amélioré prochainement. Je fais d'ailleur parti
des beta-testeurs de l'application.

Pour ce qui est de VisualSFM, je pense que le fait d'avoir une carte graphique
NVidia Cuda peut aider comme indiquer sur leur site.

## Comment faire?

J'ai choisi de télécharger l'application iPhone de 123d catch étant donné que
cela évite les problèmes qu'il peut y avoir avec les différences entres les
appareils photo. Si Autodesk a fait fonctionner leur application avec un
iPhone, ça devrait fonctionner avec le mien.

Il faut ensuite placer un objet et en faire le tour. L'application va vous dire
comment vous placer pour prendre vos photos. L'application va ensuite vous
permettre d'uploader vos images puis va passer par plusieurs phases avant
d'avoir un modèle. Environ une demi-heure plus tard, vous aurez une
notification indiquant que le traitement et fini. Vous pouvez ensuite
visualiser et publier votre modèle.

Il est très important de faire attention à l'éclairage. Le choix du type
d'objet est également important. Il est nécessaire que l'objet ait des détails
sur lesquels le logiciel peut se fier pour faire son traitement. Il faut éviter
les objets avec trop de trous et de creux.


## Exemple

Voici [un exemple de modélisation](http://www.123dapp.com/catch/2014-10-18-23-34-9/2949812):

[![Modélisation 3d](http://sherpapreview-standard.s3-website-us-east-1.amazonaws.com/Preview/2014/10/13__15_12_58/IMG_3168.JPG32aece52-5720-11e4-8e14-22000b250b09Large.jpg)](http://www.123dapp.com/catch/2014-10-18-23-34-9/2949812)

123d propose de télécharger le modèle sous 3 formats, obj, 3dp ou stl. À vous
de voir selon vos besoins.

## Conclusion

Et voilà! Nous avons maintenant un modèle 3d d'un objet réel. Dans le prochain
article, je vous présenterais une méthode simple pour intégrer le modèle 3d
dans une scène créée avec Unity afin d'utiliser Oculus Rift.
