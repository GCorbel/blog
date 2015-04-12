---
layout: post
title: "Benchmark : Digital Ocean vs Scalingo"
date: 2015-04-12 08:48
comments: true
categories:
---

Il y a peu, j'ai parlé avec Yann Klis sur [le forum de Human Coders](https://forum.humancoders.com/t/heroku-en-production/1384/4). Suite à
cette discussion, j'ai souhaité effectuer un benchmark pour comparer les
performances de DigitalOcean et de Scalingo. Dans cet article, je vais exposer
les résultats des tests ainsi que ma méthodologie.

## L'application

Voici l'application en question : https://www.associatedartcollectors.com/fr.
Il s'agit d'une application Rails légèrement optimisée, avec un système de mise
en cache, des assets sur un CDN, etc. Il s'agit d'une vraie application
(encore en développement). J'ai préféré l'utiliser plutôt que d'en faire une
uniquement pour des tests afin de mieux représenter la réalité.

## Les serveurs

Le serveur Digital Ocean est situé à New York et il coûte 10$ par mois pour
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
le code. J'ai choisi des pages différentes qui sont représentatives de
l'application.

Le code permet également d'afficher le statut de la requête envoyé, afin de
s'assurer qu'elles sont toutes valides, ainsi que le numéro de la requête
effectuée, juste pour voir la progression.

J'ai eu un problème avec le protocole SSL sur Digital Ocean. Pour le test, j'ai
simplement désactivé la vérification.

## Test 1

Le premier test consiste uniquement à visiter une page. Pour l'affichage de
cette page, 11 requêtes SQL sont exécutées.

**Le résultat à observer est le dernier, entre parenthèses. Les autres chiffres
représentent les temps d'exécutions du script sur le système**

Digital Ocean :

    Threads : 10
    Requests per thread : 100

      3.900000   0.680000   4.580000 (204.877640)
      3.900000   0.680000   4.580000 (205.035337)
      3.910000   0.680000   4.590000 (205.386872)
      3.910000   0.680000   4.590000 (205.548623)
      3.910000   0.680000   4.590000 (205.710381)
      3.910000   0.680000   4.590000 (205.951070)
      3.910000   0.680000   4.590000 (206.114616)
      3.900000   0.680000   4.580000 (206.449934)
      3.920000   0.680000   4.600000 (206.624353)
      3.920000   0.680000   4.600000 (206.786742)

Scalingo :

    Threads : 10
    Requests per thread : 100

      4.150000   0.610000   4.760000 (110.868637)
      4.150000   0.610000   4.760000 (110.982509)
      4.150000   0.610000   4.760000 (111.086731)
      4.150000   0.610000   4.760000 (111.195775)
      4.160000   0.610000   4.770000 (111.377322)
      4.160000   0.610000   4.770000 (111.482194)
      4.160000   0.610000   4.770000 (111.590034)
      4.160000   0.610000   4.770000 (111.697870)
      4.160000   0.610000   4.770000 (111.821420)
      4.170000   0.610000   4.780000 (111.938271)

Ce premier test montre clairement que Scalingo offre un temps de réponse plus rapide.

## Test 2

Dans ce deuxième test, j'ai voulu tester le comportement avec plus de requêtes concurrentes.

Digital Ocean :

    Threads : 100
    Requests per thread : 10

      4.060000   0.610000   4.670000 (192.305951)
      4.050000   0.600000   4.650000 (192.493129)
      4.060000   0.610000   4.670000 (192.630360)
      ...
      4.210000   0.640000   4.850000 (211.465539)
      4.210000   0.640000   4.850000 (211.620358)
      4.220000   0.640000   4.860000 (211.787191)
      4.220000   0.650000   4.870000 (212.134547)
      4.220000   0.640000   4.860000 (212.285122)

Scalingo :

    Threads : 100
    Requests per thread : 10

      4.090000   0.620000   4.710000 ( 92.787676)
      4.090000   0.620000   4.710000 ( 92.877818)
      4.090000   0.620000   4.710000 ( 92.959175)
      ...
      4.240000   0.640000   4.880000 (102.423316)
      4.260000   0.640000   4.900000 (102.525122)
      4.260000   0.640000   4.900000 (102.622236)

Encore cette fois, Scalingo est nettement supérieur. Pendant que les tests
s'effectuaient, je suis allé sur les deux sites avec mon navigateur. Même si cela
ne se reflète par sur les résultats du benchmark, j'ai constaté une grande
dégradation de performance pour les deux fournisseurs.

## Test 3

Par la suite, j'ai souhaité tester une autre page un peu plus complexe.

Digital Ocean :

    Thread : 10
    Request : 100

      4.080000   0.590000   4.670000 (277.214808)
      4.070000   0.590000   4.660000 (277.399558)
      4.080000   0.590000   4.670000 (277.598286)
      4.080000   0.590000   4.670000 (277.996191)
      4.090000   0.590000   4.680000 (278.207134)
      4.080000   0.590000   4.670000 (278.479023)
      4.090000   0.590000   4.680000 (278.705815)
      4.090000   0.590000   4.680000 (279.098393)
      4.090000   0.600000   4.690000 (279.326192)
      4.090000   0.600000   4.690000 (279.617889)

Scalingo :

    Thread : 10
    Request : 100

      4.190000   0.560000   4.750000 (143.032002)
      4.200000   0.560000   4.760000 (143.170704)
      4.200000   0.560000   4.760000 (143.322095)
      4.200000   0.560000   4.760000 (143.458987)
      4.200000   0.560000   4.760000 (143.596800)
      4.200000   0.560000   4.760000 (143.742877)
      4.200000   0.560000   4.760000 (143.877174)
      4.200000   0.560000   4.760000 (144.017422)
      4.200000   0.560000   4.760000 (144.161915)
      4.200000   0.570000   4.770000 (144.303397)

Sans surprise, le résultat est le même dans ce cas.

## Test 4

Pour conclure, j'ai voulu tester une page encore plus complexe. Il s'agit du
formulaire d'inscription d'un utilisateur. J'utilise des gems comme SimpleForm
pour gérer le formulaire. Les listes des services, des pays et des régions
sont disponibles ce qui correspond à autant de requête SQL et de donnée à
traiter.

Digital Ocean :

    Thread : 10
    Request : 100

      4.270000   0.610000   4.880000 (552.334913)
      4.270000   0.610000   4.880000 (552.862069)
      4.270000   0.610000   4.880000 (553.435137)
      4.280000   0.610000   4.890000 (553.972282)
      4.280000   0.610000   4.890000 (554.548237)
      4.280000   0.610000   4.890000 (555.093758)
      4.280000   0.610000   4.890000 (555.677602)
      4.280000   0.610000   4.890000 (556.259342)
      4.280000   0.610000   4.890000 (556.846054)
      4.290000   0.610000   4.900000 (557.389199)

Scalingo :

    Thread : 10
    Request : 100

      4.550000   0.620000   5.170000 (669.174683)
      4.550000   0.620000   5.170000 (669.800881)
      4.550000   0.620000   5.170000 (670.571744)
      4.550000   0.620000   5.170000 (671.256995)
      4.550000   0.620000   5.170000 (671.949939)
      4.560000   0.620000   5.180000 (672.582819)
      4.560000   0.620000   5.180000 (673.282322)
      4.560000   0.620000   5.180000 (673.983594)
      4.560000   0.620000   5.180000 (674.634513)
      4.560000   0.620000   5.180000 (675.310771)

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
à la machine. Comme moi, les développeurs ne sont toujours des administrateurs
systèmes. Je trouve beaucoup plus simple de laisser la gestion du serveur à
quelqu'un d'autre. En plus d'être probablement mieux configuré que ce que j'ai
fait, c'est également plus sécuritaire, car, si une faille de sécurité est
découverte, le problème sera géré par des professionnels dans le domaine.

Pour cet article, mes tests ont été effectués depuis Londres afin de
représenter la clientèle européenne. Pour ma part, je me situe au Québec. Dans
une seconde étape, je vais tester depuis des serveurs en Amérique du Nord et à
partir d'autres endroits dans le monde pour observer le comportement selon
les lieux.
