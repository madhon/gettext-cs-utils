# Database resource generator #

One of the tools included in this package is the [DatabaseResourceGenerator](http://code.google.com/p/gettext-cs-utils/source/browse/Gettext.CsUtils/#Gettext.CsUtils/Tools/Gettext.DatabaseResourceGenerator). This tool will allow you to dump all `po` files into the database, and set up all necessary tables and stored procedures for it to work.

The resource generator requires simply three stored procedures, for get, insert and delete resource sets. If not present in the database, the resource generator will create them using the supplied values in the configuration file, as well as the table containing all localized resources.

## Configuration ##

The following is a sample app.config file for the DatabaseResourceGenerator, containing both the connection string used for connecting to the database and all necessary app settings for creating all required artifacts.

```
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <connectionStrings>
    <add name="Gettext" connectionString="Data Source=.\SQLEXPRESS; Initial Catalog=GettextSample; Integrated Security=SSPI;"/>
  </connectionStrings>
  <appSettings>
    <add key="SP.Get" value="GettextGetResourceSet"/>
    <add key="SP.Insert" value="GettextInsertResource"/>
    <add key="SP.Delete" value="GettextDeleteResourceSet"/>
    <add key="Table.Name" value="Strings"/>
    <add key="Table.Fields.Culture" value="culture"/>
    <add key="Table.Fields.Key" value="key"/>
    <add key="Table.Fields.Value" value="value"/>
  </appSettings>
</configuration>
```

The table name and field names are used when the table is not found and is created by the generator. The definition will be used when generating the stored procedures as well, and all parameter names must be equal to the field names.

The `SP.Get`, `SP.Insert` and `SP.Delete` settings specify the name of the stored procedures for getting, inserting and deleting resource sets in the database. Unless invoked with `-d` option, the resource generator will check that these three procedures exist, and will create them if not. It is recommended that you let the generator create them, but you may create them yourself and simply indicate the generator their names for using them. The procedures must conform to the following contracts.

### Get Resource Set ###

Given a culture code parameter, named as the `Table.Fields.Culture` setting, returns the key and value columns for every record in the internationalization table for the specified culture.
```
CREATE PROCEDURE [dbo].[GettextGetResourceSet]	
	@culture VARCHAR(5)
	AS
	BEGIN
		SELECT [key], [value] FROM [Strings]	
		WHERE [culture] = @culture
END
```

### Insert Resource Set ###

Given a culture code, key and value (the parameter names are inferred from the `Table.Fields` settings), insert a new record in the resources table, to be returned later by the get resource set stored procedure.
```
CREATE PROCEDURE [dbo].[GettextInsertResource]	
	@culture VARCHAR(5),
	@key VARCHAR(4000),
	@value VARCHAR(4000)
AS
BEGIN
	INSERT INTO [Strings] ([culture], [key], [value])
	VALUES (@culture, @key, @value)
END
```

### Delete Resource Set ###

Given a culture code, deletes all resources associated to it. This procedure is useful when updating a certain resource set from an updated `po` file, as older contents are purged and the new translations are inserted. This is the standard behaviour unless the `-n` switch is used when invoking the generator.
```
CREATE PROCEDURE [dbo].[GettextDeleteResourceSet]	
	@culture VARCHAR(5)
	AS
	BEGIN
		DELETE FROM Strings	
		WHERE culture = @culture
END
```

## Usage ##

Invoke the generator with the `-?` switch for detailed options on how to invoke it.

```
DatabaseResourceGenerator.exe -?

Gettext Cs Tools
----------------

Parses a .po file and adds its entries to a database's table.
Then you can use a DatabaseResourceManager to use those resources at runtime.
Usage: DatabaseResourceGenerator -iINPUTFILE -cCULTURE -sCONNSTRING
 INPUTFILE Input file must be in po format (optional if -p).
 CULTURE Culture for the resource set to handle (optional if -p).
 CONNSTRING Connection string to override app config (optional).
Options:
 -p Only setup table and stored procedures, do not parse po file.
 -n Dont delete previous resource set.
 -a Insert all values, regardless of being empty.
 -d Skip table and stored procedures validation and creation.
```