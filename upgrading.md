# Upgrading
This document provides the necessary steps that need to be performed when upgrading from previous versions.

## Upgrading from version [4.0.4, 4.0.7] to version >= 4.1.0
### Table
Version `4.1.0` introduces an `updated_at` column to the `audits` table.
Use the following migration to convert a _default_ [`4.0.4`, `4.0.7`] table structure into the `4.1.0` version:

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Config;
use Illuminate\Support\Facades\Schema;

class UpdateAuditsTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::connection(Config::get('audit.drivers.database.connection'))
            ->table(Config::get('audit.drivers.database.table'), function (Blueprint $table) {
                $table->timestamp('updated_at')->nullable();
            });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::connection(Config::get('audit.drivers.database.connection'))
            ->table(Config::get('audit.drivers.database.table'), function (Blueprint $table) {
                $table->dropColumn('updated_at');
            });
    }
}
```

Set the same `created_at` column value to `updated_at` and make sure the `created_at` stays the same.

```php
DB::table('audits')->whereNull('updated_at')->update([
    'created_at' => DB::raw('created_at'),
    'updated_at' => DB::raw('created_at'),
]);
```

> {tip} Any customisations made to the original migration should be taken into account. Do not blindly copy-paste!

## Upgrading from version [4.0.0, 4.0.3] to version >= 4.1.0
### Table
Use the following migration to convert a _default_ [`4.0.0`, `4.0.3`] table structure into the `4.1.0` version:

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Config;
use Illuminate\Support\Facades\Schema;

class UpdateAuditsTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::connection(Config::get('audit.drivers.database.connection'))
            ->table(Config::get('audit.drivers.database.table'), function (Blueprint $table) {
                $table->string('user_agent')->nullable();
                $table->timestamp('updated_at')->nullable();
            });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::connection(Config::get('audit.drivers.database.connection'))
            ->table(Config::get('audit.drivers.database.table'), function (Blueprint $table) {
                $table->dropColumn('user_agent');
                $table->dropColumn('updated_at');
            });
    }
}
```

Set the same `created_at` column value to `updated_at` and make sure the `created_at` stays the same.

```php
DB::table('audits')->whereNull('updated_at')->update([
    'created_at' => DB::raw('created_at'),
    'updated_at' => DB::raw('created_at'),
]);
```

> {tip} Any customisations made to the original migration should be taken into account. Do not blindly copy-paste!

## Upgrading from version >= 3.1.0 to version >= 4.1.0
### Model
All `Auditable` models must implement the `OwenIt\Auditing\Contracts\Auditable` interface.

> {tip} Take a look at the brief example in the [Introduction](introduction).

### Table
Use the following migration to convert a _default_ `3.1.x` table structure into the `4.1.0` version:

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Config;
use Illuminate\Support\Facades\Schema;

class UpdateAuditsTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::connection(Config::get('audit.drivers.database.connection'))
            ->table(Config::get('audit.drivers.database.table'), function (Blueprint $table) {
                $table->increments('id')->change();
                $table->renameColumn('type', 'event');
                $table->renameColumn('old', 'old_values');
                $table->renameColumn('new', 'new_values');
                $table->unsignedInteger('user_id')->nullable()->change();
                $table->renameColumn('route', 'url');
                $table->string('user_agent')->nullable();
                $table->timestamp('updated_at')->nullable();
            });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::connection(Config::get('audit.drivers.database.connection'))
            ->table(Config::get('audit.drivers.database.table'), function (Blueprint $table) {
                $table->string('id')->change();
                $table->renameColumn('event', 'type');
                $table->renameColumn('old_values', 'old');
                $table->renameColumn('new_values', 'new');
                $table->string('user_id')->nullable()->change();
                $table->renameColumn('url', 'route');
                $table->dropColumn('user_agent');
                $table->dropColumn('updated_at');
            });
    }
}
```

Set the same `created_at` column value to `updated_at` and make sure the `created_at` stays the same.

```php
DB::table('audits')->whereNull('updated_at')->update([
    'created_at' => DB::raw('created_at'),
    'updated_at' => DB::raw('created_at'),
]);
```

> {tip} Any customisations made to the original migration should be taken into account. Do not blindly copy-paste!

### Configuration

From version `4.0.0` onward, the configuration file has been renamed from `auditing.php` to `audit.php`.
Its `array` structure has also been updated, with options being **added**, **modified** and **removed**.

> {tip} When in doubt, check the bundled configuration file.

### Queues
Audit queue support has been removed since version `4.0.0`.

### Custom message and fields
Since version `4.0.0`, `Auditable` custom message and fields are no longer necessary.
Models should be updated to reflect this change, by removing the `$auditCustomMessage` and `$auditCustomFields` attributes.

> {tip} Documentation on how to display changes can be seen in the [Audit presentation](audit-presentation) section.
