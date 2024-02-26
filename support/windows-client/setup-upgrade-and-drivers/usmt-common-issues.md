---
title: User State Migration Tool (USMT) common issues
description: Describes the resolutions for the common issues that you might see when you run the User State Migration Tool (USMT) 10.0 tools.
ms.reviewer: frankroj
manager: dcscontentpm
ms.date: 12/26/2023
ms.topic: troubleshooting
ms.custom: sap:user-state-migration-tool, csstroubleshoot
audience: itpro
localization_priority: medium
---

# User State Migration Tool (USMT) common issues

The following sections discuss common issues that you might see when you run the USMT 10.0 tools. USMT produces log files that describe in further detail any errors that occurred during the migration process. These logs can be used to troubleshoot migration failures.

## General guidelines for identifying migration problems

When you encounter a problem or error message during migration, you can use the following general guidelines to help determine the source of the problem:

- Examine the **ScanState**, **LoadState**, and UsmtUtils logs to obtain the exact USMT error messages and Windows&reg; application programming interface (API) error messages. For more information about USMT return codes and error messages, see [Return codes](usmt-return-codes.md). You can obtain more information about any listed **Windows** system error codes by typing in a command prompt window `net.exe helpmsg <error_number>`  where *<error_number>* is the error code number generated by the error message. For more information about System Error Codes, see [System Error Codes (0-499)](/windows/win32/debug/system-error-codes--0-499-).

  In most cases, the **ScanState** and **LoadState** logs indicate why a USMT migration is failing. We recommend that you use the `/v:5` option when testing your migration. This verbosity level can be adjusted in a production migration; however, reducing the verbosity level might make it more difficult to diagnose failures that are encountered during production migrations. You can use a verbosity level higher than 5 if you want the log files output to go to a debugger.

  > [!NOTE]
  > Running the **ScanState** and **LoadState** tools with the `/v:5` option creates a detailed log file. Although this option makes the log file large, the extra detail can help you determine where migration errors occurred.

- Use the `/Verify` option with the UsmtUtils tool to determine whether any files in a compressed migration store are corrupted. For more information, see [Verify the condition of a compressed migration store](/windows/deployment/usmt/verify-the-condition-of-a-compressed-migration-store).

- Use the `/Extract` option with the UsmtUtils tool to extract files from a compressed migration store. For more information, see [Extract files from a compressed USMT migration store](/windows/deployment/usmt/usmt-extract-files-from-a-compressed-migration-store).

- Create a progress log using the `/Progress` option to monitor your migration.

- For the source and destination computers, obtain operating system information, and versions of applications such as Internet Explorer and any other relevant programs. Then verify the exact steps that are needed to reproduce the problem. This information might help you to understand what is wrong and to reproduce the issue in your testing environment.

- Sign out after you run the **LoadState** tool. Some settings such as fonts, desktop backgrounds, and screen-saver settings won't take effect until the next time the end user logs on.

- Close all applications before running **ScanState** or **LoadState** tools. If some applications are running during the **ScanState** or **LoadState** process, USMT might not migrate some data. For example, if Microsoft Outlook&reg; is open, USMT might not migrate PST files.

  > [!NOTE]
  > USMT will fail if it can't migrate a file or setting unless you specify the `/c` option. When you specify the `/c` option, USMT ignores errors. However, it logs an error when it encounters a file that is in use that didn't migrate.

## User account problems

The following sections describe common user account problems. Expand the section to see recommended solutions.

### I'm having problems creating local accounts on the destination computer

**Resolution:** For more information about creating accounts and migrating local accounts, see [Migrate user accounts](/windows/deployment/usmt/usmt-migrate-user-accounts).

### Not all of the user accounts were migrated to the destination computer

**Causes/Resolutions** There are two possible causes for this problem:

When running the **ScanState** and LoadState tools on Windows 7, Windows 8, or Windows 10, you must run them in Administrator mode from an account with administrative credentials to ensure that all specified users are migrated. To run in Administrator mode:

1. Select **Start** > **All Programs** > **Accessories**.

2. Right-click **Command Prompt**.

3. Select **Run as administrator**.

4. Specify the *LoadState.exe* or *ScanState.exe* command.

If you don't run USMT in Administrator mode, only the user profile that is logged on will be included in the migration.

Any user accounts on the computer that haven't been used won't be migrated. For example, if you add User1 to the computer, but User1 never logs on, then USMT won't migrate the User1 account.

### User accounts that I excluded were migrated to the destination computer

**Cause:** The command that you specified might have had conflicting `ui` and `/ue` options. If a user is specified with the `/ui` option and with either the `/ue` or `/uel` options at the same time, the user will be included in the migration. For example, if you specify `/ui:domain1\* /ue:domain1\user1`, then User1 will be migrated because the `/ui` option takes precedence.

**Resolution:** For more information about how to use the `/ui` and `/ue` options together, see the examples in the [ScanState Syntax](/windows/deployment/usmt/usmt-scanstate-syntax) article.

### I'm using the /uel option, but many accounts are still being included in the migration

**Cause:** The `/uel` option depends on the last modified date of the users' NTUser.dat file. There are scenarios in which this last modified date might not match the users' last sign-in date.

**Resolution:** This is a limitation of the `/uel` option. You might need to exclude these users manually with the `/ue` option.

### The LoadState tool reports an error as return code 71 and fails to restore a user profile during a migration test

**Cause:** During a migration test, if you run the **ScanState** tool on your test computer and then delete user profiles in order to test the **LoadState** tool on the same computer, you may have a conflicting key present in the registry. Using the **net use** command to remove a user profile will delete folders and files associated with that profile, but won't remove the registry key.

**Resolution:** To delete a user profile, use the **User Accounts** item in Control Panel. To correct an incomplete deletion of a user profile:

1. Open the registry editor by typing *regedit.exe* at an elevated command prompt.

2. Navigate to `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\ProfileList`.

    Each user profile is stored in a System Identifier key under `ProfileList`.

3. Delete the key for the user profile you're trying to remove.

### Files that weren't encrypted before the migration are now encrypted with the account used to run the LoadState tool

**Cause:** The **ScanState** tool was run using the `/EFS:copyraw` option to migrate encrypted files and Encrypting File System (EFS) certificates. The encryption attribute was set on a folder that was migrated, but the attribute was removed from file contents of that folder prior to migration.

**Resolution:** Before using the **ScanState** tool for a migration that includes encrypted files and EFS certificates, you can run the Cipher tool at the command prompt to review and change encryption settings on files and folders. You must remove the encryption attribute from folders that contain unencrypted files or encrypt the contents of all files within an encrypted folder.

To remove encryption from files that have already been migrated incorrectly, you must sign into the computer with the account that you used to run the **LoadState** tool and then remove the encryption from the affected files.

### The LoadState tool reports an error as return code 71 and a Windows Error 2202 in the log file

**Cause:** The computer name was changed during an offline migration of a local user profile.

**Resolution:** You can use the `/mu` option when you run the **LoadState** tool to specify a new name for the user. For example,

```cmd
LoadState.exe  /i:MigApp.xml /i:MigDocs.xml \\server\share\migration\mystore 
/progress:Progress.log /l:LoadState.log /mu:fareast\user1:farwest\user1
```

## Command-line problems

The following sections describe common command-line problems. Expand the section to see recommended solutions.

### I received the following error message: "Usage Error: You can't specify a file path with any of the command-line options that exceeds 256 characters."

**Cause:** You might receive this error message in some cases even if you don't specify a long store or file path, because the path length is calculated based on the absolute path. For example, if you run the `**ScanState**.exe /o store` command from *C:\Program Files\USMT40*, then each character in *C:\Program Files\USMT40* will be added to the length of "store" to get the length of the path.

**Resolution:** Ensure that the total path length doesn't exceed 256 characters. The total path length includes the store path plus the current directory.

### I received the following error message: "USMT was unable to create the log file(s). Ensure that you have write access to the log directory."

**Cause:** If you're running the **ScanState** or **LoadState** tools from a shared network resource, you'll receive this error message if you don't specify `/l`.

**Resolution:** To fix this issue in this scenario, specify the `/l:ScanState.log` or `/l:LoadState.log` option.

## XML file problems

The following sections describe common XML file problems. Expand the section to see recommended solutions.

### I used the `/genconfig` option to create a Config.xml file, but I see only a few applications and components that are in MigApp.xml. Why does Config.xml not contain all of the same applications?

**Cause:** *Config.xml* will contain only operating system components, applications, and the user document sections that are in both of the .xml files and are installed on the computer when you run the `/genconfig` option. Otherwise, these applications and components won't appear in the *Config.xml* file.

**Resolution:** Install all of the desired applications on the computer before running the `/genconfig` option. Then run *ScanState.exe* with all of the .xml files. For example, run the following command:

```cmd
ScanState.exe /genconfig:Config.xml /i:MigDocs.xml /i:MigApp.xml /v:5 /l:ScanState.log
```

### I'm having problems with a custom .xml file that I authored, and I can't verify that the syntax is correct

**Resolution:** You can load the XML schema file *MigXML.xsd* into your XML authoring tool. *MigXML.xsd* is included with USMT. For examples, see the [Visual Studio Development Center](https://go.microsoft.com/fwlink/p/?LinkId=74513). Then, load your .xml file in the authoring tool to see if there's a syntax error. For more information about using the XML elements, see [USMT XML Reference](/windows/deployment/usmt/usmt-xml-reference).

### I'm using a MigXML helper function, but the migration isn't working the way I expected it to.  How do I troubleshoot this issue?

**Cause:** Typically, this issue is caused by incorrect syntax used in a helper function. You receive a Success return code, but the files you wanted to migrate didn't get collected or applied, or weren't collected or applied in the way you expected.

**Resolution:** You should search the **ScanState** or **LoadState** log for either the component name that contains the MigXML helper function, or the MigXML helper function title, so that you can locate the related warning in the log file.

## Migration problems

The following sections describe common migration problems. Expand the section to see recommended solutions.

### Files that I specified to exclude are still being migrated

**Cause:** There might be another rule that is including the files. If there's a more specific rule or a conflicting rule, the files will be included in the migration.

**Resolution:** For more information, see [Conflicts and Precedence](/windows/deployment/usmt/usmt-conflicts-and-precedence) and the Diagnostic Log section in [Log Files](/windows/deployment/usmt/usmt-log-files).

### I specified rules to move a folder to a specific location on the destination computer, but it hasn't migrated correctly

**Cause:** There might be an error in the XML syntax.

**Resolution:** You can use the USMT XML schema (*MigXML.xsd*) to write and validate migration .xml files. Also see the XML examples in the following articles:

[Conflicts and precedence](/windows/deployment/usmt/usmt-conflicts-and-precedence)

[Exclude files and settings](/windows/deployment/usmt/usmt-exclude-files-and-settings)

[Reroute files and settings](/windows/deployment/usmt/usmt-reroute-files-and-settings)

[Include files and settings](/windows/deployment/usmt/usmt-include-files-and-settings)

[Custom XML examples](/windows/deployment/usmt/usmt-custom-xml-examples)

### After LoadState completes, the new desktop background doesn't appear on the destination computer

There are three typical causes for this issue.

**Cause**: Some settings such as fonts, desktop backgrounds, and screen-saver settings aren't applied by **LoadState** until after the destination computer has been restarted.

**Resolution:** To fix this issue, sign out, and then log back on to see the migrated desktop background.

<!--- 

REMOVING THIS SECTION SINCE IT ONLY PERTAINS TO WINDOWS VERSIONS THAT HAVE BEEN OUT OF SUPPORT FOR SEVERAL YEARS

**Cause \#2:** If the source computer was running Windows&reg; XP and the desktop background was stored in the *Drive*:\\WINDOWS\\Web\\Wallpaper folder—the default folder where desktop backgrounds are stored in Windows XP—the desktop background won't be migrated. Instead, the destination computer will have the default Windows&reg; desktop background. This issue will occur even if the desktop background was a custom picture that was added to the \\WINDOWS\\Web\\Wallpaper folder. However, if the end user sets a picture as the desktop background that was saved in another location, for example, My Pictures, then the desktop background will migrate.

**Resolution:** Ensure that the desktop background images that you want to migrate aren't in the \\WINDOWS\\Web\\Wallpaper folder on the source computer.

**Cause \#3:** If **ScanState** wasn't run on Windows XP from an account with administrative credentials, some operating system settings won't migrate. For example, desktop background settings, screen-saver selections, modem options, media-player settings, and Remote Access Service (RAS) connection phone book (.pbk) files and settings won't migrate.

**Resolution:** Run the **ScanState** and **LoadState** tools from within an account with administrative credentials.
 --->

### I included MigApp.xml in the migration, but some PST files aren't migrating

**Cause:** The *MigApp.xml* file migrates only the PST files that are linked to Outlook profiles.

**Resolution:** To migrate PST files that aren't linked to Outlook profiles, you must create a separate migration rule to capture these files.

### USMT doesn't migrate the Start layout

**Description:** You're using USMT to migrate profiles from one installation of Windows 10 to another installation of Windows 10 on different hardware. After migration, the user signs in on the new device and doesn't have the Start menu layout they had previously configured.

**Cause:** A code change in the Start Menu with Windows 10 version 1607 and later is incompatible with this USMT function.

**Resolution:** The following workaround is available:

1. With the user signed in, back up the Start layout using the following Windows PowerShell command. You can specify a different path if desired:

    ```powershell
    Export-StartLayout -Path "C:\Layout\user1.xml"
    ```

2. Migrate the user's profile with USMT.

3. Before the user signs in on the new device, import the Start layout using the following Windows PowerShell command:

    ```powershell
    Import-StartLayout -LayoutPath "C:\Layout\user1.xml" -MountPath %systemdrive%
    ```

This workaround changes the Default user's Start layout. The workaround doesn't scale to a mass migrations or multiuser devices, but it can potentially unblock some scenarios. If other users will sign on to the device, you should delete layoutmodification.xml from the Default user profile. Otherwise, all users who sign on to that device will use the imported Start layout.

## Offline migration problems

The following sections describe common offline migration problems. Expand the section to see recommended solutions.

### Some of my system settings don't migrate in an offline migration

**Cause:** Some system settings, such as desktop backgrounds and network printers, aren't supported in an offline migration. For more information, see [What does USMT migrate?](/windows/deployment/usmt/usmt-what-does-usmt-migrate)

**Resolution:** In an offline migration, these system settings must be restored manually.

### The ScanState tool fails with return code 26

**Cause:** A common cause of return code 26 is that a temp profile is active on the source computer. This profile maps to c:\\users\\temp. The **ScanState** log shows a **MigStartupOfflineCaught** exception that includes the message **User profile duplicate SID error**.

**Resolution:** You can reboot the computer to get rid of the temp profile or you can set **MIG_FAIL_ON_PROFILE_ERROR=0** to skip the error and exclude the temp profile.

### Include and Exclude rules for migrating user profiles don't work the same offline as they do online

**Cause:** When offline, the DNS server can't be queried to resolve the user name and SID mapping.

**Resolution:** Use a Security Identifier (SID) to include a user when running the **ScanState** tool. For example:

```cmd
ScanState.exe /ui:S1-5-21-124525095-708259637-1543119021*
```

The wild card (\*) at the end of the SID will migrate the *SID*\_Classes key as well.

You can also use patterns for SIDs that identify generic users or groups. For example, you can use the `/ue:*-500` option to exclude the local administrator accounts. For more information about Windows SIDs, see [Security identifiers](/windows-server/identity/ad-ds/manage/understand-security-identifiers).

### My script to wipe the disk fails after running the ScanState tool on a 64-bit system

**Cause:** The HKLM registry hive isn't unloaded after the **ScanState** tool has finished running.

**Resolution:** Reboot the computer or unload the registry hive at the command prompt after the **ScanState** tool has finished running. For example, at a command prompt, enter:

```cmd
reg.exe unload hklm\$dest$software
```

## Hard-Link Migration Problems

The following sections describe common hard-link migration problems. Expand the section to see recommended solutions.

### EFS files aren't restored to the new partition

**Cause:** EFS files can't be moved to a new partition with a hard link. The `/efs:hardlink` command-line option is only applicable to files migrated on the same partition.

**Resolution:** Use the `/efs:copyraw` command-line option to copy EFS files during the migration instead of creating hard links, or manually copy the EFS files from the hard-link store.

### The ScanState tool can't delete a previous hard-link migration store

**Cause:** The migration store contains hard links to locked files.

**Resolution:** Use the UsmtUtils tool to delete the store or change the store name. For example, at a command prompt, enter:

```cmd
UsmtUtils.exe /rd <storedir>
```

You should also reboot the machine.

## Data collection

If you need assistance from Microsoft support, we recommend you collect the information by following the steps mentioned in [Gather information by using TSS for deployment-related issues](../windows-troubleshooters/gather-information-using-tss-deployment.md).

## Related articles

[User State Migration Tool (USMT) troubleshooting](/windows/deployment/usmt/usmt-troubleshooting)

[Frequently asked questions](/windows/deployment/usmt/usmt-faq)

[Return codes](usmt-return-codes.md)

[UsmtUtils syntax](/windows/deployment/usmt/usmt-utilities)