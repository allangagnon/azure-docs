﻿---
title: Use the Azure storage emulator for development and testing | Microsoft Docs
description: The Azure storage emulator provides a free local development environment for developing and testing your Azure Storage applications. Learn how requests are authenticated, how to connect to the emulator from your application, and how to use the command-line tool.
services: storage
documentationcenter: ''
author: mmacy
manager: timlt
editor: tysonn

ms.assetid: f480b059-df8a-4a63-b05a-7f2f5d1f5c2a
ms.service: storage
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/08/2017
ms.author: marsma

---
# Use the Azure storage emulator for development and testing

The Microsoft Azure storage emulator provides a local environment that emulates the Azure Blob, Queue, and Table services for development purposes. Using the storage emulator, you can test your application against the storage services locally, without creating an Azure subscription or incurring any costs. When you're satisfied with how your application is working in the emulator, you can switch to using an Azure storage account in the cloud.

## Get the storage emulator
The storage emulator is available as part of the [Microsoft Azure SDK](https://azure.microsoft.com/downloads/). You can also install the storage emulator by using the [standalone installer](https://go.microsoft.com/fwlink/?linkid=717179&clcid=0x409) (direct download). To install the storage emulator, you must have administrative privileges on your computer.

The storage emulator currently runs only on Windows. For those considering a storage emulator for Linux, one option is the community maintained, open source storage emulator [Azurite](https://github.com/arafato/azurite).

> [!NOTE]
> Data created in one version of the storage emulator is not guaranteed to be accessible when using a different version. If you need to persist your data for the long term, we recommended that you store that data in an Azure storage account, rather than in the storage emulator.
> <p/>
> The storage emulator depends on specific versions of the OData libraries. Replacing the OData DLLs used by the storage emulator with other versions is unsupported, and may cause unexpected behavior. However, any version of OData supported by the storage service may be used to send requests to the emulator.
>
>

## How the storage emulator works
The storage emulator uses a local Microsoft SQL Server instance and the local file system to emulate Azure storage services. By default, the storage emulator uses a database in Microsoft SQL Server 2012 Express LocalDB. You can choose to configure the storage emulator to access a local instance of SQL Server instead of the LocalDB instance. For more information, see the [Start and initialize the storage emulator](#start-and-initialize-the-storage-emulator) section later in this article.

The storage emulator connects to SQL Server or LocalDB using Windows authentication.

Some differences in functionality exist between the storage emulator and Azure storage services. For more information about these differences, see the [Differences between the storage emulator and Azure Storage](#differences-between-the-storage-emulator-and-azure-storage) section later in this article.

## Start and initialize the storage emulator
To start the Azure storage emulator:
1. Select the **Start** button or press the **Windows** key.
1. Begin typing `Azure Storage Emulator`.
1. Select the emulator from the list of displayed applications.

When the storage emulator starts, a Command Prompt window will appear. You can use this console window to start and stop the storage emulator, clear data, get status, and initialize the emulator. For more information, see the [Storage emulator command-line tool reference](#storage-emulator-command-line-tool-reference) section later in this article.

When the emulator is running, you'll see an icon in the Windows taskbar notification area.

When you close the storage emulator Command Prompt window, the storage emulator will continue to run. To bring up the Storage Emulator console window again, follow the preceding steps as if starting the storage emulator.

The first time you run the storage emulator, the local storage environment is initialized for you. The initialization process creates a database in LocalDB and reserves HTTP ports for each local storage service.

The storage emulator is installed by default to `C:\Program Files (x86)\Microsoft SDKs\Azure\Storage Emulator`.

> [!TIP]
> You can use the [Microsoft Azure Storage Explorer](http://storageexplorer.com) to work with local storage emulator resources. Look for "(Development)" under "Storage Accounts" in the Storage Explorer resources tree after you've installed and started the storage emulator.
>

### Initialize the storage emulator to use a different SQL database
You can use the storage emulator command-line tool to initialize the storage emulator to point to a SQL database instance other than the default LocalDB instance:

1. Open the Storage Emulator console window as described in the [Start and initialize the storage emulator](#start-and-initialize-the-storage-emulator) section.
1. In the console window, type the following command, where `<SQLServerInstance>` is the name of the SQL Server instance. To use LocalDB, specify `(localdb)\MSSQLLocalDb` as the SQL Server instance.

  `AzureStorageEmulator.exe init /server <SQLServerInstance>`

  You can also use the following command, which directs the emulator to use the default SQL Server instance:

  `AzureStorageEmulator.exe init /server .\\`

  Or, you can use the following command, which reinitializes the database to the default LocalDB instance:

  `AzureStorageEmulator.exe init /forceCreate`

For more information about these commands, see [Storage emulator command-line tool reference](#storage-emulator-command-line-tool-reference).

> [!TIP]
> You can use the [Microsoft SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) (SSMS) to manage your SQL Server instances, including the LocalDB installation. In the SMSS **Connect to Server** dialog, specify `(localdb)\MSSQLLocalDb` in the **Server name:** field to connect to the LocalDB instance.

## Authenticating requests against the storage emulator
Once you've installed and started the storage emulator, you can test your code against it. As with Azure Storage in the cloud, every request you make against the storage emulator must be authenticated, unless it is an anonymous request. You can authenticate requests against the storage emulator using Shared Key authentication or with a shared access signature (SAS).

### Authenticate with Shared Key credentials
[!INCLUDE [storage-emulator-connection-string-include](../../includes/storage-emulator-connection-string-include.md)]

For more information on connection strings, see [Configure Azure Storage connection strings](storage-configure-connection-string.md).

### Authenticate with a shared access signature
Some Azure storage client libraries, such as the Xamarin library, only support authentication with a shared access signature (SAS) token. You can create the SAS token using a tool like the [Storage Explorer](http://storageexplorer.com/) or another application that supports Shared Key authentication.

You can also generate a SAS token by using Azure PowerShell. The following example generates a SAS token with full permissions to a blob container:

1. Install Azure PowerShell if you haven't already (using the latest version of the Azure PowerShell cmdlets is recommended). For installation instructions, see [Install and configure Azure PowerShell](/powershell/azure/install-azurerm-ps).
2. Open Azure PowerShell and run the following commands, replacing `ACCOUNT_NAME` and `ACCOUNT_KEY==` with your own credentials, and `CONTAINER_NAME` with a name of your choosing:

```powershell
$context = New-AzureStorageContext -StorageAccountName "ACCOUNT_NAME" -StorageAccountKey "ACCOUNT_KEY=="

New-AzureStorageContainer CONTAINER_NAME -Permission Off -Context $context

$now = Get-Date

New-AzureStorageContainerSASToken -Name CONTAINER_NAME -Permission rwdl -ExpiryTime $now.AddDays(1.0) -Context $context -FullUri
```

The resulting shared access signature URI for the new container should be similar to:

```
https://storageaccount.blob.core.windows.net/sascontainer?sv=2012-02-12&se=2015-07-08T00%3A12%3A08Z&sr=c&sp=wl&sig=t%2BbzU9%2B7ry4okULN9S0wst%2F8MCUhTjrHyV9rDNLSe8g%3Dsss
```

The shared access signature created with this example is valid for one day. The signature grants full access (read, write, delete, list) to blobs within the container.

For more information on shared access signatures, see [Using shared access signatures (SAS) in Azure Storage](storage-dotnet-shared-access-signature-part-1.md).

## Addressing resources in the storage emulator
The service endpoints for the storage emulator are different from those of an Azure storage account. The difference is because the local computer does not perform domain name resolution, requiring the storage emulator endpoints to be local addresses.

When you address a resource in an Azure storage account, you use the following scheme. The account name is part of the URI host name, and the resource being addressed is part of the URI path:

`<http|https>://<account-name>.<service-name>.core.windows.net/<resource-path>`

For example, the following URI is a valid address for a blob in an Azure storage account:

`https://myaccount.blob.core.windows.net/mycontainer/myblob.txt`

However, the storage emulator, because the local computer does not perform domain name resolution, the account name is part of the URI path instead of the host name. Use the following URI format for a resource in the storage emulator:

`http://<local-machine-address>:<port>/<account-name>/<resource-path>`

For example, the following address might be used for accessing a blob in the storage emulator:

`http://127.0.0.1:10000/myaccount/mycontainer/myblob.txt`

The service endpoints for the storage emulator are:

* Blob service: `http://127.0.0.1:10000/<account-name>/<resource-path>`
* Queue service: `http://127.0.0.1:10001/<account-name>/<resource-path>`
* Table service: `http://127.0.0.1:10002/<account-name>/<resource-path>`

### Addressing the account secondary with RA-GRS
Beginning with version 3.1, the storage emulator supports read-access geo-redundant replication (RA-GRS). For storage resources both in the cloud and in the local emulator, you can access the secondary location by appending -secondary to the account name. For example, the following address might be used for accessing a blob using the read-only secondary in the storage emulator:

`http://127.0.0.1:10000/myaccount-secondary/mycontainer/myblob.txt`

> [!NOTE]
> For programmatic access to the secondary with the storage emulator, use the Storage Client Library for .NET version 3.2 or later. See the [Microsoft Azure Storage Client Library for .NET](https://msdn.microsoft.com/library/azure/dn261237.aspx) for details.
>
>

## Storage emulator command-line tool reference
Starting in version 3.0, a console window is displayed when you start the Storage Emulator. Use the command line in the console window to start and stop the emulator as well as query for status and perform other operations.

> [!NOTE]
> If you have the Microsoft Azure compute emulator installed, a system tray icon appears when you launch the Storage Emulator. Right-click on the icon to reveal a menu that provides a graphical way to start and stop the Storage Emulator.
>
>

### Command line syntax
`AzureStorageEmulator.exe [start] [stop] [status] [clear] [init] [help]`

### Options
To view the list of options, type `/help` at the command prompt.

| Option | Description | Command | Arguments |
| --- | --- | --- | --- |
| **Start** |Starts up the storage emulator. |`AzureStorageEmulator.exe start [-inprocess]` |*-inprocess*: Start the emulator in the current process instead of creating a new process. |
| **Stop** |Stops the storage emulator. |`AzureStorageEmulator.exe stop` | |
| **Status** |Prints the status of the storage emulator. |`AzureStorageEmulator.exe status` | |
| **Clear** |Clears the data in all services specified on the command line. |`AzureStorageEmulator.exe clear [blob] [table] [queue] [all]                                                    ` |*blob*: Clears blob data. <br/>*queue*: Clears queue data. <br/>*table*: Clears table data. <br/>*all*: Clears all data in all services. |
| **Init** |Performs one-time initialization to set up the emulator. |<code>AzureStorageEmulator.exe init [-server serverName] [-sqlinstance instanceName] [-forcecreate&#124;-skipcreate] [-reserveports&#124;-unreserveports] [-inprocess]</code> |*-server serverName\instanceName*: Specifies the server hosting the SQL instance. <br/>*-sqlinstance instanceName*: Specifies the name of the SQL instance to be used in the default server instance. <br/>*-forcecreate*: Forces creation of the SQL database, even if it already exists. <br/>*-skipcreate*: Skips creation of the SQL database. This takes precedence over -forcecreate.<br/>*-reserveports*: Attempts to reserve the HTTP ports associated with the services.<br/>*-unreserveports*: Attempts to remove reservations for the HTTP ports associated with the services. This takes precedence over -reserveports.<br/>*-inprocess*: Performs initialization in the current process instead of spawning a new process. The current process must be launched with elevated permissions if changing port reservations. |

## Differences between the storage emulator and Azure Storage
Because the storage emulator is an emulated environment running in a local SQL instance, there are differences in functionality between the emulator and an Azure storage account in the cloud:

* The storage emulator supports only a single fixed account and a well-known authentication key.
* The storage emulator is not a scalable storage service and does not support a large number of concurrent clients.
* As described in [Addressing resources in the storage emulator](#addressing-resources-in-the-storage-emulator), resources are addressed differently in the storage emulator versus an Azure storage account. This difference is because domain name resolution is available in the cloud but not on the local computer.
* Beginning with version 3.1, the storage emulator account supports read-access geo-redundant replication (RA-GRS). In the emulator, all accounts have RA-GRS enabled, and there is never any lag between the primary and secondary replicas. The Get Blob Service Stats, Get Queue Service Stats, and Get Table Service Stats operations are supported on the account secondary and will always return the value of the `LastSyncTime` response element as the current time according to the underlying SQL database.
* The File service and SMB protocol service endpoints are not currently supported in the storage emulator.
* If you use a version of the storage services that is not yet supported by the emulator, the storage emulator returns a VersionNotSupportedByEmulator error (HTTP status code 400 - Bad Request).

### Differences for Blob storage
The following differences apply to Blob storage in the emulator:

* The storage emulator only supports blob sizes up to 2 GB.
* Incremental copy allows snapshots from overwritten blobs to be copied, which returns a failure on the service.
* Get Page Ranges Diff does not work between snapshots copied using Incremental Copy Blob.
* A Put Blob operation may succeed against a blob that exists in the storage emulator with an active lease, even if the lease ID has not been specified in the request.
* Append Blob operations are not supported by the emulator. Attempting an operation on an append blob returns a FeatureNotSupportedByEmulator error (HTTP status code 400 - Bad Request).

### Differences for Table storage
The following differences apply to Table storage in the emulator:

* Date properties in the Table service in the storage emulator support only the range supported by SQL Server 2005 (they are required to be later than January 1, 1753). All dates before January 1, 1753 are changed to this value. The precision of dates is limited to the precision of SQL Server 2005, meaning that dates are precise to 1/300th of a second.
* The storage emulator supports partition key and row key property values of less than 512 bytes each. Additionally, the total size of the account name, table name, and key property names together cannot exceed 900 bytes.
* The total size of a row in a table in the storage emulator is limited to less than 1 MB.
* In the storage emulator, properties of data type `Edm.Guid` or `Edm.Binary` support only the `Equal (eq)` and `NotEqual (ne)` comparison operators in query filter strings.

### Differences for Queue storage
There are no differences specific to Queue storage in the emulator.

## Storage emulator release notes
### Version 5.2
* The storage emulator now supports version 2017-04-17 of the storage services on Blob, Queue, and Table service endpoints.
* Fixed a bug where table property values were not being properly encoded.

### Version 5.1
* Fixed a bug where the storage emulator was returning the `DataServiceVersion` header in some responses where the service was not.

### Version 5.0
* The storage emulator installer no longer checks for existing MSSQL and .NET Framework installs.
* The storage emulator installer no longer creates the database as part of install. Database will still be created if needed as part of startup.
* Database creation no longer requires elevation.
* Port reservations are no longer needed for startup.
* Adds the following options to `init`: `-reserveports` (requires elevation), `-unreserveports` (requires elevation), `-skipcreate`.
* The Storage Emulator UI option on the system tray icon now launches the command line interface. The old GUI is no longer available.
* Some DLLs have been removed or renamed.

### Version 4.6
* The storage emulator now supports version 2016-05-31 of the storage services on Blob, Queue, and Table service endpoints.

### Version 4.5
* Fixed a bug that caused initialization and installation of the storage emulator to fail when the backing database was renamed.

### Version 4.4
* The storage emulator now supports version 2015-12-11 of the storage services on Blob, Queue, and Table service endpoints.
* The storage emulator's garbage collection of blob data is now more efficient when dealing with large numbers of blobs.
* Fixed a bug that caused container ACL XML to be validated slightly differently from how the storage service does it.
* Fixed a bug that sometimes caused max and min DateTime values to be reported in the incorrect time zone.

### Version 4.3
* The storage emulator now supports version 2015-07-08 of the storage services on Blob, Queue, and Table service endpoints.

### Version 4.2
* The storage emulator now supports version 2015-04-05 of the storage services on Blob, Queue, and Table service endpoints.

### Version 4.1
* The storage emulator now supports version 2015-02-21 of the storage services on Blob, Queue, and Table service endpoints, except for the new Append Blob features.
* If you use a version of the storage services that is not yet supported by the emulator, the emulator returns a meaningful error message. We recommend using the latest version of the emulator. If you encounter a VersionNotSupportedByEmulator error (HTTP status code 400 - Bad Request), please download the latest version of the storage emulator.
* Fixed a bug wherein a race condition caused table entity data to be incorrect during concurrent merge operations.

### Version 4.0
* The storage emulator executable has been renamed to *AzureStorageEmulator.exe*.

### Version 3.2
* The storage emulator now supports version 2014-02-14 of the storage services on Blob, Queue, and Table service endpoints. File service endpoints are not currently supported in the storage emulator. See [Versioning for the Azure Storage Services](/rest/api/storageservices/Versioning-for-the-Azure-Storage-Services) for details about version 2014-02-14.

### Version 3.1
* Read-access geo-redundant storage (RA-GRS) is now supported in the storage emulator. The Get Blob Service Stats, Get Queue Service Stats, and Get Table Service Stats APIs are supported for the account secondary and will always return the value of the LastSyncTime response element as the current time according to the underlying SQL database. For programmatic access to the secondary with the storage emulator, use the Storage Client Library for .NET version 3.2 or later. See the Microsoft Azure Storage Client Library for .NET Reference for details.

### Version 3.0
* The Azure storage emulator is no longer shipped in the same package as the compute emulator.
* The storage emulator graphical user interface is deprecated in favor of a scriptable command-line interface. For details on the command-line interface, see Storage Emulator Command-Line Tool Reference. The graphical interface will continue to be present in version 3.0, but it can only be accessed when the Compute Emulator is installed by right-clicking on the system tray icon and selecting Show Storage Emulator UI.
* Version 2013-08-15 of the Azure storage services is now fully supported. (Previously this version was only supported by Storage Emulator version 2.2.1 Preview.)

## Next steps

* Evaluate the cross-platform, community-maintained open source storage emulator [Azurite](https://github.com/arafato/azurite). 
* [Azure Storage samples using .NET](storage-samples-dotnet.md) contains links to several code samples you can use when developing your application.
* You can use the [Microsoft Azure Storage Explorer](http://storageexplorer.com) to work with resources in your cloud Storage account, and in the storage emulator.
