---
layout: post
title: "Conserver les changements de valeurs grâce à ActiveModel::Dirty"
date: 2015-02-07 16:54
comments: true
categories:
---

Dans cet article, je vous propose d'observer le comportement
d'ActiveModel::Dirty et d'en comprendre le fonctionnement en consultant le code
source. ActiveModel::Dirty est l'un des modules méconnus de Rails. Celui-ci
conserve l'historique des changements apportés aux attributs d'un objet et de
revenir en arrière si besoin.

<!--more-->

Tout d'abord, voici un exemple de classe à laquelle ActiveModel::Dirty est inclus :

``` ruby
require 'active_model'

class User
  include ActiveModel::Dirty

  attr_accessor :name

  define_attribute_methods :name
end
```

Rien de bien particulier ici, mis à part l'appel de la fonction
`define_attribute_methods`. Cette méthode doit être appelée à chaque fois que des
[préfix, suffixe ou affixe](http://api.rubyonrails.org/classes/ActiveModel/AttributeMethods/ClassMethods.html)
sont utilisés avec, en argument, les attributs correspondants. En effet, comme
nous allons le voir, lors de l'inclusion de `ActiveModel::Dirty`, plusieurs
préfixe et suffixe sont créés.

##  Les bases

Comme indiqué ci-haut, `ActiveModel::Dirty` permet de vérifier si des valeurs d'un objet ont changés.

``` ruby
# instanciation de la classe
user = User.new
user.name = 'GCorbel'

# vérification du statut initial
p user.changed? # false
p user.name_changed? # false

# signalisation d'un changement
user.name_will_change!

# modification des valeurs
user.name = "Guirec Corbel"

# vérification du statut après changement
p user.changed? # true
p user.name_changed? # true
p user.name_changed?(from: 'GCorbel', to: 'Guirec Corbel') # true
p user.changed # ["name"]
p user.changes # {"name"=>["GCorbel", "Guirec Corbel"]}
p user.name_was # "GCorbel"
```

Dans cet exemple, un nouvel objet est instancié et la valeur de `name` est
renseignée. Initialement, aucun changement n'est indiqué. Il est nécessaire de
spécifier que l'attribut va changer en utilisant la méthode
`name_will_change!`. Suite à cet appel, nous pouvons demander si l'objet à
subit un changement grâce à la méthode `changed?`. `name_changed?` permet
d'afficher l'information spécifique à l'attribut `name`. Il est également
possible de vérifier l'état précédent et suivant des valeurs en ajoutant les
arguments `from` et `to` à `name_changed?`. Enfin, il est possible de voir
quelles valeurs on changés grâce à `changed`, quels ont été les changements grâce
à `changes` et quel était la dernière valeur grâce à `name_was`.

## Afficher les dernières valeurs

Il est possible de vider la liste des changements grâce à la fonction `changes_applied`.

```ruby
user.send(:changes_applied)
p user.changes # {}
p user.previous_changes # {"name"=>["GCorbel", "Guirec Corbel"]}
```

La fonction `change_applied` est une méthode privée. Il est donc nécessaire
d'utiliser `send` pour effectuer l'appel. Comme on peut le voir dans
l'exemple, `changes` renvoi maintenant un Hash vide. Cependant, les
changements sont encore conservés, comme on peut le voir lors de l'appel de
`previous_changes`.

Afin de vider complètement l'historique des changements, il est possible
d'appeler la fonction `clear_changes_information`.

```ruby
user.send(:clear_changes_information)
p user.changes # {}
p user.previous_changes # {}
```

## Revenir en arrière

Comme on peut s'y attendre, le module offre la possibilité de revenir en
arrière, et ceux grâce à la fonction `restore_attributes`

```ruby
user.name = "Guirec Corbel"
user.name_will_change!
user.name = "Guirec"
user.restore_attributes # restore les anciennes valeurs
p user.name # "Guirec Corbel"
```

## Fonctionnement

Le [code du module](https://github.com/rails/rails/blob/6f8d9bd6da6349d3d179f2e72db5bc7044a8e5c1/activemodel/lib/active_model/dirty.rb)
est assez simple. Lors de l'appel de `name_will_change!`, la fonction est exécutée
`attribute_will_change`. Cette fonction enregistre la valeur
courante dans le hash `changed_attributes`. Cet appel doit être effectué avant
le changement de valeur. Dans le cas contraire, l'information serait perdue par
la suite.

Les autres fonctions sont uniquement des traitements faits sur le hash, comme la fonction [#changed?](https://github.com/rails/rails/blob/6f8d9bd6da6349d3d179f2e72db5bc7044a8e5c1/activemodel/lib/active_model/dirty.rb#L126-L128).

```ruby
def changed?
  changed_attributes.present?
end
```

## Conclusion

Et voilà! Tout en restant simple, ce module peut être utile dans certains cas.
Il est notamment utilisé par ActiveRecord pour conserver toutes les
modifications faites aux modèles. Nous étudirons le comportement d'ActiveRecord
dans un prochain article.

Comme souvent, en prenant quelques minutes pour lire le code source, on se rend
compte que le fonctionnement est simple. À nouveau, la magie de Rails est
démystifiée.
