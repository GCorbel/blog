---
layout: post
title: "Benchmark : Clever Cloud vs Scalingo"
date: 2015-06-01 02:41
comments: true
categories:
---

Scalingo et Clever Cloud sont deux entreprises françaises proposant des
services d'hébergement dans le cloud. Celles-ci proposent de retirer la gestion
des serveurs aux développeurs et ceux avec un bon nombre de technologies comme
Ruby, Go, Python ou PHP. Après avoir mis à l'épreuve Heroku, Digital Ocean et
Scalingo, j'ai pu déterminer que Scalingo offrait la meilleure performance des
trois. Qu'en est-il de Clever Cloud ?

## Avant propos

Cet article se base sur [l'article précédent](http://gcorbel.github.io/blog/blog/2015/04/12/benchmark-digital-ocean-vs-scalingo/). L'application testée, le code
ainsi que les pages testées sont les mêmes que celles utilisées pour faire la
comparaison entre Digital Ocean et Scalingo.

**Je ne suis toujours pas administrateur système**. Si vous constatez des
irrégularités dans mes tests, n'hésitez pas à m'en faire part et je les
corrigerais. Cependant, je pense utiliser une méthode simple se basant sur des
faits afin d'avoir une analyse pertinente.

## L'application

[Les Collectionneurs Associés](http://www.associatedartcollectors.com/) est une application Rails qui n'a pas été conçue uniquement pour des tests. Cela nous
donne une meilleure idée du comportement des serveurs dans la réalité.

## Les serveurs

L'offre la plus petite de Clever Cloud coûte 14,40€ par mois. Pour ce prix,
nous avons 1 CPU et 1Gb de RAM. Pour le même prix, chez Scalingo, nous avons
une machine avec 1CPU et  512Mb de RAM. Cependant, avec Scalingo, il est
possible de descendre plus bas avec 256Mb pour 7,20€. J'ai choisi de prendre
l'offre à 14,40€ dans les deux entreprises. Les deux serveurs se trouvent en
France.

Le serveur à partir duquel je fais mes tests est un serveur fourni par
Digital Ocean situé à Londres. Il est donc représentatif des clients de
l'Europe de l'Ouest, incluant la France, évidemment.

## Le code

Le code n'est probablement pas le meilleur que j'ai écrit de ma vie, mais est
largement suffisant pour faire ce que je souhaitais.

```ruby
require 'benchmark'
require 'net/http'
require 'openssl'

NUMBER_OF_THREADS = 10
NUMBER_OF_REQUESTS_PER_THREAD = 100
URL = 'https://stagingcollectionneurs.scalingo.io/fr/provider_registrations/new'

threads = []
NUMBER_OF_THREADS.times do
  threads << Thread.new do
    Benchmark.bm do |x|
      x.report do
        NUMBER_OF_REQUESTS_PER_THREAD.times do |i|
          uri = URI(URL)
          http = Net::HTTP.new(uri.host, uri.port)
          http.use_ssl = true
          http.verify_mode = OpenSSL::SSL::VERIFY_NONE
          http.start do
            http.request_get(uri.path) do |res|
              p res.code + " : " + i.to_s
            end
          end
        end
      end
    end
  end
end
```

Ce code permet d'entrer une adresse et de simuler un certain nombre de
visites concurrentes faisant un certain nombre de requêtes. À la fin de
l'exécution du programme, le temps de réponse de chaque processus est affiché.
Ce code ne prend pas en compte le chargement des images et des assets. Dans mon
cas, ces éléments sont stockés sur un CDN et sont donc indépendants du serveur
où l'application se trouve.

## Test 1

Ce premier test affiche une page simple avec 11 requêtes. Voici les temps de
réponse :

Scalingo :

  126.364873
  126.482741
  126.609597
  126.741594
  126.857076
  126.988266
  127.107123
  127.235066
  127.356753
  127.469629

Clever Cloud :

  127.890590
  128.021063
  128.147565
  128.274117
  128.400284
  128.528602
  128.655974
  128.777820
  128.897637
  129.032317

Pour ce test, on s'aperçoit que les résultats sont quasiment identiques. La
différence de 2 secondes est, pour moi, négligeable.

## Test 2

Pour continuer, j'ai réexécuté le programme sur une page plus complexe. Voici
le résultat :

Scalingo :
  762.209817
  762.938170
  763.748126
  764.493016
  765.395998
  766.176983
  767.026748
  767.746600
  768.578646
  769.388282

Clever Cloud :

  766.513019
  767.324059
  768.050523
  768.837619
  769.624915
  770.354487
  771.162319
  771.894288
  772.703604
  773.544202

Encore une fois, la différence des temps de réponse est négligeable. Pour le
moment, les performances de l'application hébergée sur les serveurs des deux
entreprises sont équivalentes.

## Test 3

J'ai voulu poursuivre avec la page d'inscription, plus complexe.

Scalingo :

  415.211191
  415.640399
  416.049001
  416.437286
  416.847855
  417.337447
  417.744488
  418.145630
  418.528678
  418.999209

Clever Cloud :

  356.496478
  356.825021
  357.213072
  357.562785
  357.917548
  358.310875
  358.650679
  358.991046
  359.396792
  359.738898

Clever Cloud est légèrement plus rapide et, pour moi, ces résultats permettent
de rattraper les légers retards enregistrés précédemment.

## Conclusion

Voici une conclusion très ennuyante pour un article comparatif : les
performances des deux offres sont équivalentes. Ces résultats ne me permettent
pas de trancher entre l'un et l'autre.

Les deux entreprises offrent un
très bon support, leurs interfaces d'administration sont très bien faites et
l'utilisation est deux systèmes est très simple. Pour choisir, il n'y a pas
d'autre solution que de tester vous-même et de faire votre propre opinion, kit
à vous baser sur des notions plus subjectives.

Malgré tout, à la fois Scalingo et Clever Cloud sont plus performants qu'Heroku
et Digital Ocean. Je leur donne une position qui serait décevante aux JO, la
médaille d'or exequo.
