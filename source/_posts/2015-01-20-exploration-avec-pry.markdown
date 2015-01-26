---
layout: post
title: "Exploration avec Pry"
date: 2015-01-20 20:54
comments: true
categories:
---

Les IDE traditionnels possèdent généralement des outils avec une interface
graphique pour déboguer. Pry, quant à lui, offre les mêmes possibilités en
utilisant des lignes de commande. Pry est un REPL (Read-Eval-Print Loop) qui
permet de naviguer dans le code afin d'expliquer un bogue ou de comprendre le
fonctionnement interne d'une Gem. Nous allons voir comment faire dans cet article.

<!-- more -->

Pry offre énormément de possibilités, comme éditer le code ou voir la
documentation de fonctions. Pour afficher la liste des commandes disponibles,
vous pouvez exécuter la fonction `help` dans Pry et il est possible d'exécuter
chaque commande avec l'argument `--help` pour plus de détails.

Cet article n'a pas pour but d'expliquer le fonctionnement interne de Pry ou
d'explorer toutes les commandes, mais se concentre uniquement sur la navigation.

## Exemple de code

Voici un premier code d'exemple permettant de jouer avec Pry :

```ruby
class Cat
  def initialize(name)
    @name = name
  end

  def say
    "Miaou"
  end

  def eat
    "MiamMiam"
  end

  def debug
    require 'pry'; binding.pry
    say
  end
end

Cat.new('Tom').debug
```

## Affichage des variables

À l'exécution du programme, Pry s'arrête à la ligne suivant `require 'pry'; binding.pry`.

```
        14: def debug
        15:   require 'pry'; binding.pry
     => 16:   say
        17: end
```

La commande `ls`, très similaire à celle de Linux pour explorer les fichiers et
dossiers, nous offre la possibilité d'explorer les objets que contexte dans
lequel nous nous trouvons.

```
 > ls
=> Cat#methods: debug  eat  say
=> instance variables: @name
=> locals: _  __  _dir_  _ex_  _file_  _in_  _out_  _pry_
```

Il est possible de lister uniquement les méthodes avec l'argument -m.

```
 > ls -m
=> Cat#methods: debug  eat  say
```

L'argument `-i` permet de lister les variables d'instances.

```
 > ls -i
=> instance variables: @name
```

`-G`, correspondant à la fonction grep,  permet de rechercher dans les noms. `ls
-G d` permet de chercher toutes les méthodes ou les variables contenant un "d"
comme ici :

```
 > ls -G d
=> Cat#methods: debug
=> instance variables: @name
=> locals: _dir_
```

Il est possible de lister les méthodes d'une variable en la spécifiant à la
commande. Tous ces éléments peuvent être combinés. Il est donc possible de
regarder toutes les méthodes contenant "cap" dans la variable `@name`.

```
 > ls -m -G cap @name
=> String#methods: capitalize  capitalize!  shellescape
```

La commande cd permet de changer le contexte afin d'explorer une variable.

```
     > p self
    => main
     > p @name
    => "Tom"
     > cd @name
     > p self
    => "Tom"
     > ls -G encod
    => String#methods: encode  encode!  encoding  force_encoding  valid_encoding?
     > cd ..
     > p self
    => main
```

## Navigation dans le code

Grâce à [pry-byebug](https://github.com/deivid-rodriguez/pry-byebug), il est possible d'exécuter pas à pas un programme. Avec
quelques commandes, je vais tenter de comprendre comment ActiveRecord communique
avec la base de données pour lister des enregistrements. Voici un exemple de
code, tiré de [mon dernier article sur ActiveRecord](http://gcorbel.github.io/blog/blog/2015/01/20/utiliser-activerecord-sans-rails/), qui donne un point de départ à l'inspection :

```ruby
require 'active_record'

class User < ActiveRecord::Base

end

class CreateUsers < ActiveRecord::Migration
  def change
    create_table :users do |t|
      t.string :name
      t.integer :age
      t.timestamps null: false
    end
  end
end

ActiveRecord::Base.establish_connection(adapter: 'sqlite3', database: 'dbfile.sqlite3')
CreateUsers.new.change unless User.table_exists?

require 'pry'; binding.pry
User.all
```

À l'exécution du programme, nous avons cet affichage :

```
From: /tmp/test.rb @ line 21 :

    16: ActiveRecord::Base.establish_connection(adapter: 'sqlite3', database: 'dbfile.sqlite3')
    17: CreateUsers.new.change unless User.table_exists?
    18:
    19: User.create(name: 'test', age: 20)
    20: require 'pry'; binding.pry
 => 21: User.all
```

À partir de là, il est possible de continuer l'exécution en entrant dans la méthode `User#all` en utilisant la commande `step`.

```
 > step
From: /home/dougui/.rbenv/versions/2.1.5/lib/ruby/gems/2.1.0/gems/activerecord-4.2.0/lib/active_record/scoping/named.rb @ line 25 ActiveRecord::Scoping::Named::ClassMethods#all:

    24: def all
 => 25:   if current_scope
    26:     current_scope.clone
    27:   else
    28:     default_scoped
    29:   end
    30: end
```

La méthode est maintenant affichée. Pour continuer l'exécution en restant dans
la même méthode, il est possible d'utiliser la commande `next`.

```
 > next
From: /home/dougui/.rbenv/versions/2.1.5/lib/ruby/gems/2.1.0/gems/activerecord-4.2.0/lib/active_record/scoping/named.rb @ line 28 ActiveRecord::Scoping::Named::ClassMethods#all:

    24: def all
    25:   if current_scope
    26:     current_scope.clone
    27:   else
 => 28:     default_scoped
    29:   end
    30: end
```

Afin d'aller plus en profondeur et voir le fonctionnement interne d'ActiveRecord, j'ai tout simplement exécuté plusieurs fois la commande `step`.

```
 > step
From: /home/dougui/.rbenv/versions/2.1.5/lib/ruby/gems/2.1.0/gems/activerecord-4.2.0/lib/active_record/scoping/named.rb @ line 33 ActiveRecord::Scoping::Named::ClassMethods#default_scoped:

    32: def default_scoped # :nodoc:
 => 33:   relation.merge(build_default_scope)
    34: end

 > step
From: /home/dougui/.rbenv/versions/2.1.5/lib/ruby/gems/2.1.0/gems/activerecord-4.2.0/lib/active_record/core.rb @ line 253 ActiveRecord::Core::ClassMethods#relation:

    252: def relation #:nodoc:
 => 253:   relation = Relation.create(self, arel_table)
    254:
    255:   if finder_needs_type_condition?
    256:     relation.where(type_condition).create_with(inheritance_column.to_sym => sti_name)
    257:   else
    258:     relation
    259:   end
    260: end

 > step
    From: /home/dougui/.rbenv/versions/2.1.5/lib/ruby/gems/2.1.0/gems/activerecord-4.2.0/lib/active_record/relation/delegation.rb @ line 106 ActiveRecord::Delegation::ClassMethods#create:

    105: def create(klass, *args)
 => 106:   relation_class_for(klass).new(klass, *args)
    107: end

 > step
From: /home/dougui/.rbenv/versions/2.1.5/lib/ruby/gems/2.1.0/gems/activerecord-4.2.0/lib/active_record/core.rb @ line 237 ActiveRecord::Core::ClassMethods#arel_table:

    236: def arel_table # :nodoc:
 => 237:   @arel_table ||= Arel::Table.new(table_name, arel_engine)
    238: end
```

La navigation avec Pry nous a donc permis de comprendre qu'ActiveRecord utilise
Arel. Pour reprendre l'exécution normale du programme, il est possible
d'utiliser la commande `continue`.

La commande `finish` permet de continuer l'exécution jusqu'a ce qu'il y ait un changement
de fenêtre. Malheureusement, je la trouve très peu utile et elle me perd dans le
code plutôt qu'elle ne m'aide.

Toutes les commandes de navigations possèdent des alias. Il est possible de uniquement le premier caractère pour exécuter la commande et utiliser `s`, `n`, `f` et `c`.

## Réafficher la position actuelle

L'affichage peut rapidement devenir chargé. Pour revoir la position actuelle du curseur, il est possible d'exécuter la commande `whereami`.

## Répititions des commandes

Afin de répéter la commande précédemment utilisée en appuyant sur la touche "Entrée", il est possible d'ajouter, dans le fichier `~/.pryrc`, ce code :

```
Pry::Commands.command /^$/, "repeat last command" do
  _pry_.run_command Pry.history.to_a.last
end
```

Cet ajout va permettre de rendre beaucoup plus agréable la navigation.

## Breakpoints

Pry nous offre la possibilité d'ajouter des points d'arrêts. Il est possible
d'arrêter l'exécution en spécifiant un fichier et une ligne.

```
 > break From: /home/dougui/.rbenv/versions/2.1.5/lib/ruby/gems/2.1.0/gems/activerecord-4.2.0/lib/active_record/scoping/named.rb:33
```


En exécutant la fonction `continue`, le programme s'arrête donc à la ligne spécifiée.

```
From: /home/dougui/.rbenv/versions/2.1.5/lib/ruby/gems/2.1.0/gems/activerecord-4.2.0/lib/active_record/scoping/named.rb @ line 33 ActiveRecord::Scoping::Named::ClassMethods#default_scoped:

    32: def default_scoped # :nodoc:
 => 33:   relation.merge(build_default_scope)
    34: end
```

Dans ce cas, il se peut que l'on souhaite voir le comportement de la méthode
`merge` qui est appelée à la variable retournée par la méthode `relation`.
Premièrement, il est nécessaire de savoir de quels classe ou module fait la
méthode `merge`.

```
 > ls relation -G merge
ActiveRecord::SpawnMethods#methods: merge  merge!
```

À partir de là, il est possible de placer un point d'arrêt directement sur cette méthode.

```
 > break ActiveRecord::SpawnMethods#merge
Breakpoint 3: ActiveRecord::SpawnMethods#merge (Enabled) :

29: def merge(other)
30:   if other.is_a?(Array)
31:     to_a & other
32:   elsif other
33:     spawn.merge!(other)
34:   else
35:     self
36:   end
37: end
```

À la prochaine exécution, pry s'arrêtera au début de l'exécution de cette méthode.

```
 > continue

Breakpoint 2. First hit

From: /home/dougui/.rbenv/versions/2.1.5/lib/ruby/gems/2.1.0/gems/activerecord-4.2.0/lib/active_record/relation/spawn_methods.rb @ line 29 ActiveRecord::SpawnMethods#merge:

 => 29: def merge(other)
    30:   if other.is_a?(Array)
    31:     to_a & other
    32:   elsif other
    33:     spawn.merge!(other)
    34:   else
    35:     self
    36:   end
    37: end
```

Il est possible de lister les points d'arrêts définis grâce à la commande `breaks`.

```
 > breaks

#  Enabled  At
---------------
1  Yes      ActiveRecord::SpawnMethods#merge
2  Yes      /home/dougui/.rbenv/versions/2.1.5/lib/ruby/gems/2.1.0/gems/activerecord-4.2.0/lib/active_record/core.rb @ 237
```

Il est également possible de désactiver temporairement un point d'arrêt.

```
 > break --disable 1

#  Enabled  At
---------------
1  No       ActiveRecord::SpawnMethods#merge
2  Yes      /home/dougui/.rbenv/versions/2.1.5/lib/ruby/gems/2.1.0/gems/activerecord-4.2.0/lib/active_record/core.rb @ 237
```

On peut les désactiver tous.

```
 > break --disable-all

#  Enabled  At
---------------
1  No       ActiveRecord::SpawnMethods#merge
2  No       /home/dougui/.rbenv/versions/2.1.5/lib/ruby/gems/2.1.0/gems/activerecord-4.2.0/lib/active_record/core.rb @ 237
```

De la même manière, il est possible de supprimer un ou plusieurs points d'arrêts.

```
 > break --delete-all

No breakpoints defined.
```

Il est existe d'autres possibilités fournies par Pry à ce niveau, comme l'arrêt
suivant des conditions. Je vous invite à consulter la documentation pour en
savoir plus.


## Conclusion

Pry est un outil essentiel dans le développement avec Ruby et devient plus
puissant que les outils fournis par des IDE quand on sait l'utiliser de
manière efficace. Dans les prochains articles, je vais utiliser ces techniques
pour comprendre le fonctionnement de Rails.
