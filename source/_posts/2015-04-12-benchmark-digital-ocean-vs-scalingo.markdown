---
layout: post
title: "Benchmark : Digital Ocean vs Scalingo"
date: 2015-04-12 08:48
comments: true
categories:
---

Il y a peu, j'ai parlé avec Yann Klis, de [Scalingo](https://scalingo.com/), sur [le forum de Human Coders](https://forum.humancoders.com/t/heroku-en-production/1384/4). Suite à
cette discussion, j'ai souhaité effectuer un benchmark pour comparer les
performances de Digital Ocean et de Scalingo. Dans cet article, je vais exposer
les résultats des tests ainsi que ma méthodologie.

## L'application

Voici l'application testée : https://www.associatedartcollectors.com/fr.
Il s'agit d'une application Rails optimisée, avec un système de mise
en cache, des assets sur un CDN, etc. Il s'agit d'une vraie application
(encore en développement). J'ai préféré l'utiliser plutôt que d'en faire une
uniquement pour des tests afin de mieux représenter la réalité.

## Les serveurs

Le serveur Digital Ocean est situé à New York et coûte 10$ par mois pour
1Go de RAM. Scalingo, quant à lui, est situé en France et coûte 14.40€ pour
512Mb. Scalingo est donc légèrement plus cher pour moins de puissance.

Scalingo étant beaucoup plus facile à gérer, je pense que le temps économisé
au niveau de la gestion du serveur vaut largement le léger surplus de prix. En
effet, dans son utilisation, Scalingo ressemble davantage à Heroku qu'a Digital
Ocean. J'ai choisi de ne pas comparer Scalingo à Heroku, car, selon de précédents
tests, Digital Ocean offre une meilleure performance. J'ai voulu comparer
Scalingo au meilleur.

## Les tests

La plus grosse part de marché de Scalingo est située en France. J'ai souhaité
commencer mes tests proches de la France. Pour cela, j'ai choisi de créer un
droplet Digital Ocean à Londres.

Mon objectif a été de simuler un certain nombre de requêtes et ceux de manière
concurrente. J'ai donc créé un code qui lance plusieurs processus
simultanément. Chaque processus effectue un certain nombre de requêtes. Le
temps effectué par chaque processus permet de comparer la performance. Plus les
requêtes se sont complétées rapidement, plus le serveur est efficace.

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

Ce code permet d'entrer un nombre de processus simultané et le nombre de requêtes
fait pour chaque processus. De plus, il permet d'indiquer la page appelée par
le code. Pour les tests, j'ai choisi des pages différentes qui sont
représentatives de l'application.

Le code permet également d'afficher le statut de la requête envoyé, afin de
s'assurer qu'elles sont toutes valides, ainsi que le numéro de la requête
effectuée, juste pour voir la progression.

J'ai eu un problème avec le protocole SSL sur Digital Ocean. Pour le test, j'ai
simplement désactivé la vérification.

## Test 1

Le premier test consiste uniquement à visiter une page. Pour l'affichage de
cette page, 11 requêtes SQL sont exécutées.

Digital Ocean :

    Threads : 10
    Requests per thread : 100

    Temps de réponse :
      204.877640
      205.035337
      205.386872
      205.548623
      205.710381
      205.951070
      206.114616
      206.449934
      206.624353
      206.786742

Scalingo :

    Threads : 10
    Requests per thread : 100

    Temps de réponse :
      110.868637
      110.982509
      111.086731
      111.195775
      111.377322
      111.482194
      111.590034
      111.697870
      111.821420
      111.938271

Ce premier test montre clairement que Scalingo offre un temps de réponse plus
rapide. Scalingo est quasiment deux fois plus rapide.

## Test 2

Dans ce deuxième test, j'ai voulu tester le comportement avec plus de requêtes
concurrentes.

Digital Ocean :

    Threads : 100
    Requests per thread : 10

    Temps de réponse :
      192.305951
      192.493129
      192.630360
      ...
      211.465539
      211.620358
      211.787191
      212.134547
      212.285122

Scalingo :

    Threads : 100
    Requests per thread : 10

    Temps de réponse :
       92.787676
       92.877818
       92.959175
      ...
      102.423316
      102.525122
      102.622236

Encore cette fois, Scalingo est nettement supérieur. Pendant que les tests
s'effectuaient, je suis allé sur les deux sites avec mon navigateur. Même si cela
ne se reflète par sur les résultats du benchmark, j'ai constaté une grande
dégradation de performance pour les deux fournisseurs.

## Test 3

Par la suite, j'ai souhaité tester une autre page un peu plus complexe.

Digital Ocean :

    Thread : 10
    Request : 100

    Temps de réponse :
      277.214808
      277.399558
      277.598286
      277.996191
      278.207134
      278.479023
      278.705815
      279.098393
      279.326192
      279.617889

Scalingo :

    Thread : 10
    Request : 100

    Temps de réponse :
      143.032002
      143.170704
      143.322095
      143.458987
      143.596800
      143.742877
      143.877174
      144.017422
      144.161915
      144.303397

Sans surprise, le résultat est le même dans ce cas.

## Test 4

Enfin, j'ai voulu tester une page encore plus complexe. Il s'agit du
formulaire d'inscription d'un utilisateur. J'utilise des gems comme SimpleForm
pour gérer le formulaire. Les listes des services, des pays et des régions
sont disponibles ce qui correspond à autant de requête SQL et de donnée à
traiter.

Digital Ocean :

    Thread : 10
    Request : 100

    Temps de réponse :
      552.334913
      552.862069
      553.435137
      553.972282
      554.548237
      555.093758
      555.677602
      556.259342
      556.846054
      557.389199

Scalingo :

    Thread : 10
    Request : 100

    Temps de réponse :
      669.174683
      669.800881
      670.571744
      671.256995
      671.949939
      672.582819
      673.282322
      673.983594
      674.634513
      675.310771

Cette fois, Digital Ocean est plus rapide. Je pense que c'est là que la
différence de puissance du serveur se fait sentir. Étant donné qu'il y a plus
de code a exécuter, le traitement nécessite plus de ressource, et donc, Digital
Ocean est avantagé. Ceci dit, cette page n'est pas l'une des plus utilisées et
je pense que la performance est plus importante pour les pages permettant
uniquement l'affichage.

## Conclusion

Je pense que Scalingo gagne le duel. En plus d'offrir une performance plus
qu'honorable, la gestion du serveur est grandement simplifiée. Je conseille
vivement d'essayer cet outil si vous n'avez pas besoin d'avoir un accès complet
à la machine. Comme moi, les développeurs ne sont pas toujours des
administrateurs systèmes. Je trouve beaucoup plus simple de laisser la gestion
du serveur à quelqu'un d'autre. En plus d'être probablement mieux configuré que
ce que j'ai fait, c'est également plus sécuritaire, car, si une faille de
sécurité est découverte, le problème sera géré par des professionnels dans le
domaine.

Pour cet article, mes tests ont été effectués depuis Londres afin de
représenter la clientèle européenne. Pour ma part, je me situe au Québec. Dans
une seconde étape, je vais tester depuis des serveurs en Amérique du Nord et à
partir d'autres endroits dans le monde pour observer le comportement selon
les lieux.
