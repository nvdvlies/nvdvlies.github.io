---
layout: post
title:  "Web Optimizations in ASP.NET MVC"
date:   2014-07-21 12:00:00
---

Modern web apps often use a lot of third party (javascript / stylesheet) libraries. Examples of popular libraries include Bootstrap, jQuery, Modernizr and Underscore. Having multiple of those libraries included in your web page might affect the speed of your website in a negative way. The Microsoft.AspNet.Web.Optimization namespace offers functionality to optimize loading speed for this scenario.

<!--more-->

### Bundling

It’s possible to bundle multiple javascript or stylesheet files together into one file. The reason this increases speed is because browsers use a maximum number of simultaneous connections when downloading a page. For most browsers this maximum is around 6 simultaneous connections. Meaning the 7th javascript or stylesheet file will be put on hold until one of the other files has finished downloading.

### Minification

Minification will remove all unnecessary characters (comments, whitespace, new lines) from a file and also renames variables to shorter single characters. This can dramatically decrease the total download size. For example, jQuery's original uncompressed size is 247 KB and can be minified to around 84 KB.

### How to implement in ASP.NET MVC Website

Below an example of a HTML page with a lot of stylesheets and scripts that should be bundled together. 

{% highlight html linenos %}
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width" />
    <title>@ViewBag.Title</title>
    
    <link rel="stylesheet" type="text/css" href="~/Content/bootstrap.css" />
    <link rel="stylesheet" type="text/css" href="~/Content/site.css" />

    <script type="text/javascript" src="~/Scripts/modernizr-2.6.2.js"></script>
</head>
<body>
    @RenderBody()

    <script type="text/javascript" src="~/Scripts/jquery-1.8.2.js"></script>
    <script type="text/javascript" src="~/Scripts/jquery-ui-1.8.24.js"></script>
    <script type="text/javascript" src="~/Scripts/jquery.unobtrusive-ajax.js"></script>
    <script type="text/javascript" src="~/Scripts/jquery.validate.js"></script>
    <script type="text/javascript" src="~/Scripts/jquery.validate.unobtrusive.js"></script>
    <script type="text/javascript" src="~/Scripts/underscore.js"></script>
    <script type="text/javascript" src="~/Scripts/bootstrap.js"></script>
</body>
</html>
{% endhighlight %}

#### 1. Install Web Optimization Framework
1. Right-click on the project in the Solution Explorer 
2. Choose "Manage NuGet packages".
3. Search and install `Microsoft.ASP.NET Web Optimization Framework`

In case you've created a project using the Internet application template this package has already been installed.

#### 2. Create BundleConfig class
1. Right-click on the App_Start folder in the Solution Explorer
2. In the menu choose Add | Class...
3. Name the new class `BundleConfig`
4. Add the code listed below. Within this class the bundles will be defined later.

{% highlight C# linenos %}
using System.Web;
using System.Web.Optimization;

namespace OptimizationExample {
    public class BundleConfig {
        public static void RegisterBundles(BundleCollection bundles) {

        }
    }
}
{% endhighlight %}

#### 3. Register bundles in Global.asax
1. Open the `Global.asax` file.
2. Import the `System.Web.Optimization` namespace
3. Add the line `BundleConfig.RegisterBundles(BundleTable.Bundles);` in the `Application_Start` method.

This basically makes sure the `RegisterBundles `method is executed on application startup. The `BundleTable` variable is a global variable to which the bundles need to be added.

#### 4. Add bundles to global BundleTable
Open `BundleConfig.cs` and add the bundles to the `BundleCollection` as shown in the code snippet below. For stylesheets you use a `StyleBundle` and for scripts a `ScriptBundle`. The virtual path parameter doesn't have to point to an existing directory and can be anything you like. I would advise to use the default `Content` and `Scripts` directories, because some libraries dynamically load other files at runtime and might be looking in the wrong directory if you use a non-existing path. For example, Bootstrap might not be able to load its Glyphicons correctly. You can use wildcards when including files. This way you don't have to include specific version numbers so your application won't break when you update a package. 

{% highlight C# linenos %}
using System.Web;
using System.Web.Optimization;

namespace OptimizationExample {
    public class BundleConfig {
        public static void RegisterBundles(BundleCollection bundles) {
            bundles.Add(new StyleBundle("~/Content/css").Include(
                        "~/Content/bootstrap.css",
                        "~/Content/site.css"));

            bundles.Add(new ScriptBundle("~/Scripts/modernizr").Include(
                        "~/Scripts/modernizr-*"));

            bundles.Add(new ScriptBundle("~/Scripts/js").Include(
                        "~/Scripts/jquery*",
                        "~/Scripts/underscore.js",
                        "~/Scripts/bootstrap.js"));
        }
    }
}
{% endhighlight %}

#### 5. Edit web.config
To be able to access these bundles within the Razor engine it's necessary to register the `System.Web.Optimization` namespace in the web.config file as shown below.

{% highlight xml linenos %}
<configuration>
  <!-- omitted for brevity -->
  <system.web>
    <pages>
      <namespaces>
        <add namespace="System.Web.Helpers" />
        <add namespace="System.Web.Mvc" />
        <add namespace="System.Web.Mvc.Ajax" />
        <add namespace="System.Web.Mvc.Html" />
        <add namespace="System.Web.Routing" />
        <add namespace="System.Web.Optimization" /> <!-- Add this line -->
      </namespaces>
    </pages>
    <!-- omitted for brevity -->
  </system.web>
</configuration>
{% endhighlight %}

#### 6. Edit HTML
The last step is to edit our HTML page and replace the javascript and stylesheet references with the bundles defined earlier. For stylesheets use `Styles.Render` and for javascript use `Scripts.Render` as show in the snippet below.

{% highlight html linenos %}
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width" />
    <title>@ViewBag.Title</title>
    @Styles.Render("~/Content/css")
    @Scripts.Render("~/Scripts/modernizr")
</head>
<body>
    @RenderBody()

    @Scripts.Render("~/Scripts/js")
</body>
</html>
{% endhighlight %}

When running the application and looking at the source, it should resemble much of the snippet below. Note that when in debug mode you will still get the original HTML source. You can enable optimizations in debug mode by adding the line `BundleTable.EnableOptimizations = true;` within the `BundleConfig` class. You might have noticed that a "v" parameter has been  appended to the bundle path automatically. This will ensure the browser uses the latest version after you've made changes to a script or stylesheet. When opening one of these URL's you will notice the content has been transformed to a minified version of the original.

{% highlight html linenos %}
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width" />
    <title>Title</title>
    
    <link rel="stylesheet" type="text/css" href="~/Content/css?v=rfgerth..." />

    <script type="text/javascript" src="~/Scripts/modernizr?v=Eeffe4..."></script>
</head>
<body>

    <script type="text/javascript" src="~/Scripts/js?v=rRERErt4..."></script>
</body>
</html>
{% endhighlight %}