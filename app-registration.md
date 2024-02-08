# Entra ID Application Registration

This document explains how to create and configure an Entra ID Application Registration.

## Create an application registration

1. Login to the [Azure portal](https://portal.azure.com) and to your Entra ID (Azure AD)
2. Pick "App registrations" on the left and press the "New registration" button  
![New registration](media/appreg01.png ':size=350')
3. Enter a desired name and press "Register"  
![Register an application](media/appreg02.png ':size=350')
4. Go to "API permissions" and add the permission "Mail.Send" (under "Microsoft Graph" and then "Application permissions")
    - Limiting this permissions to only certain mailboxes is explained [later in this document](#limit-mailbox-permissions)
5. Press the "Grant admin consent for..." button after adding the permission  
![API permissions](media/appreg03.png ':size=350')
6. Go to "Certificates & secrets", upload the certificate you have create for this app registration (secrets are also supported by SMTP2Graph, but it's recommended to use certificates)  
![Certificate](media/appreg04.png ':size=350')
7. The thumbprint mentioned here should be added to the SMTP2Graph config
8. The ID you need to add to the SMTP2Graph config can be found on the overview page after "Application (client) ID"

## Generate a certificate

There a multiple ways to generate a certificate. In this example we use the site [selfsigned.org](https://selfsigned.org).

1. Enter any name where it asks for a domain  
![Enter name](media/selfsigned01.png ':size=350')
2. Press download to get your certificate
3. From the ZIP file you will need the `.key` and `.pem` file
    - You upload the `.pem` file to your app registration
    - Keep the `.key` private. Point to this file in you SMTP2Graph config ([Send: Application Registration](config.md#send-application-registration-required))

## Limit mailbox permissions

You can limit your app registration to only be able send from certain mailboxes.  
To do this you need a mail enabled security group, assign it to your app registration and as members add the mailboxes you want to grant access for.  

### Create the group
1. Login to the [Microsoft 365 admin center](https://admin.microsoft.com)
2. Go to "Teams & Groups" > "Security groups" and press "Add a mail-enabled security group"  
![Add group](media/appregpermissions01.png ':size=350')
3. Add a group with a unique email address and add mailboxes you want SMTP2Graph to be able to send mail from as members
    - **Note**: Those have to be mailboxes (user or shared). See [mail from distribution list](#mail-from-distribution-list) to setup mailing from a distribution list
    - After the group is created you can chose to hide it from the Exchange Online address list through the Exchange admin center

### Assign it to the app registration
4. Make sure you have PowerShell and the [Exchange online module](https://learn.microsoft.com/en-us/powershell/exchange/exchange-online-powershell) installed
5. In PowerShell connect to Exchange online using the command: `Connect-ExchangeOnline`
6. Link your app registration to the group with the command: `New-ApplicationAccessPolicy -AppId 01234567-89ab-cdef-0123-456789abcdef -PolicyScopeGroupId yourgroup@contoso.com -AccessRight RestrictAccess`
    - `yourgroup@contoso.com` here is the email address you picked when creating the group
    - `01234567-89ab-cdef-0123-456789abcdef` is the ID of your app registration
7. To validate if app ID `01234567-89ab-cdef-0123-456789abcdef` has access to mailbox `noreply@contoso.com` enter: `Test-ApplicationAccessPolicy -Identity noreply@contoso.com -AppId 01234567-89ab-cdef-0123-456789abcdef`
    - If the app registration has access the property `AccessCheckResult` will show "Granted", otherwise it will show "Denied"

## Mail from distribution list

It's possible to allow SMTP2Graph to send from a distribution list's email address. SMTP2Graph always sends from a mailbox, but you can assign this mailbox send as/on behalf permissions on a distribution list.

1. In this example we assume SMTP2Graph has send permissions for the mailbox `smtp2graph@example.com`
2. Login to the [Exchange admin center](https://admin.exchange.microsoft.com)
3. Go to "Groups" > "Distribution list" and select the list you want to mail from
4. Under "Settings" pick "Edit managed delegates"  
![List settings](media/distributionlist01.png ':size=350')
5. Add your `smtp2graph@example.com` mailbox (yes, this can be a shared mailbox), and press "Save changes"  
![Edit delegates](media/distributionlist02.png ':size=350')
6. In your SMTP2Graph config file, set the option [Send: Force mailbox](config.md#send-force-mailbox) to `smtp2graph@example.com`
7. Now SMTP2Graph is able to sent items from the distribution list's email address (they will be send from the `smtp2graph@example.com` mailbox, thus they will be in the `Sent items` folder of that mailbox)
