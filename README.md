# rethinkdb-migration
[![license](https://img.shields.io/github/license/mashape/apistatus.svg)]()
[![Build Status](https://travis-ci.org/FutureProcessing/rethinkdb-migration.svg?branch=master)](https://travis-ci.org/FutureProcessing/rethinkdb-migration)

RethinkDB migration tool allows you to handle database migration for your Node application.
Additionally you can define individual migration for stage, uat, etc.
Rethinkdb-migration will take care of running proper migration script from core directory and indicated implementation.

**IMPORTANT**: You need Node >= 8.9.4 for this package.

## Installation

Installation is done using the npm install command:
```
npm install rethinkdb-migration
```

## Configuration

You may configure rethinkdb-migration in two ways: by environment variables or provide configuration file.

### Environment variables.

##### Database configuration
```
TRM_DB_HOST,
TRM_DB_PORT,
TRM_DB_USER,
TRM_DB_PASS,
TRM_DB_NAME,
```
##### Tool configuration
```
TRM_DB_TABLE, specify table where rethinkdb-migration keeps migration history
TRM_DIR_MIGRATE, specify directory where rethinkdb-migration keeps migration scripts
TRM_DIR_IMPLEMENTATION, specify directory with additional implementation e.g STAGE, UAT
```

### Configuration file

If you do not want to add environment variables to your system you may provide configuration file instead.
```javascript
{
  "DB_HOST": "192.168.10.30",
  "DB_PORT": "28015",
  "DB_USER": "",
  "DB_PASS": "",
  "DB_NAME": "rethinkMigrateTest",
  "DB_TABLE": "toolRethingDBMigrate",
  "DIR_MIGRATE": "./migrate",
  "DIR_IMPLEMENTATION": "stage"
}
````
Example file with default data you can find at **config/config-default.json**

## Usage

Let's create migration script with query which creates new table in database.
Thus, we use config file to provide configuration for rethinkdb-migration.
##### Define following data in *config-example.json*
```javascript
{
  "DB_HOST": "localhost",
  "DB_PORT": "2815",
  "DB_USER": "",
  "DB_PASS": "",
  "DB_NAME": "rethinkMigrateTest",
  "DB_TABLE": "toolRethingDBMigrate",
  "DIR_MIGRATE": "./migrate",
  "DIR_IMPLEMENTATION": ""
}
````
##### Generate migration script from template. Use following command:

```
node rethinkdb-migration -cs example-migration --config="./../config/config-example.json"
```

Above command creates file with name based on pattern **yyyyMMdd_HHmm-example-migration.js** in **migrate** directory.

Generated file should have following content:

```javasctipt
async function up(rethink, migrate) {
    await migrate.finishMigrationScript();
}

module.exports = {
    up
};
```

##### Add rethinkdb query in order to create new table.

```javasctipt
async function up(rethink, migrate) {
    await rethink.getReQL().tableCreate('exampleTable').run();
    await migrate.finishMigrationScript();
}

module.exports = {
    up
};
```

or you may use method **createMissingTables()**

```javasctipt
async function up(rethink, migrate) {
    rethink.createMissingTables('exampleTable');
    await migrate.finishMigrationScript();
}

module.exports = {
    up
};
```

#### Running

Created migration can be launched by the command
```
node rethinkdb-migration --config="./../config/config-default.json"
```
When you run above command table **exampleTable** will be create in database which you defined in configuration file.

In table **toolRethingDBMigrate** you can see migration history.
```javascript
{

    "date": Wed Jan 31 2018 11:47:31 GMT+00:00 ,
    "fileName": "20180131_1241-example-migration.js" ,
    "id": "89a98790-a51b-4eaa-a55e-d4906cde7341" ,
    "version": "20180131_1241"

}
```

#### Pre and Post scripts

You may setup some scripts that will be run before and after migrations. This is seldom helpful on AT environments when you need to clear database, insert test data, migrate, and dump data again.

Just put scripts that needs to be run **before** migrations in `migrate/pre` sorted alphabetically f.ex: 
```
./migrate/pre/001_truncate_tables.js
./migrate/pre/002_insert_test_data.js
```
and scripts that needs to be run **after** migrations in folder `migrate/post`:
```
./migrate/post/001_dump_tables.js
```
**IMPORTANT**: scripts must expose function 
```
/**
 * @param {Rethink} rethinkRef
 * @param {Migrate} migrate;
 */
async function up(rethinkRef, migrate)
``` 

#### Helpers

Method | Parameters type | Description
---|---|---|
getReQl() | | Returns rethinkdbdash object.
closeConnection() | | Closes connection to database.
createDatabase(databaseName) | {String} databaseName | Creates database based on given name.
createMissingTables(tablesName) | {Array} tablesName | Creates tables based on given table names.
insertWhenTableIsEmpty(tableName, data) | {String} tableName, {String} data | Insert data to table.

### License
[MIT License](./LICENSE)
