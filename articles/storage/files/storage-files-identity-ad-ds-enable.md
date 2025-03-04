---
title: Enable AD DS authentication to Azure file shares
description: Learn how to enable Active Directory Domain Services authentication over SMB for Azure file shares. Your domain-joined Windows virtual machines can then access Azure file shares by using AD DS credentials. 
author: khdownie
ms.service: storage
ms.subservice: files
ms.topic: how-to
ms.date: 01/14/2022
ms.author: kendownie 
ms.custom: devx-track-azurepowershell
---

# Part one: enable AD DS authentication for your Azure file shares 

Before you enable Active Directory Domain Services (AD DS) authentication, make sure you've read the [overview article](storage-files-identity-auth-active-directory-enable.md) to understand the supported scenarios and requirements.

This article describes the process required for enabling Active Directory Domain Services (AD DS) authentication on your storage account. After enabling the feature, you must configure your storage account and your AD DS, to use AD DS credentials for authenticating to your Azure file share. To enable AD DS authentication over SMB for Azure file shares, you need to register your storage account with AD DS and then set the required domain properties on the storage account.

To register your storage account with AD DS, create an account representing it in your AD DS. You can think of this process as if it were like creating an account representing an on-premises Windows file server in your AD DS. When the feature is enabled on the storage account, it applies to all new and existing file shares in the account.

## Applies to
| File share type | SMB | NFS |
|-|:-:|:-:|
| Standard file shares (GPv2), LRS/ZRS | ![Yes](../media/icons/yes-icon.png) | ![No](../media/icons/no-icon.png) |
| Standard file shares (GPv2), GRS/GZRS | ![Yes](../media/icons/yes-icon.png) | ![No](../media/icons/no-icon.png) |
| Premium file shares (FileStorage), LRS/ZRS | ![Yes](../media/icons/yes-icon.png) | ![No](../media/icons/no-icon.png) |

## Option one (recommended): Use AzFilesHybrid PowerShell module

The cmdlets in the AzFilesHybrid PowerShell module make the necessary modifications and enables the feature for you. Since some parts of the cmdlets interact with your on-premises AD DS, we explain what the cmdlets do, so you can determine if the changes align with your compliance and security policies, and ensure you have the proper permissions to execute the cmdlets. Though we recommend using AzFilesHybrid module, if you are unable to do so, we provide the steps so that you may perform them manually.

### Download AzFilesHybrid module

- If you don't have [.NET Framework 4.7.2](https://dotnet.microsoft.com/download/dotnet-framework/net472) installed, install it now. It is required for the module to import successfully.
- [Download and unzip the AzFilesHybrid module (GA module: v0.2.0+)](https://github.com/Azure-Samples/azure-files-samples/releases) Note that AES 256 kerberos encryption is supported on v0.2.2 or above. If you have enabled the feature with a AzFilesHybrid version below v0.2.2 and want to update to support AES 256 Kerberos encryption, please refer to [this article](./storage-troubleshoot-windows-file-connection-problems.md#azure-files-on-premises-ad-ds-authentication-support-for-aes-256-kerberos-encryption).
- Install and execute the module in a device that is domain joined to on-premises AD DS with AD DS credentials that have permissions to create a service logon account or a computer account in the target AD.
-  Run the script using an on-premises AD DS credential that is synced to your Azure AD. The on-premises AD DS credential must have either **Owner** or **Contributor** Azure role on the storage account.

### Run Join-AzStorageAccount

The `Join-AzStorageAccount` cmdlet performs the equivalent of an offline domain join on behalf of the specified storage account. The script uses the cmdlet to create a [computer account](/windows/security/identity-protection/access-control/active-directory-accounts#manage-default-local-accounts-in-active-directory) in your AD domain. If for whatever reason you cannot use a computer account, you can alter the script to create a [service logon account](/windows/win32/ad/about-service-logon-accounts) instead. If you choose to run the command manually, you should select the account best suited for your environment.

The AD DS account created by the cmdlet represents the storage account. If the AD DS account is created under an organizational unit (OU) that enforces password expiration, you must update the password before the maximum password age. Failing to update the account password before that date results in authentication failures when accessing Azure file shares. To learn how to update the password, see [Update AD DS account password](storage-files-identity-ad-ds-update-password.md).

Replace the placeholder values with your own in the parameters below before executing it in PowerShell.
> [!IMPORTANT]
> The domain join cmdlet will create an AD account to represent the storage account (file share) in AD. You can choose to register as a computer account or service logon account, see [FAQ](./storage-files-faq.md#security-authentication-and-access-control) for details. For computer accounts, there is a default password expiration age set in AD at 30 days. Similarly, the service logon account may have a default password expiration age set on the AD domain or Organizational Unit (OU).
> For both account types, we recommend you check the password expiration age configured in your AD environment and plan to [update the password of your storage account identity](storage-files-identity-ad-ds-update-password.md) of the AD account before the maximum password age. You can consider [creating a new AD Organizational Unit (OU) in AD](/powershell/module/activedirectory/new-adorganizationalunit) and disabling password expiration policy on [computer accounts](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/jj852252(v=ws.11)) or service logon accounts accordingly. 

```PowerShell
# Change the execution policy to unblock importing AzFilesHybrid.psm1 module
Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Scope CurrentUser

# Navigate to where AzFilesHybrid is unzipped and stored and run to copy the files into your path
.\CopyToPSPath.ps1 

# Import AzFilesHybrid module
Import-Module -Name AzFilesHybrid

# Login with an Azure AD credential that has either storage account owner or contributer Azure role assignment
# If you are logging into an Azure environment other than Public (ex. AzureUSGovernment) you will need to specify that.
# See https://docs.microsoft.com/azure/azure-government/documentation-government-get-started-connect-with-ps
# for more information.
Connect-AzAccount

# Define parameters, $StorageAccountName currently has a maximum limit of 15 characters
$SubscriptionId = "<your-subscription-id-here>"
$ResourceGroupName = "<resource-group-name-here>"
$StorageAccountName = "<storage-account-name-here>"
$DomainAccountType = "<ComputerAccount|ServiceLogonAccount>" # Default is set as ComputerAccount
# If you don't provide the OU name as an input parameter, the AD identity that represents the storage account is created under the root directory.
$OuDistinguishedName = "<ou-distinguishedname-here>"
# Specify the encryption agorithm used for Kerberos authentication. Default is configured as "'RC4','AES256'" which supports both 'RC4' and 'AES256' encryption.
$EncryptionType = "<AES256|RC4|AES256,RC4>"

# Select the target subscription for the current session
Select-AzSubscription -SubscriptionId $SubscriptionId 

# Register the target storage account with your active directory environment under the target OU (for example: specify the OU with Name as "UserAccounts" or DistinguishedName as "OU=UserAccounts,DC=CONTOSO,DC=COM"). 
# You can use to this PowerShell cmdlet: Get-ADOrganizationalUnit to find the Name and DistinguishedName of your target OU. If you are using the OU Name, specify it with -OrganizationalUnitName as shown below. If you are using the OU DistinguishedName, you can set it with -OrganizationalUnitDistinguishedName. You can choose to provide one of the two names to specify the target OU.
# You can choose to create the identity that represents the storage account as either a Service Logon Account or Computer Account (default parameter value), depends on the AD permission you have and preference. 
# Run Get-Help Join-AzStorageAccountForAuth for more details on this cmdlet.

Join-AzStorageAccount `
        -ResourceGroupName $ResourceGroupName `
        -StorageAccountName $StorageAccountName `
        -DomainAccountType $DomainAccountType `
        -OrganizationalUnitDistinguishedName $OuDistinguishedName `
        -EncryptionType $EncryptionType

#Run the command below if you want to enable AES 256 authentication. If you plan to use RC4, you can skip this step.
Update-AzStorageAccountAuthForAES256 -ResourceGroupName $ResourceGroupName -StorageAccountName $StorageAccountName

#You can run the Debug-AzStorageAccountAuth cmdlet to conduct a set of basic checks on your AD configuration with the logged on AD user. This cmdlet is supported on AzFilesHybrid v0.1.2+ version. For more details on the checks performed in this cmdlet, see Azure Files Windows troubleshooting guide.
Debug-AzStorageAccountAuth -StorageAccountName $StorageAccountName -ResourceGroupName $ResourceGroupName -Verbose
```

## Option two: Manually perform the enablement actions

If you have already executed the `Join-AzStorageAccount` script above successfully, go to the [Confirm the feature is enabled](#confirm-the-feature-is-enabled) section. You don't need to perform the following manual steps.

### Check the environment

First, you must check the state of your environment. Specifically, you must check if [Active Directory PowerShell](/powershell/module/activedirectory/) is installed, and if the shell is being executed with administrator privileges. Then check to see if the [Az.Storage 2.0 module (or newer)](https://www.powershellgallery.com/packages/Az.Storage/2.0.0) is installed, and install it if it isn't. After completing those checks, check your AD DS to see if there is either a [computer account](/windows/security/identity-protection/access-control/active-directory-accounts#manage-default-local-accounts-in-active-directory) (default) or [service logon account](/windows/win32/ad/about-service-logon-accounts) that has already been created with SPN/UPN as "cifs/your-storage-account-name-here.file.core.windows.net". If the account doesn't exist, create one as described in the following section.

### Create an identity representing the storage account in your AD manually

To create this account manually, create a new Kerberos key for your storage account. Then, use that Kerberos key as the password for your account with the PowerShell cmdlets below. This key is only used during setup and cannot be used for any control or data plane operations against the storage account. 

```PowerShell
# Create the Kerberos key on the storage account and get the Kerb1 key as the password for the AD identity to represent the storage account
$ResourceGroupName = "<resource-group-name-here>"
$StorageAccountName = "<storage-account-name-here>"

New-AzStorageAccountKey -ResourceGroupName $ResourceGroupName -Name $StorageAccountName -KeyName kerb1
Get-AzStorageAccountKey -ResourceGroupName $ResourceGroupName -Name $StorageAccountName -ListKerbKey | where-object{$_.Keyname -contains "kerb1"}
```

Once you have that key, create either a service or computer account under your OU. Use the following specification (remember to replace the example text with your storage account name):

SPN: "cifs/your-storage-account-name-here.file.core.windows.net"
Password: Kerberos key for your storage account.

If your OU enforces password expiration, you must update the password before the maximum password age to prevent authentication failures when accessing Azure file shares. See [Update the password of your storage account identity in AD](storage-files-identity-ad-ds-update-password.md) for details.

Keep the SID of the newly created identity, you'll need it for the next step. The identity you've created that represent the storage account doesn't need to be synced to Azure AD.

### Enable the feature on your storage account

Modify the following command to include configuration details for the domain properties in the following command, then run it to enable the feature. The storage account SID required in the following command is the SID of the identity you created in your AD DS in [the previous section](#create-an-identity-representing-the-storage-account-in-your-ad-manually).

```PowerShell
# Set the feature flag on the target storage account and provide the required AD domain information
Set-AzStorageAccount `
        -ResourceGroupName "<your-resource-group-name-here>" `
        -Name "<your-storage-account-name-here>" `
        -EnableActiveDirectoryDomainServicesForFile $true `
        -ActiveDirectoryDomainName "<your-domain-dns-root-here>" `
        -ActiveDirectoryNetBiosDomainName "<your-domain-dns-root-here>" `
        -ActiveDirectoryForestName "<your-forest-name-here>" `
        -ActiveDirectoryDomainGuid "<your-guid-here>" `
        -ActiveDirectoryDomainsid "<your-domain-sid-here>" `
        -ActiveDirectoryAzureStorageSid "<your-storage-account-sid>"
```

#### (Optional) Enable AES256 encryption

To enable AES 256 encryption, follow the steps in this section. If you plan to use RC4, skip this section.

The domain object that represents your storage account must meet the following requirements:
- The storage account name cannot exceed 15 characters.
- The domain object must be created as a computer object in the on-premises AD domain.
- Except for the trailing '$', the storage account name must be the same as the computer object's SamAccountName.

If your domain object doesn't meet those requirements, delete it and create a new domain object that does.

Replace `<domain-object-identity>` and `<domain-name>` with your values, then use the following command to configure AES256 support: 

```powershell
Set-ADComputer -Identity <domain-object-identity> -Server <domain-name> -KerberosEncryptionType "AES256"
```

After you've ran that command, replace `<domain-object-identity>` in the following script with your value, then run the script to refresh your domain object password:

```powershell
$KeyName = "kerb1" # Could be either the first or second kerberos key, this script assumes we're refreshing the first
$KerbKeys = New-AzStorageAccountKey -ResourceGroupName $ResourceGroupName -Name $StorageAccountName -KeyName $KeyName
$KerbKey = $KerbKeys | Where-Object {$_.KeyName -eq $KeyName} | Select-Object -ExpandProperty Value
$NewPassword = ConvertTo-SecureString -String $KerbKey -AsPlainText -Force

Set-ADAccountPassword -Identity <domain-object-identity> -Reset -NewPassword $NewPassword
```

### Debugging

You can run the Debug-AzStorageAccountAuth cmdlet to conduct a set of basic checks on your AD configuration with the logged on AD user. This cmdlet is supported on AzFilesHybrid v0.1.2+ version. For more information on the checks performed in this cmdlet, see [Unable to mount Azure Files with AD credentials](storage-troubleshoot-windows-file-connection-problems.md#unable-to-mount-azure-files-with-ad-credentials) in the troubleshooting guide for Windows.

```PowerShell
Debug-AzStorageAccountAuth -StorageAccountName $StorageAccountName -ResourceGroupName $ResourceGroupName -Verbose
```

## Confirm the feature is enabled

You can check to confirm whether the feature is enabled on your storage account with the following script:

```PowerShell
# Get the target storage account
$storageaccount = Get-AzStorageAccount `
        -ResourceGroupName "<your-resource-group-name-here>" `
        -Name "<your-storage-account-name-here>"

# List the directory service of the selected service account
$storageAccount.AzureFilesIdentityBasedAuth.DirectoryServiceOptions

# List the directory domain information if the storage account has enabled AD DS authentication for file shares
$storageAccount.AzureFilesIdentityBasedAuth.ActiveDirectoryProperties
```

If successful, the output should look like this:

```PowerShell
DomainName:<yourDomainHere>
NetBiosDomainName:<yourNetBiosDomainNameHere>
ForestName:<yourForestNameHere>
DomainGuid:<yourGUIDHere>
DomainSid:<yourSIDHere>
AzureStorageID:<yourStorageSIDHere>
```
## Next steps

You've now successfully enabled the feature on your storage account. To use the feature, you must assign share-level permissions. Continue to the next section.

[Part two: assign share-level permissions to an identity](storage-files-identity-ad-ds-assign-permissions.md)
