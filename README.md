---
services: active-directory
platforms: dotnet
author: dstrockis,jmprieur
---

# Authenticating to Azure AD in daemon apps with certificates
![](https://identitydivision.visualstudio.com/_apis/public/build/definitions/a7934fdd-dcde-4492-a406-7fad6ac00e17/30/badge)

This sample is similar to [Daemon-DotNet](https://github.com/Azure-Samples/active-directory-dotnet-daemon), except instead of the daemon using a password as a credential to authenticate with Azure AD, it uses a certificate.

For more information about how the protocols work in this scenario and other scenarios, see [Authentication Scenarios for Azure AD](http://go.microsoft.com/fwlink/?LinkId=394414) and [Service to service calls using client credentials](https://github.com/Microsoft/azure-docs/blob/master/articles/active-directory/develop/active-directory-protocols-oauth-service-to-service.md)

> Looking for previous versions of this code sample? Check out the tags on the [releases](../../releases) GitHub page.

## How to Run this sample

To run this sample, you will need:
- Visual Studio 2013 or above
- An Internet connection
- An Azure Active Directory (Azure AD) tenant. For more information on how to get an Azure AD tenant, please see [How to get an Azure AD tenant](https://azure.microsoft.com/en-us/documentation/articles/active-directory-howto-tenant/) 

### Step 1:  Clone or download this repository

You can clone this repository from Visual Studio. Alternatively, from your shell or command line, use:

`git clone https://github.com/Azure-Samples/active-directory-dotnet-daemon-certificate-credential.git`

### Step 2:  Register the sample with your Azure Active Directory tenant and configure the code accordingly

If you want to understand in more depth what needs to be done in the Azure portal, and how to change the code, please have a look at [Manual-Configuration-Steps.md](./Manual-Configuration-Steps.md). Otherwise, you can use a PowerShell Script. The steps are the following:

#### Find your tenant ID
 1. Sign in to the [Azure portal](https://portal.azure.com).
 2. On the top bar, click on your account and under the **Directory** list, choose the Active Directory tenant where you wish to register your application.
 3. Click on **More Services** in the left hand nav, and choose **Azure Active Directory**.
 4. Click on **Properties** and copy the value of the **Directory ID** property to the clipboard. This is your tenant ID. We'll need it in the next step. 

#### Run the PowerShell script

 1. In the Visual Studio Solution explorer, right click on the solution file (.lsn) and choose **Open Folder in File Explorer**. The windows file explorer opens at the location of the solution
 2. In the file explorer right click on the `Configure.ps1` file and choose **Edit**, this starts PowerShell ISE and loads the PowerShell Script. You can read the comments at the beginning of the script (they provide full explanations), as well as the bottom of the script.
 3. In PowerShell ISE, Press the green arrow **Run script** to start the script
 4. At the prompt ("tenantId:") paste the tenant ID that you previously copied from the Azure portal
 5. When requested, sign-in as a user who has permissions to create applications in the AAD tenant. 
        
    The script executes and provisions the AAD applications (If you look at the AAD applications in the portal after that the script has run, you'll have two additional applications). The script also updates two configuration files in the Visual Studio solution (`TodoListDaemonWithCert\App.Config` and `TodoListService\Web.Config`)

6. If you intend to clean up the azure AD applications from the Azure AD tenant after running the sample, keep the PowerShell ISE window opened.

### Step 3:  Trust the IIS Express SSL certificate

Since the web API is SSL protected, the client of the API (the web app) will refuse the SSL connection to the web API unless it trusts the API's SSL certificate.  Use the following steps in Windows PowerShell to trust the IIS Express SSL certificate.  You only need to do this once.  If you fail to do this step, calls to the TodoListService will always throw an unhandled exception where the inner exception message is:

"The underlying connection was closed: Could not establish trust relationship for the SSL/TLS secure channel."

> Recent versions of Visual Studio will propose themselves to trust the IIS Express SSL certificate. If this is not the case for your installation
please know that you will need to do this step only once.


To configure your computer to trust the IIS Express SSL certificate, begin by opening a Windows PowerShell command window as Administrator.

Query your personal certificate store to find the thumbprint of the certificate for `CN=localhost`:

```
PS C:\windows\system32> dir Cert:\LocalMachine\My


    Directory: Microsoft.PowerShell.Security\Certificate::LocalMachine\My


Thumbprint                                Subject
----------                                -------
C24798908DA71693C1053F42A462327543B38042  CN=localhost
```

Next, add the certificate to the Trusted Root store:

```
PS C:\windows\system32> $cert = (get-item cert:\LocalMachine\My\C24798908DA71693C1053F42A462327543B38042)
PS C:\windows\system32> $store = (get-item cert:\Localmachine\Root)
PS C:\windows\system32> $store.Open("ReadWrite")
PS C:\windows\system32> $store.Add($cert)
PS C:\windows\system32> $store.Close()
```

You can verify the certificate is in the Trusted Root store by running this command:

`PS C:\windows\system32> dir Cert:\LocalMachine\Root`

### Step 4:  Run the sample

Clean the solution, rebuild the solution, and run it.  You might want to go into the solution properties and set both projects as startup projects, with the service project starting first. To do this you can for instance:
 1. Right click on the solution in the solution explorer and choose **Set Startup projects** from the context menu.
 2. choose **Multiple startup projects**
  - TodoListDaemonWithCert: **Start**
  - TodoListService: Start **Start without debugging**
 3. In VS press the **start** button: a web window appears running the service and a console application runs the dameon application under debugger. you can set breakpoints to understand the call to ADAL.NET.

The daemon will add items to its To Do list and then read them back.

### Step 4:  Clean up the applications in the Azure AD tenant
When you are done with running and understanding the sample, if you want to remove your Applications from AD and if you have keep your PowerShell ISE window opened, just run, in the same PowerShell ISE window:

``CleanUp($apps)``

If you do that you also probably want to undo the changes in the `App.config` and `Web.Config`


## How to deploy this sample to Azure

TBD.

## About the Code

If you've looked at the code in this sample and are wondering how authorization works, you're not alone.  See [this Stack Overflow question](https://stackoverflow.com/questions/34415348/azure-active-directory-daemon-client-using-certificates/).  The TodoList Service in this solution simply validates that the client was able to authenticate against the tenant that the service is configured to work with.  Effectively, any application in that tenant will be able to use the service.

## How to recreate this sample

First, in Visual Studio 2013 (or above) create an empty solution to host the  projects.  Then, follow these steps to create each project.

### Creating the TodoListService Project

1. In the solution, create a new ASP.Net MVC web API project called TodoListService and while creating the project, click the Change Authentication button, select Organizational Accounts, Cloud - Single Organization, enter the name of your Azure AD tenant, and set the Access Level to Single Sign On.  You will be prompted to sign-in to your Azure AD tenant.  NOTE:  You must sign-in with a user that is in the tenant; you cannot, during this step, sign-in with a Microsoft account.
2. In the `Models` folder add a new class called `TodoItem.cs`.  Copy the implementation of TodoItem from this sample into the class.
3. Add a new, empty, Web API 2 controller called `TodoListController`.
4. Copy the implementation of the TodoListController from this sample into the controller.  Don't forget to add the `[Authorize]` attribute to the class.
5. In `TodoListController` resolving missing references by adding `using` statements for `System.Collections.Concurrent`, `TodoListService.Models`, `System.Security.Claims`.

### Creating the TodoListDaemon Project

1. In the solution, create a new Windows --> Console Application called TodoListDaemon.
2. Add the (stable) Active Directory Authentication Library (ADAL) NuGet, Microsoft.IdentityModel.Clients.ActiveDirectory, version 1.0.3 (or higher) to the project.
3. Add  assembly references to `System.Net.Http`, `System.Web.Extensions`, and `System.Configuration`.
4. Add a new class to the project called `TodoItem.cs`.  Copy the code from the sample project file of same name into this class, completely replacing the code in the file in the new project.
5. Copy the code from `Program.cs` in the sample project into the file of same name in the new project, completely replacing the code in the file in the new project.
6. In `app.config` create keys for `ida:AADInstance`, `ida:Tenant`, `ida:ClientId`, `ida:CertName`, `todo:TodoListResourceId`, and `todo:TodoListBaseAddress` and set them accordingly.  For the public Azure cloud, the value of `ida:AADInstance` is `https://login.microsoftonline.com/{0}`.

Finally, in the properties of the solution itself, set both projects as startup projects.
