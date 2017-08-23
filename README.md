## Intent

This is a personal project is a fork of the original efcoreshaman repository to resolve issues found in .Net Core 2.0.

## Core.Shaman

There is still a lot of flaws in EntityFrameworkCore. Many of them is really irritating. That's why first name of this library was EfCore.Irritation :).

EfCore.Shaman offers simple solution for some of problems. For more information please visit [http://efcoreshaman.com/](http://efcoreshaman.com/).

## Quick start - 3 steps to fix many problems

1. Install nuget package 	

    `Install-package EfCore.Shaman`

2. Enter following code in `Up` method in each `Migration` class:

    ````csharp
   migrationBuilder.FixMigrationUp<YourDbConextClass>();
   ````

3. Enter following code in `OnModelCreating` method in `DbContext` class:

   ````csharp
   this.FixOnModelCreating(modelBuilder);
   ````

## Columns order in table

Ef uses its own sorting method for columns in table. Primary key fields are first followed by other fields in alphabetical order. Neither natural properties order in 'code first' classes nor `ColumnAttibute.Order` value is used for sorting.

To change this behaviour put code below at the end of `protected override void Up(MigrationBuilder migrationBuilder)` method in each class derived from `Migration`.
````csharp
migrationBuilder.FixMigrationUp<YourDbConextClass>();
````

New column order is based on natural properties order in 'code first' classes. This can be overrided by `ColumnAttibute.Order` value.

## Indexes

EfCore doesn't support annotation for index creation. Each index definition must be in `OnModelCreating` method in `DbContext` class. It is inconsequence leading to worse code readablity. 

EfCore.Shaman offers own `IndexAttribute` and `UniqueIndexAttribute`. In order to use this attributes put following code at the end of 'OnModelCreating' method in you `DbContext` class.

````csharp
this.FixOnModelCreating(modelBuilder);
````

### Index creation options

#### Simple one column index

Just use `IndexAttribute` and `UniqueIndexAttribute` without any parameter like below:

````csharp
[Index]
public Guid InstanceUid { get; set; }
````

#### Own index name
````csharp
[Index("MyName")]
public Guid InstanceUid { get; set; }
````

#### Multi column indexes

To create index with more than one column put `IndexAttribute` or `UniqueIndexAttribute` with some name and number related to field position in in index

````csharp
[Index("Idx_myName", 1)]
public Guid InstanceUid { get; set; }

[Index("Idx_myName", 2)]
public Guid BoxUid { get; set; }
````

You can also use names starting with `@` sign. That names will be replaced by name generated automatically by EF. 

````csharp
[Index("@1", 1)]
public Guid InstanceUid { get; set; }

[Index("@1", 2)]
public Guid BoxUid { get; set; }
````

#### Descending field sorting

`IndexAttribute` and `UniqueIndexAttribute` contains `bool IsDescending` property designed for changing sorting order of index column. This is **not yet supported**. 

````csharp
[Index("@1", 1)]
public Guid InstanceUid { get; set; }

[Index("@1", 2, true)]
public Guid BoxUid { get; set; }
````

## Decimal columns precision

`DecimalTypeAttribute` can be used for decorating decimal property. It affects changing decimal column type definition. For example 

````csharp
[DecimalType(18, 6)]
public decimal Amount { get; set; }
````
## Specify Default values for columns

`DefaultValueAttribute` and `DefaultValueSqlAttribute` can be used for setting default values on columns. For example 

````csharp
[DefaultValue(true)]
public bool IsActive { get; set; }
````

````csharp
[DefaultValueSql("getdate()")]
public DateTime DateCreated { get; set; }
````

## SqlServer support

In order to support SqlServer features  add nuget package `EfCore.Shaman.SqlServer` and turn on necessary options in coniguration, i.e.

````csharp
migrationBuilder.FixMigrationUp<YourDbConextClass>(ShamanOptions.Default.WithSqlServer());
````
and

````csharp
this.FixOnModelCreating(modelBuilder, ShamanOptions.Default.WithSqlServer());
````
Collations specific to SqlServer is recognized by reflection scaner, i.e.

````csharp
[SqlServerCollation(KnownCollations.Polish100CiAi)]
public string Code { get; set; }
````

Moreover `SqlServerCollationAttribute` can be used for class annotation in order to set default collation for all text columns (not supported yet).

Current release allows to specify column collation while only table is created. 

## Code signing
Assembly distributed with nuget package is signed with `key.snk` that is not included with github repository. `mksnk.bat` script file is included instead. It it running automatically during building process. 


# .NET Core version

Due to [MsBuild issue](https://github.com/Microsoft/msbuild/issues/394) csproj and packages.config can't coexist with xproj and project.json. 

My own priority (sorry) is to maintain .Net framework version so files necessary for .NET Core compilation are provided in separate directory. 


## Plans

* Full column collation support
* .NetCore and .NetFramework release in the same time

## Known bugs

* Not supported descending field order in indexes

