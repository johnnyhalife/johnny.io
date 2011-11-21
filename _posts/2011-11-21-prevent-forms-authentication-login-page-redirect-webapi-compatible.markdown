---
title: Prevent Forms Authentication Login Page Redirect When You Don’t Want It (Web API Compatible)
layout: post
published: true
---
<p>About a month ago my friend <a href="http://twitter.com/haacked">Phill Haack</a> published a <a href="http://haacked.com/archive/2011/10/04/prevent-forms-authentication-login-page-redirect-when-you-donrsquot-want.aspx">post</a>
with almost the same purpose. Basically there's a bug on ASP.NET Forms Authentication that you cannot skip the Login Page redirect (out of the box) when
your response has a HTTP 401 for HttpStatusCode.</p>

<p><a href="http://twitter.com/haacked">Phill</a> focused his post (and NuGet <a href="https://github.com/Haacked/CodeHaacks">package</a>) on the Ajax Scenario. This means,
when you are performing a request using Javascript and you want the browser to get HTTP 401 instead of a HTTP 302 (Redirect) with Location header set to the
login page.</p>

<p>A couple of weeks ago while working on a project with <a href="http://www.wadewegner.com/">Wade Wegner</a>, while using Web API, we faced exactly the same problem:<br/>
we were being redirected and even though we tried Phill's fix, we couldn't make it work with Web API.</p>

<p>Why? Well, after chatting with <a href="http://twitter/gblock">Glenn Block</a> he gently explained to me that you shouldn't take for granted that your Service
(or Handler) will have access to the HttpContext or even the same Thread as it runs async, and that the best way to pass context information was
stuffing information into the message.</p>

<h2>Our solution</h2>

<p>Working with <a href="http://twitter.com/jpgd">Juan Pablo</a>, <a href="http://blogs.southworks.net/mconverti">Mariano Converti</a> and the rest of our <a href="http://southworks.net">Southworks</a> gang
we decided that we wanted to keep Phill's approach but also extend it, after Glenn's advice, so it can consider "marked responses" using our custom HttpHeader.</p>

<p>We could have gone the HttpResponseMessage.Items path, but as we want to keep it simple and decoupled from Web API so you can also use it with your own stuff.</p>

<p>We packaged the solution into a cool Nuget package that will do everything for you :)</p>

<pre style="prettyprint lang-cs">
PM> install package aspnet.suppressformsredirect
</pre>


<p>That will add the reference, and include the module on your web.config. Then on your application if you want to throw a 401 and you want to
suppress the forms redirect, then you just have to</p>

<pre style="prettyprint lang-cs">
// This will add the item that our suppression module looks for on the HttpContext.Items
HttpContext.Current.Items.Add(SuppressFormsAuthenticationRedirectModule.SuppressFormsAuthenticationKey, "true");
</pre>


<p>However, if you're using WebAPI it's not a safe to do that from within a DelegatingHandler or an OperationHandler. Instead you should add the header I
mentioned at the begging of this section. As follows</p>

<pre style="prettyprint lang-cs">
// We include the header on the HttpResponseMessage and the module takes care by itself
var response = new HttpResponseMessage(System.Net.HttpStatusCode.Unauthorized);
response.Headers.Add(SuppressFormsAuthenticationRedirectModule.SuppressFormsHeaderName, "true");

throw new HttpResponseException(response);
</pre>


<p>Of course, this is a temporary hack until the ASP.NET Framework Team gives us an elegant way to turn off redirect using &lt;location /&gt; or something similar. If you
have feedback or contribution you want to make to the package, you can easily go ahead and <a href="https://github.com/johnnyhalife/aspnet.suppressformsredirect">Fork it on GitHub</a>.</p>

<p>thanks,<br/>
~johnny</p>