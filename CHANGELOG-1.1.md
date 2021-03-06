CHANGELOG for 1.1.x
===================

This changelog references the relevant changes done in 1.1 minor versions. If upgrading from
1.0.x branch, please review
[Upgrade Path](https://github.com/doctrine/mongodb-odm/blob/master/CHANGELOG-1.1.md#upgrade-path).

1.1.0
-----

All issues and pull requests in this release may be found under the
[1.1.0 milestone](https://github.com/doctrine/mongodb-odm/issues?q=milestone%3A1.1.0).

#### Sharding support

[#1385](https://github.com/doctrine/mongodb-odm/pull/1385) -
Introduces full [sharding](https://docs.mongodb.com/manual/sharding/) support for ODM.
For usage instructions please refer to
[Sharding chapter](http://docs.doctrine-project.org/projects/doctrine-mongodb-odm/en/latest/reference/sharding.html)
in the documentation.

#### Custom Collections

[#1219](https://github.com/doctrine/mongodb-odm/pull/1219) -
Allows developer to use their own collection classes instead of `ArrayCollection` for
`@EmbedMany` and `@ReferenceMany` fields and gives full control on how the collections
are instantiated. For full usage reference please check out
[Custom Collections chapter](http://docs.doctrine-project.org/projects/doctrine-mongodb-odm/en/latest/reference/custom-collections.html)
in the documentation.

#### Event is dispatched for missing referenced documents

[#1336](https://github.com/doctrine/mongodb-odm/pull/1336) -
When referenced document is accessed but not found in database ODM throws
`DocumentNotFoundException` exception. As of now user can hook into the process
and populate incomplete object on his/her own or take any other suitable action.
For example usage please refer to
[documentNotFound event documentation](http://docs.doctrine-project.org/projects/doctrine-mongodb-odm/en/latest/reference/events.html#documentnotfound).

#### Partial indexes

[#1303](https://github.com/doctrine/mongodb-odm/pull/1303)
Partial indexes [introduced in MongoDB 3.2](https://docs.mongodb.com/manual/core/index-partial/)
are also supported as ODM's `@Index` option. For example usage please refer to
[index documentation](http://docs.doctrine-project.org/projects/doctrine-mongodb-odm/en/latest/reference/indexes.html#partial-indexes)

#### More powerful lifecycle callbacks

[#1222](https://github.com/doctrine/mongodb-odm/pull/1222) -
Document's lifecycle callbacks are now receiving an event argument which allows user
to access `DocumentManager` should (s)he need it. For full method signatures please see
[Event chapter](http://docs.doctrine-project.org/projects/doctrine-mongodb-odm/en/latest/reference/events.html#lifecycle-callbacks)
of documentation.

#### Ease using objects as search criterion

[#1363](https://github.com/doctrine/mongodb-odm/pull/1363) -
querying for references is now less tedious, now you can use

```
$dm->getRepository(User::class)->findOneBy(['account' => $account]);
$dm->getRepository(User::class)->findBy(['groups' => $group]);
```

[#1333](https://github.com/doctrine/mongodb-odm/pull/1333) -
ODM will go through known class' descendants when query builder's `references` or
`includesReferenceTo` are used with non-existing field (i.e. discriminated abstract
class being queried for references defined in child classes).

#### Read-Only mode for fetching documents

[#1403](https://github.com/doctrine/mongodb-odm/pull/1403) -
Read-Only mode instructs ODM to not only hydrate the latest data but also to
create new document's instance (i.e. if found document would be already managed
by Doctrine, new instance will be returned) and not register it in `UnitOfWork`.
This technique can prove especially useful when using `select()` while building
query with no intent to update fetched documents. To read more about read-only mode
you can check [documentation](http://docs.doctrine-project.org/projects/doctrine-mongodb-odm/en/latest/reference/query-builder-api.html#fetching-documents-as-read-only).

#### EmbedMany and ReferenceMany fields included in the change set

[#1339](https://github.com/doctrine/mongodb-odm/pull/1339) -
Previously hard to spot changes in collections are now easier to detect thanks to
including them in the owning document's change set. While collections' instances
are usually the same you can check both new and removed documents with `getInsertedDocuments`
and `getDeletedDocuments` methods respectively.

#### Support for immutable DateTime objects

[#1391](https://github.com/doctrine/mongodb-odm/pull/1391) -
The `Date` type now supports converting `DateTimeImmutable` objects to `MongoDate`. Previously, only
`DateTime` was converted, causing errors when using immutable date objects.

#### Support for writeConcern per document

[#1419](https://github.com/doctrine/mongodb-odm/pull/1419) -
The new `writeConcern` property in the document mapping allows users to overwrite the default write
concern specified in the configuration. With this, it's possible to allow unacknowledged writes for
certain documents or require writes to more than one node for others.

PHP 7 compatibility
-------------------

While ODM still relies on legacy MongoDB driver (i.e. [ext-mongo](https://pecl.php.net/package/mongo))
it is possible to run ODM using PHP 7 and HHVM thanks to
[alcaeus/mongo-php-adapter](https://github.com/alcaeus/mongo-php-adapter) which provides old API
atop new [ext-mongodb](http://php.net/manual/en/mongodb.installation.php) driver and
[mongodb/mongo-php-library](https://github.com/mongodb/mongo-php-library) library. Until ODM 2.0 is
released this is officially supported way and is included in our test suite. If you are using
annotations to map documents, do not use the deprecated annotations mentioned below as they do not
work in PHP 7.

Upgrade Path
------------

#### PHP requirement changed

PHP 5.3, 5.4 and 5.5 support has been dropped due to their [end of life](http://php.net/eol.php)
(or getting close to it in case of 5.5).

#### preLoad lifecycle callback signature change

`preLoad` lifecycle callback now receives `PreLoadEventArgs` object instead of an array of data
passed by reference. For reasoning why the change was made please see relevant pull request
([#1222](https://github.com/doctrine/mongodb-odm/pull/1222)).

Deprecations
------------

#### `@Field` preferred way of mapping field

Due to PHP 7 reserving keywords such as `int`, `string`, `bool` and `float` their respective
field annotations are no longer valid. To avoid having large inconsistencies
[#1318](https://github.com/doctrine/mongodb-odm/pull/1318) deprecates all annotations which
only purpose was setting mapped field's type. Deprecated classes will be removed in version 2.0.

#### `@Increment` superseded by storage strategies

[#1352](https://github.com/doctrine/mongodb-odm/pull/1352) deprecates `@Increment` field type
in favour of more robust `strategy` field option. To learn more about storage strategies
please see relevant chapter in [documentation](http://docs.doctrine-project.org/projects/doctrine-mongodb-odm/en/latest/reference/storage-strategies.html).
`increment` field type will be removed in version 2.0.

#### `Simple` references superseded by `storeAs`

[#1349](https://github.com/doctrine/mongodb-odm/pull/1349) deprecates the `simple` option used
in references and replaces it with the new `storeAs` option. This allows for an additional reference
mode called `dbRef` which doesn't store a `$db` field in the `DBRef` object. For compatibility with
existing references, the default mode is `dbRefWithDb` but will be replaced with `dbRef` in version
2.0. For more information, see the
[reference mapping documentation](http://docs.doctrine-project.org/projects/doctrine-mongodb-odm/en/latest/reference/reference-mapping.html#storing-references).

1.0.x End-of-Life
-----------------

ODM 1.1 drops older PHP versions which may be problematic for some users. Although we strongly
recommend running latest PHP we understand this may not be easy change therefore we will still
support ODM 1.0.x and release bugfix versions for **6 months** from now on.
