# Intune Device Details GUI - Functions FAQ

## 📋 Table of Contents

- [Overview](#overview)
- [Core Functions](#core-functions)
- [Utility Functions](#utility-functions)
- [Data Retrieval Functions](#data-retrieval-functions)
- [Conversion & Parsing Functions](#conversion--parsing-functions)
- [Report Generation Functions](#report-generation-functions)
- [Frequently Asked Questions](#frequently-asked-questions)

---

## Overview

**Q: What is IntuneDeviceDetailsGUI?**

A: It's a PowerShell-based tool that generates comprehensive HTML reports for Microsoft Intune managed devices. It provides device intelligence including apps, policies, scripts, assignments, group memberships, and conflict detection.

**Q: What version of PowerShell is required?**

A: Both Windows PowerShell 5.1 and PowerShell 7.x are supported.

**Q: What PowerShell modules are required?**

A: Microsoft.Graph.Authentication module is required. The script checks for this automatically and provides installation instructions if missing.

---

## Core Functions

### `Invoke-IntuneDeviceDetailsReport`

**What does it do?**

Main orchestrator function that generates complete device reports.

**Parameters:**

- `DeviceId` - Intune device GUID
- `ReloadCache` - Forces cache reload
- `SkipAssignments` - Skips assignment data
- `DoNotOpenReportAutomatically` - Prevents auto-opening report
- `ExtendedReport` - Generates extended report with full details

**When should I use it?**

This is the main entry point for report generation. Call it after finding a device.

---

### `Initialize-IntuneSession`

**What does it do?**

Establishes connection to Microsoft Graph API and initializes the session.

**Key Actions:**

- Checks for required PowerShell modules
- Connects to Microsoft Graph with necessary permissions
- Retrieves tenant information
- Sets up cache folders
- Initializes quick search filters

**When does it run?**

Automatically runs on first API call to ensure authenticated session exists.

---

### `Search-ManagedDevices`

**What does it do?**

Searches for Intune managed devices based on various criteria.

**Search Methods:**

- Device name (supports partial matches)
- User principal name (email)
- Serial number
- GUID (exact match)
- Quick filters (predefined queries)

**Returns:**

Array of matching devices with key properties sorted by device name.

---

## Utility Functions

### `Validate-GUID`

**What does it do?**

Validates if a string is a properly formatted GUID.

**Why is it needed?**

Determines if search input is a device ID for direct lookup vs. text search.

---

### `Fix-UrlSpecialCharacters`

**What does it do?**

URL-encodes special characters for safe Graph API queries.

**Characters Encoded:**

- Space → %20
- Quote → %22
- Apostrophe → %27
- Backslash → %5C
- @ → %40
- Nordic characters (ä, ö, å, Ä, Ö, Å)

**Why is it important?**

Prevents API errors when searching for devices with special characters in names.

---

### `Add-GUIDToHashtable`

**What does it do?**

Adds object GUIDs to a global hashtable for fast lookups.

**Why use a hashtable?**

O(1) lookup performance vs. O(n) array searches when resolving GUIDs to names.

---

### `Get-NameFromGUID`

**What does it do?**

Retrieves display name for a GUID from the hashtable.

**Use Case:**

Converting policy/group/user GUIDs to readable names in reports.

---

### `Get-ObjectFromGUID`

**What does it do?**

Retrieves complete object data for a GUID from hashtable.

**Use Case:**

Getting full object details when you only have the GUID.

---

### `ConvertTo-LocalDateTimeString`

**What does it do?**

Converts UTC datetime strings to local timezone with custom formatting.

**Format:**

`yyyy-MM-dd HH:mm:ss` (e.g., "2026-01-16 14:30:45")

---

### `Get-CacheFolder`

**What does it do?**

Returns the cache folder path for the current tenant.

**Cache Structure:**

`cache\{TenantId}\` stores downloaded data to minimize API calls.

---

### `Ensure-Directory`

**What does it do?**

Creates directory if it doesn't exist and returns the path.

**When is it used?**

Setting up cache and reports folders.

---

## Data Retrieval Functions

### `Invoke-MGGraphGetRequestWithMSGraphAllPages`

**What does it do?**

Executes Graph API GET requests with automatic pagination handling.

**Why is pagination needed?**

Graph API returns max 100-1000 items per request. This function automatically follows @odata.nextLink to get all results.

**Parameters:**

- `url` - Graph API endpoint URL

---

### `Invoke-MGGraphPostRequest`

**What does it do?**

Executes Graph API POST requests (used for reporting endpoints).

**Use Cases:**

- Configuration policy reports
- Compliance policy reports
- App installation reports

---

### `Get-ManagedDeviceSnapshot`

**What does it do?**

Retrieves current device data from Intune.

**Data Includes:**

- Device name, OS, manufacturer, model
- Enrollment date, last sync
- Compliance state
- Primary user
- Serial number

---

### `Get-AdditionalDeviceHardware`

**What does it do?**

Fetches extended hardware information.

**Additional Data:**

- Total storage, free storage
- Physical memory
- Device category
- Ownership type

---

### `Get-PrimaryUserContext`

**What does it do?**

Retrieves primary user and their Azure AD group memberships.

**Why is this important?**

Determines which user-targeted assignments apply to the device.

---

### `Get-LatestLogonContext`

**What does it do?**

Finds the most recent logged-in user and their group memberships.

**Use Case:**

Device may be assigned to one user but another user recently logged in. Both contexts matter for assignments.

---

### `Get-AzureDeviceContext`

**What does it do?**

Retrieves Azure AD device object and device group memberships.

**Why separate from Intune?**

Azure AD device has additional properties and group memberships critical for assignments.

---

### `Get-AutopilotContext`

**What does it do?**

Retrieves Windows Autopilot device registration and assigned profile.

**Data Includes:**

- Autopilot profile assignments
- Group tags
- Purchase order ID
- Deployment profile settings

---

### `Get-AutopilotDevicePreparationContext`

**What does it do?**

Retrieves Autopilot Device Preparation profile for newer enrollment type.

**When is it used?**

For devices using Windows Autopilot Device Preparation instead of traditional Autopilot.

---

### `Get-EnrollmentStatusPageContext`

**What does it do?**

Retrieves Enrollment Status Page (ESP) profile assigned to device.

**ESP Controls:**

- Which apps/policies must install during OOBE
- Timeout settings
- User permissions during enrollment

---

### `Get-SecurityContext`

**What does it do?**

Previously retrieved BitLocker keys and Windows LAPS passwords (now commented out for security).

**Why removed from main flow?**

Sensitive data that requires explicit user consent and additional permissions.

---

### `Get-ApplicationsWithAssignments`

**What does it do?**

Downloads all Intune applications with their assignment data.

**Cached:**

Yes - cached for 24 hours to improve performance.

**Includes:**

- Win32 apps
- Store apps
- iOS/macOS apps
- Web apps
- Assignment groups and filters

---

### `Get-MobileAppAssignments`

**What does it do?**

Retrieves and processes app assignments for specific application.

**Assignment Types:**

- Required
- Available
- Uninstall
- Device vs. User context

---

### `Get-RemediationScriptsWithAssignments`

**What does it do?**

Downloads all Proactive Remediation scripts with assignments.

**Data Includes:**

- Detection script
- Remediation script
- Run schedule
- Assignment groups

---

### `Get-PlatformScriptsWithAssignments`

**What does it do?**

Downloads PowerShell and Shell scripts with assignments.

**Platforms:**

- Windows (PowerShell)
- macOS (Shell scripts)

---

### `Get-PowerShellScriptContent`

**What does it do?**

Decodes and retrieves Base64-encoded PowerShell script content.

---

### `Get-MacOSShellScriptContent`

**What does it do?**

Decodes and retrieves Base64-encoded macOS shell script content.

---

### `Get-RemediationDetectionScriptContent`

**What does it do?**

Retrieves detection script content from remediation.

---

### `Get-RemediationRemediateScriptContent`

**What does it do?**

Retrieves remediation script content from remediation.

---

### `Download-IntuneFilters`

**What does it do?**

Downloads all assignment filters configured in Intune.

**Cached:**

Yes - filters are cached to improve performance.

**Why important?**

Filters determine whether assignments apply to specific devices based on device properties.

---

### `Download-IntuneConfigurationProfiles2`

**What does it do?**

Downloads all configuration profiles including:

- Device Configuration policies
- Settings Catalog policies
- Administrative Templates
- Compliance policies

**Cached:**

Yes - cached for 24 hours.

---

### `Download-IntunePostTypeReport`

**What does it do?**

Generic function for downloading POST-based Intune reports.

**Use Cases:**

- Configuration policy device status
- App installation status
- Compliance reports

---

### `Get-ConfigurationPolicyReport`

**What does it do?**

Generates device-specific configuration policy status report.

**Reports Include:**

- Applied policies
- Policy settings
- Success/failure status
- Error messages
- Conflict detection

**This is a large function (500+ lines). What does it contain?**

Complex logic to:

1. Query all policy types
2. Match assignments to device/user
3. Retrieve policy settings
4. Detect conflicts
5. Format results for HTML

---

### `Get-RemediationScriptsReport`

**What does it do?**

Generates remediation script status report for device.

**Status Includes:**

- Detection results
- Remediation results
- Execution timestamps
- Success/failure states
- Pre/post remediation detection results

---

### `Get-AppleEnrollmentProfileDetails`

**What does it do?**

Retrieves iOS/macOS enrollment profile assignments.

**Includes:**

- DEP profiles
- Profile restrictions
- Setup assistant options

---

### `Get-SettingsCatalogPolicyDetails`

**What does it do?**

Retrieves detailed settings for Settings Catalog policies.

**Why complex?**

Settings Catalog uses dynamic schema requiring additional API calls to get setting definitions.

---

### `Analyze-SettingsCatalogConflicts`

**What does it do?**

Detects and reports conflicting settings across multiple Settings Catalog policies.

**Conflict Detection:**

- Same setting in multiple policies with different values
- Groups conflicts by setting instance path
- Shows policy priority and values

---

### `Get-CheckedInUsersInfo`

**What does it do?**

Retrieves information about users who have logged into the device.

**Data Source:**

`usersLoggedOn` property from Intune device object.

---

## Conversion & Parsing Functions

### `ConvertTo-ReadableSettingsCatalog`

**What does it do?**

Converts Settings Catalog raw JSON to human-readable format.

**Transformation:**

- Resolves setting definitions
- Formats nested structures
- Displays setting paths and values

---

### `Parse-SettingInstance`

**What does it do?**

Parses individual setting instances from Settings Catalog.

**Handles:**

- Simple settings
- Choice settings
- Group settings (nested)
- Collection settings (arrays)

---

### `Extract-SettingInfo`

**What does it do?**

Extracts setting definition information from catalog schema.

---

### `ConvertTo-ReadableOmaSettings`

**What does it do?**

Converts OMA-URI custom settings to readable format.

**OMA-URI Used In:**

- Custom device configuration
- Administrative templates
- Legacy CSP settings

---

### `Get-OmaSettingPlainTextValue`

**What does it do?**

Decodes Base64-encoded OMA-URI setting values.

---

### `ConvertTo-ReadableWin32LobApp`

**What does it do?**

Converts Win32 app definitions to readable format with all properties.

**Includes:**

- Installation commands
- Detection rules
- Requirements
- Return codes
- Dependencies and supersedence

**This is the largest conversion function (500+ lines). Why so large?**

Win32 apps have extensive configuration:

- Multiple detection rule types
- Complex requirement rules
- Custom return codes
- Dependencies/supersedence chains
- Installation/uninstallation commands

---

### `ConvertTo-ReadableMacOSDmgApp`

**What does it do?**

Converts macOS DMG app packages to readable format.

---

### `ConvertTo-ReadableMacOSPkgApp`

**What does it do?**

Converts macOS PKG installers to readable format.

---

### `ConvertTo-ReadableIosVppApp`

**What does it do?**

Converts iOS VPP (Volume Purchase Program) apps to readable format.

---

### `ConvertTo-ReadableMacOsVppApp`

**What does it do?**

Converts macOS VPP apps to readable format.

---

### `ConvertTo-ReadableWebApp`

**What does it do?**

Converts web link apps to readable format.

---

### `ConvertTo-ReadableWinGetApp`

**What does it do?**

Converts Windows Package Manager (WinGet) apps to readable format.

---

### `ConvertTo-ReadableMacOSCustomConfiguration`

**What does it do?**

Converts macOS custom configuration profiles to readable format.

---

### `ConvertTo-ReadableMacOSCustomAppConfiguration`

**What does it do?**

Converts macOS app-specific configuration to readable format.

---

### `ConvertTo-ReadableAppleEnrollmentProfile`

**What does it do?**

Converts Apple DEP enrollment profiles to readable format.

---

### `Objectify_JSON_Schema_and_Data_To_PowershellObjects`

**What does it do?**

Converts JSON schema and data into PowerShell objects for reporting.

**Use Case:**

Processing Intune's complex JSON reports into structured objects.

---

## Report Generation Functions

### `New-IntuneDeviceHtmlReport`

**What does it do?**

Generates the complete HTML report with all device data.

**Report Sections:**

1. Device Summary
2. User Information
3. Azure AD Device Info
4. Autopilot Information
5. Group Memberships
6. Applications (with status)
7. Configuration Policies (with settings)
8. Compliance Policies
9. Scripts
10. Remediation Scripts
11. Conflicts
12. Raw JSON Data (Extended Report only)

**This is the largest function (2000+ lines). What's in it?**

- HTML structure generation
- CSS styling (embedded)
- JavaScript for interactive features
- Data formatting for all sections
- Conditional rendering based on report type
- Conflict highlighting
- Status badges and icons

---

### `Save-IntuneDeviceHtmlReport`

**What does it do?**

Saves the generated HTML report to disk and optionally opens it.

**Filename Format:**

`IntuneDeviceReport_{DeviceName}_{Timestamp}.html`

---

### `Write-DeviceSummaryToConsole`

**What does it do?**

Displays device summary in console with colored output.

---

### `Write-DeviceSearchTable`

**What does it do?**

Formats and displays device search results in console table.

---

### `Invoke-InteractiveDeviceSelection`

**What does it do?**

Handles interactive device selection from search results.

**Features:**

- Numbered device list
- Last sync age highlighting
- Quick filter shortcuts
- Quit option

---

### `Resolve-DeviceId`

**What does it do?**

Resolves various device identifier inputs to Intune device GUID.

**Accepts:**

- Intune device GUID
- Device name
- User email
- Serial number

---

### `Update-QuickFilters`

**What does it do?**

Generates dynamic quick filter list for device searches.

**Filter Categories:**

- Last sync time (1, 3, 7, 14, 30 days)
- Operating System (Windows, iOS, Android, macOS)
- Enrollment age (7, 30 days)
- Compliance state
- Ownership type (Corporate/Personal)

---

### `Resolve-AssignmentGroupNames`

**What does it do?**

Converts assignment group GUIDs to display names.

**Handles:**

- Include/Exclude groups
- All Users/All Devices virtual groups
- Filter assignments

---

### `Resolve-EspAssignmentGroupNames`

**What does it do?**

Specialized version for ESP profile assignment group resolution.

---

### `Add-AzureADGroupGroupTypeExtraProperties`

**What does it do?**

Adds formatted group type properties to Azure AD groups.

**Properties Added:**

- Group type (Security/Microsoft 365)
- Membership type (Assigned/Dynamic)
- Formatted display strings

---

### `Add-AzureADGroupDevicesAndUserMemberCountExtraProperties`

**What does it do?**

Adds device and user member counts to group objects.

**Cached:**

Yes - member counts cached to reduce API calls.

---

## Frequently Asked Questions

### General Usage

**Q: How do I run the script?**

A: `.\IntuneDeviceDetailsGUI.ps1` for interactive mode, or use parameters for direct report generation.

**Q: Do I need admin rights?**

A: No, but you need appropriate Intune/Azure AD permissions (read permissions for devices, apps, policies).

**Q: How long does report generation take?**

A: First run: 2-5 minutes (downloads all data). Subsequent runs: 30-60 seconds (uses cache).

**Q: Can I generate reports for multiple devices?**

A: Yes, use pipeline: `Get-IntuneManagedDevice | Where-Object {$_.deviceName -like 'LAPTOP*'} | .\IntuneDeviceDetailsGUI.ps1 -Id $_.id`

---

### Caching

**Q: Where is data cached?**

A: `cache\{TenantId}\` folder in script directory.

**Q: How long is cache valid?**

A: 24 hours by default (configurable via `$script:ReloadCacheEveryNDays`).

**Q: How do I force cache refresh?**

A: Use `-ReloadCache` parameter.

**Q: What's cached?**

A: Applications, configuration profiles, scripts, remediation scripts, filters, group details.

**Q: What's NOT cached?**

A: Individual device data (always fetched fresh).

---

### Reports

**Q: What's the difference between standard and extended reports?**

A:

- **Standard**: Device info, apps, policies, scripts, assignments, conflicts
- **Extended**: Everything above PLUS policy settings details, script contents, full JSON data

**Q: Where are reports saved?**

A: `reports\` folder by default, or custom location with `-OutputFolder`.

**Q: Can I customize the report?**

A: Yes, modify the `New-IntuneDeviceHtmlReport` function HTML/CSS sections.

**Q: Does the report contain sensitive data?**

A: No BitLocker keys or LAPS passwords by default. Script contents are included in extended reports.

---

### Performance

**Q: The script is slow on first run. Why?**

A: It downloads all apps, policies, and scripts from your tenant. This data is cached for subsequent runs.

**Q: How can I speed up reports?**

A: Use `-SkipAssignments` for minimal reports, or ensure cache is recent.

**Q: How many API calls does it make?**

A: First run: 50-200+ calls depending on tenant size. Cached runs: 10-20 calls.

---

### Troubleshooting

**Q: I get "Cannot bind argument to parameter 'Url' because it is an empty string"**

A: This was fixed in recent versions. Update to latest version or ensure you don't press Enter with empty search.

**Q: Authentication fails. What should I do?**

A: Run `Disconnect-MgGraph` then rerun script. Ensure you have required permissions.

**Q: Some policies show as "not assigned" but they are. Why?**

A: Check if filters are used - they may be excluding the device. Also verify group membership sync.

**Q: Conflict detection shows no conflicts but I see conflicting policies. Why?**

A: Conflict detection only works for Settings Catalog policies. OMA-URI and legacy profiles require manual review.

---

### Advanced Usage

**Q: Can I use this in automation?**

A: Yes, use `-Id` parameter with device GUID and `-DoNotOpenReportAutomatically`.

**Q: Can I export to formats other than HTML?**

A: No built-in support, but you can modify the script to export JSON objects.

**Q: Can I query multiple tenants?**

A: Yes, disconnect and reconnect between tenants. Each tenant has separate cache.

**Q: How do I integrate with Azure Automation?**

A: Use Managed Identity authentication (requires code modifications for non-interactive auth).

---

### Security & Compliance

**Q: Are credentials stored?**

A: No, uses Microsoft Graph SDK which stores tokens securely via Windows Credential Manager.

**Q: What permissions are required?**

A:

- DeviceManagementManagedDevices.Read.All
- DeviceManagementApps.Read.All
- DeviceManagementConfiguration.Read.All
- DeviceManagementServiceConfig.Read.All
- DeviceManagementScripts.Read.All
- User.Read.All
- Group.Read.All
- GroupMember.Read.All
- Directory.Read.All

**Q: Can I run this with least privilege?**

A: Yes, all permissions are read-only. No write operations are performed.

**Q: Is the HTML report safe to share?**

A: Review contents first. It contains device info, user names, group memberships, and policy details. Redact sensitive data before sharing externally.

---

## Version Information

**Current Version:** 4.0

**Author:** Petri Paavola

**Repository:** https://github.com/petripaavola/IntuneDeviceDetailsGUI

---

## Quick Reference: Common Use Cases

### Use Case 1: Find and report on a device by name

```powershell
.\IntuneDeviceDetailsGUI.ps1 -SearchText "LAPTOP001"
```

### Use Case 2: Generate extended report for specific device

```powershell
.\IntuneDeviceDetailsGUI.ps1 -Id "2e6e1d5f-b18a-44c6-989e-9bbb1efafbff" -ExtendedReport
```

### Use Case 3: Quick minimal report (no assignments)

```powershell
.\IntuneDeviceDetailsGUI.ps1 -Id "2e6e1d5f-b18a-44c6-989e-9bbb1efafbff" -SkipAssignments
```

### Use Case 4: Batch reporting via pipeline

```powershell
Get-Content deviceids.txt | ForEach-Object {
    .\IntuneDeviceDetailsGUI.ps1 -Id $_ -DoNotOpenReportAutomatically
}
```

### Use Case 5: Force fresh data

```powershell
.\IntuneDeviceDetailsGUI.ps1 -ReloadCache
```

---

## Function Categories Summary

| Category             | Count  | Purpose                               |
| -------------------- | ------ | ------------------------------------- |
| Core Functions       | 3      | Main orchestration and initialization |
| Utility Functions    | 9      | Helper functions for common tasks     |
| Data Retrieval       | 20     | Fetch data from Microsoft Graph API   |
| Conversion & Parsing | 17     | Transform API data to readable format |
| Report Generation    | 12     | Create and format HTML reports        |
| **Total**            | **61** | **Complete device intelligence**      |

---

## Support & Contributions

For issues, feature requests, or contributions, visit the GitHub repository.

**Need help?** Check the troubleshooting section or open an issue on GitHub.
