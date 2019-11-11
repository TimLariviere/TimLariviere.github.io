---
layout: post
title: "Call an Azure AD protected API in Xamarin/UWP apps"
date: "2017-02-12"
---

When we talk about mobile apps, a web API is generally hiding in the background, doing most of the work like connecting to a database, verifying authorization, computing data, and so on. Like everything else, those APIs need protection against unauthorized calls that don't come from our mobile apps.

In this post, I will explain how to leverage Azure Active Directory to protect our WebAPI without any code, and how to call this WebAPI from our mobile apps.

### So, how do we do that?

In order, we will :

1. Enable Azure AD authentication on our API
2. Declare our apps in the Azure AD in order to allow them to authenticate their users and give them access to our API
3. Implement ADAL.NET in our apps to authenticate the user and retrieve an access token for our API
4. Call our API with the access token
5. ???
6. Profit

### Before we begin

For this post, I will use the default ASP.NET Core WebAPI that I will host in an Azure App Service. The mobile apps will be made with Xamarin.iOS, Xamarin.Android and UWP, core logic will be implemented inside a .NETStandard 1.4 project.

I made a sample containing the WebAPI and the mobile apps, it's available on Github : [https://github.com/TimLariviere/WoodenMoose.Samples.Xamarin\_Authentication](https://github.com/TimLariviere/WoodenMoose.Samples.Xamarin_Authentication)

If you want to run the sample, you will need to have an active Azure subscription with an Azure Active Directory. Then you will need to declare your apps in the Azure AD like we will see in this post.

### Configuring Azure AD for the WebAPI

Before we can do anything with Xamarin or UWP, we need to set up our WebAPI and configure the authentication against Azure Active Directory. This used to require lots of setup code in our WebAPI in order to allow only authenticated requests, most of the times specific to only one identity provider. Well, this time Azure really make it easy on us with a "turn key" feature that require no code at all!

I have already created an Azure App Service named "**woodenmoose-authsample-api**" and published in it an ASP.NET Core WebAPI, generated from the default template.

To enable authentication with Azure AD, I just need to go into the "Authentication / Authorization" panel, turn the feature on and configure it to create a new Azure AD app representing my WebAPI. Like this : ![Authentication](/assets/2017-02-12-call-an-azure-ad-protected-api-in-xamarinuwp-apps/Authentication-1024x532.png)

That's it! As promised, we protected the WebAPI with no code whatsoever, as you can see when asking for the **Index** action of the **ValuesController**. ![Capture6](/assets/2017-02-12-call-an-azure-ad-protected-api-in-xamarinuwp-apps/Capture6-1024x641.png)

### Configuring Azure AD for the mobile apps

Our API is now protected by Azure AD and we can authenticate ourselves directly in the browser, but our mobile apps aren't ready yet to query the API. This is because Azure AD won't issue access tokens to unknown clients.

So there's a little more configuration to do.

First, we'll create a new **native** app entry into the Azure AD apps registry. I have named it "**woodenmoose-authsample-mobileapp**".

The RedirectUri will be used in the mobile apps to notify the success of the authentication. It can be any URI that you want. But if you're going for multitenancy authentication, Azure requires you follow a specific pattern : https://yourtenantname.com/what\_you\_want

I have set the value to "**https://mcnext.com/woodenmoose-authsample-mobileapp**"

![Capture5](/assets/2017-02-12-call-an-azure-ad-protected-api-in-xamarinuwp-apps/Capture5.png)

Next, we'll need to allow users connected to woodenmoose-authsample-mobileapp to access the web API. For this, we have the "Required permissions" panel in the native app entry. In it, we will select the api entry from the Azure AD.

Like this : ![DelegatedPermissions](/assets/2017-02-12-call-an-azure-ad-protected-api-in-xamarinuwp-apps/DelegatedPermissions-1024x507.png)

And then, we set the permission needed. In our case, simply "**Access woodenmoose-authsample-api**". ![Capture4](/assets/2017-02-12-call-an-azure-ad-protected-api-in-xamarinuwp-apps//Capture4-1024x225.png)

And finally, the last step in configuring Azure AD is to tell our api registration that if our native app registration want to issue a token to access the API, it's ok.

So we have to change our api registration and fill the "knownClientApplications" array in the manifest with the ApplicationId of our native app registration.

We have to use the manifest, because there's no other interface that will allow us to do so. ![Capture2](/assets/2017-02-12-call-an-azure-ad-protected-api-in-xamarinuwp-apps/Capture2-1024x495.png)

And that's it for the configuration! You're still there? Great, because now we're finally going to write some code!

### Implementing ADAL.NET

Now that all the configuration part on Azure is done, we can implement the authentication process in our apps before we can make a call to our API. For that, we will use [ADAL.NET](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet) (standing for Active Directory Authentication Library for dotNET) which is compatible with our 3 platforms.

ADAL.NET will take care of everything. It will display a screen where the user will log in, handle every error cases (wrong account, wrong password, account disabled, etc.) and it will handle the OAuth process. At the end, you receive the access token that you need to query the API. Really simple!

So, to use ADAL.NET we need to add to our core library the nuget package "[Microsoft.IdentityModel.Clients.ActiveDirectory](https://www.nuget.org/packages/Microsoft.IdentityModel.Clients.ActiveDirectory/)" (version 3.13.8 at the time of writing)

A few tips so NuGet will accept to install ADAL :

- If you're making a PCL project, you'll need to target the PCL Profile 7 (see image below for which platforms to select)
- If you're making a .NETStandard project, you'll need to target at least .NET Standard 1.4

The reason is that the ADAL package is only compatible with those versions.

Here are the required platforms if you're making a PCL. ![pcl-targets](/assets/2017-02-12-call-an-azure-ad-protected-api-in-xamarinuwp-apps/pcl-targets.png)

Once ADAL installed, we have access to the **AuthenticationContext** class. With it, we can ask for the user to log in.

```csharp
public class AuthenticationManager
{
    private string _authority;
    private string _resource;
    private string _clientId;
    private Uri _redirectUri;
    private static IPlatformParameters _platformParameters;
    private string _accessToken;
    
    public async Task LoginAsync()
    {
        var authContext = new AuthenticationContext(_authority);
        var authResult =
            await authContext.AcquireTokenAsync(
                _resource, _clientId, _redirectUri, _platformParameters);
                
        var connectedUser = authResult.UserInfo;
        _accessToken = authResult.AccessToken;
    }
    
    public static void SetParameters(IPlatformParameters platformParameters)
    {
        _platformParameters = platformParameters;
    }
}
```

Our LoginAsync method needs some parameters, let me explain them :

| Parameter | Description | Where to get it |
| --- | --- | --- |
| Authority | URL of the authentication authority | Two values are possible<br/>- If multitenancy is activated : **https://login.windows.net/common/**<br/>- Otherwise : **https://login.windows.net/yourtenantname.com/** |
| Resource* | Guid associated to the web app registration that we want to get a token for. |This value can be found in the properties of the web app registration under the name "Application ID" (Thanks Darren for correcting that) |
| ClientId | Guid associated to the native app registration that we want to authenticate against | This value can be found in the properties of the native app registration under the name "Application ID" |
| RedirectUri | URI that the OAuth process will call if the authentication is successful | This value is the one that we configured in the native app registration. It can be found in the "Redirect URIs" panel. |
| PlatformParameters | Platform specific interface that will change how ADAL.NET works | We need to pass an instance of a platform specific implementation of this interface. We will see how in the next parts |

_For the Resource parameter, it seems that some Azure AD will accept that you give an URI (App ID) instead of a Guid. But I recommend that you give the Guid anyway so there won't be any surprises._

So for my case, I'll use those values

- Authority = **https://login.windows.net/mcnext.com/**
- Resource = **c21a38b2-54bb-4027-b5e4-257f7d10f3f9**
- ClientId = **150526ad-75ee-44d5-a6e1-091ef0900d77**
- RedirectUri = **https://mcnext.com/woodenmoose-authsample-mobileapp**

We are still missing the PlatformParameters instance for ADAL.NET to work. Instanciating PlatformParameters is different for each platform so we will see how to do so for iOS, Android, and UWP.

#### Implementing iOS

The first step (after adding a reference to our core project) is to add a reference to the NuGet package "[Microsoft.IdentityModel.Clients.ActiveDirectory](https://www.nuget.org/packages/Microsoft.IdentityModel.Clients.ActiveDirectory/)" like in our core projet. The reason is that this package will come with a platform specific DLL (for instance Microsoft.IdentityModel.Clients.ActiveDirectory.Platform) which contains the iOS implementation of the IPlatformParameters interface.

Then all that remains is to give our core project an instance of the iOS PlatformParameters

```csharp
public partial class MainViewController : UIViewController
{
    public override void ViewDidLoad()
    {
        var platformParameters = new PlatformParameters(this);
        AuthenticationManager.SetParameters(platformParameters);
    }
}
```

ADAL.NET requires that you give it the instance of the currently displayed UIViewController, so it can resume our application once the authentication is complete. And we're done for iOS.

#### Implementing Android

Android is mostly the same as iOS, to a slight exception. We need to reference the NuGet package "[Microsoft.IdentityModel.Clients.ActiveDirectory](https://www.nuget.org/packages/Microsoft.IdentityModel.Clients.ActiveDirectory/)".

And we need to pass our Android implementation of PlatformParameters the instance of our currently displayed Activity.

```csharp
public class MainActivity : Activity
{
    protected override void OnCreate(Bundle bundle)
    {
        var platformParameters = new PlatformParameters(this);
        AuthenticationManager.SetParameters(platformParameters);
    }
    
    protected override void OnActivityResult(
        int requestCode, Result resultCode, Intent data)
    {
        base.OnActivityResult(requestCode, resultCode, data);
        
        AuthenticationAgentContinuationHelper
            .SetAuthenticationAgentContinuationEventArgs(requestCode, resultCode, data);
    }
}
```

Notice that I have overridden the OnActivityResult method. AuthenticationAgentContinuationHelper.SetAuthenticationAgentContinuationEventArgs needs to be called on OnActivityResult, otherwise ADAL.NET will not be able to resume our application on authentication success.

#### Implementing UWP

Like iOS and Android, UWP needs to reference the NuGet Package "[Microsoft.IdentityModel.Clients.ActiveDirectory](https://www.nuget.org/packages/Microsoft.IdentityModel.Clients.ActiveDirectory/)". If you try to build now, you may notice that the build fails. Just update your "Microsoft.NETCore.UniversalWindowsPlatform" package to version 5.2.2 minimum.

Unlike iOS and Android, UWP doesn't need to access the currently displayed Window. So instead, we can just set up our PlatformParameters in the App.xaml.cs file.

```csharp
public partial class App : Application
{
    public App()
    {
        this.InitializeComponent();
        this.Suspending += OnSuspending;
        
        var platformParameters = new PlatformParameters(PromptBehavior.Auto, false);
        AuthenticationManager.SetParameters(platformParameters);
    }
}
```

#### What about Xamarin.Forms?

Well, it works the same way as we saw for iOS, Android, and UWP. The only thing that differ is how to retrieve the current UIViewController and Activity. See the blog post from Xamarin "[Put Some Azure Active Directory in Xamarin.Forms](https://blog.xamarin.com/put-adal-xamarin-forms/)" for more information.

### Calling the API

Now that we finally have the access token, we can use it with HttpClient. To use it, we just have to set the Authorization header.

Like this:
```csharp
var client = new HttpClient();
client.DefaultRequestHeaders.Authorization =
    new AuthenticationHeaderValue("Bearer", _accessToken);

var result =
    await client.GetAsync(new Uri("https://woodenmoose-authsample-api.azurewebsites.net/api/values"));
```

That way, Azure AD will allow our request to be answered by our WebAPI.

### Conclusion

Setting up authentication and authorization can be quite a challenge, but ADAL and Azure Active Directory simplified it for us, especially since we're targeting 3 different platforms. Of course, we only saw the bare minimum. ADAL.NET has plenty more features to help you deliver secured and easy to use apps like token cache, refresh session, and so on. See the ADAL.NET GitHub for more information : [https://github.com/AzureAD/azure-activedirectory-library-for-dotnet](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet)

Also I have made a sample on Github demonstrating everything I showed in this post : [https://github.com/TimLariviere/WoodenMoose.Samples.Xamarin\_Authentication](https://github.com/TimLariviere/WoodenMoose.Samples.Xamarin_Authentication)

Happy coding!
