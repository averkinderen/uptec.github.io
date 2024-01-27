---
layout: post
title: "Auth Series #1 - Microsoft Entra ID Authentication using ASP.NET Core MVC"
author: mirzaevolution
categories:
  - Azure
  - Microsoft Entra ID
  - Authentication
tags:
  - Azure
  - Microsoft Entra ID
  - Authentication
post_image: /assets/images/auth-series-1/2024-01-15_08h50_42.png
---

# Auth Series #1 - Microsoft Entra ID Authentication using ASP.NET Core MVC

This tutorial will be a series of blog posts to cover some common Authentication/Authorization scenarios using Azure Entra Id (Azure AD).
Here are the details of the sample project used:

 1. Web Framework: ASP.NET Core 7x MVC
 2. Nuget: Microsoft.Identity.Web

We are going to make a simple ASP.NET Core MVC app (default template) that will have login and logout buttons/links to access protected page (Privacy page).
The login process will take you to Azure Entra Id for authentication using your work account.

Before we proceed, let's setup the things on the Azure Portal first.

## 1. Register the app

We need to create new Azure Entra ID application for our sample. You can follow the screenshots below to get to started.

![2024 01 10 08H42 24](/assets/images/auth-series-1/2024-01-10_08h42_24.png)

After pressing **New Registration** button, you can fill the registration name with any name you want and on the Redirect Uri, select Web option and enter the url: `https://localhost:8080/signin-oidc`.
That url will be used by our app and /signin-oidc as a callback path. After proper authentication you will be redirected to this page.

![2024 01 10 08H45 04](/assets/images/auth-series-1/2024-01-10_08h45_04.png)

Next step is to take a note of some of the registration details from the Overview page like the client id and tenant id.

![2024 01 10 08H45 19](/assets/images/auth-series-1/2024-01-10_08h45_19.png)

The next step is to create Client Secret (optional) for later use if we want to access API.
After the Client Secret been created, save the value of it.

![2024 01 10 08H46 32](/assets/images/auth-series-1/2024-01-10_08h46_32.png)
![2024 01 10 08H47 14](/assets/images/auth-series-1/2024-01-10_08h47_14.png)

Our last step is to enable ID Tokens for authentication process. Go to the Authentication page, and tick the ID Tokens.

![2024 01 10 09H17 25](/assets/images/auth-series-1/2024-01-10_09h17_25.png)

## 2. Initiate the ASP.NET Core MVC Project

Now, we need to create an ASP.NET Core MVC Project. You can give any name for this sample project.

![2024 01 10 08H49 21](/assets/images/auth-series-1/2024-01-10_08h49_21.png)
![2024 01 10 08H49 49](/assets/images/auth-series-1/2024-01-10_08h49_49.png)

In the below section, make sure you select **None** for the Authentication type. As we would like to setup everything manually.

![2024 01 10 08H50 03](/assets/images/auth-series-1/2024-01-10_08h50_03.png)

After project is loaded. We need to adjust the **launchSettings.json**.
Clean-up everything and just leave the https section like below:

![2024 01 10 08H50 49](/assets/images/auth-series-1/2024-01-10_08h50_49.png)

The next step is to install a nuget package that will handle everything for us.

`Microsoft.Identity.Web`

![2024 01 10 08H52 15](/assets/images/auth-series-1/2024-01-10_08h52_15.png)

Once the nuget package is installed, we need to adjust our **appsettings.json** to hold information thelike ClientId, TenantId etc.
Make sure you replace the value with the ones you got when register the app in Azure Entra Id portal.

**NB**: In real world, you should put this on Azure KeyVault or something similar or even encrypt it.

![2024 01 10 08H56 35](/assets/images/auth-series-1/2024-01-10_08h56_35.png)

## 3. Adding authentication

Let's jump to **Program.cs** file. Here, we're gonna add the registration steps
to make sure OpenIdConnect authentication using Azure Entra Id works perfectly.
At first, make sure you include these following namespaces.

    using Microsoft.AspNetCore.Authentication.Cookies;
    using Microsoft.Identity.Web;

![2024 01 10 09H18 01](/assets/images/auth-series-1/2024-01-10_09h18_01.png)

After that, we can add the OpenIdConnect authentication registration.  
Copy paste the following snippet:

    services.AddMicrosoftIdentityWebAppAuthentication(configuration, configSectionName: "AzureAd");
      
The code above will register the OpenIdConnect using Microsoft Identity Web library that we have installed previously and will read the configuration section **"AzureAd"** that has the information like ClientId, TenantId, etc (you can override the name or even the options).

Copy past the next snippet here:

    services.Configure<CookieAuthenticationOptions>(options =>
                {
                    options.LoginPath = new PathString("/Auth/Login");
                    options.LogoutPath = new PathString("/Auth/Logout");
                    options.ReturnUrlParameter = "redirectUrl";
                });

That code will re-configure the CookieAuthenticationOptions to setup custom login/logout path and return url. We'll later create Controller for this.

![2024 01 10 09H18 27](/assets/images/auth-series-1/2024-01-10_09h18_27.png)

The last step, add the `app.UseAuthentication()` code right before
the `app.UseAuthorization()` to enable authentication.

![2024 01 10 09H19 01](/assets/images/auth-series-1/2024-01-10_09h19_01.png)

## 4. Adding AuthController to handle login/logout

Now, let's create new mvc controller and name it **AuthController**.

![2024 01 10 09H19 35](/assets/images/auth-series-1/2024-01-10_09h19_35.png)

Inside the AuthController, we need to inject the **IConfiguration** instance and add login method for that.

    public IActionResult Login(string redirectUrl = "/")
            {
                return Challenge(new AuthenticationProperties
                {
                    RedirectUri = Url.IsLocalUrl(redirectUrl) ?
                        redirectUrl : Url.Action("Index", "Home", null, Request.Scheme)
                },
                    OpenIdConnectDefaults.AuthenticationScheme);
            }

The code above will challenge the OpenIdConnect that we have registered.
We can also see the returnUrl/redirectUrl used to make sure after authentication finished, the IDP will return us back to correct url.

![2024 01 10 09H20 06](/assets/images/auth-series-1/2024-01-10_09h20_06.png)

we need to create the logout method as well. This time, we need to mark it as authorized, as this endpoint only available after authentication completed (non-anonymous).

        [Authorize]
        public IActionResult Logout()
        {
            return SignOut(
                OpenIdConnectDefaults.AuthenticationScheme,
                CookieAuthenticationDefaults.AuthenticationScheme);
        }

![2024 01 10 09H20 54](/assets/images/auth-series-1/2024-01-10_09h20_54.png)

## 5. Adding sample authorized page

![2024 01 10 09H21 19](/assets/images/auth-series-1/2024-01-10_09h21_19.png)

Go to HomeController, instead of creating new page/action, we can just mark the default Privacy action/page with `[Authorized]` attribute.

## 6. Modify the _Layout.cshtml

![2024 01 10 09H21 45](/assets/images/auth-series-1/2024-01-10_09h21_45.png)

Add the following snippets like in the screenshot above so that we can have login/logout link with authenticated username in the navbar section.

                    <ul class="navbar-nav ms-auto">
                        @if (User.Identity.IsAuthenticated)
                        {
                            <li class="nav-item">
                                <a class="nav-link text-dark" href="#">@User.Identity.Name</a>
                            </li>
                            <li class="nav-item">
                                <a class="nav-link text-dark" asp-area="" asp-controller="Auth" asp-action="Logout">Logout</a>
                            </li>
                        }
                        else
                        {
                            <li class="nav-item">
                                <a class="nav-link text-dark" asp-area="" asp-controller="Auth" asp-action="Login">Login</a>
                            </li>
                        }
                    </ul>

## 7. Testing out the app

Now, after everything is completed, we can test the app by clicking the login link or accessing the Privacy page.

![2024 01 10 09H22 07](/assets/images/auth-series-1/2024-01-10_09h22_07.png)

![2024 01 10 09H22 16](/assets/images/auth-series-1/2024-01-10_09h22_16.png)

![2024 01 10 09H22 31](/assets/images/auth-series-1/2024-01-10_09h22_31.png)

After we are successfully logged-in. We can access protected page.

![2024 01 10 09H22 34](/assets/images/auth-series-1/2024-01-10_09h22_34.png)

To logout from the app, click the logout link.

![2024 01 10 09H22 46](/assets/images/auth-series-1/2024-01-10_09h22_46.png)

You can find the full sample project here:

> Sample project: https://github.com/mirzaevolution/Uptec-Entra-Id-Web-Login

Hope this helps,
Mirza
