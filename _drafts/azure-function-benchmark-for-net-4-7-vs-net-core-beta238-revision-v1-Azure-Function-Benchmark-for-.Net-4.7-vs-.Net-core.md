---
id: 247
title: Azure Function Benchmark for .Net 4.7 vs .Net core
date: 2017-10-24T20:33:55+02:00
author: Janusz Nowak
layout: revision
guid: http://blog.janono.pl/2017/10/238-revision-v1/
permalink: /2017/10/238-revision-v1/
---
There was some friend benchmark about running application on app service plan vs app service plan for docker.

I have some other approach to run the same code sample but on azure function running .net 4.7 vs azure function running .net core

The size of machine for app service plan is S1, testing approach is ab benchmark tool running under windows hosted in azure, do not hit network limits or CPU the machine size is DS5_V2

<img class="size-full wp-image-239 alignleft" src="/wp-content/uploads/2017/10/s1.png" alt="" width="183" height="327" srcset="/wp-content/uploads/2017/10/s1.png 183w, /wp-content/uploads/2017/10/s1-168x300.png 168w" sizes="(max-width: 183px) 100vw, 183px" /><img class="size-full wp-image-240 alignleft" src="/wp-content/uploads/2017/10/DS5_V2.jpg" alt="" width="188" height="327" srcset="/wp-content/uploads/2017/10/DS5_V2.jpg 188w, /wp-content/uploads/2017/10/DS5_V2-172x300.jpg 172w" sizes="(max-width: 188px) 100vw, 188px" /> 

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

The setup is pre compiled� Azure Function running .Net 4.7

<img class="alignnone size-full wp-image-246" src="/wp-content/uploads/2017/10/fun-runtime.jpg" alt="" width="205" height="72" /> 

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">public static class Function1
{        
    private const int MaxPage = 101;

    private static Dictionary&lt;int, string&gt; Values { get; set; } = new Dictionary&lt;int, string&gt;();

    private static string GetBodyOfSize(int i)
    {
        if (Values.ContainsKey(i))
            return Values[i];

        var s = new String('X', i);
        //Values.TryAdd(i, s);
        Values.Add(i, s);
        return s;
    }

    [FunctionName("Function1")]
    public static async Task&lt;HttpResponseMessage&gt; Run([HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)]HttpRequestMessage req, TraceWriter log)
    {
        var s = req.GetQueryNameValuePairs()
                .FirstOrDefault(q =&gt; string.Compare(q.Key, "s", true) == 0)
                .Value;

        if (string.IsNullOrEmpty(s))
        {
            // return simple hello, world
            var now = DateTime.UtcNow;
            return req.CreateResponse(HttpStatusCode.OK, $"Hello World, from ASP.NET Fun App Net 4.7! {now.ToString("yyyy-MM-dd HH:mm:ss.FFF")}");
        }

        if (int.TryParse(s, out int i))
        {
            if (i &gt; 0 && i &lt; MaxPage)
            {
                var body = GetBodyOfSize(i * 1000);
                return req.CreateResponse(HttpStatusCode.OK, body);
            }
            else
            {
                return req.CreateResponse(HttpStatusCode.BadRequest, $"Size must be an integer between 1 and {MaxPage}");
            }
        }
        else
        {
            return req.CreateResponse(HttpStatusCode.BadRequest, $"Size must be an integer between 1 and {MaxPage}");
        }
    }
}</pre>

vs Azure Function .Net Core having kestrel under hood in beta version

<img class="alignnone size-full wp-image-243" src="/wp-content/uploads/2017/10/functionbeta.png" alt="" width="235" height="89" /> 

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">#r "Newtonsoft.Json"

using System.Net;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Primitives;
using Newtonsoft.Json;


   private const int MaxPage = 101;

        private static Dictionary&lt;int, string&gt; Values { get; set; } = new Dictionary&lt;int, string&gt;();

        private static string GetBodyOfSize(int i)
        {
            if (Values.ContainsKey(i))
                return Values[i];

            var s = new String('X', i);
            Values.TryAdd(i, s);
            //Values.Add(i, s);
            return s;
        }

        public static IActionResult Run(HttpRequest req, TraceWriter log)
        {
             var s = req.Query["s"];
          if (string.IsNullOrEmpty(s))
            {                
                var now = DateTime.UtcNow;
                return  (ActionResult)new OkObjectResult($"Hello World, from ASP.NET Core and Net Core 2.0! {now.ToString("yyyy-MM-dd HH:mm:ss.FFF")}");
            }

            if (int.TryParse(s, out int i))
            {
                if (i &gt; 0 && i &lt; MaxPage)
                {
                    var body = GetBodyOfSize(i * 1000);
                    return  (ActionResult)new OkObjectResult( body);
                }
                else
                {
                    return  (ActionResult)new OkObjectResult( $"Size must be an integer between 1 and {MaxPage}");
                }
            }
            else
            {
                 return  (ActionResult)new OkObjectResult( $"Size must be an integer between 1 and {MaxPage}");
            }
}
</pre>

&nbsp;

&nbsp;