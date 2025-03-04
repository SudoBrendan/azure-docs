---
title: Manage firewall rules - Azure CLI - Azure Database for MySQL
description: This article describes how to create and manage Azure Database for MySQL firewall rules using Azure CLI command-line.
author: savjani
ms.author: pariks
ms.service: mysql
ms.devlang: azurecli
ms.topic: how-to
ms.date: 3/18/2020 
ms.custom: devx-track-azurecli
---

# Create and manage Azure Database for MySQL firewall rules by using the Azure CLI

[!INCLUDE[applies-to-mysql-single-server](includes/applies-to-mysql-single-server.md)]
Server-level firewall rules can be used to manage access to an Azure Database for MySQL Server from a specific IP address or a range of IP addresses. Using convenient Azure CLI commands, you can create, update, delete, list, and show firewall rules to manage your server. For an overview of Azure Database for MySQL firewalls, see [Azure Database for MySQL server firewall rules](./concepts-firewall-rules.md).

Virtual Network (VNet) rules can also be used to secure access to your server. Learn more about [creating and managing Virtual Network service endpoints and rules using the Azure CLI](howto-manage-vnet-using-cli.md).

## Prerequisites
* [Install Azure CLI](/cli/azure/install-azure-cli).
* An [Azure Database for MySQL server and database](quickstart-create-mysql-server-database-using-azure-cli.md).

## Firewall rule commands:
The **az mysql server firewall-rule** command is used from the Azure CLI to create, delete, list, show, and update firewall rules.

Commands:
- **create**: Create an Azure MySQL server firewall rule.
- **delete**: Delete an Azure MySQL server firewall rule.
- **list**: List the Azure MySQL server firewall rules.
- **show**: Show the details of an Azure MySQL server firewall rule.
- **update**: Update an Azure MySQL server firewall rule.

## Sign in to Azure and list your Azure Database for MySQL Servers
Securely connect Azure CLI with your Azure account by using the **az login** command.

1. From the command-line, run the following command:
    ```azurecli
    az login
    ```
   This command outputs a code to use in the next step.

2. Use a web browser to open the page [https://aka.ms/devicelogin](https://aka.ms/devicelogin), and then enter the code.

3. At the prompt, sign in using your Azure credentials.

4. After your login is authorized, a list of subscriptions is printed in the console. Copy the ID of the desired subscription to set the current subscription to use. Use the [az account set](/cli/azure/account#az-account-set) command.
    ```azurecli-interactive
    az account set --subscription <your subscription id>
    ```

5. List the Azure Databases for MySQL servers for your subscription and resource group if you are unsure of the names. Use the [az mysql server list](/cli/azure/mysql/server#az-mysql-server-list) command.

    ```azurecli-interactive
    az mysql server list --resource-group myresourcegroup
    ```

   Note the name attribute in the listing, which you need to specify the MySQL server to work on. If needed, confirm the details for that server and using the name attribute to ensure it is correct. Use the [az mysql server show](/cli/azure/mysql/server#az-mysql-server-show) command.

    ```azurecli-interactive
    az mysql server show --resource-group myresourcegroup --name mydemoserver
    ```

## List firewall rules on Azure Database for MySQL Server 
Using the server name and the resource group name, list the existing server firewall rules on the server. Use the [az mysql server firewall list](/cli/azure/mysql/server/firewall-rule#az-mysql-server-firewall-rule-list) command.  Notice that the server name attribute is specified in the **--server** switch and not in the **--name** switch. 
```azurecli-interactive
az mysql server firewall-rule list --resource-group myresourcegroup --server-name mydemoserver
```
The output lists the rules, if any, in JSON format (by default). You can use the **--output table** switch to output the results in a more readable table format.
```azurecli-interactive
az mysql server firewall-rule list --resource-group myresourcegroup --server-name mydemoserver --output table
```
## Create a firewall rule on Azure Database for MySQL Server
Using the Azure MySQL server name and the resource group name, create a new firewall rule on the server. Use the [az mysql server firewall create](/cli/azure/mysql/server/firewall-rule#az-mysql-server-firewall-rule-create) command. Provide a name for the rule, as well as the start IP and end IP (to provide access to a range of IP addresses) for the rule.
```azurecli-interactive
az mysql server firewall-rule create --resource-group myresourcegroup --server-name mydemoserver --name FirewallRule1 --start-ip-address 13.83.152.0 --end-ip-address 13.83.152.15
```

To allow access for a single IP address, provide the same IP address as the Start IP and End IP, as in this example.
```azurecli-interactive
az mysql server firewall-rule create --resource-group myresourcegroup --server-name mydemoserver --name FirewallRule1 --start-ip-address 1.1.1.1 --end-ip-address 1.1.1.1
```

To allow applications from Azure IP addresses to connect to your Azure Database for MySQL server, provide the IP address 0.0.0.0 as the Start IP and End IP, as in this example.
```azurecli-interactive
az mysql server firewall-rule create --resource-group myresourcegroup --server mysql --name "AllowAllWindowsAzureIps" --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0
```

> [!IMPORTANT]
> This option configures the firewall to allow all connections from Azure including connections from the subscriptions of other customers. When selecting this option, make sure your login and user permissions limit access to only authorized users.
> 

Upon success, each create command output lists the details of the firewall rule you have created, in JSON format (by default). If there is a failure, the output shows error message text instead.

## Update a firewall rule on Azure Database for MySQL server 
Using the Azure MySQL server name and the resource group name, update an existing firewall rule on the server. Use the [az mysql server firewall update](/cli/azure/mysql/server/firewall-rule#az-mysql-server-firewall-rule-update) command. Provide the name of the existing firewall rule as input, as well as the start IP and end IP attributes to update.
```azurecli-interactive
az mysql server firewall-rule update --resource-group myresourcegroup --server-name mydemoserver --name FirewallRule1 --start-ip-address 13.83.152.0 --end-ip-address 13.83.152.1
```
Upon success, the command output lists the details of the firewall rule you have updated, in JSON format (by default). If there is a failure, the output shows error message text instead.

> [!NOTE]
> If the firewall rule does not exist, the rule is created by the update command.

## Show firewall rule details on Azure Database for MySQL Server
Using the Azure MySQL server name and the resource group name, show the existing firewall rule details from the server. Use the [az mysql server firewall show](/cli/azure/mysql/server/firewall-rule#az-mysql-server-firewall-rule-show) command. Provide the name of the existing firewall rule as input.
```azurecli-interactive
az mysql server firewall-rule show --resource-group myresourcegroup --server-name mydemoserver --name FirewallRule1
```
Upon success, the command output lists the details of the firewall rule you have specified, in JSON format (by default). If there is a failure, the output shows error message text instead.

## Delete a firewall rule on Azure Database for MySQL Server
Using the Azure MySQL server name and the resource group name, remove an existing firewall rule from the server. Use the [az mysql server firewall delete](/cli/azure/mysql/server/firewall-rule#az-mysql-server-firewall-rule-delete) command. Provide the name of the existing firewall rule.
```azurecli-interactive
az mysql server firewall-rule delete --resource-group myresourcegroup --server-name mydemoserver --name FirewallRule1
```
Upon success, there is no output. Upon failure, error message text displays.

## Next steps
- Understand more about [Azure Database for MySQL Server firewall rules](./concepts-firewall-rules.md).
- [Create and manage Azure Database for MySQL firewall rules using the Azure portal](./howto-manage-firewall-using-portal.md).
- Further secure access to your server by [creating and managing Virtual Network service endpoints and rules using the Azure CLI](howto-manage-vnet-using-cli.md).