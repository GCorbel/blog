---
layout: post
title: Design Principles et Design Patterns avec Rails
date: 2014-05-18 06:52
comments: true
categories:
---

Losque l'on débute notre vie de programmeur, l'architecture du code est secondaire. En prenant de l'expérience, on se rend compte qu'une application a besoin d'une structure quand sa taille augmente. Il est possible de développer ces propres techniques, mais d'autres personnes sont passées par le même chemin et certains ont publié leurs solutions. Ce sont les Principes et les Design Patterns. Cet article en présente quelques-unes appliquées à Ruby on Rails.

<!--more-->

## Introduction

Cet article est une brève présentation des Design Principles, ou Principes de Conception, et des Design Patterns, ou Patrons de Conception. Le but de cet article n'est pas de détailler chacun de ces concepts, mais de donner un bref aperçu de ceux-ci. D'autres articles suivront pour rentrer plus dans les détails.

## Programation Orienté Objet

Un des principes fondamentaux, en Ruby comme dans d'autres langages, est la Programation Orienté Objet (POO). Il est parfaitement possible d'écrire une application en code en procédural, mais il est vite difficile de la maintenir efficacement. Afin de faire une séparation logique du code, il est possible de le regrouper en classe et en méthode. Pour faire une bonne séparation, il est important de bien comprendre quels sont les principes existants. De plus, la POO est bien plus que mettre des bouts de cote dans des catégories. Il s'agit d'une manière de penser le code.

## Design Principles

Dans le livre [Practical Object-Oriented Design in Ruby: An Agile Primer](http://www.amazon.ca/Practical-Object-Oriented-Design-Ruby-Primer/dp/0321721330), Sandi Metz définit qu'une application doit être TRUE :

* **T**ransparent : Il doit être facile de comprendre quels vont être les impacts lors du changement d'un code;
* **R**easonable : Le coût du changement doit impliquer un gain proportionnel;
* **U**sable : Le code doit pouvoir être utilisé dans différents contextes;
* **E**xemplary : Le code produit doit encourager une meilleure qualité de la conception;

Pour nous aider dans cette démarche, [Robert C. Martin](http://en.wikipedia.org/wiki/Robert_Cecil_Martin) a défini cinq Design Principles communément acceptés. Une application doit être [SOLID](http://en.wikipedia.org/wiki/SOLID_%28object-oriented_design%29), l'acronyme pour :

* [**S**ingle Responsability Principle](http://en.wikipedia.org/wiki/Single_responsibility_principle) - Une classe doit avoir une seule responsabilité;
* [**O**pen/Clode Priciple](http://en.wikipedia.org/wiki/Open/closed_principle) - Une classe doit être ouverte pour l'extension, mais fermée pour la modification;
* [**L**iskov Substition Principle](http://en.wikipedia.org/wiki/Liskov_substitution_principle) - Les types dérivés doivent pouvoir être remplacés par leur type de base;
* [**I**nterface Segregation Principle](http://en.wikipedia.org/wiki/Interface_segregation_principle) - Une classe ne doit pas dépendre d'une interface contenant des méthodes qu'elle n'utilise pas;
* [**D**ependency Inversion Principle](http://en.wikipedia.org/wiki/Dependency_inversion_principle) - Un module de haut niveau ne doit pas dépendre d'un module de bas niveau;

Ce qu'il y a d'intéressant avec ces principes, c'est qu'ils se renforcent mutuellement. Si l'on en utilise certains, les autres vont être plus évidents ce qui encouragera une meilleure qualité.

Rails prône également les principes de [Convention Over Configuration](http://en.wikipedia.org/wiki/Convention_over_Configuration), qui vise a simplifier le développement en utilisant des conventions de nommage plutôt que des fichiers de configuration, et [Don't Repeat Yourself](http://en.wikipedia.org/wiki/Don%27t_Repeat_Yourself), qui dit que l'on ne devrait jamais dupliquer du code.

Le plus discutable prôné par Rails est Fat models, skinny controllers. Celui-ci vise à garder le contrôleur le plus simple possible et à ajouter la logique dans le modèle. Pourquoi discutable? Parce que je crois, comme d'autres, que les modèles deviennent trop complexes et difficiles à tester.

Il n'est pas toujours facile de faire l'unanimité sur des techniques. Heureusement, [Rails is Omakasee](http://david.heinemeierhansson.com/2012/rails-is-omakase.html). Le framework est suffisamment souple pour nous laisser le choix des techniques à utiliser. La "Rails Way" est de faire du "Fat models, skinny controllers" mais il est tout à fait possible d'introduire d'autres concepts et de créer de bons vieux purs objets Ruby.

## Desgin Patterns

Les Design Patterns sont des types de solutions répondant à certains problèmes communs. En programmation web, je crois que le plus connu est [MVC](http://en.wikipedia.org/wiki/MVC). Celui-ci veut qu'une requête HTTP soit traitée par un routeur qui instancie un contrôleur qui a pour but d'instancier les modèles et de renvoyer une vue.

Il existe bien d'autres Design Patterns. Avec Rails, mes préférés sont :

* Service Objects - Permet d'éviter la boulimie des modèles en extrayant les responsabilités dans des classes séparées;
* Form Objects - Simplifie le traitement des formulaires complexes;
* Decorators - Permet d'ajouter aux modèles des fonctions relatives à la vue;
* Presenters - Extrait les logiques complexes de la vue dans une classe;

Lorsque l'on s'attaque à un problème, il est intelligent de se renseigner d'abord sur les solutions qui existent. Une fois appliqué, on apprend à reconnaitre les patterns et l'on applique plus vite les solutions. Le temps passé à faire la recherche initiale est largement rentabilisé avec le temps.

## Quand utiliser un Design Pattern?

Savoir quand utiliser un Design Pattern, peut paraitre anodin, mais il y a un autre principe à suivre [Keep It Simple and Stupid](http://en.wikipedia.org/wiki/Keep_it_simple) (KISS). Ils doivent être utilisés quand il y a un problème ou en prévision d'un problème. Parfois, il ne sert à rien de se casser la tête. Si le programme est propre et facile a modifier, il ne sert à rien d'utiliser des techniques complexes.

## Quel Design Pattern utiliser?

Il est parfois difficile de choisir quel Design Pattern utilisé. D'autant plus que certains peuvent se contredisent. Par exemple, il est difficile d'utiliser Tell Don't Ask dans un contrôleur quand viens le temps de savoir quelle vue retourner. Malheureusement, il n'y a pas de règles simples. Il faut savoir utiliser le bon sens et prendre des décisions.

## L'impact psychologique

Je crois que l'impact psychologique des principes et des Design Patterns est non négligeable. C'est beaucoup plus agréable de travailler sur un projet bien structuré que de modifier du code sans savoir quels vont être les répercussions ou devoir modifier une grosse partie du code. Ruby est un langage fait pour rendre le développeur heureux. Chacun sait qu'un développeur moins stressé sera plus productif.

De plus, une bonne qualité de code fera de vous fier de ce que vous faite. C'est super encourageant de rentrer tous les matins au travail en sachant que l'on fait du bon travail.

## Conclusion

Ces sujets sont beaucoup trop complexes pour être écrit en un seul article. Ils mériteraient des livres. D'ailleurs, il y en a un paquet. Parmi ceux-ci, on peut citer [Clean Code: A Handbook of Agile Software Craftsmanship](http://www.amazon.ca/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882) de Robert C. Martin, [Practical Object-Oriented Design in Ruby: An Agile Primer](http://www.amazon.ca/Practical-Object-Oriented-Design-Ruby-Primer/dp/0321721330), de Sandi Metz ou [Head First Design Patterns](http://www.amazon.ca/Head-First-Design-Patterns-Freeman/dp/0596007124) de Eric Freeman, Elisabeth Robson, Bert Bates et Kathy Sierra.

Comme indiqué ci-dessus, je vais bientôt écrire plusieurs articles et Design Pattern, so [Follow me on Twitter](https://twitter.com/GuirecCorbel).
