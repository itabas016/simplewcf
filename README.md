# Simple WCF
Many of the WCF services that we build today rely on the WCF context (`OperationContext` and `WebOperationContext`) for performing different things, specially REST services where the context is necessary for settings and getting Http status codes or headers. 

The fact that the WCF context does not expose interfaces or base classes complicates the unit testing a lot. 

In order to test a WCF service with what we have today, we have to either test the service as a black box (integration tests, which requires a lot of plumbing code to setup all the WCF infrastructure for the test, channels, host, client, etc) or create some wrappers to encapsulate the WCF context behavior.

WCFMock is a solution for unit testing WCF services, it provides a set of useful classes that will help you to remove all the explicit dependencies with the operation context, and still provide a good way to mock them from unit tests.

###Example

* WCF REST service that returns a RSS feed with a catalog of products.

> This service implementation only relies on the `WebOperationContext` to set up the response content type, that is being done in the following line,

```csharp
WebOperationContext.Current.OutgoingResponse.ContentType = "application/atom+xml";
```

* Now to find a way to get rid of that dependency 

> so we can unit test that method. Here is where WCFMock comes to the rescue. 

> The first thing you have to do is to define a new alias on top of your class,

```csharp
using WebOperationContext = System.ServiceModel.Web.MockedWebOperationContext;
```

> Optionally, you can wrap that sentence with a conditional compilation directive

```csharp
#if DEBUG
using WebOperationContext = System.ServiceModel.Web.MockedWebOperationContext;
#endif
```

> This is useful for instance, if you want to use the mocked version in development, and always the WCF version in production. 

> That's all, you do not need to touch your existing service implementation at all, once you defined that alias, the service is ready to be tested.

* For testing the service

> Use Moq, a pretty good mock framework
