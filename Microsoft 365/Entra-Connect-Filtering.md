---
title: Entra Connect Filtering
parent: Microsoft 365
---

# Entra Connect Filtering

If you've installed Azure AD Connect to sync objects from your local Active Directory to Office 365, you may have seen that you can use filtering to stop objects being sync.  Yeah Yeah I hear you say, you can filter objects by the OU they're in.. Yes you can, but you can also filter by attributes on objects, which as you can imagine can be very handy.

The following link to [Microsoft Entra Connect Sync: Configure filtering] contains information on how to do this.

The filtering examples in the above link can be used to filter in/out users from being sync'd to Microsoft 365 Entra ID from your local AD. It uses the Extension Attributes (on the user objects) to perform the filtering. Once you Entra Connect has been setup to do this filtering, the below PowerShell examples can be used to populate the relevant ExtensionAttribute with values which will be filtered to stop users being synced to Microsoft 365.

The Microsoft Example uses ExtensionAttribute15 being set to 'NoSync'. In my examples, I've used ExtensionAttribute8 and setting to 'DoNotSync' as that's how was the ExtensionAttribute and value selected in our Azure AD Connect.

To find any users within your AD which have ExtensionAttribute8 set to 'DoNotSync'.

```
Import-Module ActiveDirectory
$users = Get-ADUser -Filter "ExtensionAttribute8 -eq 'DoNotSync'" -properties ExtensionAttribute8
```

To find any users within your AD which have ExtensionAttribute8 set to 'DoNotSync' within a specific OU

```
$users = Get-ADUser -Filter "ExtensionAttribute8 -eq 'DoNotSync'" -properties ExtensionAttribute8 -SearchBase "OU=UserAccounts,DC=FABRIKAM,DC=COM"
```

Or a specific user

```
get-aduser -Identity username -Properties ExtensionAttribute8
```
To Add 'DoNotSync' in ExtensionAttribute8 to a specific user

```
set-aduser -Identity username -Replace @{ExtensionAttribute8='DoNotSync'}
```

----

[Microsoft Entra Connect Sync: Configure filtering]: https://learn.microsoft.com/en-us/entra/identity/hybrid/connect/how-to-connect-sync-configure-filtering#attribute-based-filtering
