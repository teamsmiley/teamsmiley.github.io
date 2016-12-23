---
layout: post
title: 'webapi version (namespace) 사용하기' 
author: teamsmiley 
date: 2016-11-19
tags: [webapi, apikey]
image: /files/covers/blog.jpg
category: {.net}
---

# web api 2 versioning with name space 

참고 - http://aspnet.codeplex.com/SourceControl/changeset/view/dd207952fa86#Samples/WebApi/NamespaceControllerSelector/ReadMe.txt


1. class를 NamespaceHttpControllerSelector  추가하자.
```cs 
using System;
using System.Collections.Generic;
using System.Globalization;
using System.Linq;
using System.Net;
using System.Net.Http;
using System.Web.Http;
using System.Web.Http.Controllers;
using System.Web.Http.Dispatcher;
using System.Web.Http.Routing;

namespace s3.api.App_Start
{
    public class NamespaceHttpControllerSelector : IHttpControllerSelector
    {
        private const string NamespaceKey = "namespace";
        private const string ControllerKey = "controller";

        private readonly HttpConfiguration _configuration;
        private readonly Lazy<Dictionary<string, HttpControllerDescriptor>> _controllers;
        private readonly HashSet<string> _duplicates;

        public NamespaceHttpControllerSelector(HttpConfiguration config)
        {
            _configuration = config;
            _duplicates = new HashSet<string>(StringComparer.OrdinalIgnoreCase);
            _controllers = new Lazy<Dictionary<string, HttpControllerDescriptor>>(InitializeControllerDictionary);
        }

        private Dictionary<string, HttpControllerDescriptor> InitializeControllerDictionary()
        {
            var dictionary = new Dictionary<string, HttpControllerDescriptor>(StringComparer.OrdinalIgnoreCase);

            // Create a lookup table where key is "namespace.controller". The value of "namespace" is the last
            // segment of the full namespace. For example:
            // MyApplication.Controllers.V1.ProductsController => "V1.Products"
            IAssembliesResolver assembliesResolver = _configuration.Services.GetAssembliesResolver();
            IHttpControllerTypeResolver controllersResolver = _configuration.Services.GetHttpControllerTypeResolver();

            ICollection<Type> controllerTypes = controllersResolver.GetControllerTypes(assembliesResolver);

            foreach (Type t in controllerTypes)
            {
                var segments = t.Namespace.Split(Type.Delimiter);

                // For the dictionary key, strip "Controller" from the end of the type name.
                // This matches the behavior of DefaultHttpControllerSelector.
                var controllerName = t.Name.Remove(t.Name.Length - DefaultHttpControllerSelector.ControllerSuffix.Length);

                var key = String.Format(CultureInfo.InvariantCulture, "{0}.{1}", segments[segments.Length - 1], controllerName);

                // Check for duplicate keys.
                if (dictionary.Keys.Contains(key))
                {
                    _duplicates.Add(key);
                }
                else
                {
                    dictionary[key] = new HttpControllerDescriptor(_configuration, t.Name, t);
                }
            }

            // Remove any duplicates from the dictionary, because these create ambiguous matches. 
            // For example, "Foo.V1.ProductsController" and "Bar.V1.ProductsController" both map to "v1.products".
            foreach (string s in _duplicates)
            {
                dictionary.Remove(s);
            }
            return dictionary;
        }

        // Get a value from the route data, if present.
        private static T GetRouteVariable<T>(IHttpRouteData routeData, string name)
        {
            object result = null;
            if (routeData.Values.TryGetValue(name, out result))
            {
                return (T)result;
            }
            return default(T);
        }

        public HttpControllerDescriptor SelectController(HttpRequestMessage request)
        {
            IHttpRouteData routeData = request.GetRouteData();
            if (routeData == null)
            {
                throw new HttpResponseException(HttpStatusCode.NotFound);
            }

            // Get the namespace and controller variables from the route data.
            string namespaceName = GetRouteVariable<string>(routeData, NamespaceKey);
            if (namespaceName == null)
            {
                throw new HttpResponseException(HttpStatusCode.NotFound);
            }

            string controllerName = GetRouteVariable<string>(routeData, ControllerKey);
            if (controllerName == null)
            {
                throw new HttpResponseException(HttpStatusCode.NotFound);
            }

            // Find a matching controller.
            string key = String.Format(CultureInfo.InvariantCulture, "{0}.{1}", namespaceName, controllerName);

            HttpControllerDescriptor controllerDescriptor;
            if (_controllers.Value.TryGetValue(key, out controllerDescriptor))
            {
                return controllerDescriptor;
            }
            else if (_duplicates.Contains(key))
            {
                throw new HttpResponseException(
                    request.CreateErrorResponse(HttpStatusCode.InternalServerError,
                    "Multiple controllers were found that match this request."));
            }
            else
            {
                throw new HttpResponseException(HttpStatusCode.NotFound);
            }
        }

        public IDictionary<string, HttpControllerDescriptor> GetControllerMapping()
        {
            return _controllers.Value;
        }
    }

```
2. 기존 라우터에 namespace를 추가하자. - WebApiConfig.cs

```cs

// Register default route
config.Routes.MapHttpRoute(
	name: "DefaultApi",
	routeTemplate: "api/{namespace}/{controller}/{id}",
	defaults: new { id = RouteParameter.Optional }
);
```

3. 서비스에 등록하자. - WebApiConfig.cs
```cs

config.Services.Replace(typeof(IHttpControllerSelector), new NamespaceHttpControllerSelector(config));

```


4. 이제 api를 두개 만들자. - Controllers/v1/

```cs
namespace MyApp.Controllers.V1
{
    // Version 1 controller
    public class ValuesController : ApiController {  }
}

namespace MyApp.Controllers.V2
{
    // Version 2 controller
    public class ValuesController : ApiController {  }
}
```

5. 테스트해보자. 

* http://localhost:59486/v1/values
* http://localhost:59486/v2/values

동작한다. 

6. 해결하지 못한 문제..

* swagger에서 이상하게 나온다.
* api 주소를 잘못치면 자꾸 비쥬얼 스튜디오가 멈춘다. 

 
