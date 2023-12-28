# Global Error Handling
## Note: Currently, I have implemented two types in this repository, and in the future, other methods will be added.

### You can use this methods for handling errors globally.

## > First mrthod use middleware:

```csharp
public async Task InvokeAsync(HttpContext context)
{
    try
    {
        await _next(context);
    }
    catch (Exception exception)
    {
        var problemDetails = new ProblemDetails
        {
            Status = GetStatusCode(exception),
            Title = GetTitle(exception),
            Type = "Server Error",
            Detail = exception.Message
        };

        context.Response.StatusCode =
            StatusCodes.Status500InternalServerError;

        await context.Response.WriteAsJsonAsync(problemDetails);
    }
}
```
### now must add middleware in program.cs file .

```csharp
app.UseMiddleware<ExceptionHandlingMiddleware>();
```

### In these conditions, any exception thrown in the code will be managed by this middleware wherever it occurs.



## > the second mrthod use middleware:

### You must be use  dotnet 8 for can use Interface "IExceptionHandler" in project.

```csharp
internal sealed class GlobalExceptionHandler : IExceptionHandler
```

### You must be implement method TryHandleAsync.

```csharp
    public async ValueTask<bool> TryHandleAsync(
        HttpContext httpContext,
        Exception exception,
        CancellationToken cancellationToken)
    {
        _logger.LogError(
            exception, "Exception occurred: {Message}", exception.Message);

        var problemDetails = new ProblemDetails
        {
            Status = GetStatusCode(exception),
            Title = GetTitle(exception),
            Type = "Server Error",
            Detail = exception.Message
        };

        httpContext.Response.StatusCode = problemDetails.Status.Value;

        await httpContext.Response
            .WriteAsJsonAsync(problemDetails, cancellationToken);

        return true;
    }
```

### now must register GlobalExceptionHandler class and ProblemDetails in program.cs file like below :

```csharp
builder.Services.AddExceptionHandler<GlobalExceptionHandler>();
builder.Services.AddProblemDetails();
```
### and add middleware :
```csharp
app.UseExceptionHandler();
```

## Writing better documentary ...