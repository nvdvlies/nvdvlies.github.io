---
layout: post
title:  "Require HTTPS in Asp.Net Web API"
date:   2014-07-25 12:00:00
---

When creating a Web API which requires authorization, most authorization methods aren't save without an SSL connection. This includes basic authentication, but single sign on methods like OAuth and SAML also rely on SSL since they don't include encryption. You can enforce the use of SSL from within your web server, but you might be extra save to also enforce this on application level. Especially when you're not in control of the web server configuration yourself.

<!--more-->

### Create a MessageHandler
1. Right-click on the Web API project in the Solution Explorer
2. In the menu choose Add | Class...
3. Name the new class `RequireHttpsHandler`
4. Add the code listed below. 

{% highlight c# linenos %}
public class RequireHttpsHandler : DelegatingHandler {

    protected override Task<HttpResponseMessage> SendAsync(HttpRequestMessage request, CancellationToken cancellationToken) {

        if (request.RequestUri.Scheme != Uri.UriSchemeHttps) {
            return Task<HttpResponseMessage>.Factory.StartNew(() =>
                {
                    var response = new HttpResponseMessage(HttpStatusCode.Forbidden)
                    {
                        Content = new StringContent("HTTPS Required")
                    };

                    return response;
                });
        }

        return base.SendAsync(request, cancellationToken);
    }
}
{% endhighlight %}

### Register MessageHandler
1. Open class `WebApiConfig` from the `App_Start` directory.
2. Add line `config.MessageHandlers.Add(new RequireHttpsHandler());` to the `Register` method.

{% highlight c# linenos %}
namespace Example.Web.App_Start {

    public static class WebApiConfig {
        
        public static void Register(HttpConfiguration config) {
            //...

            config.MessageHandlers.Add(new RequireHttpsHandler());
        }
    }
}
{% endhighlight %}