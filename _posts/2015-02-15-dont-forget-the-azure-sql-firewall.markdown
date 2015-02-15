---
layout: post
title: "Don't forget about the Azure SQL Database Firewall"
date: 2015-02-15 14:42:41 +0000
comments: true
---

Microsoft Azure offers a DbaaS (Database as a service), in this case its a cloud offering of SQL Server.

To help protect your data, the Azure SQL Database firewall prevents all access to your Azure SQL Database server until you specify which IP addresses have permission.

<!-- more --> 

If you're getting errors from your application when trying to connect to SQL Azure, you might have fallen into this trap... I know I have several times ;-)

The first server-level firewall setting can be created using the Management Portal or programmatically using the REST API or Azure PowerShell.

## Manage Server-Level Firewall Rules through Management Portal

From the Management Portal, click *SQL Databases* and then drill into your database and select *dashboard*. Select *Manage allowed IP addresses* and add your client IP address.

### Manage Server-Level Firewall Rules through Transact-SQL

Load up your favourite query editor, connect to the Azure SQL Master database and modify the server-level firewall:

	EXEC sp_set_firewall_rule @name = N'MyFirewallRule', @start_ip_address = '192.168.1.1', @end_ip_address = '192.168.1.10'

### Manage Server-Level Firewall Rules through Azure Powershell

Or run some Powershell goodness:

	New-AzureSqlDatabaseServerFirewallRule –StartIPAddress 192.168.1.1 –EndIPAddress 192.168.1.10 –RuleName MyFirewallRule –ServerName MyServer

### Manage Server-Level Firewall Rules through REST API

Server-level rules can be created, updated, or deleted using REST API at the following endpoint:

	https://management.core.windows.net:8443/{subscriptionId}/services/sqlservers/servers/MyServer/firewallrules

With the following request body:

	<ServiceResource xmlns="http://schemas.microsoft.com/windowsazure">
		<Name>MyFirewallRule</Name>
		<StartIPAddress>192.168.1.4</StartIPAddress>
		<EndIPAddress>192.168.1.10</EndIPAddress>
	</ServiceResource>

Hope that helps, y'all.