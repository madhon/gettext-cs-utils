# Introduction #

This page will guide you towards the creation of an internationalized console project for c-sharp using Gettext, using a SQL database to store the translated strings instead of po files. The source code for this sample can be found [here](http://code.google.com/p/gettext-cs-utils/source/browse/#hg/Gettext.CsUtils/Samples/Gettext.Samples.Database).

This tutorial assumes that you already know [how to create a simple console project](SampleConsoleProject.md) using these tools.

# T4 Template for Database #

To instruct the template to instantiate a database resource manager to fetch resource sets from a database, simply add a `UseDatabase` flag to the template parameters and specify the name of the stored procedure to use to obtain resource sets (we will create this SP later).

```
<#@ assembly name="System.Configuration" #>

<#
this.ClassName = "Strings";
this.ResourceName = "Strings";
this.NamespaceName = "Gettext.Samples.Database";
this.UseDatabase = true;
this.StoredProcedureName = "GettextGetResourceSet";
#>

<#@ include file="..\..\Core\Gettext.Cs\Templates\Strings.tt" #>
```

The `StoredProcedureName` value specifies which stored procedure should be invoked to obtain a full resource set given the culture. This is, given a culture code, obtain all key/value records for that culture.

You must also add a `Gettext` connection string in the application configuration file to instruct the application how to create the connection that will be used for obtaining the resource sets.

```
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <connectionStrings>
    <add name="Gettext" connectionString="Data Source=.\SQLEXPRESS; Initial Catalog=GettextSample; Integrated Security=SSPI;"/>
  </connectionStrings>
</configuration>
```

These are the only changes required on the application itself. The major changes now are how to handle the `po` files and inserting them into the database.

# Database Resource Generator #

One of the tools included in this package is the [Database Resource Generator](http://code.google.com/p/gettext-cs-utils/source/browse/Gettext.CsUtils/#Gettext.CsUtils/Tools/Gettext.DatabaseResourceGenerator). See [this page](DatabaseResourceGenerator.md) for more details on how to configure it appropriately before you continue reading.

## Preparing the Database ##

After creating an empty GettextSample database, and configuring the application settings and connection string for the generator, you can use it to set up all necessary artifacts. Invoke the generator with the `-p` switch so it sets up both the strings table and all the required stored procedures.

![http://content.gettext-cs-utils.googlecode.com/hg/images/SampleDatabaseProject/ResourceGeneratorPrepare.png](http://content.gettext-cs-utils.googlecode.com/hg/images/SampleDatabaseProject/ResourceGeneratorPrepare.png)

Your database should now have the `Strings` table, and `GettextGetResourceSet`, `GettextInsertResource` and `GettextDeleteResourceSet` stored procedures.

![http://content.gettext-cs-utils.googlecode.com/hg/images/SampleDatabaseProject/Database.png](http://content.gettext-cs-utils.googlecode.com/hg/images/SampleDatabaseProject/Database.png)

## Dumping po files ##

When the translators have the `po` files ready, it is time to insert them into the database. Still using `po` files for the translations allows you  to keep a Pootle instance for managing your translators, regardless of whether the application itself uses raw `po` files or a database as a back end for the resource sets.

Depending on where you will be keeping the `po` translated files, it is a good practice to create a batch to dump all translations files to the database by invoking the database resource generator.

```
SET dbrsgen=..\..\..\Tools\Gettext.DatabaseResourceGenerator\bin\Debug\DatabaseResourceGenerator.exe

echo Dumping culture sets into DB...
CALL %dbrsgen% -i ..\Translated\es\Strings.po -c es -a
CALL %dbrsgen% -i ..\Translated\en\Strings.po -c en -a
CALL %dbrsgen% -i ..\Translated\pt\Strings.po -c pt -a
CALL %dbrsgen% -i ..\Translated\fr\Strings.po -c fr -a
pause
```

You should see the following records now in your `Strings` table in the database; they will be used for translation when the project is executed.

| **culture** | **key** | **value** |
|:------------|:--------|:----------|
| en          | Hello world |           |
| es          | Hello world | Hola mundo |
| fr          | Hello world | Bonjour tout le monde |
| pt          | Hello world | Ol� mundo |


# Execute the project #

Now you have both configured your project to use a database as a repository for translated resource sets, and have dumped all translated strings into the database. You are ready to run the project and check the results!