Daemon-CertificateCredential-DotNet
===================================

This sample is similar to Daemon-DotNet, except instead of the daemon using a password as a credential to authenticate with Azure AD, it uses a certificate.

For more information about how the protocols work in this scenario and other scenarios, see [Authentication Scenarios for Azure AD](http://go.microsoft.com/fwlink/?LinkId=394414).

## How To Run This Sample

To run this sample you will need:
- Visual Studio 2013
- The Azure AD module for Powershell
- The makecert.exe utility from the Windows SDK
- An Internet connection
- An Azure subscription (a free trial is sufficient)

Every Azure subscription has an associated Azure Active Directory tenant.  If you don't already have an Azure subscription, you can get a free subscription by signing up at [http://wwww.windowsazure.com](http://www.windowsazure.com).  All of the Azure AD features used by this sample are available free of charge.

### Step 1:  Clone or download this repository

From your shell or command line:

`git clone git@github.com:AzureADSamples/Daemon-CertificateCredential-DotNet.git`

### Step 2:  Create a user account in your Azure Active Directory tenant

If you already have a user account in your Azure Active Directory tenant, you can skip to the next step.  This sample will not work with a Microsoft account, so if you signed in to the Azure portal with a Microsoft account and have never created a user account in your directory before, you need to do that now.  If you create an account and want to use it to sign-in to the Azure portal, don't forget to add the user account as a co-administrator of your Azure subscription.

### Step 3:  Register the sample with your Azure Active Directory tenant

There are two projects in this sample.  Each needs to be separately registered in your Azure AD tenant.

#### Register the TodoListService web API

1. Sign in to the [Azure management portal](https://manage.windowsazure.com).
2. Click on Active Directory in the left hand nav.
3. Click the directory tenant where you wish to register the sample application.
4. Click the Applications tab.
5. In the drawer, click Add.
6. Click "Add an application my organization is developing".
7. Enter a friendly name for the application, for example "TodoListService", select "Web Application and/or Web API", and click next.
8. For the sign-on URL, enter the base URL for the sample, which is by default `https://localhost:44321`.
9. For the App ID URI, enter `https://<your_tenant_name>/TodoListService`, replacing `<your_tenant_name>` with the name of your Azure AD tenant.  Click OK to complete the registration.
10. While still in the Azure portal, click the Configure tab of your application.
11. Find the Client ID value and copy it aside, you will need this later when configuring your application.
12. Using the Manage Manifest button in the drawer, download the manifest file for the application.
13. Add a permission to the application by replacing the appPermissions section with the block of JSON below.  You will need to create a new GUID and replace the example permissionId GUID.
14. Using the Manage Manfiest button, upload the updated manifest file.  Save the configuration of the app.

NOTE:  Steps 13 and 14 are not necessary for this sample, but are included in case you want to compose this sample with client applications from other samples.

```JSON
"appPermissions": [
{
	"claimValue": "user_impersonation",
	"description": "Allow full access to the To Do List service on behalf of the signed-in user",
	"directAccessGrantTypes": [],
	"displayName": "Have full access to the To Do List service",
	"impersonationAccessGrantTypes": [
		{
			"impersonated": "User",
		    "impersonator": "Application"
		}
	],
	"isDisabled": false,
	"origin": "Application",
	"permissionId": "b69ee3c9-c40d-4f2a-ac80-961cd1534e40",
	"resourceScopeType": "Personal",
	"userConsentDescription": "Allow full access to the To Do service on your behalf",
	"userConsentDisplayName": "Have full access to the To Do service"
	}
],
```

#### Register the TodoListDaemonWithCert app

1. Sign in to the [Azure management portal](https://manage.windowsazure.com).
2. Click on Active Directory in the left hand nav.
3. Click the directory tenant where you wish to register the sample application.
4. Click the Applications tab.
5. In the drawer, click Add.
6. Click "Add an application my organization is developing".
7. Enter a friendly name for the application, for example "TodoListDaemonWithCert", select "Web Application and/or Web API", and click next.
8. Since this application is a daemon and not a web application, it doesn't have a sign-in URL or app ID URI.  For these two fields, enter "http://TodoListDaemonWithCert".
9. While still in the Azure portal, click the Configure tab of your application.
10. Find the Client ID value and copy it aside, you will need this later when configuring your application.

#### Create a self-signed certificate

To complete this step you will need the makecert.exe and pvk2pfx.exe utilities, which is included in the Windows SDK.  You may have installed the Windows SDK with Visual Studio 2013, in which case you will find it if you search your computer for makecert.exe.  If not, you can download the SDK [here](http://msdn.microsoft.com/en-US/windows/desktop/aa904949).  You only need to install the Windows Software Development Kit part of the SDK.  You can find more information about the makecert.exe utility [here](http://msdn.microsoft.com/en-us/library/bfsktky3(v=vs.110).aspx).

Run makecert.exe with the following parameters to create a self-signed certificate on your computer:

```
makecert -r -pe -n "CN=TodoListDaemonWithCert" -ss My -len 2048 TodoListDaemonWithCert.cer
```

#### Add the certificate as a key for the TodoListDaemonWithCert application in Azure AD

To associate the certificate credential with the TodoListDaemonWithCert app object in Azure AD, you will need the Azure AD module for Powershell.  You can find instructions on how to download the module [here](http://technet.microsoft.com/en-us/library/jj151815.aspx).

Once you have downloaded and installed the module, open a Powershell command window and execute the following steps:

```
PS C:\windows\system32> connect-msolservice
PS C:\windows\system32> $cer = New-Object System.Security.Cryptography.X509Certificates.X509Certificate
PS C:\windows\system32> $cer.Import("<Path to Certificate (.cer) file created in prior step>")
PS C:\windows\system32> $binCert = $cer.GetRawCertData()
PS C:\windows\system32> $credValue = [System.Convert]::ToBase64String($binCert);
PS C:\windows\system32> New-MsolServicePrincipalCredential -AppPrincipalId "<Client ID from prior step>" -Type asymmetric -Value $credValue -StartDate $cer.GetEffectiveDateString() -EndDate $cer.GetExpirationDateString() -Usage verify
```

You can verify the certificate was successfully added to the application registration by running this command:

```
PS C:\windows\system32> Get-MsolServicePrincipalCredential –ServicePrincipalName “http://TodoListDaemonWithCert”
```

### Step 4:  Configure the sample to use your Azure AD tenant

#### Configure the TodoListService project

1. Open the solution in Visual Studio 2013.
2. Open the `web.config` file.
3. Find the app key `ida:Tenant` and replace the value with your AAD tenant name.
4. Find the app key `ida:Audience` and replace the value with the App ID URI you registered earlier, for example `https://<your_tenant_name>/TodoListService`.
5. Find the app key `ida:ClientId` and replace the value with the Client ID for the TodoListService from the Azure portal.

#### Configure the TodoListDaemon project

1. Open `app.config'.
2. Find the app key `ida:Tenant` and replace the value with your AAD tenant name.
3. Find the app key `ida:ClientId` and replace the value with the Client ID for the TodoListClient from the Azure portal.
4. Find the app key `ida:CertName` and replace the value with the subject name of the self-signed certificate you created, e.g. "CN=TodoListDaemonWithCert".
5. Find the app key `todo:TodoListResourceId` and replace the value with the  App ID URI of the TodoListService, for example `https://<your_tenant_name>/TodoListService`
6. Find the app key `todo:TodoListBaseAddress` and replace the value with the base address of the TodoListService project.

### Step 5:  Trust the IIS Express SSL certificate

Since the web API is SSL protected, the client of the API (the web app) will refuse the SSL connection to the web API unless it trusts the API's SSL certificate.  Use the following steps in Windows Powershell to trust the IIS Express SSL certificate.  You only need to do this once.  If you fail to do this step, calls to the TodoListService will always throw an unhandled exception where the inner exception message is:

"The underlying connection was closed: Could not establish trust relationship for the SSL/TLS secure channel."

To configure your computer to trust the IIS Express SSL certificate, begin by opening a Windows Powershell command window as Administrator.

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

### Step 6:  Run the sample

Clean the solution, rebuild the solution, and run it.  You might want to go into the solution properties and set both projects as startup projects, with the service project starting first.

The daemon will add items to its To Do list and then read them back.

## How To Deploy This Sample to Azure

Coming soon.

## About The Code

Coming soon.

## How To Recreate This Sample

First, in Visual Studio 2013 create an empty solution to host the  projects.  Then, follow these steps to create each project.

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
6. In `app.config` create keys for `ida:AADInstance`, `ida:Tenant`, `ida:ClientId`, `ida:CertName`, `todo:TodoListResourceId`, and `todo:TodoListBaseAddress` and set them accordingly.  For the public Azure cloud, the value of `ida:AADInstance` is `https://login.windows.net/{0}`.

Finally, in the properties of the solution itself, set both projects as startup projects.