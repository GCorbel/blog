---
layout: post
title: "Benchmark entre DigitalOcean et Heroku"
date: 2014-03-29 14:10
comments: true
categories:
---

Aujourd'hui, j'avais envie de mettre des chiffres sur un fait que je pensais avéré. J'ai voulu faire un comparitif simple entre l'offre gratuite d'Heroku et celle la plus populaire de Digital Ocean.

## L'application ##

L'application utilisée dans ce cas est une application Ruby on Rails standard utilisant Rails 3.2.17, Ruby 2.1.0 et Unicorn 4.8.2.

## Les outils mesures ##

Pour mesurer les performences du serveur, j'ai utilisé [New Relic](http://newrelic.com/). Il s'agit d'un excellent utilitaire qui indique en détail les performences d'une application ainsi les différents points sensibles.

J'ai également utilisé [Load Impact](http://loadimpact.com/), un outil au fonctionnement assez simple. Il suffit d'enregistrer un scénario pour pouvoir le rejouer en simulant un certain nombre de visiteurs. Load Impact fait en sorte d'augmenter peu à peu le nombre de visiteurs sur le site comme on peu le voir sur cette image :

{% img http://img11.hostingpics.net/pics/628030Slection004.png %}

Grâce à cet outil, j'ai pu créer des scénarios et les jouers en simulant 50 visiteurs. L'application comporte 3 types d'utilisateurs, j'ai donc créé un scénario dans chacun de ces cas.

## Les chiffres ##

### DigitalOcean ###

J'ai utilisé l'offre à 10$ par mois, la plus populaire,  de Digital Ocean pour effectuer ce test et voici le résultat :

{% img http://img11.hostingpics.net/pics/410612Slection007.png %}

Comme on peut le voir, le temps de réponse est de moins de 800ms. Le temps de traitement est partagé entre Ruby et la base de données. Je crois qu'il s'agit d'un comportement normal et d'un bon temps de réponse.

### Heroku ###

J'ai répété exactement le même test avec Heroku et voici le résultat :

{% img http://img11.hostingpics.net/pics/502656Slection006.png %}

On voit clairement qu'Heroku est le grand perdant du duel. Je me doutais déjà du résultat, mais je ne pensais pas que la différence serait aussi probante. Le temps de réponse est supérieur à 25 secondes. La majeure partie du temps est prise dans la queue de traitement. On peut également voir qu'Heroku a eu du mal à se remettre du test sur ce graphique :

{% img http://img11.hostingpics.net/pics/804402Slection005.png %}

Avant 10h, il s'agit du trafic normal. La pointe verte, entre 10h et 10h30 représente le pic lors du test effectué. On peut voir qu'après cette pointe, tous les temps sont très longs et la situation n'estpas revenue à la normale.

## Conclusion ##

De part cet exemple, on peut voir assez clairement que DigitalOcean est plus rapide qu'Heroku.

Les résultats obtenus ici **ne montrent pas définitivement qu'Heroku est moins efficace que DigitalOcean**. Premièrement, je suis loin d'avoir fait des tests exhaustifs. Pour aller plus loin, il serait nécessaire de faire des essais avec les offres payantes d'Heroku ainsi qu'en essayant d'autres scénarios et cas d'utilisation. De plus, Heroku reste génial comme environnement de développement et de test.

Comme d'habitude, je ne prétend pas détenir la vérité. Si vous avez d'autres arguments permettant de confirmer ou d'infirmer ma conclusion, s'il vous plaît, laissez vos commentaires.
