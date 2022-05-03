# Softmax.ZarinPal
Unofficial implementation of ZarinPal API for .NET 6

[![Nuget Version][nuget-shield]][nuget]
[![Nuget Downloads][nuget-shield-dl]][nuget]

## Installing the NuGet Package
You can install this package by entering the following command into your `Package Manager Console`:

```powershell
Install-Package Softmax.ZarinPal -PreRelease
```

*Note:* This package requires **.NET 6.0**.

## Use in ASP.NET Core 6
### Startup Configuration
To use `ZarinPalService` you need to register it in your `Program.cs` class: 

```csharp
// using Softmax.ZarinPal;
// using Softmax.ZarinPal.Enums;

builder.Services.AddZarinPal(options =>
{
    // Required
    options.MerchantId = "your-merchant-id";
    
    // Optional (defualt: IRR)
    options.CurrencyType = CurrencyType.IRR;
    
    // Optional
    options.DefaultCallbackUri = new Uri("your-callback-uri");
});
```

*Note:* IRR is **Iran Rial** and IRT is **Iran Toman**.

### Controller Configuration
After registering the service, you need to add it to your controller.

```csharp
// using Softmax.ZarinPal;

public class HomeController : Controller
{
    private readonly IZarinPalService _zarinpal;

    public HomeController(IZarinPalService zarinpal)
    {
        _zarinpal = zarinpal;
    }
}
```

### Payment
You can request a payment via `PaymentAsync` method.

- **Amount** is the transaction amount. (Required)
- **Description** is a short description about the transaction. (Required)
- **Callback Url** is the url to redirect to after the transaction. (Optional)

```csharp
[Route("send")]
public async Task<IActionResult> Send()
{
      // Defualt CurrencyType is IRR (Rial) , you can change it in service options
      long amount = 1000; // Required   
      string description = "This is a test payment"; // Required
      
      string callbackUrl = new Uri("https://localhost:5001/verify"); // Optional 
      string email = "your-email"; // Optional 
      string mobile = "your-mobie"; // Optional 
  
      var result = await _zarinPal.PaymentAsync(new PaymentRequest
      {
          CallbackUrl = callbackUrl,
          Description = description,
          Amount = amount,
          Email = email,
          Mobile = mobile,
      });
  
      if (result.IsSuccess())
      {
          return Redirect(result.Data.PaymentUri.AbsoluteUri);
      }
  
      return Ok($"Failed, Error code: {result.Error.Code}");
}
```

### Verify Payment
You can verify the transaction via `VerifyAsync` method.

> This action was our **Callback Url** in the `PaymentAsync` method.
> After the transaction was completed, the payment provider will redirect to this action. 

```csharp
[Route("verify")]
public async Task<IActionResult> Verify()
{
    // Get Status and Authority and show error if not available.
    if (!Request.Query.TryGetValue("Status", out var status) ||
        !Request.Query.TryGetValue("Authority", out var authority))
    {
      return BadRequest();
    }
  
    long amount = 1000;
    var result = await _zarinPal.VerifyAsync(authority: authority, amount: amount);
  
    // Check if transaction was successful.
    if (result.IsSuccess())
    {
      return Ok($"Success, RefId: {result.Data.RefId}");
    }
  
    // Show unsuccessful transaction with code.
    return BadRequest($"Failed, Error code: {result.Error.Code}");
}
```

## Another .NET Platforms
### Payment
You can request a payment via `PaymentAsync` method.

- **Amount** is the transaction amount. (Required)
- **Description** is a short description about the transaction. (Required)
- **Callback Url** is the url to redirect to after the transaction. (Optional)

```csharp
using Softmax.ZarinPal;

namespace ZarinPal
{
  internal class Program
  {
    static void Main(string[] args)
    {
        IZarinPalService zarinPal = 
          new ZarinPalService(merchantId: "your-merchant-id", defaultCallbackUri: new Uri("your-callback-uri"));
          
        // Defualt CurrencyType is IRR (Rial) , you can change it in service options
        long amount = 1000; // Required   
        string description = "This is a test payment"; // Required
  
        string callbackUrl = new Uri("https://localhost:5001/verify"); // Optional 
        string email = "your-email"; // Optional 
        string mobile = "your-mobie"; // Optional 
  
        var result = await zarinPal.PaymentAsync(new PaymentRequest
        {
            CallbackUrl = callbackUrl,
            Description = description,
            Amount = amount,
            Email = email,
            Mobile = mobile,
        });
  
        if (result.IsSuccess())
        {
            Console.WriteLine(result.Data.PaymentUri.AbsoluteUri);
        }
  
        Console.WriteLine($"Failed, Error code: {result.Error.Code}");
    }
  }
}
```

### Verify Payment
You can verify the transaction via `VerifyAsync` method.

> This action was our **Callback Url** in the `PaymentAsync` method.
> After the transaction was completed, the payment provider will redirect to this action. 

```csharp
using Softmax.ZarinPal;

namespace ZarinPal
{
  internal class Program
  {
    static void Main(string[] args)
    {
        IZarinPalService zarinPal = 
          new ZarinPalService(merchantId: "your-merchant-id", defaultCallbackUri: new Uri("your-callback-uri"));
              
        long amount = 1000;
        var result = await zarinPal.VerifyAsync(authority: "your-authority", amount: 1000);
      
        // Check if transaction was successful.
        if (result.IsSuccess())
        {
          Console.WriteLine($"Success, RefId: {result.Data.RefId}");
        }
      
        // Show unsuccessful transaction with code.
        Console.WriteLine($"Failed, Error code: {result.Error.Code}");
    }
  }
}
```


## License
This project is licensed under the [MIT License](LICENSE).

[nuget]: https://www.nuget.org/packages/Softmax.ZarinPal
[nuget-shield]: https://img.shields.io/nuget/v/Softmax.ZarinPal
[nuget-shield-dl]: https://img.shields.io/nuget/dt/Softmax.ZarinPal
