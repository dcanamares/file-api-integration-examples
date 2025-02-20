# File API SDKs and Examples

The document is divided into two parts, the first part includes the documentation of File API SDKs and the second part is the Examples of File API SDKs usage. 

## File API SDKs

[![Build Status](https://dev.azure.com/raet/RaetOnline/_apis/build/status/Team%20Transporters/FTaaS/Ftaas.Sdk?branchName=master)](https://dev.azure.com/raet/RaetOnline/_build/latest?definitionId=4631&branchName=master)

Library that allows to send and receive files to File API.
It comes in two different flavors: [Ftaas.Sdk.FileSystem](#ftaassdkfilesystem) and [Ftaas.Sdk.Streaming](#ftaassdkstreaming). The first one allows 
to send and receive files directly on your file system, while the second is more configurable, as the source of files are streams.

### FTaaS.Sdk.FileSystem

Integrate with File API with file system sources, sending and downloading files from and into directories.

#### Getting started

1. Install the file system Nuget package into your ASP.NET Core application.

    ```
    Package Manager : Install-Package VismaRaet.FileApi.Sdk.FileSystem -Version 1.10.0
    CLI : dotnet add package VismaRaet.FileApi.Sdk.FileSystem --version 1.10.0
    ```

2. In the `ConfigureServices` method of `Startup.cs`, register the FileSystem integrator.

    ```csharp
    using Ftaas.Sdk.FileSystem;
    ```

    ```csharp
    services.AddFileSystemService(
                            options =>
                            {
                                options.MftServiceBaseAddress = mftService;
                                options.ChunkMaxBytesSize = Configuration.GetValue<int>("chunk_max_bytes_size");
								options.ClientTimeout = Configuration.GetValue<int>("client_timeout");
                                options.ConcurrentConnectionsCount = Configuration.GetValue<byte>("concurrent_connections");
                            },
                            async (serviceProvider) =>
                            {
                                await serviceProvider.GetRequiredService<ISecureStore>().TryRetrieveAsync<string>($"ftaascli:{mftService}", out var serializedLoginSession);
                                var loginSession = JsonConvert.DeserializeObject<LoginSession>(serializedLoginSession);
                                var bearerToken = loginSession.AuthorizationToken;
                                bearerToken.ValidateJwtToken();
                                return bearerToken;
                            });
    ```

    `IServiceCollection AddFileSystemService(
            this IServiceCollection services,
            Action<ServiceConfigurationOptions> optionsConfiguration,
            Func<IServiceProvider, Task<string>> bearerTokenFactory)`\
    `optionConfiguration`: MFT API base address, maximum chunk size (0 - 4194304 bytes) and number of concurrent connections (1 - 6).\
    `bearerTokenFactory`: Function that retrieves an authorization token.

#### Usage 

##### Upload

`UploadFileAsync` uploads a file and returns the metadata of the file created on MFT: The Id can be used to download the file by the subscribers.

_NOTE: If the file size is greater than the maximum chunk size configured, it will be uploaded by chunks._

###### Task<FileUploadInfo> UploadFileAsync(FileUploadRequest request, string filePath, string tenantId, CancellationToken cancellationToken) 
`request`: composed by fileName and bussinessTypeId.\
`filePath`: absolute path of the file to upload.\
`tenantId`: (optional) tenantId.\
`cancellationToken`: (optional) the CancellationToken that the upload task will observe.

##### Download

`DownloadFileAsync` downloads the requested file on the specified path.

_NOTE: If the file already exists, it will be replaced with the downloaded one._

###### Task DownloadFileAsync(string fileId, string filePath, string tenantId, CancellationToken cancellationToken)
`fileId`: GUID of the file to download.\
`filePath`: absolute path of the file to download.\
`tenantId`: (optional) tenantId.\
`cancellationToken`: (optional) the CancellationToken that the download task will observe.

##### List

`GetAvailableFilesAsync` retrieves a list of metadatas of the available files.

_NOTE: Files that have already been downloaded won't be listed._

###### Task<PaginatedItems<FileInfo>> GetAvailableFilesAsync(long businessTypeId, Pagination pagination, string tenantId, CancellationToken cancellationToken)
`businessTypeId`: (optional) if specified, only the files of this bussiness type will be listed.\
`pagination`: (optional) if specified, the list will have the specified items size and will be the index page. If not, the first twenty files metadata will be retrieved.\
`tenantId`: (optional) tenantId.\
`cancellationToken`: (optional) the CancellationToken that the list task will observe.

##### Has Subscription

Returns true if a business type has subscribers for the specified authorized tenant and false otherwise.

###### Task<bool> HasSubscriptionAsync(long businessTypeId, string tenantId, CancellationToken cancellationToken)
`businessTypeId`: business type.\
`tenantId`: (optional) tenantId.\
`cancellationToken`: (optional) the CancellationToken that the task will observe.

### FTaaS.Sdk.Streaming

Integrate with File API with stream sources.

#### Getting started

1. Install the streams Nuget package into your ASP.NET Core application.

    ```
    Package Manager : Install-Package VismaRaet.FileApi.Sdk.Streaming -Version 1.10.0
    CLI : dotnet add package VismaRaet.FileApi.Sdk.Streaming --version 0.10.0
    ```

2. In the `ConfigureServices` method of `Startup.cs`, register the Streaming integrator.

    ```csharp
    using Ftaas.Sdk.Streaming;
    ```
    
    ```csharp
    services.AddStreamingService(
                            options =>
                            {
                                options.MftServiceBaseAddress = mftService;
                                options.ChunkMaxBytesSize = Configuration.GetValue<int>("chunk_max_bytes_size");
                                options.ConcurrentConnectionsCount = Configuration.GetValue<byte>("concurrent_connections");
                            },
                            async (serviceProvider) =>
                            {
                                await serviceProvider.GetRequiredService<ISecureStore>().TryRetrieveAsync<string>($"ftaascli:{mftService}", out var serializedLoginSession);
                                var loginSession = JsonConvert.DeserializeObject<LoginSession>(serializedLoginSession);
                                var bearerToken = loginSession.AuthorizationToken;
                                bearerToken.ValidateJwtToken();
                                return bearerToken;
                            });
    ```

    `IServiceCollection AddStreamingService(
            this IServiceCollection services,
            Action<ServiceConfigurationOptions> optionsConfiguration,
            Func<IServiceProvider, Task<string>> bearerTokenFactory)`\
    `optionConfiguration`: MFT API base address, maximum chunk size (0 - 4194304 bytes) and number of concurrent connections (1 - 6).\
    `bearerTokenFactory`: Function that retrieves an authorization token.

#### Usage

##### Upload

`UploadFileAsync` uploads a file and returns the metadata of the file created on MFT: The Id can be used to download the file by the subscribers.

_NOTE: If the file size is greater than the maximum chunk size configured, it will be uploaded by chunks._

###### Task<FileUploadInfo> UploadFileAsync(FileUploadRequest request, Stream fileStream, string tenantId, CancellationToken cancellationToken)
`request`: composed by fileName and bussinessTypeId.\
`fileStream`: stream containing the bytes of the file.\
`tenantId`: (optional) tenantId.\
`cancellationToken`: (optional) the CancellationToken that the upload task will observe.

##### Download

`DownloadFileAsync` downloads the requested file on the specified path.

###### Task DownloadFileAsync(string fileId, Stream fileStream, string tenantId, CancellationToken cancellationToken)
`fileId`: GUID of the file to download.\
`fileStream`: stream where the bytes of the file will be stored.\
`tenantId`: (optional) tenantId.\
`cancellationToken`: (optional) the CancellationToken that the download task will observe.

##### List

`GetAvailableFilesAsync` retrieves a list of metadatas of the available files.

_NOTE: Files that have already been downloaded won't be listed._

###### Task<PaginatedItems<FileInfo>> GetAvailableFilesAsync( long businessTypeId, Pagination pagination, string tenantId, CancellationToken cancellationToken)
`businessTypeId`: (optional) if specified, only the files of this bussiness type will be listed.\
`pagination`: (optional) if specified, the list will have the specified items size and will be the index page. If not, the first twenty files metadata will be retrieved.\
`tenantId`: (optional) tenantId.\
`cancellationToken`: (optional) the CancellationToken that the list task will observe.

##### Has Subscription

Returns true if a business type has subscribers for the specified authorized tenant and false otherwise.

###### Task<bool> HasSubscriptionAsync(long businessTypeId, string tenantId, CancellationToken cancellationToken)
`businessTypeId`: business type.\
`tenantId`: (optional) tenantId.\
`cancellationToken`: (optional) the CancellationToken that the task will observe.

## Examples of File API SDKs usage

**net-core-with-file-api-sdk** folder includes a collection of examples that show how to integrate the **File API SDKs** with **.Net Core**.

### Getting Started with Examples

Download **net-core-with-file-api-sdk** folder.

Inside the folder there is a solution called **FileAPI.MFT**. This solution contains two projects:
  - **FileAPI.MFT.FileSystem.NetCore22**. This project provides examples of how to integrate **FileSystem SDK** using **.Net Core 2.2**.
  - **FileAPI.MFT.Streaming.NetCore22**. This projects provides examples of how to integrate **Streaming SDK** using **.Net Core 2.2**.

The examples are in the **Examples** folder and they are created as tests methods. Run them if you want to see the SDK working.

Most of the examples require custom parameters (like the tenant ID you want to use, the business type you want to upload the files to...).

This required data is in three places:
  - **config.json**: Contains some parameters that the SDK needs.
  - **Startup.cs**: When injecting the SDK, the second parameter needs to be provided.
  - **In each test**: Required data is at the beginning of each test, inside a **#region** called **Custom parameters**.

Please, fill the required data before running the examples.

**NOTE: If you are having errors when excuting the examples. Most likely will be caused because the custom parameters are not correctly provided.**

### Projects structure of Examples

Both example projects has the same structure:
  - **Example** folder contains all the examples as test methods.
  - **Files** folder works as an internal file system. It also provides some sample files.
  - **config.json** contains some parameters that the SDK needs.
  - **Startup.cs** initialize the examples and injects the SDK.

### Running Examples

Please, refer to [Microsoft documentation](https://docs.microsoft.com/en-us/visualstudio/test/run-unit-tests-with-test-explorer?view=vs-2019).

The examples are populated with some logs. They are stored internally and shown after an example is executed. To see these logs go the **Test Explorer**, choose the executed test and press **Open additional output for this result**. You can get more information [here](https://xunit.net/docs/capturing-output).

**WARNING: Take on consideration that the tests are running through real environments. This mean that they will affect the data that is in these environments. Remember you can choose the environment in the config.json.**

## Authors

**Visma - Transporters Team**