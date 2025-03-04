---
title: Connectivity modes and requirements
description: Explains Azure Arc-enabled data services connectivity options for from your environment to Azure
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 03/30/2022
ms.topic: conceptual
---

# Connectivity modes and requirements


## Connectivity modes

There are multiple options for the degree of connectivity from your Azure Arc-enabled data services environment to Azure. As your requirements vary based on business policy, government regulation, or the availability of network connectivity to Azure, you can choose from the following connectivity modes.

Azure Arc-enabled data services provides you the option to connect to Azure in two different *connectivity modes*: 

- Directly connected 
- Indirectly connected

The connectivity mode provides you the flexibility to choose how much data is sent to Azure and how users interact with the Arc Data Controller. Depending on the connectivity mode that is chosen, some functionality of Azure Arc-enabled data services may or may not be available.

Importantly, if the Azure Arc-enabled data services are directly connected to Azure, then users can use [Azure Resource Manager APIs](/rest/api/resources/), the Azure CLI, and the Azure portal to operate the Azure Arc data services. The experience in directly connected mode is much like how you would use any other Azure service with provisioning/de-provisioning, scaling, configuring, and so on all in the Azure portal.  If the Azure Arc-enabled data services are indirectly connected to Azure, then the Azure portal is a read-only view. You can see the inventory of SQL managed instances and Postgres Hyperscale instances that you have deployed and the details about them, but you cannot take action on them in the Azure portal.  In the indirectly connected mode, all actions must be taken locally using Azure Data Studio, the appropriate CLI, or Kubernetes native tools like kubectl.

Additionally, Azure Active Directory and Azure Role-Based Access Control can be used in the directly connected mode only because there is a dependency on a continuous and direct connection to Azure to provide this functionality.

Some Azure-attached services are only available when they can be directly reached such as Container Insights, and backup to blob storage.

||**Indirectly connected**|**Directly connected**|**Never connected**|
|---|---|---|---|
|**Description**|Indirectly connected mode offers most of the management services locally in your environment with no direct connection to Azure.  A minimal amount of data must be sent to Azure for inventory and billing purposes _only_. It is exported to a file and uploaded to Azure at least once per month.  No direct or continuous connection to Azure is required.  Some features and services which require a connection to Azure will not be available.|Directly connected mode offers all of the available services when a direct connection can be established with Azure. Connections are always initiated _from_ your environment to Azure and use standard ports and protocols such as HTTPS/443.|No data can be sent to or from Azure in any way.|
|**Current availability**| Available |Available|Not currently supported.|
|**Typical use cases**|On-premises data centers that don’t allow connectivity in or out of the data region of the data center due to business or regulatory compliance policies or out of concerns of external attacks or data exfiltration.  Typical examples: Financial institutions, health care, government. <br/><br/>Edge site locations where the edge site doesn’t typically have connectivity to the Internet.  Typical examples: oil/gas or military field applications.  <br/><br/>Edge site locations that have intermittent connectivity with long periods of outages.  Typical examples: stadiums, cruise ships. | Organizations who are using public clouds.  Typical examples: Azure, AWS or Google Cloud.<br/><br/>Edge site locations where Internet connectivity is typically present and allowed.  Typical examples: retail stores, manufacturing.<br/><br/>Corporate data centers with more permissive policies for connectivity to/from their data region of the datacenter to the Internet.  Typical examples: Non-regulated businesses, small/medium sized businesses|Truly "air-gapped" environments where no data under any circumstances can come or go from the data environment. Typical examples: top secret government facilities.|
|**How data is sent to Azure**|There are three options for how the billing and inventory data can be sent to Azure:<br><br> 1) Data is exported out of the data region by an automated process that has connectivity to both the secure data region and Azure.<br><br>2) Data is exported out of the data region by an automated process within the data region, automatically copied to a less secure region, and an automated process in the less secure region uploads the data to Azure.<br><br>3) Data is manually exported by a user within the secure region, manually brought out of the secure region, and manually uploaded to Azure. <br><br>The first two options are an automated continuous process that can be scheduled to run frequently so there is minimal delay in the transfer of data to Azure subject only to the available connectivity to Azure.|Data is automatically and continuously sent to Azure.|Data is never sent to Azure.|

## Feature availability by connectivity mode

|**Feature**|**Indirectly connected**|**Directly connected**|
|---|---|---|
|**Automatic high availability**|Supported|Supported|
|**Self-service provisioning**|Supported<br/>Creation can be done through Azure Data Studio, the appropriate CLI, or Kubernetes native tools (helm, kubectl, oc, etc.), or using Azure Arc-enabled Kubernetes GitOps provisioning.|Supported<br/>In addition to the indirectly connected mode creation options, you can also create through the Azure portal, Azure Resource Manager APIs, the Azure CLI, or ARM templates. 
|**Elastic scalability**|Supported|Supported<br/>|
|**Billing**|Supported<br/>Billing data is periodically exported out and sent to Azure.|Supported<br/>Billing data is automatically and continuously sent to Azure and reflected in near real time. |
|**Inventory management**|Supported<br/>Inventory data is periodically exported out and sent to Azure.<br/><br/>Use client tools like Azure Data Studio, Azure Data CLI, or `kubectl` to view and manage inventory locally.|Supported<br/>Inventory data is automatically and continuously sent to Azure and reflected in near real time. As such, you can manage inventory directly from the Azure portal.|
|**Automatic upgrades and patching**|Supported<br/>The data controller must either have direct access to the Microsoft Container Registry (MCR) or the container images need to be pulled from MCR and pushed to a local, private container registry that the data controller has access to.|Supported|
|**Automatic backup and restore**|Supported<br/>Automatic local backup and restore.|Supported<br/>In addition to automated local backup and restore, you can _optionally_ send backups to Azure blob storage for long-term, off-site retention.|
|**Monitoring**|Supported<br/>Local monitoring using Grafana and Kibana dashboards.|Supported<br/>In addition to local monitoring dashboards, you can _optionally_ send monitoring data and logs to Azure Monitor for at-scale monitoring of multiple sites in one place. |
|**Authentication**|Use local username/password for data controller and dashboard authentication. Use SQL and Postgres logins or Active Directory (AD is not currently supported) for connectivity to database instances.  Use Kubernetes authentication providers for authentication to the Kubernetes API.|In addition to or instead of the authentication methods for the indirectly connected mode, you can _optionally_ use Azure Active Directory.|
|**Role-based access control (RBAC)**|Use Kubernetes RBAC on Kubernetes API. Use SQL and Postgres RBAC for database instances.|You can use Azure Active Directory and Azure RBAC. **Pending availability in directly connected mode**|

## Connectivity requirements

**Some functionality requires a connection to Azure.**

**All communication with Azure is always initiated from your environment.** This is true even for operations which are initiated by a user in the Azure portal.  In that case, there is effectively a task, which is queued up in Azure.  An agent in your environment initiates the communication with Azure to see what tasks are in the queue, runs the tasks, and reports back the status/completion/fail to Azure.

|**Type of Data**|**Direction**|**Required/Optional**|**Additional Costs**|**Mode Required**|**Notes**|
|---|---|---|---|---|---|
|**Container images**|Microsoft Container Registry -> Customer|Required|No|Indirect or direct|Container images are the method for distributing the software.  In an environment which can connect to the Microsoft Container Registry (MCR) over the Internet, the container images can be pulled directly from MCR.  In the event that the deployment environment doesn’t have direct connectivity, you can pull the images from MCR and push them to a private container registry in the deployment environment.  At creation time, you can configure the creation process to pull from the private container registry instead of MCR. This will also apply to automated updates.|
|**Resource inventory**|Customer environment -> Azure|Required|No|Indirect or direct|An inventory of data controllers, database instances (PostgreSQL and SQL) is kept in Azure for billing purposes and also for purposes of creating an inventory of all data controllers and database instances in one place which is especially useful if you have more than one environment with Azure Arc data services.  As instances are provisioned, deprovisioned, scaled out/in, scaled up/down the inventory is updated in Azure.|
|**Billing telemetry data**|Customer environment -> Azure|Required|No|Indirect or direct|Utilization of database instances must be sent to Azure for billing purposes. |
|**Monitoring data and logs**|Customer environment -> Azure|Optional|Maybe depending on data volume (see [Azure Monitor pricing](https://azure.microsoft.com/pricing/details/monitor/))|Indirect or direct|You may want to send the locally collected monitoring data and logs to Azure Monitor for aggregating data across multiple environments into one place and also to use Azure Monitor services like alerts, using the data in Azure Machine Learning, etc.|
|**Azure Role-based Access Control (Azure RBAC)**|Customer environment -> Azure -> Customer Environment|Optional|No|Direct only|If you want to use Azure RBAC, then connectivity must be established with Azure at all times.  If you don’t want to use Azure RBAC then local Kubernetes RBAC can be used.|
|**Azure Active Directory (AAD) (Future)**|Customer environment -> Azure -> Customer environment|Optional|Maybe, but you may already be paying for Azure AD|Direct only|If you want to use Azure AD for authentication, then connectivity must be established with Azure at all times. If you don’t want to use Azure AD for authentication, you can use Active Directory Federation Services (ADFS) over Active Directory. **Pending availability in directly connected mode**|
|**Backup and restore**|Customer environment -> Customer environment|Required|No|Direct or indirect|The backup and restore service can be configured to point to local storage classes. |
|**Azure backup - long term retention (Future)**| Customer environment -> Azure | Optional| Yes for Azure storage | Direct only |You may want to send backups that are taken locally to Azure Backup for long-term, off-site retention of backups and bring them back to the local environment for restore. |
|**Provisioning and configuration changes from Azure portal**|Customer environment -> Azure -> Customer environment|Optional|No|Direct only|Provisioning and configuration changes can be done locally using Azure Data Studio or the appropriate CLI.  In directly connected mode, you will also be able to provision and make configuration changes from the Azure portal.|

## Details on internet addresses, ports, encryption, and proxy server support

There are three connections required to services available on the Internet. These connections include:

- [Microsoft Container Registry (MCR)](#microsoft-container-registry-mcr)
- [Azure Resource Manager APIs](#azure-resource-manager-apis)
- [Azure monitor APIs](#azure-monitor-apis)

All HTTPS connections to Azure and the Microsoft Container Registry are encrypted using SSL/TLS using officially signed and verifiable certificates.

The following sections provide details for these connections. 

### Microsoft Container Registry (MCR)

The Microsoft Container Registry hosts the Azure Arc-enabled data services container images.  You can pull these images from MCR and push them to a private container registry and configure the data controller deployment process to pull the container images from that private container registry.

#### Connection source

The Kubernetes kubelet on each of the Kubernetes nodes pulling the container images.

#### Connection target

`mcr.microsoft.com`

#### Protocol

HTTPS

#### Port

443

#### Can use proxy

Yes

#### Authentication

None

### Helm chart used to create data controller in direct connected mode

The helm chart used to provision the Azure Arc data controller bootstrapper and cluster level objects, such as custom resource definitions, cluster roles, and cluster role bindings, is pulled from an Azure Container Registry.

#### Connection source

The Kubernetes kubelet on each of the Kubernetes nodes pulling the container images.

#### Connection target

`arcdataservicesrow1.azurecr.io`

#### Protocol

HTTPS

#### Port

443

#### Can use proxy

Yes

#### Authentication

None

### Azure Resource Manager APIs
Azure Data Studio, and Azure CLI connect to the Azure Resource Manager APIs to send and retrieve data to and from Azure for some features.

#### Connection source

A computer running Azure Data Studio, or Azure CLI that is connecting to Azure.

#### Connection target

- `login.microsoftonline.com`
- `management.azure.com`
- `san-af-eastus-prod.azurewebsites.net`
- `san-af-eastus2-prod.azurewebsites.net`
- `san-af-australiaeast-prod.azurewebsites.net`
- `san-af-centralus-prod.azurewebsites.net`
- `san-af-westus2-prod.azurewebsites.net`
- `san-af-westeurope-prod.azurewebsites.net`
- `san-af-southeastasia-prod.azurewebsites.net`
- `san-af-koreacentral-prod.azurewebsites.net`
- `san-af-northeurope-prod.azurewebsites.net`
- `san-af-westeurope-prod.azurewebsites.net`
- `san-af-uksouth-prod.azurewebsites.net`
- `san-af-francecentral-prod.azurewebsites.net`

#### Protocol

HTTPS

#### Port

443

#### Can use proxy

Yes

#### Authentication 

Azure Active Directory

### Azure monitor APIs

Azure Data Studio and Azure CLI connect to the Azure Resource Manager APIs to send and retrieve data to and from Azure for some features.

#### Connection source

A computer running Azure CLI that is uploading monitoring metrics or logs to Azure Monitor.

#### Connection target

- `login.microsoftonline.com`
- `management.azure.com`
- `*.ods.opinsights.azure.com`
- `*.oms.opinsights.azure.com`
- `*.monitoring.azure.com`

For example, to upload usage metrics data services will connect to `https://<azureRegion>.monitoring.azure.com/` where `<azureRegion>` is the region where data services is deployed.

Likewise, data services will connect to the log analytics workspace at `https://<subscription_id>.ods.opinsights.azure.com` where `<subscription_id>` represents your Azure subscription.

#### Protocol

HTTPS

#### Port

443

#### Can use proxy

Yes

#### Authentication 

Azure Active Directory

> [!NOTE]
> For now, all browser HTTPS/443 connections to the data controller for running the command `az arcdata dc export` and Grafana and Kibana dashboards are SSL encrypted using self-signed certificates.  A feature will be available in the future that will allow you to provide your own certificates for encryption of these SSL connections.

Connectivity from Azure Data Studio to the Kubernetes API server uses the Kubernetes authentication and encryption that you have established.  Each user that is using Azure Data Studio or CLI must have an authenticated connection to the Kubernetes API to perform many of the actions related to Azure Arc-enabled data services.
