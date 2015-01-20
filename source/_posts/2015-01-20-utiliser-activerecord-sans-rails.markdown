---
layout: post
title: "Utiliser ActiveRecord sans Rails"
date: 2015-01-20 04:19
comments: true
categories:
---

ActiveRecord est l'une des premières classes que l'on apprend à utiliser avec
Rails. Dans la vie d'un Rubyste, il est possible que l'on souhaite l'utiliser
sans Rails pour l'exécuter dans un programme en ligne de commande
ou dans une application utilisant Sinatra par exemple. Nous allons voir comment
faire en quelques lignes.

<!-- more -->

Cet article est piloté par l'erreur. Je vais commencer par utiliser
ActiveRecord comme je l'aurais fait avec Rails. Je vais ensuite implémenter le
code en fonction des erreurs obtenues.

## Code

Sans plus attendre, voici une classe permettant de créer le modèle, insérer
un enregistrement et de les lister :

``` ruby
class User < ActiveRecord::Base
end

User.create(name: 'Guirec Corbel', age: 29)
p User.all
```

Évidemment, Ruby nous envoi l'exception `uninitialized constant ActiveRecord (NameError)`. Pour corriger l'erreur, il suffit d'inclure le fichier `active_record`.

``` ruby
require 'active_record'

class User < ActiveRecord::Base
end

User.create(name: 'Guirec Corbel', age: 29)
p User.all
```

L'exception suivante inscrit le message `No connection pool for User (ActiveRecord::ConnectionNotEstablished)`. Il est nécessaire d'établir une nouvelle base de données.

``` ruby
require 'active_record'

class User < ActiveRecord::Base
end

ActiveRecord::Base.establish_connection(adapter: 'sqlite3', database: 'dbfile.sqlite3')

User.create(name: 'Guirec Corbel', age: 29)
p User.all
```

Cette fois, l'erreur provient de SQLite et indique que la table `users` n'existe pas. Comme avec Rails, il est possible de faire une migration.

``` ruby
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
CreateUsers.new.change

User.create(name: 'Guirec Corbel', age: 29)
p User.all
```

Ça marche! Et non, ça ne marche pas tout à fait. La première exécution retourne
bien l'enregistrement, mais la seconde retourne une erreur provenant de SQLite
indiquant que la table existe déjà. Il est nécessaire de créer la table
uniquement si elle n'existe pas encore. Il est possible de le faire en
utilisant la fonction `#table_exists?` de la classe `User`.

Voici donc le résultat final :

``` ruby
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

p User.all
```

## Conclusion

Toutes les informations contenues dans cet article peuvent être trouvées [sur le Github de rails](https://github.com/rails/rails/tree/master/activerecord). Comme dirait Avdi Grimm, Happy Hacking!
