---
layout: post
title: "Single Responsability Principle"
date: 2014-07-27 19:32
comments: true
categories:
---

S'il n'y avait qu'un principe de Programation Orienté Objet à retenir, ça
serait celui-ci. Dans cet article, nous allons voir comment le Single
Responsability Principle peut aider à avoir un code plus clair et plus simple à
maintenir.
<!-- more -->

## Introduction

Le Single Responsability Principle (SRP) est le S des principes SOLID. Il a été
introduit par [Robert C. Martin](http://en.wikipedia.org/wiki/Robert_Cecil_Martin),
alias Uncle Bob. D'après celui-ci, une classe doit avoir une, et une seule, responsabilité.

Le langage utilisé dans cet article est le Ruby, mais le principe est applicable
dans tous les langages orientés objet.

## Exemple de violation du SRP

Commençons par un exemple ayant plus d'une responsabilité.

``` ruby
class User < ActiveRecord::Base
  validates :username, presence: true

  before_create :generate_token
  after_create :notify_administrator
  after_create :notify_user

  def to_param
    token
  end

  def notify_administrator
    UserMailer.mail_after_creation_to_administrator(self).deliver
  end

  def notify_user
    UserMailer.nail_after_creation_to_user(self).deliver
  end

  private

  def generate_token
    token = SecureRandom.urlsafe_base64
  end
end
```

Si l'on se demande quels sont les buts de cette classe, on comprend que celle-ci
s'occupe de la persistance des données, de la génération des tokens, de l'envoi
d'email après la création à l'administrateur et à l'utilisateur. Chacun de ces
buts est une responsabilité.

Cet exemple est très simple, mais est représentatif d'une classe qui grossit et
gagne en complexité rapidement. Il en va souvent de même pour les modèles, le M du
principe MVC. Ceux-ci sont des aspirateurs à responsabilité. Ceci est dû au
principe "Fat model, skinny controller" prôné par [Ruby on Rails](http://rubyonrails.org/) entre autres.

## Bénéfices

Pourquoi changer cette classe? Il existe plusieurs bénéfices résultant de
l'utilisation du Single Responsability Principle.

### Clarté

Premièrement, le fait de segmenter les fonctions en plusieurs classes permet
de clarifier le code. Les classes vont être moins longues et donc, plus lisibles.
Si l'on suit [les principes de Sandi Metz](http://robots.thoughtbot.com/sandi-metz-rules-for-developers),
une classe ne devrait pas dépasser cent lignes. Plus c'est court, plus c'est
facile à comprendre.

### Diminution de l'arité

Il sera plus facile d'instancier et d'utiliser ces classes. Une classe avec une
seule responsabilité aura besoin de moins d'arguments dans son constructeur.
Avec des classes simples, il devient facile de les instancier à partir de
n'importe quelle autre classe ou depuis la console.

### Simplifie les tests

Étant donné que les classes sont plus courtes, possèdent moins de fonctions et
sont plus faciles à instancier, elles sont plus faciles à tester. Un élément
difficile à tester est un symptôme, mais pas le problème. Utiliser le SRP
permet d'avoir des tests plus efficace, plus courts et plus compréhensibles.

### Maintenance

Les classes suivant ce principe sont également plus faciles à changer. Les
classes sont isolées les unes des autres, ce qui signifie qu'il en résulte
moins de "code spaghetti". Le changement d'une responsabilité est moins
susceptible d'en impacter une autre.

### Duck Typing

Enfin, une classe avec une seule responsabilité favorise l'utilisation du Duck
Typing étant donné qu'il peut y avoir une seule méthode. Il est possible de
nommer cette méthode de la même façon, `call` par exemple. Ce qui fait qu'il
sera toujours possible d'appeler la méthode ainsi `object.call`.

## Exemple de refactoring

``` ruby
class User < ActiveRecord::Base
  validates :username, presence: true

  def to_param
    token
  end
end

class UserCreator
  def initialize(user:)
    @user = user
  end

  def call
    if @user.save
      @user.token = generate_token
      send_notification
    end
  end

  private

  def send_notification
    UserCreationNotifier.new(user: @user).call
  end

  def generate_token
    TokenGenerator.new.call
  end
end

class UserCreationNotifier
  def initialize(user:)
    @user = user
  end

  def call
    notify_administrator
    notify_user
  end

  private

  def notify_administrator
    UserMailer.mail_after_creation_to_administrator(self).deliver
  end

  def notify_user
    UserMailer.nail_after_creation_to_user(self).deliver
  end
end

class TokenGenerator
  def call
    SecureRandom.urlsafe_base64
  end
end
```

Le premier modèle a été décomposé en plusieurs classes plus petites et plus simples.
La principale différence est au niveau de la class `User`. Celle-ci est
beaucoup plus simple et n'a qu'un seul but, celui de représenter la donnée.
`UserCreator` s'occupe uniquement du processus de création d'un utilisateur et
`UserCreationNotifier` s'occupe uniquement de la notification suite à la
création. `TokenGenerator`, comment son nom l'indique, génère des tokens. Il
s'agit d'une classe très réutilisable, pouvant servir dans de nombreux cas.

## Only one reason to change

Robert C. Martin décrit une responsabilité comme une raison de changer. Une
classe ou un module doit avoir seulement une raison de changer. Si l'on prend
la première version de la classe `User`, celle-ci devra changer si l'on décide
de générer des token différemment, ou encore si l'on change les notifications à
envoyer, ou si l'on change le type de persistance des données. Bref, cette
classe a définitivement plus d'une raison de changer.

Si l'on prend la proposition de refactoring en y pensant en terme de raison de
changement. La classe `User` changera uniquement si l'on décide de changer les
données à sauvegarder. La classe `UserCreator` changera si l'on modifie le
processus de création d'un utilisateur et `UserCreationNotifier` changera en
fonction des notifications que l'on veut effectuer lors de la création d'un
utilisateur. Enfin, `TokenGenerator` change si l'on souhaite changer la
manière de générer des tokens.

Il faut bien garder en tête que c'est la responsabilité qui doit avoir une
raison de changer et non le code. On peut imaginer dix mille raisons de
changer de code dans une classe, par exemple, changer le nom d'une fonction ou
d'une variable, mais ces changements sont acceptables. Tandis que, si l'on change
la responsabilité de la classe, on change sa raison d'être et il faut que ça
soit justifié.

## Personnifier la classe pour éviter les violations

Un des moyens pouvant être utilisés pour déterminer si une classe a plus d'une
responsabilité peut être de la personnifier et de lui demande quelles sont ces
responsabilités. Un petit exemple de dialogue :

Vous - Bonjour, madame `User`, que faites-vous dans la vie?<br/>
La classe - Je m'occupe de la persistance des données ET de la notification ET du changement des tokens.

Le mot clé est "ET". S'il y en a un dans la réponse à la question que vous avez
posé à la classe, c'est qu'elle à trop de responsabilités.

## Conclusion

Le Single Responsability Principe est un concept très important à comprendre.
Ceci dit, il faut l'utiliser avec du bon sens. Il y a une limite au bénéfice du
principe et il faut faire attention à ne pas tomber dans l'excès.

Chacun des principes se renforce mutuellement. Si l'on suit celui-ci, il sera
plus facile d'en utiliser un autre. Cependant, parfois, ils peuvent être
mutuellement exclusifs. Il n'existe pas de règle magique et il faut faire
preuve de bon sens. Avec le temps et l'expérience, tout cela devient plus
naturel.
