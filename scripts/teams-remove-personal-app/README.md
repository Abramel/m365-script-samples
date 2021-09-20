---
plugin: add-to-gallery
---

# Removes Microsoft Teams personal app from users and Microsoft Teams app catalog

## Summary

Uninstalls an app from the specified users and / or unpublish it from the Microsoft Teams app catalog based on the App Id available in the
``` manifest.json ``` of the Teams app.

[!INCLUDE [Delete Warning](../../docfx/includes/DELETE-WARN.md)]
 
# [CLI for Microsoft 365 with PowerShell](#tab/cli-m365-ps)
```powershell
<#
  .SYNOPSIS
    Removes an app from the personal scope of a set of users
  .EXAMPLE
    .\Remove-PersonalApp.ps1 -AppId "2dbace6f-3f3b-4779-9e3f-bb4d27c403fe" -Unpublish -Uninstall -CurrentUser
  .EXAMPLE
    .\Remove-PersonalApp.ps1 -AppId "2dbace6f-3f3b-4779-9e3f-bb4d27c403fe" -Unpublish -Uninstall -CurrentUser -Users @("user1@contoso.com", "user2@contoso.com")
  .EXAMPLE
    .\Remove-PersonalApp.ps1 -AppId "2dbace6f-3f3b-4779-9e3f-bb4d27c403fe" -Unpublish
  .PARAMETER AppId
    GUID of the Microsoft Teams app. Is the same "id" you can find in the manifest.json from your Microsoft Teams app.
  .PARAMETER Users
    Array of string representing the usernames of the users to deploy the Microsoft Teams app to.
  .PARAMETER CurrentUser
    Switch allowing to Install the app for the current user
#>
Param(
  [string]$AppId,
  [string[]]$Users,
  [switch]$Uninstall,
  [switch]$Unpublish,
  [switch]$CurrentUser
)

$m365Status = m365 status

if ($m365Status -eq "Logged Out") {
  # Connection to Microsoft 365
  m365 login
}

# Validating that the app is not already in the store
$app = m365 teams app list --query "[?externalId == '$AppId']" -o json | ConvertFrom-Json

if ($app.Length -gt 0) {
  if ($Uninstall) {
    if ($CurrentUser) {
      # Getting the reference of the currently connected user
      $connectedAs = m365 status -o json | ConvertFrom-Json
      $user = m365 aad user get --userName $connectedAs.connectedAs -o json | ConvertFrom-Json

      if ($user) {
        $Users += $user.userPrincipalName
      }
    }

    if ($Users.Length -gt 0) {
      $Users | ForEach-Object {
        $user = m365 aad user get --userName $_ -o json | ConvertFrom-Json
        $userApp = m365 teams user app list --userId $user.id --query "[?appId == '$($app.id)']" -o json | ConvertFrom-Json

        if ($userApp) {
          # Removing the app from the personal apps of the specified user
          m365 teams user app remove --appId $userApp.id --userId $user.id --confirm
          Write-Host "The App '$($app.displayName)' with ID '$($app.id)' was removed for user '$($user.userPrincipalName)'."
        }
        else {
          Write-Warning "The App '$($app.displayName)' with ID '$($app.id)' is not installed for user '$($user.userPrincipalName)'."
        }
      }
    }
  }  

  if ($Unpublish) {
    # Removing the app from the app catalog
    m365 teams app remove --id $app.id --confirm
    Write-Host "The App '$($app.displayName)' with ID '$($app.id)' was removed from the app catalog."
  }
}
else {
  Write-Warning "The App with ID '$AppId' does not exist."
}
```
[!INCLUDE [More about CLI for Microsoft 365](../../docfx/includes/MORE-CLIM365.md)]
***

## Source Credit

Sample first appeared on [Removes Microsoft Teams personal app from users and Microsoft Teams app catalog | CLI for Microsoft 365](https://pnp.github.io/cli-microsoft365/sample-scripts/teams/remove-personal-app/)

## Contributors

| Author(s) |
|-----------|
| Sébastien Levert |


[!INCLUDE [DISCLAIMER](../../docfx/includes/DISCLAIMER.md)]
<img src="https://telemetry.sharepointpnp.com/script-samples/scripts/teams-remove-personal-app" aria-hidden="true" />