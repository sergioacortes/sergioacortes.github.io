---
title: Aspnet core data protection
description: >-
  Let's supposed that you need to persist trusted information for later retrieval, but you don't trust the persistence mechanism. In web terms, this might be written as I need to round-trip trusted state via an untrusted client.
author: sergioacortes
date: 2025-06-13 00:00:00 +0800
categories: [Blogging, Aspnet, Data protection]
tags: [Aspnet]
pin: true
media_subpath: '/posts/20250613'
---

# Use case: Running an application behind a reverse proxy.

Let's imagine an application running on kubernetes with multiple replicas running simmultaniusly. 

As you might know that if your kubernetes cluster is running on a cloud provider ([Microsoft Azure](https://portal.azure.com/), [Amazon Web Services](https://aws.amazon.com/es/), [Google Cloud Platform](http://cloud.google.com/), etc), there is a load balancer that redirect the traffic to the kubernetes ingress, in resume, the application is running behind a reverse proxy.

In the use case we are about to talk, the application runs behing a reverse proxy and start an authentication flow with the identity server (OpenIdConnect) and needs to save the state of the authentication flow once the identity server sends the response. The problem faced in this use case comes because the different replicas running in kubernetes are not sharing the state and here is when [Aspnet core data protection](https://learn.microsoft.com/en-us/aspnet/core/security/data-protection/introduction?view=aspnetcore-9.0) comes into play.

# Use Microsoft DataProtection

As you can read in the officila documentation, you can use DataProtection to store sensitive data, and theres many persistenace options to store this data, such as Azure Storage, Redis, Mongo, etc.

In this particular example, you will find how to configure DataProtection to store sensitive data in Azure Storage connecting to it using an Azure Service principal.

```
internal static class ServiceCollectionExtensions
{

    private const string ApplicationName = "invoicing-control-panel";
    
    public static IServiceCollection AddDataProtection(this IServiceCollection services, IConfiguration configuration, IWebHostEnvironment webHostEnvironment)
    {

        if (webHostEnvironment.IsDevelopment())
        {
            services
                .AddDataProtection()
                .SetApplicationName(ApplicationName)
                .PersistKeysToFileSystem(new DirectoryInfo(webHostEnvironment.ContentRootPath));

            return services;
        }

        var credentials = GetTokenDefaultAzureCredential(
            tenantId: configuration.GetValue<string>("AzureConfiguration:TenantId")!,
            applicationId: configuration.GetValue<string>("AzureConfiguration:ApplicationId")!,
            applicationSecret: configuration.GetValue<string>("AzureConfiguration:ApplicationSecret")!
        );
        var blobStorageConnectionString = configuration.GetValue<string>("AzureConfiguration:BlobStorage:ConnectionString")!;
        var blobStorageContainerName = configuration.GetValue<string>("AzureConfiguration:BlobStorage:ContainerName")!;

        var dataProtectionUrl =
            $"{configuration.GetValue<string>("AzureConfiguration:KeyVaultUrl")!}{configuration.GetValue<string>("AzureConfiguration:DataProtectionKey")}";

        services.AddDataProtection()
            .PersistKeysToAzureBlobStorage(blobStorageConnectionString, blobStorageContainerName, $"data-protection-{applicationName}")
            .ProtectKeysWithAzureKeyVault(new Uri(dataProtectionUrl), credentials);

        return services;

    }

    private static ClientSecretCredential GetTokenDefaultAzureCredential(
        string tenantId,
        string applicationId,
        string applicationSecret
        ) => new(
        tenantId,
        applicationId,
        applicationSecret);

}
```
 