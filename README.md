# Awesome ASP.NET Core API

> This repo was originally [a blog post](https://mazeez.dev/posts/asp-net-core-api-checklist).

Building modern APIs require a lot of things to make them reliable, observable, and scalable. In no particular order, here are some of them that help you build better APIs:

## 1. Healthchecks
Healthchecks are important in making sure that we know when anything happens to our APIs. We can setup dashboards to monitor them and setup alerting to let us know when one of the APIs is unhealthy. They are also important when deploying your apps to kubernetes. Kubernetes can monitor healthchecks of your APIs and automatically try to kill the unhealthy instances and create new instances to take their place.

There are two kinds of healthchecks:

  - **Liveliness:** indicates if your API has crashed and must be restarted.

  - **Readiness:** indicates if your API has been intialized and is ready for processing requests. When launching a new instance of an API, it might need some time to intialize dependencies and load data to be ready.

### More Information:
 - [MS Docs - Health checks in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/health-checks)
 - [IAmTimCorey - Intro to Health Checks in .NET Core](https://www.youtube.com/watch?v=Kbfto6Y2xdw)

## 2. Logging
Logging provides valuable information when trying to debug unexpected behavior. But too much logging can significantly slow down our APIs. For that reason we can set the logging level to `Warning` in production and only lower it when we need to. 

By default ASP.NET Core provides an abstraction layer for logging that supports [Structured Logging](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/logging/?view=aspnetcore-5.0#log-message-template).

A very popular logging library that many people use with ASP.NET Core is [Serilog](https://serilog.net/). Serilog has more [Sinks](https://github.com/serilog/serilog/wiki/Provided-Sinks) than the default ASP.NET Core loggging abstraction and can easily be integrated with ASP.NET Core.

### More information:
 - [MS Docs - Logging in .NET Core and ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/logging)
 - [IAmTimCorey - C# Logging with Serilog and Seq - Structured Logging Made Easy](https://www.youtube.com/watch?v=_iryZxv8Rxw)

## 3. Observability

This includes a few things:
  2. Performance monitoring (P99, P95, P50)
  3. Metrics: Specific counters you or your business cares about
  4. Tracing: Being able to see the entire lifecycle of each request from frontend to API to data source.

[OpenTelemetry](https://opentelemetry.io/) is an open standard for doing all of the above and [ASP.NET Core supports it](https://devblogs.microsoft.com/aspnet/observability-asp-net-core-apps/). The good news is, if you use OpenTelemetry, there is a rich ecosystem of tools and services that you can integrate with.

All of the major cloud providers have services that you can use to view the captured data.

## 4. Error reporting
There are tools specifically for capturing, storing and showing exceptions that have been raised in your APIs. They the exceptions by their type and location and show how many times they have occurred. Some tools include:

 - [Sentry](https://sentry.io)
 - [Rollbar](https://rollbar.com/)
 - [Raygun](https://raygun.com/)

## 5. Status Endpoint

It's also good to have a status endpoint in your API that shows the name of the API, version of the API and when was it started. This can be used to create a dashboard showing all of the different services and their versions. Something like this:

```csharp
using Microsoft.AspNetCore.Mvc;

using System;
using System.Diagnostics;

namespace MyCoolApi.Controllers
{
    public class StatusResponse
    {
        public string Name { get; set; }
        public string Version { get; set; }
        public DateTime StartTime { get; set; }
        public string Host { get; set; }
    }

    [ApiController]
    [Route("status")]
    public class StatusController
    {
        [HttpGet]
        public StatusResponse Get()
        {
            var version = typeof(Startup).Assembly.GetName().Version;

            return new StatusResponse
            {
                Name = "my-cool-api",
                Version = $"{version.Major}.{version.Minor}.{version.Build}",
                StartTime = Process.GetCurrentProcess().StartTime,
                Host = Environment.MachineName
            };
        }
    }
}
```

## 6. Http Resiliency

Although it's generally preferred for your APIs to communicate with other APIs using asynchronous messaging, sometimes you need to call other APIs using HTTP calls.

We need to bake in some level of resiliency by automatically retrying transient failures. This can easily be done by using something like [Polly](https://github.com/App-vNext/Polly).

### More information:

 - [Implement HTTP call retries with exponential backoff with IHttpClientFactory and Polly policies](https://docs.microsoft.com/en-us/dotnet/architecture/microservices/implement-resilient-applications/implement-http-call-retries-exponential-backoff-polly)
 - [Bryan Hogan - Fault Tolerant Web Service Requests with Polly](https://app.pluralsight.com/library/courses/polly-fault-tolerant-web-service-requests)

## 7. Statelessness and Containerization

Containers are a great way to make sure your APIs can be easily scaled out and deployed to multiple environments in a repeatable manner. However to make sure you can get the most out of containerization you should try to make sure your APIs are stateless. 

Being stateless means that they don't hold any critical data in memory. For caching you can use a centralized caching technology like [Redis](https://redis.io/) instead. This way you can start as many instances as you need without worrying about having stale cached data or data duplication. 

You must also be careful about background jobs. You must make sure the different instances don't process background jobs multiple times. And for message queues, you have to implement the [Competing Consumers](https://docs.microsoft.com/en-us/azure/architecture/patterns/competing-consumers) pattern which some message buses support natively.

### More information:

- [MS Docs - Tutorial: Containerize a .NET Core app](https://docs.microsoft.com/en-us/dotnet/core/docker/build-container)
- [Hangfire](https://www.hangfire.io/)
- [Quartz.NET](https://www.quartz-scheduler.net/)
- [MS Docs - Competing Consumers](https://docs.microsoft.com/en-us/azure/architecture/patterns/competing-consumers)

## 9. OpenAPI Spec / Swagger

Documenting your APIs is very important. Swagger integrates with ASP.NET Core and automatically finds all of the routes (Controller actions) and shows them in a beautiful dashboard that you can customize.

### More information:
 - [MS Docs - ASP.NET Core web API documentation with Swagger / OpenAPI]((https://docs.microsoft.com/en-us/aspnet/core/tutorials/web-api-help-pages-using-swagger?view=aspnetcore-5.0))
 - [Kevin Dockx - Documenting an ASP.NET Core API with OpenAPI / Swagger](https://www.pluralsight.com/courses/aspdotnet-core-api-openapi-swagger)

## 10. Configuration and Options

ASP.NET Core has an extensible configuration mechanism. It can pull configurations from json files, environment variables and command line arguments. You can also provide custom sources for configuration. It also provide ways to easily fetch the configurations in a type-safe manner. it also provides an easy mechanism to [validate the configuration sections](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/options?view=aspnetcore-5.0#options-validation).

### More information:

- [MS Docs - Options Pattern in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/options)
- [Steve Gordon - Using Configuration and Options in .NET Core and ASP.NET Core Apps](https://app.pluralsight.com/library/courses/dotnet-core-aspnet-core-configuration-options/)

## 11. Integration and Unit Tests

ASP.NET Core has made it easy to write Unit tests by making the whole framework DI friendly. It has also made Integration tests easy by [`WebApplicationFactory`](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1) Having automated tests saves a lot of time and makes your APIs more robust. And when writing integration tests, try to use the same database technology that you use for production. If you're using Postgres in production, don't use Sqlite or In-Memory DB Providers for integration tests.

### More information:

- [MS Docs - Integration tests in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/test/integration-tests)
- [Steve Gordon - Integration Testing ASP.NET Core Applications: Best Practices](https://app.pluralsight.com/library/courses/integration-testing-asp-dot-net-core-applications-best-practices/table-of-contents)

## 12. Build beautiful REST APIs

If you're building REST APIs, there are some conventions that make your APIs more pleasant and intuitive to use.

### More information:

- [Stackoverflow Blog - Best practices for REST API design](https://stackoverflow.blog/2020/03/02/best-practices-for-rest-api-design/)
- [Martin Fowler - Richardson Maturity Model](https://martinfowler.com/articles/richardsonMaturityModel.html)

## 13. Authentication and Authorization

Authentication is the process of identifying a user and authorization is knowing and enforcing what each user can and can't do.  The most popular standards for authentication is OpenIDConnect which is an authentication layer on top of OAuth 2.

There are some popular Identity Providers that you can easily integrate with your API:

- [Auth0](https://auth0.com/)
- [Okta](https://www.okta.com/)
- [FusionAuth](https://fusionauth.io/)

And there are some open source Identity and Access Management servers that you can run on-prem:

- [Keycloak](https://www.keycloak.org/)
- [Gluu](https://gluu.org/)

And there are some libraries that you can use to build your own OIDC server:

- [IdentityServer](https://duendesoftware.com/)
- [OpenIddict](https://github.com/openiddict/openiddict-core)

### More information

- [Kevin Dockx - Securing ASP.NET Core 3 with OAuth2 and OpenID Connect](https://app.pluralsight.com/library/courses/securing-aspnet-core-3-oauth2-openid-connect)
- [Kevin Dockx - Securing Microservices in ASP.NET Core](https://app.pluralsight.com/library/courses/securing-microservices-asp-dot-net-core)

## 14. Security

### 14.1 CORS

Cross-Origin Resource Sharing allows frontends to call your API even if they are not on the same domain as the API. By default in ASP.NET Core CORS is disabled.

#### More information

- [MS Docs - Enable Cross-Origin Requests (CORS) in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/security/cors)

### 14.2 HTTPS Enforcing

For this there are two scenarios:

- You're using Kestrel on edge: Then you have to make sure it's only listening to and respond over HTTPS.
- You've put ASP.NET Core behind a reverse proxy: Then you generally terminate HTTPS on the proxy and it's the proxy's job to enforce HTTPS.

#### More Information

- [MS Docs - Enforce HTTPS in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/security/enforcing-ssl)

### 14.3 Cross-Site Scripting (XSS)

Cross-Site Scripting (XSS) is a security vulnerability which enables an attacker to place client side scripts (usually JavaScript) into web pagesMore Information. You can prevent it by sanitizing inputs from the user.

- [Prevent Cross-Site Scripting (XSS) in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/security/cross-site-scripting)

## 15. API Versioning

Versioning your APIs allow you to maintain backward compatibility when making breaking changes. You can maintaing multiple versions at the same time and then deprecate versions over time.

### More Information

- [Overview of API Versioning in ASP.NET Core 3.0+](https://exceptionnotfound.net/overview-of-api-versioning-in-asp-net-core-3-0/)

