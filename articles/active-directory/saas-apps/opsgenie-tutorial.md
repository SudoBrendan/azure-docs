---
title: 'Tutorial: Azure Active Directory integration with OpsGenie | Microsoft Docs'
description: Learn how to configure single sign-on between Azure Active Directory and OpsGenie.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/28/2020
ms.author: jeedes
---

# Tutorial: Azure Active Directory single sign-on (SSO) integration with OpsGenie

In this tutorial, you'll learn how to integrate OpsGenie with Azure Active Directory (Azure AD). When you integrate OpsGenie with Azure AD, you can:

* Control in Azure AD who has access to OpsGenie.
* Enable your users to be automatically signed-in to OpsGenie with their Azure AD accounts.
* Manage your accounts in one central location - the Azure portal.

## Prerequisites

To get started, you need the following items:

* An Azure AD subscription. If you don't have a subscription, you can get a [free account](https://azure.microsoft.com/free/).
* OpsGenie single sign-on (SSO) enabled subscription.

## Scenario description

In this tutorial, you configure and test Azure AD SSO in a test environment.

* OpsGenie supports **IDP** initiated SSO

## Adding OpsGenie from the gallery

To configure the integration of OpsGenie into Azure AD, you need to add OpsGenie from the gallery to your list of managed SaaS apps.

1. Sign in to the Azure portal using either a work or school account, or a personal Microsoft account.
1. On the left navigation pane, select the **Azure Active Directory** service.
1. Navigate to **Enterprise Applications** and then select **All Applications**.
1. To add new application, select **New application**.
1. In the **Add from the gallery** section, type **OpsGenie** in the search box.
1. Select **OpsGenie** from results panel and then add the app. Wait a few seconds while the app is added to your tenant.

## Configure and test Azure AD SSO for OpsGenie

Configure and test Azure AD SSO with OpsGenie using a test user called **B.Simon**. For SSO to work, you need to establish a link relationship between an Azure AD user and the related user in OpsGenie.

To configure and test Azure AD SSO with OpsGenie, perform the following steps:

1. **[Configure Azure AD SSO](#configure-azure-ad-sso)** - to enable your users to use this feature.
    1. **[Create an Azure AD test user](#create-an-azure-ad-test-user)** - to test Azure AD single sign-on with B.Simon.
    1. **[Assign the Azure AD test user](#assign-the-azure-ad-test-user)** - to enable B.Simon to use Azure AD single sign-on.
1. **[Configure OpsGenie SSO](#configure-opsgenie-sso)** - to configure the single sign-on settings on application side.
    1. **[Create OpsGenie test user](#create-opsgenie-test-user)** - to have a counterpart of B.Simon in OpsGenie that is linked to the Azure AD representation of user.
1. **[Test SSO](#test-sso)** - to verify whether the configuration works.

## Configure Azure AD SSO

Follow these steps to enable Azure AD SSO in the Azure portal.

1. In the Azure portal, on the **OpsGenie** application integration page, find the **Manage** section and select **single sign-on**.
1. On the **Select a single sign-on method** page, select **SAML**.
1. On the **Set up single sign-on with SAML** page, click the edit/pen icon for **Basic SAML Configuration** to edit the settings.

   ![Edit Basic SAML Configuration](common/edit-urls.png)

1. On the **Basic SAML Configuration** section, perform the following steps:

    a. In the **Identifier** text box, type a URL using the following pattern:
    `https://app.opsginie.com/auth/saml/<UNIQUEID>`

    b. In the **Reply URL** text box, type a URL using the following pattern:
    `https://app.opsginie.com/auth/saml?id=<UNIQUEID>`

    > [!NOTE]
	> These values are not real. Update these values with the actual Identifier and Reply URL, which is explained later in this tutorial.

5. On the **Set up Single Sign-On with SAML** page, In the **SAML Signing Certificate** section, click copy button to copy **App Federation Metadata Url** and save it on your computer.

	![The Certificate download link](common/copy-metadataurl.png)

1. On the **Set up OpsGenie** section, copy the appropriate URL(s) based on your requirement.

	![Copy configuration URLs](common/copy-configuration-urls.png)

### Create an Azure AD test user

In this section, you'll create a test user in the Azure portal called B.Simon.

1. From the left pane in the Azure portal, select **Azure Active Directory**, select **Users**, and then select **All users**.
1. Select **New user** at the top of the screen.
1. In the **User** properties, follow these steps:
   1. In the **Name** field, enter `B.Simon`.  
   1. In the **User name** field, enter the username@companydomain.extension. For example, `B.Simon@contoso.com`.
   1. Select the **Show password** check box, and then write down the value that's displayed in the **Password** box.
   1. Click **Create**.

### Assign the Azure AD test user

In this section, you'll enable B.Simon to use Azure single sign-on by granting access to OpsGenie.

1. In the Azure portal, select **Enterprise Applications**, and then select **All applications**.
1. In the applications list, select **OpsGenie**.
1. In the app's overview page, find the **Manage** section and select **Users and groups**.

1. Select **Add user**, then select **Users and groups** in the **Add Assignment** dialog.

1. In the **Users and groups** dialog, select **B.Simon** from the Users list, then click the **Select** button at the bottom of the screen.
1. If you are expecting a role to be assigned to the users, you can select it from the **Select a role** dropdown. If no role has been set up for this app, you see "Default Access" role selected.
1. In the **Add Assignment** dialog, click the **Assign** button.

## Configure OpsGenie SSO

1. To automate the configuration within OpsGenie, you need to install **My Apps Secure Sign-in browser extension** by clicking **Install the extension**.

	![My apps extension](common/install-myappssecure-extension.png)

2. After adding extension to the browser, click on **Set up OpsGenie** will direct you to the OpsGenie application. From there, provide the admin credentials to sign into OpsGenie. The browser extension will automatically configure the application for you and automate steps 3-7.

	![Setup configuration](common/setup-sso.png)

3. If you want to setup OpsGenie manually, in a different web browser window, sign in to your OpsGenie company site as an administrator.

2. Click **Settings**, and then click the **Single Sign On** tab.
   
    ![OpsGenie Single Sign-On](./media/opsgenie-tutorial/tutorial-opsgenie-06.png)

3. To enable SSO, select **Enabled**.
   
    ![Screenshot that shows the "Enabled" checkbox selected.](./media/opsgenie-tutorial/tutorial-opsgenie-07.png) 

4. In the **Provider** section, click the **Azure Active Directory** tab.
   
    ![Screenshot that shows the "Provider" section with the "Azure Active Directory" tab selected.](./media/opsgenie-tutorial/tutorial-opsgenie-08.png) 

5. On the Azure Active Directory dialog page, perform the following steps:
   
    ![Screenshot that shows the "Single sign-on" section with the "Enable single sign-on" toggle, "S A M L 2.0 Endpoint", and "Metadata U R L".](./media/opsgenie-tutorial/tutorial-opsgenie-09.png)
	
    a. Copy the **App ID URI** value and paste it into **Identifier (Entity ID)** textbox in the **Basic SAML Configuration** section in the Azure portal.

    a. Copy the **Reply URL** value and paste it into **Reply URL** textbox in the **Basic SAML Configuration** section in the Azure portal.

	a. In the **SAML 2.0 Endpoint** textbox, paste **Login URL**value which you have copied from the Azure portal.
	
	b. In the **Metadata Url:** textbox, paste **App Federation Metadata Url** value which you have copied from the Azure portal.
    
    c. To enable SSO, turn on the **Enable single sign-on** toggle.

    d. Click **Apply SSO settings**.

### Create OpsGenie test user

The objective of this section is to create a user called B.Simon in OpsGenie. 

1. In a web browser window, sign into your OpsGenie tenant as an administrator.

2. Navigate to Users list by clicking **Users** in left panel.
   
    ![OpsGenie Settings](./media/opsgenie-tutorial/tutorial-opsgenie-10.png) 

3. Click **Add User**.

4. On the **Add User** dialog, perform the following steps:
   
    ![Screenshot that shows the "Add User" dialog with the "Email" and "Full name" text boxes highlighted, and the "Save" button selected.](./media/opsgenie-tutorial/tutorial-opsgenie-11.png)
   
    a. In the **Email** textbox, type the email address of B.Simon addressed in Azure Active Directory.
   
    b. In the **Full Name** textbox, type **B.Simon**.
   
    c. Click **Save**. 

> [!NOTE]
> B.Simon gets an email with instructions for setting up their profile.

## Test SSO 

In this section, you test your Azure AD single sign-on configuration with following options.

* Click on Test this application in Azure portal and you should be automatically signed in to the OpsGenie for which you set up the SSO

* You can use Microsoft My Apps. When you click the OpsGenie tile in the My Apps, you should be automatically signed in to the OpsGenie for which you set up the SSO. For more information about the My Apps, see [Introduction to the My Apps](https://support.microsoft.com/account-billing/sign-in-and-start-apps-from-the-my-apps-portal-2f3b1bae-0e5a-4a86-a33e-876fbd2a4510).

## Next steps

* Once you configure OpsGenie you can enforce session control, which protects exfiltration and infiltration of your organization’s sensitive data in real time. Session control extends from Conditional Access. [Learn how to enforce session control with Microsoft Defender for Cloud Apps](/cloud-app-security/proxy-deployment-any-app).
