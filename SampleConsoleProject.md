# Introduction #

This page will guide you towards the creation of an internationalized console project for c-sharp using Gettext. The source code for this sample can be found [here](http://code.google.com/p/gettext-cs-utils/source/browse/#hg/Gettext.CsUtils/Samples/Gettext.Samples.Console).

# Creating the Project #

Create a new c-sharp console project, in this sample `Gettext.Samples.Console`, and add a reference to `Gettext.Cs.dll`, which is provided in the Core folder. You may also need to add a reference to System.Configuration.dll from the .NET framework.

## Include T4 Template ##

Next step is to create a [T4](http://msdn.microsoft.com/en-us/library/bb126445.aspx) template that will generate the static translation functions and contain the resource manager singleton. Note that the `Strings.T` function that will be used for translation is defined in the code file spawned by the template; when you save the template `tt` file, it writes a dependant `Strings.cs` file with the generated code.

![http://content.gettext-cs-utils.googlecode.com/hg/images/SampleConsoleProject/ProjectStructure1.png](http://content.gettext-cs-utils.googlecode.com/hg/images/SampleConsoleProject/ProjectStructure1.png)

There are several options that can be speficied for the template; by now, we will only specify the folder that will contain the resources, the resource and class name, and the project namespace.

```
<#@ assembly name="System.Configuration" #>

<#
this.ClassName = "Strings";
this.ResourceName = "Strings";
this.NamespaceName = "Gettext.Samples.Console";
this.DefaultResourceDir = "Resources"; 
#>

<#@ include file="..\..\Core\Gettext.Cs\Templates\Strings.tt" #>
```

Note that the beginning of the file specifies settings that will be used as parameters when the `Strings.tt` template is included in the last line. You may have to change that path so it reflects where you have copied the template, which you can find [here](http://code.google.com/p/gettext-cs-utils/source/browse/Gettext.CsUtils/Core/Gettext.Cs/Templates/Strings.tt).

## Write the program ##

Now that you have the translation function defined in your code, you can start using it for translation. Simply wrap a raw string inside the function, and it will be translated to the thread's Current UI Culture.

```
string translated = Strings.T("Hello world");
System.Console.Out.WriteLine(translated);
```

You may also want to change the Current UI Culture of the program so you can see how the string changes between different languages. In the console sample, we will set the current culture using the two-letter code specified as an argument when running the executable. Make sure the culture is set before translating the string!

```
var culture = CultureInfo.GetCultureInfo(args[0]);
System.Threading.Thread.CurrentThread.CurrentUICulture = culture;
```

## Extract strings for translation ##

As in every gettext project, after you have written the strings directly in your code and marked them for translation, you have to extract these strings into a `pot` template file, containing all untranslated messages. In order to do this, we will create an `Extract.bat` batch file that will make use of the [ExtractStrings.bat] file included in this solution.

This batch file requires certain environment variables to be set, specifically:
  * `path_xgettext` The path to the [xgettext binary](http://code.google.com/p/gettext-cs-utils/source/browse/Gettext.CsUtils/#Gettext.CsUtils/Bin/Gnu.Gettext.Win32)
  * `file_list` The list of files to be processed for strings extraction
  * `path_output` The folder where the `pot` file will be generated

The `extract.bat` batch file will look like this, depending on where you have placed the different files:

```
echo Setting up global variables...
SET path_xgettext=..\..\..\Bin\Gnu.Gettext.Win32\xgettext.exe
SET path_source=..\..\*.cs
SET path_output=..\Templates

echo Generating strings po file...
CALL ..\..\..\Scripts\ExtractStrings.bat Strings
```

After that, simply run the batch file, you will find a `Strings.pot` file in the output folder, in this case, Templates. This file contains all untranslated strings, and for this project will look like this:

```
#: E:\Gettext\cs-utils\Gettext.CsUtils\Samples\Gettext.Samples.Console\Program.cs:15
msgid "Hello world"
msgstr ""
```

## Translate strings ##

Next step is to give the `Strings.pot` file to the translators in order to obtain the different translated files. I personally recommend installing an instance of [Pootle](http://translate.sourceforge.net/wiki/pootle/index) for managing your translations, but any other tool or distribution method that suits you is just fine.

```
#: E:\Gettext\cs-utils\Gettext.CsUtils\Samples\Gettext.Samples.Console\Program.cs:15
msgid "Hello world"
msgstr "Hola mundo"
```

For every translated `Strings.po` file you have, simply add them to the project in a specific folder, each of them in a folder with the two-letter name of the culture. Since we specified `Resources` to be the lookup folder in the `Strings.tt` file, we simply paste them there.

![http://content.gettext-cs-utils.googlecode.com/hg/images/SampleConsoleProject/ProjectStructure2.png](http://content.gettext-cs-utils.googlecode.com/hg/images/SampleConsoleProject/ProjectStructure2.png)

Make sure every `po` file is set to be copied to the output folder, so the binary has access to it. When distributing your application, you will have to include the content of the Resources folder.

# Executing #

By now you should have a console project with a `Strings` class that provides translation functions, along with a `po` resource file for each of the different cultures you are planning to support. Compile and execute the project and see the different results!

![http://content.gettext-cs-utils.googlecode.com/hg/images/SampleConsoleProject/Execution.png](http://content.gettext-cs-utils.googlecode.com/hg/images/SampleConsoleProject/Execution.png)

Note from the german execution, that a non-found culture will eventually return the string in a neutral language (in this case, English). Also, any locale-specific cultures, if not found, will fall back to their neutral version appropriately.