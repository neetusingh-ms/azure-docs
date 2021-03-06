---
title: Azure Service Fabric - Configure an existing Azure Service Fabric cluster to enable managed identity support | Microsoft Docs
description: This article shows you how to configure an existing Azure Service Fabric cluster to enable support for managed identities
services: service-fabric
author: athinanthny

ms.service: service-fabric
ms.topic: article
ms.date: 07/25/2019
ms.author: atsenthi
---

# Configure an existing Azure Service Fabric cluster to enable Managed Identity support
In order to access the managed identity feature for Azure Service Fabric applications, you must first enable the **Managed Identity Token Service** on the cluster. This service is responsible for the authentication of Service Fabric applications using their managed identities, and for obtaining access tokens on their behalf. Once the service is enabled, you can see it in Service Fabric Explorer under the **System** section in the left pane, running under the name **fabric:/System/ManagedIdentityTokenService**.

> [!NOTE]
> Service Fabric runtime version 6.5.658.9590 or higher is required to enable the **Managed Identity Token Service**.  
> 
> You can find the Service Fabric version of a cluster from the Azure portal by opening the cluster resource and checking the **Service Fabric version** property in the **Essentials** section.
> 
> If the cluster is on **Manual** upgrade mode, you will need to first upgrade it to 6.5.658.9590 or later.


## Enable the Managed Identity Token Service in an existing cluster
To enable the Managed Identity Token Service in an existing cluster, you will need to initiate a cluster upgrade specifying two changes: enabling the Managed Identity Token Service, and requesting a restart of each node. To do so, add the following two snippets in the Azure Resource Manager template:

```json
"fabricSettings": [
    {
        "name": "ManagedIdentityTokenService",
        "parameters": [
            {
                "name": "IsEnabled",
                "value": "true"
            }
        ]
    }
]
```

In order for the changes to take effect, you will also need to change the upgrade policy to specify a forceful restart of the Service Fabric runtime on each node as the upgrade progresses through the cluster. This restart ensures that the newly enabled system service is started and running on each node. In the snippet below, `forceRestart` is the essential setting; use your existing values for the remainder of the settings.  

```json
"upgradeDescription": {
    "forceRestart": true,
    "healthCheckRetryTimeout": "00:45:00",
    "healthCheckStableDuration": "00:05:00",
    "healthCheckWaitDuration": "00:05:00",
    "upgradeDomainTimeout": "02:00:00",
    "upgradeReplicaSetCheckTimeout": "1.00:00:00",
    "upgradeTimeout": "12:00:00"
}
```

> [!NOTE]
> Upon the successful completion of the upgrade, do not forget to roll back the `forceRestart` setting, to minimize the impact of subsequent upgrades. 

## Errors and troubleshooting

If the deployment fails with the following message, it means the cluster is not running on a high enough Service Fabric version:

```json
{
    "code": "ParameterNotAllowed",
    "message": "Section 'ManagedIdentityTokenService' and Parameter 'IsEnabled' is not allowed."
}
```

## Next steps
* [Deploy an Azure Service Fabric application with a system-assigned managed identity](./how-to-deploy-service-fabric-application-system-assigned-managed-identity.md)
* [Deploy an Azure Service Fabric application with a user-assigned managed identity](./how-to-deploy-service-fabric-application-user-assigned-managed-identity.md)
* [Leverage the managed identity of a Service Fabric application from service code](./how-to-managed-identity-service-fabric-app-code.md)
* [Grant an Azure Service Fabric application access to other Azure resources](./how-to-grant-access-other-resources.md)

## Related Articles
* Review [managed identity support](./concepts-managed-identity.md) in Azure Service Fabric

* [Enable managed identity support in an existing Azure Service Fabric cluster](./configure-existing-cluster-enable-managed-identity-token-service.md)
