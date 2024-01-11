---
layout: post
title: "Auth Series #2 - Protect ASP.NET Core Api with Azure Entra ID and Access It via Postman"
author: mirzaevolution
categories:
  - Azure
  - Entra ID
  - Authentication
  - Authorization
tags:
  - Azure
  - Entra ID
  - Authentication
  - Authorization
post_image: /assets/images/auth-series-2/2024-01-11_17h42_18.png
---

# Auth Series #2 - Protect ASP.NET Core Api with Azure Entra ID and Access It via Postman



This is 2nd tutorial of the **Auth Series**. In this tutorial, we'll create
a basic ASP.NET Core Web Api and protect it using Azure AD/Entra ID.
If you want to know how to protect web app with Azure AD/Entra ID, you can checkout 
our previous tutorial: [Auth Series #1 - Azure Entra Id Authentication using ASP.NET Core MVC](/auth-series-1-azure-entra-id-authentication-using-aspnet-core-mvc)


**Requirements:**
 
    - Web Framework: ASP.NET Core 7x Api Default Project
    - Nuget: Microsoft.Identity.Web

<br/>

We are going to make a simple WeatherForecast Api and protect it using JWT Bearer/Oauth 
via Microsoft Entra ID / Azure ID and we will then call the api endpoint via Postman. 

Before we move further, let's start the first step.

### 1. Create Web Api Registration

We need to create an api registration in the Azure Portal so that we can
protect our api. First, we need to go to **Microsoft Entra ID Portal > App Registrations > New registration**.

![2024 01 11 17H17 43](/assets/images/auth-series-2/2024-01-11_17h17_43.png)

In this tutorial, we will give name **uptec-auth-api** but you can give anyname you want. Leave other options and hit Register.

![2024 01 11 17H21 01](/assets/images/auth-series-2/2024-01-11_17h21_01.png)

After it has been created, you should take a note on the Client Id and Tenant Id for our api **appsettings.json**.

![2024 01 11 17H21 15](/assets/images/auth-series-2/2024-01-11_17h21_15.png)


Now, go to **Expose an API > Add Application ID URI** and **Add a scope**. 
Application ID URI is important as an identifier for our scope later. And the scope
itself is like an attribute that we can use to protect an api as part of authorization. It means, after an api gets
validated (JWT Token validated), we can check its scope in the JWT token as well. If it has 
the certain scopes that we've registered, the request can be continued. Otherwise, 
the request can be stopped and issue 403-forbidden.


![2024 01 11 17H23 20](/assets/images/auth-series-2/2024-01-11_17h23_20.png)

Add the Application ID URI like the image below.

![2024 01 11 17H23 45](/assets/images/auth-series-2/2024-01-11_17h23_45.png)

After that, add a new scope, give it name -> **Access.Read** and fill other 
informations like Title and Description like in the screenshot below.

![2024 01 11 17H25 22](/assets/images/auth-series-2/2024-01-11_17h25_22.png)

Once created, don't forget to copy/take a note the scope for later use.


### 2. Create Web Api Project

Now, we are heading to create ASP.NET Core Web API project. In this section, i will 
use .NET 7. 

Open Visual Studio, Select ASP.NET Core Web API project. Give it a name, and make sure leave 
other options as screenshots below.

![2024 01 11 17H26 56](/assets/images/auth-series-2/2024-01-11_17h26_56.png)

![2024 01 11 17H27 11](/assets/images/auth-series-2/2024-01-11_17h27_11.png)

![2024 01 11 17H27 22](/assets/images/auth-series-2/2024-01-11_17h27_22.png)

Once the project initialized, modify the appsettings.json and add the below section.

    "AzureAd": {
        "Instance": "https://login.microsoftonline.com/",
        "TenantId": "",
        "ClientId": "",
        "Scopes": "api://<GUID>/Access.Read" 
      }

Make sure you copy past the TenantId, ClientId and Scope from our previous #1 section.

![2024 01 11 17H29 46](/assets/images/auth-series-2/2024-01-11_17h29_46.png)

Install the nuget package: 

```
Microsoft.Identity.Web
```

This library is used to enable JWT Bearer/API Protection using token in our api project.

![2024 01 11 17H30 24](/assets/images/auth-series-2/2024-01-11_17h30_24.png)

To customize the program execution, we need to modify our **launchSettings.json**. 
Cleanup everything and leave the "https" section with default port 8181.

```
{
  "$schema": "https://json.schemastore.org/launchsettings.json",
  "profiles": {
    "https": {
      "commandName": "Project",
      "dotnetRunMessages": true,
      "launchBrowser": true,
      "launchUrl": "swagger",
      "applicationUrl": "https://localhost:8181;",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    }
  }
}

```

![2024 01 11 17H33 18](/assets/images/auth-series-2/2024-01-11_17h33_18.png)


### 3. Register the MicrosoftIdentityWebApiAuthentication

Once the above setup completed, we will enable api protection by customizing the **Program.cs** file.
Add the following namespace:

```
using Microsoft.Identity.Web;
```

then add the following code to register the **MicrosoftIdentityWebApiAuthentication** to enable api 
authentication/protection using JWT Token from Microsoft Entra ID/Azure AD.

```
services
    .AddMicrosoftIdentityWebApiAuthentication(configuration, "AzureAd");
```

![2024 01 11 17H35 10](/assets/images/auth-series-2/2024-01-11_17h35_10.png)

The above snippet will read our configuration section of **AzureAd** in **appsettings.json** file.


### 4. Protect WeatherForecast Sample API

Once point #3 completed, now we are jumping to the last step to mark our controller 
to be **Authorized**. It means, only authenticated members are allowed to access the endpoint.

![2024 01 11 17H35 36](/assets/images/auth-series-2/2024-01-11_17h35_36.png)


### 5. Test The App - Got 401 Unauthorized

Right. Now we will test the app using Postman. And because we are not authenticated, of course 
we'll get 401.

![2024 01 11 17H42 18](/assets/images/auth-series-2/2024-01-11_17h42_18.png)


### 6. Create New App Registration For Postman (Microsoft Entra ID)

Ok, because we got 401, we need our Postman to be authenticated to access the endpoint. 
Now, we will create new App Registration special for any client that will access any protected apis 
that we define.

Give the name anything but ended with **-caller** (optional - previous app registration was **uptect-auth-ap**).

![2024 01 11 17H42 29](/assets/images/auth-series-2/2024-01-11_17h42_29.png)

Once created, take a note on those client id and tenant id.

![2024 01 11 17H42 43](/assets/images/auth-series-2/2024-01-11_17h42_43.png)

Create the client secret and take a note of it in **Certificate & secrets** menu.

![2024 01 11 17H43 23](/assets/images/auth-series-2/2024-01-11_17h43_23.png)

![2024 01 11 17H43 41](/assets/images/auth-series-2/2024-01-11_17h43_41.png)

Now, for our Postman to work, we need to register the callback url in the **Authentication** menu.
Paste the below url:

```
https://oauth.pstmn.io/v1/callback
```

![2024 01 11 17H44 52](/assets/images/auth-series-2/2024-01-11_17h44_52.png)
![2024 01 11 17H45 56](/assets/images/auth-series-2/2024-01-11_17h45_56.png)

After we have defined our callback url, we need to make sure our Postman's app registration 
can easily access our Protected Api's app registration. We need to request api permissions for our 
api's scope we defined earlier in the beginning. Go to **API Permissions** menu, 
select **Add a permission** button, in the **APIs my organization uses** type your previous api's app registration name
 and select it.

![2024 01 11 17H48 20](/assets/images/auth-series-2/2024-01-11_17h48_20.png)

Once selected, choose the permission that we defined earlier.

```
Access.Read
```

![2024 01 11 17H48 54](/assets/images/auth-series-2/2024-01-11_17h48_54.png)

The last step is to grant admin consent for requested api's scope(s).

![2024 01 11 17H49 21](/assets/images/auth-series-2/2024-01-11_17h49_21.png)



### 7. Access the Protected API via Postman

After the step #6 completed, now we can access the api with Postman. 
To do so, we nee to collect OAuth v2 authorization and token endpoints from our api's 
app registration.

Take a note both the OAuth v2 authorization and token endpoints like the image below: 

**NB: This is api's app registration not Postman's app regisration**

![2024 01 11 17H50 19](/assets/images/auth-series-2/2024-01-11_17h50_19.png)

The next step is to go the our previous Postman, in the Authorization section, choose 
the **OAuth 2.0** type.

![2024 01 11 17H52 28](/assets/images/auth-series-2/2024-01-11_17h52_28.png)

Now, fill the information in the section below:

![2024 01 11 17H54 29](/assets/images/auth-series-2/2024-01-11_17h54_29.png)

<br/>

    - Auth Url -> Authorization endpoint of api's app registration.
    - Access Token URL -> Token endpoint of api's app registration.
    - Client ID -> client id of our Postman
    - Client Secret -> client secret of our postman
    - Scope -> scope of the api - api://(GUID)/Access.Read
    
<br/>

After that, scroll down and click the **Get New Access Token**. It will authenticate using browser.
And on the successfull sign-in, make sure you proceed and click **Use Token** button.

![2024 01 11 17H54 58](/assets/images/auth-series-2/2024-01-11_17h54_58.png)

![2024 01 11 17H55 15](/assets/images/auth-series-2/2024-01-11_17h55_15.png)

![2024 01 11 17H55 39](/assets/images/auth-series-2/2024-01-11_17h55_39.png)

![2024 01 11 17H55 45](/assets/images/auth-series-2/2024-01-11_17h55_45.png)


Ok, if no issue, we can now test to call our api endpoint.

![2024 01 11 17H56 30](/assets/images/auth-series-2/2024-01-11_17h56_30.png)

We can also copy paste our access token in the https://jwt.ms to 
inspect the payload information of it.

![2024 01 11 17H56 59](/assets/images/auth-series-2/2024-01-11_17h56_59.png)

![2024 01 11 17H57 28](/assets/images/auth-series-2/2024-01-11_17h57_28.png)
.



Ok, i think that's all for this tutorial. In the next tutorial, we will be accessing our protected api 
via console application built on top of .NET Core 7x.


> Sample project: https://github.com/mirzaevolution/Uptec-Protected-Web-Api



Regards,

**Mirza Ghulam Rasyid**
