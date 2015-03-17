---
layout: post
title: "Comment ça marche : Le coeur d'ActiveRecord"
date: 2015-03-03 16:55
comments: true
categories:
---

Il y a quelque temps, je me suis lancé dans la lecture du code source de
Rails. Mon but étant de mieux comprendre son mécanisme et, peut-être, de
devenir un contributeur. J'ai choisi de commencer par ActiveRecord étant donné
que je trouve que c'est le module le plus "magique" de Rails. J'ai donc voulu
comprendre mieux cette magie apparente. Dans cet article, je vais tenter de
comprendre et d'expliquer ce qu'il se cache derrière `ActiveRecord::Base` ainsi
que le noyau d'ActiveRecord.

Comme dans les articles précédents, je vais utiliser [Pry pour
explorer](http://gcorbel.github.io/blog/blog/2015/01/20/exploration-avec-pry/). Je vous conseille fortement de lire le code en même temps que l'article.

Pour commencer, je vais reprendre
[le code servant à créer des issues](https://github.com/rails/rails/blob/master/guides/bug_report_templates/active_record_master.rb)
dans ActiveRecord en y plaçant un breakpoint afin de pouvoir étudier le
comportement du modèle.

```ruby
unless File.exist?('Gemfile')
  File.write('Gemfile', <<-GEMFILE)
    source 'https://rubygems.org'
    gem 'rails', github: 'rails/rails', ref: '6a7ac40dab'
    gem 'arel', github: 'rails/arel'
    gem 'sqlite3'
    gem 'pry-byebug'
  GEMFILE

  system 'bundle'
end

require 'bundler'
Bundler.setup(:default)

require 'active_record'
require 'minitest/autorun'
require 'logger'

# This connection will do for database-independent bug reports.
ActiveRecord::Base.establish_connection(adapter: 'sqlite3', database: ':memory:')
ActiveRecord::Base.logger = Logger.new(STDOUT)

ActiveRecord::Schema.define do
  create_table :posts, force: true  do |t|
  end

  create_table :comments, force: true  do |t|
    t.integer :post_id
  end
end

class Post < ActiveRecord::Base
end

require 'pry'; binding.pry
puts 'Just a line to make pry stop'
```

## ActiveRecord

Commençons par le commencement. Lorsque l'on charge la gem `activerecord`, le fichier
[activerecord.rb](https://github.com/rails/rails/blob/master/activerecord/lib/active_record.rb) est inclus.
Comme on peut le voir en lisant le code, celui-ci commence par chargé ses
dépendances.

```
require 'active_support'
require 'active_support/rails'
require 'active_model'
require 'arel'

require 'active_record/version'
require 'active_record/attribute_set'
```

On comprend donc que `ActiveRecord` dépend d'`ActiveSupport`, `ActiveModel` et
`Arel`. `ActiveSupport` est un module fournissant des fonctions utiles dans les
autres modules de Rails. `ActiveModel` contient les informations spécifiques au
modèle du design pattern `MVC` sans contenir les informations relatives à la
base de données. `Arel`, quant à lui, permet de créer des requêtes SQL en Ruby.


Les premières lignes du module sont les suivantes :

```
  extend ActiveSupport::Autoload

  autoload :Attribute
```

Plutôt que de faire un `require`, `ActiveRecord` utilise le `autoload`
d'`ActiveSupport`. `ActiveSupport` permet de déterminer le nom du fichier à
charger en incluant le nom du module. Si l'on appelle `autoload :Concern` depuis
le module `ActiveSupport`, "active_support/concern" est déterminé. Cette chaîne
de caractère est ensuite envoyée à la fonction
[autoload](http://ruby-doc.org/core-2.1.0/Module.html#method-i-autoload) de
Ruby. De cette façon, les modules ne sont chargés qu'au moment où ils sont
appelés. C'est uniquement lorsque l'on utilisera `ActiveSupport::Concern` dont
le fichier sera chargé. Il s'agit d'un procédé permettant d'économiser la
mémoire puisque l'on ne charge pas ce qui est inutile.

On continuant la lecture du code, on peut s'apercevoir que certain des modules
sont chargé dans un block `eager_load`. Les chargements automatiques sont mis
dans un tableau. Les modules contenus dans ce tableau seront chargés quand la
fonction `ActiveSupport.eager_load!` sera appelée.

Certains des éléments sont chargés dans block `autoload_under`. Il s'agit,
simplement, d'un block ajoutant un dossier au chemin chargé.

Dans le bas du fichier, la fonction `eager_load!` est définie. Celle-ci permet
de forcer le chargement de sous-modules. Cette fonction à été introduit dans
[ce commit](https://github.com/rails/rails/commit/2801786e1a51b7cf7d7c3fd72b5fc9974f83f435).
D'après ce que je comprends, ce code peut-être utile quand on fait du
multithreading ou pour être "CoW friendly". J'avoue ne pas savoir ce que veut
dire "CoW". Si quelqu'un peut m'éclairer, ça me plairait!

La dernière partie consiste en deux "hook".

```ruby
ActiveSupport.on_load(:active_record) do
  Arel::Table.engine = self
end

ActiveSupport.on_load(:i18n) do
  I18n.load_path << File.dirname(__FILE__) + '/active_record/locale/en.yml'
end
```

Ces appels vont activer des événements stockés dans des tableaux puis stock à
nouveau pour une utilisation ultérieure.

## ActiveRecord::Base

Tous les développeurs utilisant Ruby on Rails ont probablement fait un modèle
héritant de `ActiveRecord::Base`. Quand on regarde le fichier
[base.rb](https://github.com/rails/rails/blob/master/activerecord/lib/active_record/base.rb),
on s'aperçoit qu'il s'agit uniquement d'un fichier en incluant d'autres et
faisant des `include` et des `extend` d'autres modules. Deux lignes sont
intéressantes, `ActiveSupport.run_load_hooks(:active_record, Base)` et `include
Core`.

L'avant-dernière ligne exécute les événements stockés précédemment.
`ActiveRecord` est donc indiqué comme étant le moteur par défaut de `Arel`.


## Conclusion

Voici donc le coeur d'ActiveRecord. Je vais tenter de poursuivre ma lecture du
code et j'exposerai mes découvertes dans de prochains articles.
