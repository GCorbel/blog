---
layout: post
title: "Comment ActiveRecord reconnait la structure d'une table d'un modèle"
date: 2015-01-25 20:06
comments: true
categories:
---

ActiveRecord est probablement un des Gem qui impressionne le plus quand on
commence avec Rails. Comme par magie, ActiveRecord est capable de comprendre
seul quelle table est utilisée pour un modèle. Quand on connait Rails, on
comprend qu'il n'y a aucune magie. Dans cet article, je vais démystifier
ActiveRecord grâce à Pry.

<!-- more -->

Dans cet article, j'utilise la version 4.2 d'ActiveRecord.

Avant de commencer, il est important de comprendre comment explorer avec Pry. Je vous invite à lire [sur le sujet](http://gcorbel.github.io/blog/blog/2015/01/20/exploration-avec-pry/) car ce sont ces techniques que je vais utiliser ici.

## Point de départ

Comme code de départ, je vais prendre celui de [mon article sur ActiveRecord sans Rails](http://gcorbel.github.io/blog/blog/2015/01/20/utiliser-activerecord-sans-rails/) légèrement modifié :

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

require 'pry'; binding.pry
p User.columns.map { |c| [c.name, c.type] }
```

À l'exécution du programme, nous avons la liste des noms de colonnes avec les
types associés. ActiveRecord à donc trouver la structure de la table.

``` ruby
[["id", :integer], ["name", :string], ["age", :integer], ["created_at", :datetime], ["updated_at", :datetime]
```

Grâce à Pry, il est possible d'entrer dans la fonction `User#columns`. Voici
cette méthode :

```
[1] pry(main)> step

From: /home/dougui/.rbenv/versions/2.1.5/lib/ruby/gems/2.1.0/gems/activerecord-4.2.0/lib/active_record/attributes.rb @ line 91 ActiveRecord::Attributes::ClassMethods#columns:

    90: def columns
 => 91:   @columns ||= add_user_provided_columns(connection.schema_cache.columns(table_name))
    92: end
```

Explorons donc certaines variables ou méthodes qui sont utilisées.

```
[1] pry(User):1> @columns
=> nil
[2] pry(User):1> table_name
=> "users"
[3] pry(User):1> connection
=> #<ActiveRecord::ConnectionAdapters::SQLite3Adapter:0x007ff3a3c9cc20
 @active=nil,
 @config={:adapter=>"sqlite3", :database=>"dbfile.sqlite3"},
 #...
[4] pry(User):1> ls -G table_name
ActiveRecord::ModelSchema::ClassMethods#methods: full_table_name_prefix  full_table_name_suffix  quoted_table_name  reset_table_name  table_name  table_name=
ActiveRecord::Base.methods:
  pluralize_table_names   pluralize_table_names?        schema_migrations_table_name=  table_name_prefix   table_name_prefix?  table_name_suffix=
  pluralize_table_names=  schema_migrations_table_name  schema_migrations_table_name?  table_name_prefix=  table_name_suffix   table_name_suffix?
instance variables:
  @arel_engine  @attribute_methods_generated  @column_names  @columns       @content_columns     @generated_association_methods  @parent_name        @relation                 @sequence_name
  @arel_table   @attributes_builder           @column_types  @columns_hash  @default_attributes  @generated_attribute_methods    @quoted_table_name  @relation_delegate_cache  @table_name
class variables:
  @@configurations    @@dump_schema_after_migration  @@maintain_test_schema     @@raise_in_transactional_callbacks  @@time_zone_aware_attributes
  @@default_timezone  @@logger                       @@primary_key_prefix_type  @@schema_format                     @@timestamped_migrations
```

Plusieurs éléments sont intéressants ici. Premièrement, les colonnes ne sont pas
encore connues par la classe étant donné que la variable `@columns` est nulle.
`table_name` est une méthode retournant la variable `@table_name` qui contient
déjà le nom de la table, "users" dans ce cas. `connection.schema_cache` semble contenir diverses
informations sur la base de données.

Avant d'exécuter `connection.schema_cache.columns(table_name)`, `schema_cache`
ne contient pas les informations sur la structure de la table "users" tandis
qu'elles sont présentes par la suite.

```
[1] pry(User):1> connection.schema_cache.instance_variable_get('@columns')['users']
=> nil
[2] pry(User):1> connection.schema_cache.columns(table_name)
# ...
[3] pry(User):1> connection.schema_cache.instance_variable_get('@columns')['users']
=> [#<ActiveRecord::ConnectionAdapters::Column:0x007fadc7cec288
#...
```

Les informations sont donc stockées dans un cache et ne sont cherchées qu'une
fois si le cache n'est pas vidé.

Pour aller plus loin, il est nécessaire de comprendre comment fonctionne la
méthode de recherche des colonnes.

```
[1] pry(User):1> break ActiveRecord::ConnectionAdapters::SchemaCache#columns                                                                                                                                                          [7/1892]
Breakpoint 1: ActiveRecord::ConnectionAdapters::SchemaCache#columns (Enabled) :

42: def columns(table_name)
43:   @columns[table_name] ||= connection.columns(table_name)
44: end

[2] pry(User):1> continue

Breakpoint 1. First hit

From: /home/dougui/.rbenv/versions/2.1.5/lib/ruby/gems/2.1.0/gems/activerecord-4.2.0/lib/active_record/connection_adapters/schema_cache.rb @ line 42 ActiveRecord::ConnectionAdapters::SchemaCache#columns:

 => 42: def columns(table_name)
    43:   @columns[table_name] ||= connection.columns(table_name)
    44: end

[2] pry(#<ActiveRecord::ConnectionAdapters::SchemaCache>)> step

From: /home/dougui/.rbenv/versions/2.1.5/lib/ruby/gems/2.1.0/gems/activerecord-4.2.0/lib/active_record/connection_adapters/schema_cache.rb @ line 43 ActiveRecord::ConnectionAdapters::SchemaCache#columns:

    42: def columns(table_name)
 => 43:   @columns[table_name] ||= connection.columns(table_name)
    44: end

[2] pry(#<ActiveRecord::ConnectionAdapters::SchemaCache>)> step

From: /home/dougui/.rbenv/versions/2.1.5/lib/ruby/gems/2.1.0/gems/activerecord-4.2.0/lib/active_record/connection_adapters/sqlite3_adapter.rb @ line 389 ActiveRecord::ConnectionAdapters::SQLite3Adapter#columns:

    388: def columns(table_name) #:nodoc:
 => 389:   table_structure(table_name).map do |field|
    390:     case field["dflt_value"]
    391:     when /^null$/i
    392:       field["dflt_value"] = nil
    393:     when /^'(.*)'$/m
    394:       field["dflt_value"] = $1.gsub("''", "'")
    395:     when /^"(.*)"$/m
    396:       field["dflt_value"] = $1.gsub('""', '"')
    397:     end
    398:
    399:     sql_type = field['type']
    400:     cast_type = lookup_cast_type(sql_type)
    401:     new_column(field['name'], field['dflt_value'], cast_type, sql_type, field['notnull'].to_i == 0)
    402:   end
    403: end

[2] pry(#<ActiveRecord::ConnectionAdapters::SQLite3Adapter>)> step

From: /home/dougui/.rbenv/versions/2.1.5/lib/ruby/gems/2.1.0/gems/activerecord-4.2.0/lib/active_record/connection_adapters/sqlite3_adapter.rb @ line 516 ActiveRecord::ConnectionAdapters::SQLite3Adapter#table_structure:

    515: def table_structure(table_name)
 => 516:   structure = exec_query("PRAGMA table_info(#{quote_table_name(table_name)})", 'SCHEMA').to_hash
    517:   raise(ActiveRecord::StatementInvalid, "Could not find table '#{table_name}'") if structure.empty?
    518:   structure
    519: end
```

Nous voilà à une fonction des plus intéressante. ActiveRecord execute la requête `PRAGMA table_info("users")`.


```
[3] pry(#<ActiveRecord::ConnectionAdapters::SQLite3Adapter>)> exec_query("PRAGMA table_info(#{quote_table_name(table_name)})", 'SCHEMA').to_hash
=> [{"cid"=>0, "name"=>"id", "type"=>"INTEGER", "notnull"=>1, "dflt_value"=>nil, "pk"=>1},
 {"cid"=>1, "name"=>"name", "type"=>"varchar", "notnull"=>0, "dflt_value"=>nil, "pk"=>0},
 {"cid"=>2, "name"=>"age", "type"=>"integer", "notnull"=>0, "dflt_value"=>nil, "pk"=>0},
 {"cid"=>3, "name"=>"created_at", "type"=>"datetime", "notnull"=>1, "dflt_value"=>nil, "pk"=>0},
 {"cid"=>4, "name"=>"updated_at", "type"=>"datetime", "notnull"=>1, "dflt_value"=>nil, "pk"=>0}]
```

Avec SQLite, cette requête renvoie les informations sur la table et c'est à
partir de ces informations qu'ActiveRecord reconstitut les données.

Cette requête fonctionne pour SQLite mais il en existe une similaire pour
Postgres.

```ruby
#activerecord/lib/active_record/connection_adapters/postgresql_adapter.rb:12
def column_definitions(table_name) # :nodoc:
  exec_query(<<-end_sql, 'SCHEMA').rows
      SELECT a.attname, format_type(a.atttypid, a.atttypmod),
             pg_get_expr(d.adbin, d.adrelid), a.attnotnull, a.atttypid, a.atttypmod
        FROM pg_attribute a LEFT JOIN pg_attrdef d
          ON a.attrelid = d.adrelid AND a.attnum = d.adnum
       WHERE a.attrelid = '#{quote_table_name(table_name)}'::regclass
         AND a.attnum > 0 AND NOT a.attisdropped
       ORDER BY a.attnum
  end_sql
end
```

Il en est de même pour MySql.

```ruby
#activerecord/lib/active_record/connection_adapters/abstract_mysql_adapter.rb:464
def columns(table_name)#:nodoc:
  sql = "SHOW FULL FIELDS FROM #{quote_table_name(table_name)}"
  execute_and_free(sql, 'SCHEMA') do |result|
    each_hash(result).map do |field|
      field_name = set_field_encoding(field[:Field])
      sql_type = field[:Type]
      cast_type = lookup_cast_type(sql_type)
      new_column(field_name, field[:Default], cast_type, sql_type, field[:Null] == "YES", field[:Collation], field[:Extra])
    end
  end
end
```

## Conclusion

Je pense que cette brève inspection nous suffit pour comprendre ce qu'il se
passe sous le capot. Le fonctionnement est assez simple et serait
facile à reproduire pour d'autres cas d'utilisation. Pry nous a permis, en
quelques minutes, de comprendre le code et de passer au travers l'apparente
magie de Rails.

Si vous voulez avoir d'autres explications sur le fonctionnement de Rails,
dites-le-moi et je ferais un processus similaire pour essayer de comprendre.
