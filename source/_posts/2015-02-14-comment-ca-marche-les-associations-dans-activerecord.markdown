---
layout: post
title: "Comment ça marche : Les associations dans ActiveRecord"
date: 2015-02-14 11:33
comments: true
categories:
---

Les associations sont une part importante d'ActiveRecord. Il s'agit d'éléments
que l'on utilise dans les premières phases d'apprentissage de Rails. Bien que
son utilisation soit simple, l'implémentation est assez compliquée. Je vous
propose d'explorer le fonctionnement en observant le code exécuté à l'appel de
l'association `belongs_to` afin de comprendre le mécanisme complexe.


Dans cet article, je vais utiliser [Pry pour explorer](http://gcorbel.github.io/blog/blog/2015/01/20/exploration-avec-pry/).
Je vous conseille de suivre l'exécution du code tout en lisant l'article. Cela
vous permettra d'avoir accès aux détails à votre guise. J'utilise la version la
plus récente lors de l'écriture de cette article, le 15 février 2015. La
version de Rails à la date de votre lecture peut être différente. Vous devez
spécifier le commit "6a7ac40dab" si vous faite un clone de Rails et
[ce lien](https://github.com/rails/rails/tree/6a7ac40dabf4a8948418c61f307ffe52ddcb48f2)
pour explorer sur Github.

Comme point de départ, je vous propose de reprendre [le code servant à créer des issues](https://github.com/rails/rails/blob/master/guides/bug_report_templates/active_record_master.rb) dans ActiveRecord légèrement modifié.

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
  require 'pry'; binding.pry
  has_many :comments
end

class Comment < ActiveRecord::Base
  belongs_to :post
end

class BugTest < Minitest::Test
  def test_association_stuff
    post = Post.create!
    post.comments << Comment.create!

    post.comments
  end
end
```

Au lancement du programme, Pry stop l'exécution. Nous pouvons ainsi entrer dans
le code d'ActiveRecord par le module `ActiveRecord::Associations`. Plus de 1000 lignes de
commentaires sont disponibles pour expliquer les utilisations possibles de la
méthode.

```ruby
From: /home/dougui/.rbenv/versions/2.1.5/lib/ruby/gems/2.1.0/bundler/gems/rails-698afe1173c6/activerecord/lib/active_record/associations.rb @ line 1265 ActiveRecord::Associations::ClassMethods#has_many:

    1264: def has_many(name, scope = nil, options = {}, &extension)
 => 1265:   reflection = Builder::HasMany.build(self, name, scope, options, &extension)
    1266:   Reflection.add_reflection self, name, reflection
    1267: end
```

Cette méthode de seulement deux lignes peut paraître simple mais la complexité
est cachée dans la méthode `build`, de classe
`ActiveRecord::Associations::Builder::HasMany`. Tout les arguments lui sont
transmis avec, en plus, `self`, correspondant à la classe du modèle, `Post` dans
notre cas.

La classe `ActiveRecord::Associations::Builder::HasMany` fournit des
informations basiques sur l'association, c'est-à-dire son nom et les options
disponibles.

```ruby
module ActiveRecord::Associations::Builder
  class HasMany < CollectionAssociation #:nodoc:
    def self.macro
      :has_many
    end

    def self.valid_options(options)
      super + [:primary_key, :dependent, :as, :through, :source, :source_type, :inverse_of, :counter_cache, :join_table, :foreign_type]
    end

    def self.valid_dependent_options
      [:destroy, :delete_all, :nullify, :restrict_with_error, :restrict_with_exception]
    end
  end
end
```

Les informations fournies sont utiles pour les classes parentes. En effet, la
classe hérite de
`ActiveRecord::Associations::Builder::CollectionAssociation::CollectionAssociation`,
qui hérite de `ActiveRecord::Associations::Builder::Association`.

Le coeur du traitement se trouve dans la méthode `build` de la classe
`ActiveRecord::Associations::Builder::Association`.

```ruby
From: /home/dougui/.rbenv/versions/2.1.5/lib/ruby/gems/2.1.0/bundler/gems/rails-6a7ac40dabf4/activerecord/lib/active_record/associations/builder/association.rb @ line 22 ActiveRecord::Associations::Builder::Association.build:

    21: def self.build(model, name, scope, options, &block)
 => 22:   if model.dangerous_attribute_method?(name)
    23:     raise ArgumentError, "You tried to define an association named #{name} on the model #{model.name}, but " \
    24:                          "this will conflict with a method #{name} already defined by Active Record. " \
    25:                          "Please choose a different association name."
    26:   end
    27:
    28:   extension = define_extensions model, name, &block
    29:   reflection = create_reflection model, name, scope, options, extension
    30:   define_accessors model, reflection
    31:   define_callbacks model, reflection
    32:   define_validations model, reflection
    33:   reflection
    34: end
```

Pour la suite des explications, je vais passer en revue la méthode `build` ligne
par ligne. Commençons par la première.

La première fonction appelée, `model.dangerous_attribute_method?` vérifie si le
nom de l'association n'est pas une fonction existante dans ActiveRecord, par
exemple, il n'est possible de créer une association comme `belongs_to :save`.
Le nom de l'association ne peut également pas être similaire à une méthode
relative aux IDs, `belongs_to :id` par exemple.

```ruby
From: /home/dougui/.rbenv/versions/2.1.5/lib/ruby/gems/2.1.0/bundler/gems/rails-6a7ac40dabf4/activerecord/lib/active_record/associations/builder/association.rb @ line 22 ActiveRecord::Associations::Builder::Association.build:

    21: def self.build(model, name, scope, options, &block)
    22:   if model.dangerous_attribute_method?(name)
    23:     raise ArgumentError, "You tried to define an association named #{name} on the model #{model.name}, but " \
    24:                          "this will conflict with a method #{name} already defined by Active Record. " \
    25:                          "Please choose a different association name."
    26:   end
    27:
 => 28:   extension = define_extensions model, name, &block
    29:   reflection = create_reflection model, name, scope, options, extension
    30:   define_accessors model, reflection
    31:   define_callbacks model, reflection
    32:   define_validations model, reflection
    33:   reflection
    34: end
```

Comme on peut le voir, `build` accepte un block qui est transmis à
`define_extensions`. Le block est passé depuis la définition de l'association
comme celui-ci :

```
  has_many :comments do
    def recents
      where("created_at > ?", 5.days.ago)
    end
  end
```

Il sera donc possible d'utiliser l'association comme ainsi :
`post.comments.recents`. Pour plus de détails dans la
[doc officiel](http://guides.rubyonrails.org/association_basics.html#association-extensions).

Les extensions sont définies comme suit :

```ruby
From: /home/dougui/.rbenv/versions/2.1.5/lib/ruby/gems/2.1.0/bundler/gems/rails-6a7ac40dabf4/activerecord/lib/active_record/associations/builder/collection_association.rb @ line 25 ActiveRecord::Associations::Builder::CollectionAssociation.define_extensions:

    24: def self.define_extensions(model, name)
 => 25:   if block_given?
    26:     extension_module_name = "#{model.name.demodulize}#{name.to_s.camelize}AssociationExtension"
    27:     extension = Module.new(&Proc.new)
    28:     model.parent.const_set(extension_module_name, extension)
    29:   end
    30: end
```

La fonction créer un module avec le block qui est fourni. Comme
son nom l'indique, `extension_module_name` contient le nom du module. Celui-ci
est composé du nom du modèle et de celui de l'association. Dans l'exemple, nous
obtenons "PostCommentsAssociationExtension". Le nouveau module créé grâce à
`Module.new(&Proc.new)`. Celui-ci est ensuite ajouté au code de l'application
afin d'être inclus par la suite. Ce module a pour but de recevoir les méthodes
qui vont être créées par la suite. Je trouve que c'est un moyen très astucieux
pour isoler ces méthodes.

Nous pouvons donc passer à la suite.

```ruby
From: /home/dougui/.rbenv/versions/2.1.5/lib/ruby/gems/2.1.0/bundler/gems/rails-6a7ac40dabf4/activerecord/lib/active_record/associations/builder/association.rb @ line 22 ActiveRecord::Associations::Builder::Association.build:
    21: def self.build(model, name, scope, options, &block)
    22:   if model.dangerous_attribute_method?(name)
    23:     raise ArgumentError, "You tried to define an association named #{name} on the model #{model.name}, but " \
    24:                          "this will conflict with a method #{name} already defined by Active Record. " \
    25:                          "Please choose a different association name."
    26:   end
    27:
    28:   extension = define_extensions model, name, &block
 => 29:   reflection = create_reflection model, name, scope, options, extension
    30:   define_accessors model, reflection
    31:   define_callbacks model, reflection
    32:   define_validations model, reflection
    33:   reflection
    34: end
```

Je pense que la méthode `create_reflection` est la plus compliquée. Celle-ci
a pour but de créer un objet qui va contenir les informations relatives à l'association.

```ruby
From: /home/dougui/.rbenv/versions/2.1.5/lib/ruby/gems/2.1.0/bundler/gems/rails-698afe1173c6/activerecord/lib/active_record/associations/builder/association.rb @ line 37 ActiveRecord::Associations::Builder::Association.create_reflection:

    36: def self.create_reflection(model, name, scope, options, extension = nil)
 => 37:   raise ArgumentError, "association names must be a Symbol" unless name.kind_of?(Symbol)
    38:
    39:   if scope.is_a?(Hash)
    40:     options = scope
    41:     scope   = nil
    42:   end
    43:
    44:   validate_options(options)
    45:
    46:   scope = build_scope(scope, extension)
    47:
    48:   ActiveRecord::Reflection.create(macro, name, scope, options, model)
    49: end
```

La première ligne renvoi une erreur si l'on essai de créer une collection avec
autre chose qu'un `Symbol`.

Par la suite, on remplace les options par le scope
si c'est un `Hash`. Cette partie semble être uniquement un hack. Si `scope` est
un `Hash`, c'est qu'il s'agit des options. Lors de la création de
l'association, il n'est pas spécifié quel argument représente le scope ou les
options. Il est donc nécessaire de faire ce petit tour de passe-passe pour
les distinguer.

`validate_options` permet de s'assurer que les options passées à l'association
sont valides. On ne peut donc pas faire une association comme `has_many
:comments, something_invalid: :test`. Une erreur sera retournée.

Comme son nom l'indique, build_scope construit le scope.

```ruby
From: /home/dougui/.rbenv/versions/2.1.5/lib/ruby/gems/2.1.0/bundler/gems/rails-698afe1173c6/activerecord/lib/active_record/associations/builder/association.rb @ line 52 ActiveRecord::Associations::Builder::Association.build_scope:

    51: def self.build_scope(scope, extension)
 => 52:   new_scope = scope
    53:
    54:   if scope && scope.arity == 0
    55:     new_scope = proc { instance_exec(&scope) }
    56:   end
    57:
    58:   if extension
    59:     new_scope = wrap_scope new_scope, extension
    60:   end
    61:
    62:   new_scope
    63: end
```

Par défaut, le scope est gradé tel quel. S'il y a un scope et que celui-ci ne
possède par d'argument, il est transformé pour être exécuté par l'instance
plutôt que la classe.

S'il y a une extension, celle-ci est "wrapper" comme ceci :

```ruby
From: /home/dougui/.rbenv/versions/2.1.5/lib/ruby/gems/2.1.0/bundler/gems/rails-698afe1173c6/activerecord/lib/active_record/associations/builder/collection_association.rb @ line 72 ActiveRecord::Associations::Builder::CollectionAssociation.wrap_scope:

    71: def self.wrap_scope(scope, mod)
 => 72:   if scope
    73:     proc { |owner| instance_exec(owner, &scope).extending(mod) }
    74:   else
    75:     proc { extending(mod) }
    76:   end
    77: end
```

Grâce à cette méthode, le module créé précédemment est inclus lors de l'appel de
la méthode. Cette partie est plutôt complexe et demanderait un article au
complet. Je vais donc laisser cette partie pour le moment.

Après cet interlude, revenons à l'exécution de `create_reflection`.

```ruby
From: /home/dougui/.rbenv/versions/2.1.5/lib/ruby/gems/2.1.0/bundler/gems/rails-698afe1173c6/activerecord/lib/active_record/associations/builder/association.rb @ line 48 ActiveRecord::Associations::Builder::Association.create_reflection:

    36: def self.create_reflection(model, name, scope, options, extension = nil)
    37:   raise ArgumentError, "association names must be a Symbol" unless name.kind_of?(Symbol)
    38:
    39:   if scope.is_a?(Hash)
    40:     options = scope
    41:     scope   = nil
    42:   end
    43:
    44:   validate_options(options)
    45:
    46:   scope = build_scope(scope, extension)
    47:
 => 48:   ActiveRecord::Reflection.create(macro, name, scope, options, model)
    49: end
```

La macro (`:has_many`), celui de l'association (`:comments`), le scope, les
options et le modèle sont passés à `ActiveRecord::Reflection#create`.

```ruby
From: /home/dougui/.rbenv/versions/2.1.5/lib/ruby/gems/2.1.0/bundler/gems/rails-698afe1173c6/activerecord/lib/active_record/reflection.rb @ line 17 ActiveRecord::Reflection.create:

    16: def self.create(macro, name, scope, options, ar)
 => 17:   klass = case macro
    18:           when :composed_of
    19:             AggregateReflection
    20:           when :has_many
    21:             HasManyReflection
    22:           when :has_one
    23:             HasOneReflection
    24:           when :belongs_to
    25:             BelongsToReflection
    26:           else
    27:             raise "Unsupported Macro: #{macro}"
    28:           end
    29:
    30:   reflection = klass.new(name, scope, options, ar)
    31:   options[:through] ? ThroughReflection.new(reflection) : reflection
    32: end
```

Premièrement, on trouve la classe correspondant à la macro, `HasManyReflection`
dans notre cas. Par la suite, cette classe est instanciée, avec le nom de
l'association, le scope, les options et le modèle. Les informations seront
passées à l'objet créé. `HasManyReflection` est un objet représentant
l'association. Les informations y sont stockées afin d'y être utilisées dans votre
code ou dans des gems. Pour accéder à la liste des reflections du modèle, il
est possible de faire `Post.reflect_on_all_associations`. Vous obtiendrez ce
résultat :

```ruby
[#<ActiveRecord::Reflection::HasManyReflection:0x007fc8898aeba8
 @active_record=Post(id: integer),
 @association_scope_cache={},
 @automatic_inverse_of=nil,
 @constructable=true,
 @foreign_type="comments_type",
 @klass=nil,
 @name=:comments,
 @options={},
 @plural_name="comments",
 @scope=#<Proc:0x007fc8896b4230@/home/dougui/.rbenv/versions/2.1.5/lib/ruby/gems/2.1.0/bundler/gems/rails-698afe1173c6/activerecord/lib/active_record/associations/builder/collection_association.rb:75>,
 @scope_lock=#<Mutex:0x007fc88844c368>,
 @type=nil>]
```

On peut comprendre aisément l'utilité d'une telle liste.

À la fin de l'exécution, nous nous retrouvons dans la méthode `build`.

```ruby
From: /home/dougui/.rbenv/versions/2.1.5/lib/ruby/gems/2.1.0/bundler/gems/rails-698afe1173c6/activerecord/lib/active_record/associations/builder/association.rb @ line 30 ActiveRecord::Associations::Builder::Association.build:

    21: def self.build(model, name, scope, options, &block)
    22:   if model.dangerous_attribute_method?(name)
    23:     raise ArgumentError, "You tried to define an association named #{name} on the model #{model.name}, but " \
    24:                          "this will conflict with a method #{name} already defined by Active Record. " \
    25:                          "Please choose a different association name."
    26:   end
    27:
    28:   extension = define_extensions model, name, &block
    29:   reflection = create_reflection model, name, scope, options, extension
 => 30:   define_accessors model, reflection
    31:   define_callbacks model, reflection
    32:   define_validations model, reflection
    33:   reflection
    34: end
```

Voici une fonction des plus intéressante. Grâce à la magie de la
métaprogrammation avec Ruby, cette fonction va définir les accesseurs.

```ruby
From: /home/dougui/.rbenv/versions/2.1.5/lib/ruby/gems/2.1.0/bundler/gems/rails-698afe1173c6/activerecord/lib/active_record/associations/builder/association.rb @ line 102 ActiveRecord::Associations::Builder::Association.define_accessors:

    101: def self.define_accessors(model, reflection)
 => 102:   mixin = model.generated_association_methods
    103:   name = reflection.name
    104:   define_readers(mixin, name)
    105:   define_writers(mixin, name)
    106: end
```

Comme nous pouvons le voir, le modèle est passé en argument ainsi que
`reflection` contenant toutes les informations sur l'association.

Voici la première fonction appelée.

```ruby
From: /home/dougui/.rbenv/versions/2.1.5/lib/ruby/gems/2.1.0/bundler/gems/rails-698afe1173c6/activerecord/lib/active_record/core.rb @ line 199 ActiveRecord::Core::ClassMethods#generated_association_methods:

    194: def generated_association_methods
    195:   @generated_association_methods ||= begin
    196:     mod = const_set(:GeneratedAssociationMethods, Module.new)
    197:     include mod
    198:     mod
 => 199:   end
    200: end
```

Comme on peut le voir, un nouveau module `GeneratedAssociationMethods` est créé
puis inclus. Ce nouveau module est ensuite retourné pour une utilisation
ultérieure.

Si l'on continue l'exécution de `define_accessors`, on trouve l'appel de
`define_readers` et `define_writers`. Ces deux méthodes reçoivent, comme
argument, le nouveau module ainsi que le nom de l'association. Voici la
première :

```ruby
From: /home/dougui/.rbenv/versions/2.1.5/lib/ruby/gems/2.1.0/bundler/gems/rails-698afe1173c6/activerecord/lib/active_record/associations/builder/collection_association.rb @ line 52 ActiveRecord::Associations::Builder::CollectionAssociation.define_readers:

    51: def self.define_readers(mixin, name)
 => 52:   super
    53:
 => 54:   mixin.class_eval <<- CODE, __FILE__, __LINE__ + 1
    55:     def #{name.to_s.singularize}_ids
    56:       association(:#{name}).ids_reader
    57:     end
    58:   CODE
    59: end
```

Et voici la méthode de la classe parente :

```ruby
From: /home/dougui/.rbenv/versions/2.1.5/lib/ruby/gems/2.1.0/bundler/gems/rails-6a7ac40dabf4/activerecord/lib/active_record/associations/builder/association.rb @ line 109 ActiveRecord::Associations::Builder::Association.define_readers:

    108: def self.define_readers(mixin, name)
 => 109:   mixin.class_eval <<- CODE, __FILE__, __LINE__ + 1
    110:     def #{name}(*args)
    111:       association(:#{name}).reader(*args)
    112:     end
    113:   CODE
    114: end
```

J'aime beaucoup ces méthodes. Elles définissent des fonctions dans
le module créé plus tôt. Cela revient à écrire :

```ruby
module GeneratedAssociationMethods
  def comments(*args)
    association(:comments).reader(*args)
  end
end
```

Comme le module a été inclus à la classe du modèle, la fonction est disponible
pour celui-ci. Il est donc possible de faire un appel comme `post.comments`.
Cela revient à écrire `post.association(:comments).reader`.

Vous avez probablement remaqué ceci `<<-CODE, __FILE__, __LINE__ + 1`. Ce code
permet de spécifier une ligne et un fichier au code généré. Il est donc
possible def faire ceci :

```ruby
 > Post.instance_method(:comments).source_location
=> ["/home/dougui/.rbenv/versions/2.1.5/lib/ruby/gems/2.1.0/bundler/gems/rails-698afe1173c6/activerecord/lib/active_record/associations/builder/association.rb", 110]
```

La méthode `define_writers` suit exactement le même principe.

Nous revoici donc à la méthode `build`.

```ruby
From: /home/dougui/.rbenv/versions/2.1.5/lib/ruby/gems/2.1.0/bundler/gems/rails-6a7ac40dabf4/activerecord/lib/active_record/associations/builder/association.rb @ line 31 ActiveRecord::Associations::Builder::Association.build:

    21: def self.build(model, name, scope, options, &block)
    22:   if model.dangerous_attribute_method?(name)
    23:     raise ArgumentError, "You tried to define an association named #{name} on the model #{model.name}, but " \
    24:                          "this will conflict with a method #{name} already defined by Active Record. " \
    25:                          "Please choose a different association name."
    26:   end
    27:
    28:   extension = define_extensions model, name, &block
    29:   reflection = create_reflection model, name, scope, options, extension
    30:   define_accessors model, reflection
 => 31:   define_callbacks model, reflection
    32:   define_validations model, reflection
    33:   reflection
    34: end
```

Les méthodes `define_callbacks` et `define_validations` ont des noms qui parlent
d'eux même. Chacune de ces méthodes mérite un article à elle même. Je ne
m'éterniserais donc pas sur le sujet.

Nous pouvons donc revenir à la méthode supérieure.

```ruby
From: /home/dougui/.rbenv/versions/2.1.5/lib/ruby/gems/2.1.0/bundler/gems/rails-6a7ac40dabf4/activerecord/lib/active_record/associations.rb @ line 1264 ActiveRecord::Associations::ClassMethods#has_many:

    1264: def has_many(name, scope = nil, options = {}, &extension)
    1265:   reflection = Builder::HasMany.build(self, name, scope, options, &extension)
 => 1266:   Reflection.add_reflection self, name, reflection
    1267: end
```

La prochaine méthode est simplement un ajout de `reflection` à la liste connu des reflections.

Nous avons maintenant compris comment ActiveRecord construisait l'association,
mais pas la méthode utilisée pour rechercher les enregistrements. Pour ce
faire, je vais explorer `post.comments`.

Comme prévu, la première méthode est la suivante :

```ruby
    110: def #{name}(*args)
 => 111:   association(:#{name}).reader(*args)
    112: end
```

Il s'agit de celle qui a été créée ci-haut. L'association est ensuite cherchée
dans le cache et la méthode `reader` lui est appliquée.

```ruby
From: /home/dougui/.rbenv/versions/2.1.5/lib/ruby/gems/2.1.0/bundler/gems/rails-6a7ac40dabf4/activerecord/lib/active_record/associations/collection_association.rb @ line 30 ActiveRecord::Associations::CollectionAssociation#reader:

    29: def reader(force_reload = false)
 => 30:   if force_reload
    31:     klass.uncached { reload }
    32:   elsif stale_target?
    33:     reload
    34:   end
    35:
    36:   if owner.new_record?
    37:     # Cache the proxy separately before the owner has an id
    38:     # or else a post-save proxy will still lack the id
    39:     @new_record_proxy ||= CollectionProxy.create(klass, self)
    40:   else
    41:     @proxy ||= CollectionProxy.create(klass, self)
    42:   end
    43: end
```

Si le rechargement n'est pas forcé ou si le modèle n'est pas "vicié", le modèle
n'est pas rechargé. Je me suis cassé les dents sur la méthode `stale_target?`.
Je vous invite à lire [cette issue](https://github.com/rails/rails/issues/18633#event-234692317)
pour plus de détails. Le proxy est également mémorisé. La classe
`CollectionProxy` hérite de `ActiveRecord::Relation` qui est chargée de la
relation avec la base de données. L'interface se fait grâce à Arel, mais je
pense réserver les explications pour un prochain article.

Si l'on fait `post.comments.class.name` on peut voir que c'est bien la classe
`ActiveRecord::Associations::CollectionProxy` qui est utilisée.

## Conclusion

Explorer le code de Rails est un exercice assez difficile, mais toujours très
intéressant. Cette partie est probablement l'une des plus complexes de Rails.
L'exploration permet de comprendre Rails, de mieux l'utiliser de s'en inspirer
si l'on souhaite recréer un code similaire.
