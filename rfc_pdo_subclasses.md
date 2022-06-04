# PHP RFC: PDO driver specific sub-classes
  * Version: 0.9
  * Date: 2022-06-04
  * Author: Danack
  * Status: Draft
  * First Published at: http://wiki.php.net/rfc/PDOSubclasses

## Introduction 

PDO is a generic database class. Some of the databases it supports have functionality that is specific to that database.

For example, when connected to an SQLite database, the sqlite specific function [PDO::sqliteCreateFunction](https://www.php.net/manual/en/pdo.sqlitecreatefunction.php) is available.

This is a slightly shitty way of doing things. And is an example of code that was snuck in before the RFC [process was in place.](https://github.com/php/php-src/commit/41421c7d7aacd831d03f008735c90bdf02bc196b)

This RFC continues [a discussion on internals](https://news-web.php.net/php.internals/100773) of reimplementing this functionality 'properly'.

## Proposal 

The proposal has three parts:

* Add new subclasses of PDO
* Add PDO::connect to be able to create them
* Add DB specific function to those subclasses.

### Add new subclasses of PDO

There will be one subclasses for each known PDO support DB connection:

* PDODblib
* PDOFirebird
* PDOMysql
* PDOOci
* PDOOdbc
* PDOpgsql  - definitely
* PDOsqlite - definitely

Each of these subclasses will expose functionality that exists that is peculiar to that DB, and not present in the generic PDO class. e.g. PDOSqlite::createFunction() should be there rather than on [PDO::sqliteCreateFunction](https://www.php.net/manual/en/pdo.sqlitecreatefunction.php).

### Add a way of creating them through PDF static factory method

Add a static factory method to PDO called `connect`. During the connect process, the exact type of database being connected to will be checked, and if it is a DB that has a specific sub-class, return that sub-class instead of a generic PDO object

```
class PDO
{
    public static function connect(string $dsn [, string $username [, string $password [, array $options ]]]) {

        if (connecting to SQLite DB) {
            return new PDOSqlite(...);
        }


        return new PDO(...);
    }
}
```

PDO::connect will return the appropriate sub-class when connecting to specific type of DB.


### Add DB specific function to those subclasses.


## Backward Incompatible Changes 

None known. It might be a bit of a pain in the arse for people who are extending PDO class directly


## Proposed PHP Version(s) 

PHP 8.2

## Open Issues 

There are the known current issues.

### Should the sub-classes only be avaiable when support for that DB is compiled in?

e.g. should PDOSqlite exist if PHP was compiled without PDO_SQLITE compilted in? Probaby not.

### Create all DB sub-classes?

Should all DBs have sub-classes created now, or should they be added when someone requests it?

### When to deprecate old function on PDO

The method PDO::sqliteCreateFunction should probably be deprecated and removed at 'some point'. Although this cleanup should happen, the position of this RFC is that it's very low priority.

8.2 - PDO subclassses become available
9.0 - calls to PDO::sqliteCreateFunction start raising deprecation notice.
10.0 - method PDO::sqliteCreateFunction is removed, and that functionality can only be accessed through PDOSqlite


## Unaffected PHP Functionality 

Everything else

## Future Scope 

Any DB specific stuff known about that were too much work to implement?

## Proposed Voting Choices 

Accept the RFC as proposed 2/3 required.

## Patches and Tests 

Links to any external patches and tests go here.

## Implementation
After the project is implemented, this section should contain
  - the version(s) it was merged into
  - a link to the git commit(s)
  - a link to the PHP manual entry for the feature
  - a link to the language specification section (if any)

## References 
Links to external references, discussions or RFCs

## Rejected Features 
Keep this updated with features that were discussed on the mail lists.
