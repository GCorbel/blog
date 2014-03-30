---
layout: post
title: "PyLeapMouse: Commandez votre OS par la voix ET les mouvements"
date: 2013-11-01 19:34
comments: true
categories:
- Leap Motion
- Sbire
---

Suite à [mon article et ma vidéo](http://gcorbel.github.io/blog/blog/2013/10/27/sbire-dites-ce-que-vous-voulez-il-le-fera/) sur [Sbire](https://github.com/gcorbel/sbire), je poursuis dans ma lancée pour expliquer comment j'ai modifier le projet PyLeapMouse pour coupler LeapMotion et Sbire.
[LeapMotion](https://www.leapmotion.com/) est un très bon outil permettant de détecter les mouvements. L'une des fonctionnalités évidentes que l'on imagine rapidement avec LeapMotion est de remplacer la souris pour commander le système d'exploitation.
Pour cela, il existe le projet [openleap/PyLeapMouse](https://github.com/openleap/PyLeapMouse). Celui-ci émule le pointeur de la souris et permet de simuler les clics.
Dans cet article, vous verrez le cheminement que j'ai effectué pour contrôler l'OS avec LeapMotion.
<!--more-->

Exemple d'utilisation
---------------------
Avant de commencer, voici une courte vidéo montrant l'utilisation de PyLeapMouse et Sbire.

{% youtube l1BxiD1dx5s %}

Diriger la souris/Executer des commandes?
-----------------------------------------
Mon objectif principal pour cette application est de contrôler l'OS avec LeapMotion. En tant qu'adepte de l'Open-Source, j'ai premièrement cherché s'il existait un projet le permettant. Je suis tombé sur [LeapMouse](https://github.com/archetipo/leapmouse), écrit en C. J'ai rapidement vu les problèmes existants pour diriger la souris. Pour ma part, je mettais plus de 10 secondes pour réussir à fixer un bouton ou une icône. Sur ce point, l'utilisation classique de la souris est beaucoup plus efficace.
De plus, il est nécessaire de maintenir la main à une certaine distance du contrôleur. On doit donc la maintenir en hauteur ce qui est rapidement fatigant.

Le fait d'effectuer des commandes en fonction des mouvements m'a paru une méthode plus efficace et plus précise. Tout d'abord, une commande doit être actionnée par un mouvement court. Il n'est donc pas nécessaire de maintenir la main en suspension pendant une longue période. De plus, le besoin de précision est moindre ce qui rend l'utilisation plus efficace.

Pourquoi Python?
----------------
Il existe plusieurs SDK pour LeapMotion. Malheureusement pour moi, Ruby, mon langage préféré, n'est pas disponible officiellement. J'avais donc l'embarras du choix des langages.

LeapMouse étant écrit en C, je me suis demandé pourquoi choisir ce langage plutôt qu'un autre. Il est vrai que ce langage possède l'avantage d'être plus bas niveau. La plupart des outils disponibles dans les autres langages sont des dérivés. Également, le C est très avantageux au niveau de la performance. Ces deux points constituent des avantages non négligeables pour l'utilisation du C.

Cela faisait longtemps que je n'avais pas utilisé le C. Étant habitué à Ruby, j'avais un peu d'aprioris à utiliser un langage moins moderne. Je pensais avoir plus de mal avec la syntaxe, mais finalement c'est la difficulté à inclure des librairies qui m'a repoussé. Grâce à bundler, Ruby possède une manière très simple de gérer les dépendances d'un projet. En C, c'est parfois aussi simple de refaire des bouts de code. Réinventer la roue n'étant pas ce que je préfère, j'ai abandonné le C et pris la résolution de l'utiliser uniquement si je n'ai pas le choix. Évidemment, c'est une conclusion très personnelle. Libre à chacun de choisir autre chose.

Python est un langage tout aussi moderne que Ruby. La gestion des dépendances se fait également très bien avec Pip. Bien que la philosophie soit différente que celle de Ruby, il est très facile de passer de l'un à l'autre. De plus, c'est toujours agréable de découvrir autre chose et de changer un peu.

Pourquoi PyLeapMouse?
---------------------
Ensuite, j'ai choisi de modifier [openleap/PyLeapMouse](https://github.com/openleap/PyLeapMouse). Tout d'abord PyLeapMouse possède déjà un bon nombre de fonctionnalités. De plus, il est relativement populaire et possède déjà une bonne communauté. Ce choix fut assez rapide. Je crois que c'est le projet le plus abouti concernant le contrôle de la souris avec LeapMotion.

Mon développement sur PyLeapMouse
---------------------------------
Malgré le fait que PyLeapMouse soit fonctionnel, l'utiliser pour déplacer le pointeur de la souris est très fastidieux. Ceci est dû aux limites expliquées dans [cet article](http://gcorbel.github.io/blog/blog/2013/10/12/leap-motion-le-point-de-vue-dun-developpeur/).

J'ai donc fait un fork du projet pour ajouter la possibilité d'exécuter des commandes en fonctions des mouvements effectuées. Le fonctionnement est assez simple. PyLeapMouse va lire un fichier `commands.ini` comme celui-ci :

    [screentap]

    [keytap]

    [swiperight]
    1finger: rhythmbox-client --next

    [swipeleft]
    1finger: rhythmbox-client --previous

    [clockwise]
    1finger: sbire start
    2finger: sbire start
    3finger: sbire start
    4finger: sbire start
    5finger: sbire start

    [counterclockwise]
    1finger: sbire stop
    2finger: sbire stop
    3finger: sbire stop
    4finger: sbire stop
    5finger: sbire stop

À chaque changement détecté par LeapMotion, le programme recherche dans une liste d'objets avec des classes comme celle-ci :

``` python
class KeytapCommand():
    def __init__(self):
        self.name = "keytap"

    def applicable(self, frame):
        return(frame.gestures()[0].type == Leap.Gesture.TYPE_KEY_TAP)
```

Chaque commande possède un nom, permettant de l'identifier, et une fonction `applicable` permettant de voir si les conditions sont réunies pour exécuter la commande en fonction de la variable `frame` représentant les données passées par LeapMotion.

Ensuite, il va rechercher dans la configuration quelle commande il doit exécuter en fonction du nombre de doigts reconnus et du mouvement. Pour plus de détails, vous pouvez voir le [code complet sur GitHub](https://github.com/openleap/PyLeapMouse/blob/master/MotionControl.py).

Cette amélioration rend l'utilisation plus efficace, mais n'est pas fiable à 100%. Il y a très souvent des imperfections dans les mouvements détectés.

Afin d'augmenter la précision, il existe des projets comme [LeapTrainer](https://github.com/roboleary/LeapTrainer.js). Celui-ci fait en sorte que le système apprend les mouvements. Il est ensuite possible d'exporter le mouvement appris puis de l'importer dans d'autres outils. C'est effectivement un très bon projet, mais cela manque encore d'efficacité.

Pourquoi lier avec Sbire?
-------------------------
Malrgès le fait que lier les commandes aux mouvements soit plus précis, il reste encore beaucoup d'erreurs. C'est pour cette raison qu'il est mieux de le coupler avec Sbire. Sbire est plus précis et les possibilités sont plus vastes. LeapMotion possède une petite liste de mouvements prédéfinis. Sbire, quant à lui, peut reconnaitre un nombre illimité de phrases pour chercher les commandes à exécuter.

Conclusion
----------
Je pense que le couplage des deux outils est plutôt bien. Cependant, je trouve qu'utiliser un raccourci clavier pour démarrer et arrêter Sbire est encore une solution plus efficace. Je crois que LeapMotion a encore du travail à faire pour rendre l'outil 100% fonctionnel.

À sa décharge, LeapMotion est encore en développement et les développeurs se font une priorité d'augmenter la précision dans les prochaines versions. Je suis encore plein d'espoir pour cet outil.
