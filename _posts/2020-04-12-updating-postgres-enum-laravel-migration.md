---
layout: post
title: "Updating a Postgres Enum Column with a Laravel Migration"
date: 2020-04-12 09:00:00 +0800
tags: [postgres,laravel,database,php]
navigation: true
---

The enumerated data type (enum) provides a convenient way to constraint a column to a set of values.
However, the implementation of enum in [Laravel](https://laravel.com) is not as it seems, and changing the set of values can prove difficult later on.
This post looks at how Laravel sets a column to enum in the database and provides details on how to change the available set of values using a migration script.

## Laravel Enum
As already mentioned, when a column is set to enum in a migration script, the resulting data type in the database for that column is different.
Below shows an example of a migration script setting a column to enum and the resulting data type in the database.

```php
public function up()
{
    Schema::create('messages', function (Blueprint $table) {
        ...
        $table->enum('status', ['draft', 'sent', 'read']);
        ...
    });
}
```

```
Column     |              Type              | Collation | Nullable | Default                 
-----------+--------------------------------+-----------+----------+-----------------------------------------
...
status     | character varying(255)         |           | not null  |
...
```

As can be seen in the data description above, the data type set for the status column in the database is character varying(255).
This can be seen in the [source code of Laravel](https://github.com/laravel/framework/blob/0fd4c5c58982ce255733baf10f3fdb2eee24e510/src/Illuminate/Database/Schema/Grammars/PostgresGrammar.php#L616).

Therefore, to limit the column to only the set of values provided, a constraint is added to the column, as shown below.

```
Check constraints:
    "messages_status_check" CHECK (status::text = ANY (ARRAY['draft'::character varying, 'sent'::character varying, 'read'::character varying]::text[]))
```

## Updating an Enum
Looking at the [Laravel documentation](https://laravel.com/docs/7.x/migrations#modifying-columns), it states that updating a column with type enum is not supported when using the [Doctrine DBAL library](https://www.doctrine-project.org).
However, since we know about the actual implementation of an enum column, discussed above, we can look for a method of updating such a column.

An approach that can be taken to altering the constraint is shown below.
First, the constraint is dropped.
Second, the new constraint is generated as a string.
Finally, the new constraint is applied to the messages table.

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Schema;

class ChangeEnum extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        DB::statement("ALTER TABLE messages DROP CONSTRAINT messages_status_check");

        $types = ['draft', 'sent', 'read', 'replied'];
        $result = join( ', ', array_map(function ($value){
            return sprintf("'%s'::character varying", $value);
        }, $types));

        DB::statement("ALTER TABLE messages ADD CONSTRAINT messages_status_check CHECK (status::text = ANY (ARRAY[$result]::text[]))");
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        //
    }
}
```

> View this code as a [GitHub Gist](https://gist.github.com/jonathanstaniforth/c48b22960ce13bf303c8e04a591c0ef3).

## Summary
To summarise, this post investigated the enum data type used by Laravel.
First, an example of how a column can be set to the type enum using a migration script is shown.
Second, the actual implementation of the enum type in the database was revealed.
Finally, an example of how to update a column that has been set to enum was presented, as it is not currently supported by the Doctrine DBAL library.
