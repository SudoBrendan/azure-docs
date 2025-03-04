---
title: Configure data at scale for Azure Automation State Configuration
description: This article tells how to configure data at scale for Azure Automation State Configuration.
keywords: dsc,powershell,configuration,setup
services: automation
ms.subservice: dsc
ms.date: 08/08/2019
ms.topic: conceptual
---

# Configure data at scale for Azure Automation State Configuration

> Applies To: Windows PowerShell 5.1

Managing hundreds or thousands of servers can be a challenge.
Customers have provided feedback that the most difficult aspect is actually managing
[configuration data](/powershell/dsc/configurations/configdata).
Organizing information across logical constructs like location, type, and environment.

> [!NOTE]
> This article refers to a solution that is maintained by the Open Source community.
> Support is only available in the form of GitHub collaboration, not from Microsoft.

## Community project: Datum

A community maintained solution named
[Datum](https://github.com/gaelcolas/Datum)
has been created to resolve this challenge.
Datum builds on great ideas from other configuration management platforms
and implements the same type of solution for PowerShell DSC.
Information is
[organized in to text files](https://github.com/gaelcolas/Datum#3-intended-usage)
based on logical ideas.
Examples would be:

- Settings that should apply globally
- Settings that should apply to all servers in a location
- Settings that should apply to all database servers
- Individual server settings

This information is organized in the file format you prefer (JSON, Yaml, or PSD1).
Then cmdlets are provided to generate configuration data files by
[consolidating the information](https://github.com/gaelcolas/Datum#datum-tree)
from each file in to single view of a server or server role.

Once the data files have been generated,
you can use them with
[DSC Configuration scripts](/powershell/dsc/configurations/write-compile-apply-configuration)
to generate MOF files
and
[upload the MOF files to Azure Automation](./tutorial-configure-servers-desired-state.md#create-and-upload-a-configuration-to-azure-automation).
Then register your servers from either
[on-premises](./automation-dsc-onboarding.md#enable-physicalvirtual-linux-machines)
or [in Azure](./automation-dsc-onboarding.md#enable-azure-vms)
to pull configurations.

To try out Datum, visit the
[PowerShell Gallery](https://www.powershellgallery.com/packages/datum/)
and download the solution or click "Project Site"
to view the
[documentation](https://github.com/gaelcolas/Datum#2-getting-started--concepts).

## Next steps

- To understand PowerShell DSC, see [Windows PowerShell Desired State Configuration overview](/powershell/dsc/overview).
- Find out about PowerShell DSC resources in [DSC Resources](/powershell/dsc/resources/resources).
- For details of Local Configuration Manager configuration, see [Configuring the Local Configuration Manager](/powershell/dsc/managing-nodes/metaconfig).
