---
layout: post
title:  "Guess how AAD Auth token is signature verified by the .Net application"
date:   2023-03-29 02:12:00 +0530
categories: jekyll update
published: true
---

Azure Active Directory simplifies the burden of Authentication and Authorization process while securing an application.

But have you ever guessed how could the process flow is so robust and flawless, so that the token (*JWT token*) is verified for the issuer and cannot be tampered.

Let's have a look at how we can secure a simple .Net minimal api using the AAD Authentication (*Microsoft Identity Platform*).

So, just jump in to create a minimal api using visual studio

> **Prerequisites**  
>Visual Studio 2022  
>Dotnet 7 Runtime

Create an api project project using Visual Studio   
![image](\images\2023-03-29-guess-how-aad-auth-token-gets-verified-by-the-application\Get Started 2023-03-29 193113.png){: width=600px }

We need only minimal setup to implement *Microsoft.Platform* authentication.    
![image](\images\2023-03-29-guess-how-aad-auth-token-gets-verified-by-the-application\Minimal Setup 2023-03-29 193644.png)

The solution looks simple as:

![image](\images\2023-03-29-guess-how-aad-auth-token-gets-verified-by-the-application\Minimal Soln 2023-03-29 194013.png)

Right click the connected services to add the *Microsoft Identity* platform as a connected service dependency.

![image](\images\2023-03-29-guess-how-aad-auth-token-gets-verified-by-the-application\MI as connected service 2023-03-29 194757.png)

Click plus button.

![image](\images\2023-03-29-guess-how-aad-auth-token-gets-verified-by-the-application\Add MI dependency 2023-03-29 195156.png)

Search `identity`

![image](\images\2023-03-29-guess-how-aad-auth-token-gets-verified-by-the-application\Search MI 2023-03-29 195347.png)

Click Next.


![image](\images\2023-03-29-guess-how-aad-auth-token-gets-verified-by-the-application\Click Next 2023-03-29 200345.png)

Click Next
It lists the App Registrations in the Active Directory.

We are using a new app registration to authenticate our app.

So lets click on the create button.

![image](\images\2023-03-29-guess-how-aad-auth-token-gets-verified-by-the-application\Create aapregs 2023-03-29 200715.png)

Name the app and tap `Register`

![image](\images\2023-03-29-guess-how-aad-auth-token-gets-verified-by-the-application\Reg App 2023-03-29 201337.png)

Tap `Next`

Summary would look like

![image](\images\2023-03-29-guess-how-aad-auth-token-gets-verified-by-the-application\Finish reg 2023-03-29 201627.png)

If the App Registration is completed successfully, you would see a screen similar to this.

![image](\images\2023-03-29-guess-how-aad-auth-token-gets-verified-by-the-application\Reg Complete 2023-03-29 201716.png)

Now that, you have added the `Microsoft Identity Platform` dependency successfully, you can see some additional codes in your `Program.cs`.
There would be some additional packages installed and referenced in the `Program.cs`.
```csharp
        using Microsoft.AspNetCore.Authentication;
        using Microsoft.AspNetCore.Authentication.JwtBearer;
        using Microsoft.Graph;
        using Microsoft.Identity.Web;
```

and these lines of codes will implement the *Microsoft Identity* authentication using AAD.
```csharp
    builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
        .AddMicrosoftIdentityWebApi(builder.Configuration.GetSection("AzureAd"));
```

In `applicationsettings.json` you can find the `AzureAd` section updated as per the details of the App Reg we have created `MinimalAuthApp`. 

```json

     "AzureAd": {
        "Instance": "https://login.microsoftonline.com/",
        "Domain": "mydomain.com",
        "TenantId": "xxxxxx-xxxxxx-xxxxxx-xxxxxx-xxxxxx",
        "ClientId": "xxxxxx-xxxxxx-xxxxxx-xxxxxx-xxxxxx",
        "CallbackPath": "/signin-oidc",
        "Scopes": "access_as_user"
     }
```
Lets keep it simple; open `Program.cs` and remove the `swagger` configuration.

```csharp

// Remove
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}
```

Update `launchSettings.json`

```json
{
  "$schema": "https://json.schemastore.org/launchsettings.json",
  "iisSettings": {
    "windowsAuthentication": false,
    "anonymousAuthentication": true,
    "iisExpress": {
      "applicationUrl": "http://localhost:6850",
      "sslPort": 44315
    }
  },
  "profiles": {
    "http": {
      "commandName": "Project",
      "dotnetRunMessages": true,
      "launchBrowser": true,
      "launchUrl": "weatherforecast", //👈 change url
      "applicationUrl": "http://localhost:5086",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    },
    "https": {
      "commandName": "Project",
      "dotnetRunMessages": true,
      "launchBrowser": true,
      "launchUrl": "weatherforecast", //👈 change url
      "applicationUrl": "https://localhost:7116;http://localhost:5086",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    },
    "IIS Express": {
      "commandName": "IISExpress",
      "launchBrowser": true,
      "launchUrl": "weatherforecast", //👈 change url
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    }
  }
}

```

Start you application, this will launch the application on `/weatherforecast` endpoint.

We didn't add any Authorization header, so, as expected `401` page.

![image](\images\2023-03-29-guess-how-aad-auth-token-gets-verified-by-the-application\weatherforecast 401 2023-04-04 202735.png)


Where we can get the token fromm????

If you check the [microsoft documentation](), it describes different flows and request to get a token from the end points, ie, the AAD endpoints.

For simplicity we are using `implicit` flow and the url to generate a token in the callback looks like this...

```http
https://login.microsoftonline.com/{tenant_id}/oauth2/v2.0/authorize?
client_id={client_id}
&response_type=token
&redirect_uri=https%3A%2F%2Flocalhost%3A7116%2Fmyapp%2F
&scope=api%3A%2F%2F0ec35133-b038-49eb-86d0-fb1c37fd17bd%2Faccess_as_user
&response_mode=fragment
&state=12345
&nonce=678910
&prompt=none


```
Don't forget to configure the redirect url and token flow (here `implicit`) and scope in your App Registration and make a call to the token end point.

You will get the callback url and token as a fragment:

```http
https://localhost:7116/myapp/#access_token=eyJ0eXAiOiJKV1QiLCJhbGciOixxxxxxx.eyJhdWQiOiIwZWMzNTEzMy1iMDM4LTQ5xxxxxxx.TmRzKkvl7TemO9y64nXHHkKjlhmJXM_Hx240QpMVxxxxxxx&token_type=Bearer&expires_in=4928&scope=api%3a%2f%xxxxxxxxxxxxxxxxxx%2faccess_as_user&state=12345&session_state=xxxxxxxxxxxxxxxxxxxxxxxxxxxx
```
Copy the token and use `postman` to make a post call with the token as the Bearer Authorization header.

![image](\images\2023-03-29-guess-how-aad-auth-token-gets-verified-by-the-application\Postman response 2023-04-04 205015.png)
Yay.....
Go got a response authenticated by Azure AAD.

So far so good....

 But now comes the difficult question....

How does your application knows that the token it received is actually generated by the AAD itself.
Or, why I could not make a JWT token and authenticate the application.

In order to cut the clutter, let's have look at the roken first.

Go to the [jwt.io](https://jwt.io) and decrypt the token received.

```
HEADER:ALGORITHM & TOKEN TYPE

{
  "typ": "JWT",
  "alg": "RS256",
  "kid": "-KI3Q9nNR7bxxxxxxxxxxx"
}
PAYLOAD:DATA

{
  "aud": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
  "iss": "https://login.microsoftonline.com/{tenant_id}/v2.0",
  "iat": 1680620723,
  "nbf": 1680620723,
  "exp": 1680625952,
  "aio": "AZQAaaaaaa/8TAAxxxxxxxxxxxxxxxxxxxxx/40bZJVqxIY1Hsn+VuQJOFxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
  "azp": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
  "azpacr": "0",
  "name": "Arun Paul (Polly)",
  "oid": "xxxxxxxxxxxxxxxxxxxxx",
  "preferred_username": "apaul@ariqt.com",
  "rh": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
  "scp": "access_as_user",
  "sub": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
  "tid": "8xxxxxxxxxxxxxxxxxxxxx",
  "uti": "xxxxxxxxxxxxxxxxxxxxxxxxxxxx",
  "ver": "2.0"
}
VERIFY SIGNATURE

RSASHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  
{
  "e": "AQABadad",
  "kty": "RSA",
  "n": "btsbnnrsn6wSoDAhgRYSDkaMuEHy75VikiB8wg25WuR96gdMpookdlRvh7SnRvtjQN9b5m4zJCMpSRcJ5DuXl4mcd7Cg3Zp1C5-JmMq8J7m7OS9HpUQbA1yhtCHqP7XA4UnQI28J-TnGiAa3viPLlq0663Cq6hQw7jYo5yNjdJcV5-FS-xNV7UHR4zAMRruMUHxte1IZJzbJmxjKoEjJwDTtcd6DkI3yrkmYt8GdQmu0YBHTJSZiz-M10CY3LbvLzf-tbBNKQ_gfnGGKF7MvRCmPA_YF_APynrIG7p4vPDRXhpaggtehbths"
}
,
  
Private Key in PKCS #8, PKCS #1, or JWK string format. The key never leaves your browser.

)
```
The payload has the data we expected from the AAD App registration.
And interestingly the token came signature verified.

![image](\images\2023-03-29-guess-how-aad-auth-token-gets-verified-by-the-application\Signature verified 2023-04-04 210328.png)

The AAD uses a private key to sign the token and the only the public key pair can decrypt it.
So the token came signature verified with the public key.

If you want to know more about generating a JWT token with key pairs, please read my blog [here]().

Still the question is not completely answered....

How does the application verify that the public key used for successfully decrypt the token was generated by AAD itself??

Your application became smarter with the Microsoft.Identity platform to verify this.

To really understand what does it happen under the hood when you try to authenticate/authorize your application, we need some http interceptor to analyze all the http calls the application makes.

[Fiddler Everywhere]() is a great tool for that, which can be installed free on a trial basis.

Once you installed the tool, open it 
***Happy coding....***