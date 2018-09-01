---
title: "Postgres Data Types on Rails"
subtitle: "Better native types mappings for Active Record"
date: 2016-06-08T14:56:33+02:00
tags: ["postgres", "rails", "ruby"]
summary: "After watching, @pvh’s talk about PostgreSQL data types, I started looking what default types ActiveRecord uses for PostgreSQL when running a database migration."
---

After watching, [@pvh](https://twitter.com/pvh)’s talk about [PostgreSQL data types](https://www.youtube.com/watch?v=BLX8N3HB8sQ), I started looking what default types ActiveRecord uses for PostgreSQL when running a database migration.

By default, the ActiveRecord PostgreSQL adapter uses **serial** for primary keys, **timestamp without time zone** for datetime/timestamp, and **varchar** for strings ([ref](https://github.com/rails/rails/blob/v4.2.6/activerecord/lib/active_record/connection_adapters/postgresql_adapter.rb#L79..L114)).

Based on the suggestions of the talk, You probably want to use **bigserial** instead of **serial**, because of the larger amount of serial values ([uuids](http://blog.arkency.com/2014/10/how-to-start-using-uuid-in-activerecord-with-postgresql/) are another option), and [always timestamp with time zones](https://www.depesz.com/2014/04/04/how-to-deal-with-timestamps/).

I was able to change the default through this initializer file in my application:

{{< highlight ruby >}}
# config/initializers/active_record_native_database_types.rb
require "active_record/connection_adapters/postgresql_adapter"

module ActiveRecord
 module ConnectionAdapters
   class PostgreSQLAdapter
     NATIVE_DATABASE_TYPES.merge!(
       primary_key: "bigserial primary key",
       datetime:  { name: "timestamptz" },
       timestamp: { name: "timestamptz" }
       string:    { name: "text" }
     )
   end
 end
end
{{</ highlight >}}

> **NOTE: varchar** and **text** are the same. This change is just personal preference (because is short!). Varchar limit [was removed](https://github.com/rails/rails/pull/14579) in Rails 4.2.

Unfortunately, this patch is not backward compatible, so it’s better for new applications. Still, you can use these data types in new migrations:

{{< highlight ruby >}}
class CreateUsers < ActiveRecord::Migration
 def change
   create_table :persons, id: :bigserial do |t|
     t.text :first_name
     t.text :last_name
     t.column :created_at, :timestamptz, null: false
     t.column :updated_at, :timestamptz, null: false
   end
 end
end
{{</ highlight >}}

**Update 1:** There is an open pull request in the Rails’ GitHub repository to use bigserial as default: https://github.com/rails/rails/pull/24962. Looks like there are plans for timestamptz by default too.

**Update 2:** Rails 5 has a feature to change the default primary key type when generating database migrations: http://blog.mccartie.com/2015/10/20/default-uuid's-in-rails.html.
