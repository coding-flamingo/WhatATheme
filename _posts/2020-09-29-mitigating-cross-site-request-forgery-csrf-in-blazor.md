---
title: Mitigating Cross-Site Request Forgery (CSRF) in Blazor WASM
layout: post
post-image: "/assets/images/csrfdiagram.jpg"
description: In this post we take a look at what is Cross-Site Request Forgery (CSRF)
  and how to mitigate it in Blazor WASM.
tags:
- Blazor
- Cross-Site Request Forgery
- CSRF
- Security
- Tutorial
---

In this post we are going to look at how to mitigate Cross-Site Request Forgery in your blazor site. 
## What is CSRF?
Cross-Site Request Forgery is a type of exploit of a website where unauthrized commands are submitted from a user that the web application trusts. Which means that a bad actor phises or somehow gets a user to go to thier site where the site responds with a script that tells the browser to use the save credential to your site and send a request to your site that the bad actor needs. 

In the diagram below you can see this scenario playing out with the bad actor stealing money from the user without the user knowing.

![CSRF Diagram](/assets/images/csrfdiagram.jpg)

To mitigate this, on pages were you do an action that affects the server status (for example a post or a delete), the server should send a cookie if you are in that page when you send the request you attached that cookie back so the server knows the request is coming from your site and not from a bad site. 

## Video Version
coming soon :)

## Text Version
### Prerequisites
- Have a Blazor Site with modern Authentication (This tutorial was tested with Azure AD)

### Tutorial
#### Setting it up Server Side
In Startup.cs we are going to first add at the top the following dependencies:
```
using Microsoft.AspNetCore.Antiforgery;
using Microsoft.AspNetCore.Http;
```
then under your `public void ConfigureServices(IServiceCollection services)` we are going to add the following service:
```
services.AddAntiforgery(options =>
{
     options.HeaderName = "X-CSRF-TOKEN";
});
```
then to the Configure method (usually `public void Configure(IApplicationBuilder app, IWebHostEnvironment env)` we are gping to add an antiforgery input so it looks like this: `public void Configure(IApplicationBuilder app, IWebHostEnvironment env, IAntiforgery antiforgery)`
and inside of it we are going to add:
```
 app.Use(next => context =>
 {
     var tokens = antiforgery.GetAndStoreTokens(context);
     context.Response.Cookies.Append("XSRF-TOKEN", tokens.RequestToken, new CookieOptions() { HttpOnly = false });
     return next(context);
 });
```
That concludes the startup.cs code.
To protect your controllers you can add this `[ValidateAntiForgeryToken]` or `[AutoValidateAntiforgeryToken]` (the only difference is that `[AutoValidateAntiforgeryToken]` will only validate the token in http methods that ar at risk aka exclude: GET, HEAD, OPTIONS and TRACE) to each route you want to protect. For Example:
```
    [AutoValidateAntiforgeryToken]
    [HttpPost]
    public IEnumerable<WeatherForecast> Post()
     {
          var rng = new Random();
          return Enumerable.Range(1, 5).Select(index => new WeatherForecast
          {
               Date = DateTime.Now.AddDays(index),
               TemperatureC = rng.Next(-20, 55),
               Summary = Summaries[rng.Next(Summaries.Length)]
          }) .ToArray();
     }
```
Or you can put it at the top of your controller:
```
[ApiController]
[Route("[controller]")]
[Authorize]
[AutoValidateAntiforgeryToken]
public class WeatherForecastController : ControllerBase
{
	//controller code
}
```
#### Setting up Client Side
First we are going to add a `getcookie.js` fie inside the wwwroot folder with the following code:
```
function getCookie(cname) {
    var decodedCookie = decodeURIComponent(document.cookie);
    var ca = decodedCookie.split(';');
    for (var i = 0; i < ca.length; i++) {
        var arr = ca[i].split('=');
        if (arr[0] == cname)
            return arr[1]
    }
    return "";
}
```
and add the refference to that script in the `index.html` file insde the body tag:
```
 <script src="getcookie.js"></script>
```
Then in each page you are using it you can get it by
1. Injecting the IJSRuntime `@inject IJSRuntime _jSRuntime` for razor page or `[Inject] IJSRuntime _jSRuntime { get; set; }` if you are breaking up your controller code into a Component Base with all your C# code. 
2.  In the function where you call the backend you get the csrfcookie by calling: `string csrfCookieValue = await _jSRuntime.InvokeAsync<string>("getCookie", "XSRF-TOKEN");`
3. add it to your HTTPRequestMessage headers and post with the following code (where `_httpClient` is the name of the HttpClient you inject to your page):
```
HttpRequestMessage requestMessage = new HttpRequestMessage(HttpMethod.Post, url);
HttpResponseMessage response;
if (!string.IsNullOrWhiteSpace(csrfCookieValue))
{
      requestMessage.Headers.Add("X-CSRF-TOKEN", csrfCookieValue);
}
response = await _httpClient.SendAsync(requestMessage);
```

4. repeat for all your requests, and now we are secure :)