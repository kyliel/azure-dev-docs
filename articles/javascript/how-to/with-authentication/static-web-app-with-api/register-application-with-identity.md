---
title: "2: Register identity application"
titleSuffix: Azure Developer Center
description: In this article, register your identity application.
ms.topic: how-to
ms.date: 06/15/2021
ms.custom: devx-track-js
---

# How to register your identity application for a Static web app

In this article, learn how to register your identity application necessary to authenticate users with the Microsoft Authentication Library for React (MSAL React) and call an Azure service on behalf of the user. 

## Create Microsoft Identity provider app registration

Create your Microsoft Identity provider **app registration** to manage authentication. 

1. Sign in to the Azure portal, and [register an application](https://ms.portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredApps).
1. If your account is present in more than one Azure AD tenant, select your profile at the top-right corner in the menu on top of the page, and then **switch directory** to change your portal session to the desired Azure AD tenant.
1. In the **Name** section, enter a meaningful application name that will be displayed to users of the app, for example `msal-react-spa-YOUR-NAME`. 
   Make sure to always add your email name or first name to your resources if you are not the only user of your Azure subscription. 
2. For **Supported account types**, select **Accounts in this organizational directory only**. This supports a single tenant. 
3. In the **Redirect URI** section, select **Single-page application** in the combo-box and enter the following redirect URI: `http://localhost:3000/`.

4. Select **Register** to create the app registration for the authentication application.

## Get app registration settings

In the app's registration **Overview** screen, find and note the following:

* **Application (client) ID**
* **Directory (tenant) ID**

You'll need these settings later in your React and API configuration files. 

## Function API configurations your app registration 

Because this sample has an Azure Function API, which needs to call into an Azure service (Microsoft Graph) without direct login from a user, your app registration needs a few more settings so the Azure function can act on behalf of the user. To understand why these settings are required, the flow of information from authentication to use is helpful. 

The flow of information to act on behalf of a user includes:

* During user authentication: Requesting `access_as_user` scoped permission during the user's login, to act on behalf of the user for the `/api/hello` endpoint in the Azure Function. This is configured in the Azure portal for your Azure app registration and configured in your code in the React app's `/src/authConfig.js` file.
* Between React client and Azure Function API call: When the React client calls into the Azure Function API, the client passes the access token received during client user authentication. 
* In Azure Function API call: The Azure Function takes the token, validates the token, acquires a new token to act on behalf of the user, then passes that new token to the Azure service (Microsoft Graph).

### Create an app registration client secret

The secret is used to authenticate requests from your Function API to your app registration.

1. In the app's registration screen, select the **Certificates & secrets** blade to open the page where we can generate secrets and upload certificates.
1. In the **Client secrets** section, select **New client secret**, then enter a key description (for instance `react SWA/API app secret`),
1. Select one of the available key durations, then select **Add**.

    The generated key is displayed. 

1. Copy the generated value for use in the steps later. This key value will not be displayed again, and is not retrievable by any other means, so make sure to note it from the Azure portal before navigating to any other screen or blade.

    You'll need this client secret later in your API configuration file. 

### Create and expose an authentication endpoint for API

In order for the Azure Function API to act on behalf of your user when calling other APIs, you need to create an authentication endpoint for your Azure Function API. This requires creating an application ID URI, which is used in the React client MSAL config object to request access on the user's behalf to your Azure Function API.

1. In the app's registration screen, select the **Expose an API** blade to open the page where you can declare the parameters to expose this app as an API for which your web applications can obtain [access tokens](/azure/active-directory/develop/access-tokens).
1. Declare the unique [resource](/azure/active-directory/develop/v2-oauth2-auth-code-flow) URI that your client and API applications will use to obtain access tokens for this app registration: 
1. Select `Set` next to the **Application ID URI** to generate a URI that is unique for this app.
1. Accept the proposed Application ID URI (`api://{clientId}`) by selecting **Save**.
1. Copy the URI. You'll need this later in your React and API configuration files. 

## Configure app registration API scope

Add the `access_as_user` [scope](/azure/active-directory/develop/v2-oauth2-auth-code-flow#request-an-authorization-code) for the React client application to obtain an access token with that scope successfully. 

Add information to the user's **first authentication** to explain the scope request.

1. Still on the **Expose an API** blade, select **Add a scope**. 
1. For **Scope name**, use `access_as_user`.
1. Select **Admins and users** options for **Who can consent?**.
1. For **Admin consent display name** type `Access msal-react-function`.
1. For **Admin consent description** type `Allows the app to access msal-react-spa as the signed-in user.`
1. For **User consent display name** type `Access msal-react-spa`.
1. For **User consent description** type `Allow the application to access msal-react-spa on your behalf.`
1. Keep **State** as **Enabled**.
1. Select **Add scope** to save this scope.

## Configure API permissions for your app

Your application, both the React client and the Azure Function, are authorized to call APIs when they are granted permissions by users/admins as part of the consent process. The list of configured permissions should include all the permissions the application needs.

1. Select the **API permissions** blade. Notice the Microsoft Graph API's scope of **User.Read** is already added for you, by default. 
1. Select the **Add a permission**, then select **My APIs**.
1. From the list of APIs, select the API associated with your current app registration.
1. In the **Delegated permissions** section, select the **Access 'msal-react-spa'** in the list. 1. Select the **Add permissions** button at the bottom.

    The app registration is now configured for both your local React client and your local Azure Function app.

## Configure token version for single tenant app

1. Select the **Manifest** blade.
1. Find the key `accessTokenAcceptedVersion` and replace the existing value (null) with `2`.

    ```json
    "accessTokenAcceptedVersion": 2
    ```
1. Select **Save**.

## Review collected settings

The React client and the Azure Function both need to have configuration settings to use the MSAL SDK. 

You should have collected the following information from previous steps:

* Application (client) ID
* Directory (tenant) ID
* Client secret
* App ID URI

## Next steps

* [Create the local authenticated Static web app](configure-source-code-for-msal.md)