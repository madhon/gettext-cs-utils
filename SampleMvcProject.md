# Introduction #

This page will guide you towards the creation of an internationalized ASP NET MVC site using Gettext, storing resources in plain `po` files in the site folder. The source code for this sample can be found [here](http://code.google.com/p/gettext-cs-utils/source/browse/#hg/Gettext.CsUtils/Samples/Gettext.Samples.Mvc).

This tutorial assumes that you already know [how to create a simple console project](SampleConsoleProject.md) using these tools, and that you are familiar with ASP NET MVC framework.

# Creating the Project #

First of all, create an ASP NET MVC project in the solution. In our case, we will name it `Gettext.Samples.Mvc`. As with previous examples, the first step is to include the T4 template that adds the `Strings.T` translation function.

## T4 Template ##

Include the `Strings.tt` template as usual, and add the value `this.ServerMapPath = true` to force the ASP NET application to map the logical path to the physical path to the resources in local disk. To specify the path to the resources, instead of relying on `this.ResourceDir`, this time we will specify that value via web config. Simply add an app setting with the value you want, in the case, `~/bin/Gettext/Po`:
```
<appSettings>
	<add key="ResourcesDir" value="~/bin/Gettext/Po/"/>
</appSettings>
```

Also, create a `Gettext/Po` folder in your application root that will be where you will copy the translated `po` files when ready.

![http://content.gettext-cs-utils.googlecode.com/hg/images/SampleMvcProject/ProjectStructure.png](http://content.gettext-cs-utils.googlecode.com/hg/images/SampleMvcProject/ProjectStructure.png)

## Add Translation ASP NET Control ##

The usual way for translating strings in web pages is to use the `<%= %>` syntax to introduce code, and invoke the translation function there. For example, should we want to output an internationalized _Hello world_ string in the home page, we may write:
```
<p><%= Strings.T("Hello world") %></p>
```

Remember to add the namespace where the `Strings` class is defined to the web config, in the pages/namespaces section, so you can use the `Strings` class without its fully qualified name. Another sample that makes use of the `Strings` class interpolation would be the following:

<%= Strings.T("Go to the {0} page.", Html.ActionLink(Strings.T("home"), "Index", "Home")) %>

This line outputs an action link to the `Home` page, with internationalized text, creating the link only in the word _home_. The translator will have to deal with two separate strings in this case: _Go to the {0} page_ and _home_.

Although this way of writing internationalized strings in web pages works, for simple strings it might be a little to cumbersome. An option is to create an ASP NET control that runs at server and internationalizes its content. The base for this control is provided in the `Gettext.Cs.Web` core project. To make use of it, create an `AspTranslateControl` class in your application that inherits from the base control, and fill the abstract `Translate` function, like this:

```
namespace Gettext.Samples.Mvc.Controls
{
    public class t : Gettext.Cs.Web.AspTranslate
    {
        protected override string Translate(string text)
        {
            return Strings.T(text);
        }
    }
}
```

Make sure you also add the `Gettext.Samples.Mvc.Controls` to the web config, in this case to the pages/controls section, and using a short and convenient prefix, such as `t`:

```
<controls>
	<add tagPrefix="asp" namespace="System.Web.UI" assembly="System.Web.Extensions, Version=3.5.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35"/>
	<add tagPrefix="asp" namespace="System.Web.UI.WebControls" assembly="System.Web.Extensions, Version=3.5.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35"/>
	<add tagPrefix="t" namespace="Gettext.Samples.Mvc.Controls" assembly="Gettext.Samples.Mvc"/>
</controls>
```

Now if you want to translate a simple string in an ASP NET page, you only have to wrap it in a `t:t` control, including the `runat=server` attribute so it is processed and translated server-side:

```
<t:t runat=server>Hello world</t:t>
```

Of course, the `Strings.T` function is still available for use in views and in any other part of the application, such as controllers, models, services, etc.

## Choosing culture ##

There are multiple ways of allowing the user to specify which culture he or she perfers. A usual way is to present a combo box or list of languages, and when the user selects one of them, store that information in a cookie or server-side. Keeping language information in the URL itself is also a good practice when SEO is important.

For the sake of this example we will use the most simple one: use the browser's default. To instruct ASP NET MVC to use as UICulture the browser's language, add the following node to the `system.web` in the web config:
```
<globalization uiCulture="auto"/>
```

This will use the Accept-Language headers sent to the server by the browser and apply them to the current thread. So in order to test the site in different languages, you will have to change your browser's languages. In Firefox, for example, just go to Tools, Options, and choose to change language:

![http://content.gettext-cs-utils.googlecode.com/hg/images/SampleMvcProject/ChangeLang.png](http://content.gettext-cs-utils.googlecode.com/hg/images/SampleMvcProject/ChangeLang.png)

# Extracting strings #

The GNU gettext strings extractor, `xgettext`, supports a number of languages, but does not handle `<t:t>` tags in ASP NET pages. It may also ocassionally presents some issues when attempting to parse `Strings.T` c-sharp functions inside an ASP NET page.

The `ExtractAspNetStrings` batch file takes care of these cases, by invoking the [AspExtract](http://code.google.com/p/gettext-cs-utils/source/browse/#hg/Gettext.CsUtils/Tools/Gettext.AspExtract) tool in the solution, by creating intermediate files from all `aspx`, `asxc` and `Master` files that can be understood by `xgettext`.

This batch requires a number of environment variables to be set before being invoked, see the [example file](http://code.google.com/p/gettext-cs-utils/source/browse/#hg/Gettext.CsUtils/Samples/Gettext.Samples.Mvc/Gettext/Extract.bat) for more details:
  * **path\_xgettext** Path to the xgettext executable
  * **path\_aspextract** Path to the aspextract tool executable included in this solution's tools
  * **path\_output** Output path for the template file
  * **file\_list** List of files to be processed by xgettext
  * **asp\_files\_root** Where to obtain all ASP NET files to be processed

When running this batch on the sample project, the result is the following `Strings.pot` gettext template, with all the strings in both the code and the ASP NET pages.
```
#: E:\Gettext\cs-utils\Gettext.CsUtils\Samples\Gettext.Samples.Mvc\Controllers\HomeController.cs:14
msgid "Welcome to internationalized ASP.NET MVC!"
msgstr ""

#: E:\Gettext\cs-utils\Gettext.CsUtils\Samples\Gettext.Samples.Mvc\Views\Home\Index.aspx.postrings:1
msgid "Hello world"
msgstr ""

#: E:\Gettext\cs-utils\Gettext.CsUtils\Samples\Gettext.Samples.Mvc\Views\Home\Index.aspx.postrings:2
#, csharp-format
msgid "Go to the {0} page."
msgstr ""

#: E:\Gettext\cs-utils\Gettext.CsUtils\Samples\Gettext.Samples.Mvc\Views\Home\Index.aspx.postrings:3
msgid "home"
msgstr ""
```

Note that strings from ASP NET pages have an additional `postrings` extension, these are temp files generated with a syntax easily understandable by `xgettext` for procession. The downside of this solution is that line numbers are not preserved, this is a pending issue in the tool.

# Executing the Project #

Copy the `po` files for the different cultures in the Gettext/Po folder you created earlier. Make sure to specify that these files are copied to the output directory, so it can be properly accessed. Now run the project, set your browser's language to spanish, and check the results:

![http://content.gettext-cs-utils.googlecode.com/hg/images/SampleMvcProject/Running.png](http://content.gettext-cs-utils.googlecode.com/hg/images/SampleMvcProject/Running.png)