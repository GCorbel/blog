---
layout: post
title: "Open/Closed Principle"
date: 2014-08-25 04:52
comments: true
categories:
---

Le principe d'ouverture/fermeture, où OCP, permet de créer un code plus clair et
plus facilement modifiable. Dans cet article, nous allons voir dans quels cas il
peut être utilisé et quels en sont les bénéfices. OCP fait partie des principes
SOLID, qu'il faut absolument connaître pour devenir un meilleur programmeur.
<!-- more -->

## Introduction

L'Open/Closed Principle est le O des principes SOLID. Il a été
introduit par [Bertrand Meyer](http://fr.wikipedia.org/wiki/Bertrand_Meyer) (un français). Si l'on suit ce principe, un élément
dans un programme doit être "Ouvert pour l'extension, mais fermé pour la
modification". On doit donc pouvoir ajouter des fonctionnalités à une classe,
un module, une fonction, etc. sans pour autant la modifier.

Cet article est piloté par l'exemple, beaucoup de code et peu de blabla. Il
est écrit en Ruby, mais est utilisable dans n'importe quel autre langage.

L'article est le deuxième d'une série concernant les principes SOLID. Le
premier porte sur [SRP](http://gcorbel.github.io/blog/blog/2014/07/27/single-responsability-principle/).
Dans cet article, vous trouverez des références au SRP. Si ce n'est pas déjà
fait, je vous conseille de lire le premier article avant celui-ci.

## Exemple 1

Sans plus attendre, voici un premier exemple de code ne respectant pas OCP :

```ruby
class UserSubscription
  def initialize(user)
    @user = user
  end

  def call
    @user.save
    TwitterNotifier.new.notify
  end
end

class TwitterNotifier
  def notify
    puts "Notify Twitter"
  end
end

UserSubscription.new(user).call
```

Comme on peut le voir, la classe `TwitterNotifier` est instanciée dans la
méthode `#call` de `UserSubscription`. Si l'on souhaite ne plus envoyer de
notifications via Twitter, mais via Facebook, il va falloir rouvrir la classe
`UserSubscription` et remplacer la classe appelée.

Voici un exemple faisant la même chose, mais suivant le principe.

```ruby
class UserSubscription
  def initialize(user)
    @user = user
  end

  def call(notifier)
    @user.save
    notifier.notify
  end
end

class TwitterNotifier
  def notify
    puts "Notify Twitter"
  end
end

UserSubscription.new(user).call(TwitterNotifier.new)
```

L'appel de `TwitterNotifier.new` se fait à un niveau supérieur et il est
possible de changer le comportement de `UserSubscription` sans changer son
implémentation.

## Exemple 2

Si un changement au programme est demandé pour que la notification soit
différente selon le type d'utilisateur, sans suivre OCP, nous pouvons écrire
ceci :

```ruby
class UserSubscription
  def initialize(user)
    @user = user
  end

  def call(notifier)
    @user.save
    if @user.admin?
      notifier.notify("Admin Message")
    elsif @user.editor?
      notifier.notify("Editor Message")
    else
      notifier.notify("Other Message")
    end
  end
end

class TwitterNotifier
  def notify(text)
    puts text
  end
end

UserSubscription.new(user).call(TwitterNotifier.new)
```

À chaque nouveau type d'utilisateur, il va falloir rouvrir la classe
`UserSubscription`. Il est plus facile de modifier la classe comme ceci :

```ruby
class UserSubscription
  def initialize(user)
    @user = user
  end

  def call(notifier)
    @user.save
    notifier.notify(@user.notification_text)
  end
end

class TwitterNotifier
  def notify(text)
    puts text
  end
end

UserSubscription.new(user).call(TwitterNotifier.new)
```

La responsabilité du choix du message à afficher a été reportée à la classe
de l'objet `@user`. En plus du fait de rendre cette fonction plus adaptable, cette
modification suit le Single Responsability Principle. Ce n'est pas à la classe
s'occupant du processus d'inscription de savoir quel message affiché à
l'utilisateur.

## Exemple 3

Plutôt que de faire une seule notification, nous allons modifier le code pour
en envoyer plusieurs. Encore une fois, voici le code qui ne suit pas OCP :

```ruby
class UserSubscription
  def initialize(user)
    @user = user
  end

  def call
    @user.save
    TwitterNotifier.new.notify(@user.notification_text)
    FacebookNotifier.new.add_message(@user)
    AdminNotifier.new.send_email(@user.email)
  end
end

class TwitterNotifier
  def notify(text)
    puts text
  end
end

UserSubscription.new(user).call
```

Dans ce cas, à chaque fois que l'on souhaite ajouter un type de notification,
il faut rouvrir la classe `UserSubscription`. Voici la version corrigée :

```ruby
class UserSubscription
  def initialize(user)
    @user = user
  end

  def call(observers)
    @user.save
    observers.each do |observer|
      observer.call(@user)
    end
  end
end

class TwitterNotifier
  def call(user)
    puts user.notification_text
  end
end

observers = [TwitterNotifier.new, FacebookNotifier.new, AdminNotifier.new]
UserSubscription.new(user).call(observers)
```

Dans cette version, un nombre indéfini de méthodes est appelé après la création
de l'utilisateur. Il est facile d'ajouter un comportement en modifiant la
classe de plus haut niveau. De plus, la méthode appelée est toujours la même,
`call`. Cela permet d'ajouter n'importe quel type de traitement après la
création, pas seulement les notifications.

Dans le cas où nous souhaitons effectuer des traitements particuliers si une
notification échoue, il est possible d'extraire les observateurs dans une
classe indépendante comme ceci :

```ruby
class UserSubscriptionObservers
  def initialize(observers)
    @observers = observers
  end

  def call(user)
    @observers.each do |observer|
      observer.call(user)
    end
  end
end

class UserSubscription
  def initialize(user)
    @user = user
  end

  def call(observer)
    @user.save
    observer.call(@user)
  end
end

class TwitterNotifier
  def call(user)
    puts user.notification_text
  end
end

observers = [TwitterNotifier.new, FacebookNotifier.new, AdminNotifier.new]
composite_observer = UserSubscriptionObservers.new(observers)
UserSubscription.new(user).call(composite_observer)
```

Cet exemple est un peu plus complexe, mais permet une meilleure gestion de la file.

## Avantages

Plusieurs avantages sont à noter. Le premier, et le plus évident, est la rapidité
de réaction au changement. Il va être plus facile d'ajouter des comportements
lors de l'exécution d'une fonction.

Comme souvent, l'utilisation des bonnes pratiques simplifie les tests. Dans un
test, il est possible de passer un tableau vide comme observers.

Les principes SOLID se renforcent mutuellement. Une bonne utilisation d'OCP est
une technique qui encourage l'application du SRP. Utiliser l'ensemble de ces techniques
permet de faire un code plus clair et plus résistant aux changements.

## Inconvénients

Malheurseument, il existe au moins un inconvénient à OCP. Les comportements sont
toujours remis au niveau supérieur. Ceci rend complexe le plus au niveau,
principalement le controller. Il est possible d'ajouter des fichiers de
configuration, mais l'implémentation est plus compliquée.

## Quand utiliser OCP?

Comme toujours dans l'utilisation de Design Patterns, il n'y a pas de règle
exacte pour l'application d'OCP. Le choix se fera selon les cas d'utilisation et
le bon sens. Il y a, cependant, quelques indications que l'on peut
suivre :

 - Il est impossible d'anticiper tous les changements. Ne faites pas de
code inutilement complexe.
 - Lorsque l'on connaît une application ou un client, on sait quels sont les
éléments qui vont changer. Ce sont ceux-ci qu'il faut cibler.
 - Si l'on a changé une classe, il faut se demander si ce n'est pas l'occasion
d'appliquer OCP.

## Conclusion

J'espère que cet article vous sera utile, et que vous avez appris un nouveau
principe ou que vous avez eu un bon rafraîchissement. J'écrirais, bientôt, des
articles sur les autres principes SOLID.

Si vous avez des commentaires sur le fond ou la forme de l'article, n'hésitez
pas à m'en faire part.
