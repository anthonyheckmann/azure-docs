---
title: Introduction to connectivity check in Azure Network Watcher | Microsoft Docs
description: This page provides an overview of the Network Watcher connectivity capability
services: network-watcher
documentationcenter: na
author: georgewallace
manager: timlt
editor: 

ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload:  infrastructure-services
ms.date: 07/11/2017
ms.author: gwallace
---

# Introduction to connectivity check in Azure Network Watcher

The connectivity feature of Network Watcher provides the capability to test a direct TCP connection from a virtual machine to a virtual machine (VM), fully qualified domain name (FQDN), URI, or IPv4 address. Network scenarios are complex, they are implemented using Network Security Groups, firewalls, User-defined routes, and resources provided by Azure. Complex configurations make troubleshooting connectivity issues challenging. Network Watcher helps reduce the amount of time to find and detect connectivity issues. The results returned can provide insights into whether a connectivity issue is due to a platform or a user configuration issue. Connectivity can be checked with [PowerShell](network-watcher-connectivity-powershell.md), [Azure CLI](network-watcher-connectivity-cli.md), and [REST API](network-watcher-connectivity-rest.md).

[!INCLUDE [network-watcher-preview](../../includes/network-watcher-public-preview-notice.md)]

> [!IMPORTANT]
> Connectivity check requires a virtual machine extension `AzureNetworkWatcherExtension`. For installing the extension on a Windows VM visit [Azure Network Watcher Agent virtual machine extension for Windows](../virtual-machines/windows/extensions-nwa.md) and for Linux VM visit [Azure Network Watcher Agent virtual machine extension for Linux](../virtual-machines/linux/extensions-nwa.md).

## Register the preview capability

Connectivity check is currently in public preview, to use this feature it needs to be registered. To do this, run the following PowerShell sample:

```powershell
Register-AzureRmProviderFeature -FeatureName AllowNetworkWatcherConnectivityCheck  -ProviderNamespace Microsoft.Network
Register-AzureRmResourceProvider -ProviderNamespace Microsoft.Network
```

To verify the registration was successful, run the following Powershell sample:

```powershell
Get-AzureRmProviderFeature -FeatureName AllowNetworkWatcherConnectivityCheck  -ProviderNamespace  Microsoft.Network
```

If the feature was properly registered, the output should match the following:

```
FeatureName         ProviderName      RegistrationState
-----------         ------------      -----------------
AllowNetworkWatcherConnectivityCheck  Microsoft.Network Registered
```

## Response

The following table shows the properties returned when a connectivity check has finished running.

|Property  |Description  |
|---------|---------|
|ConnectionStatus     | The status of the connectivity check. Possible results are **Reachable** and **Unreachable**.        |
|AvgLatencyInMs     | Average latency during the connectivity test in milliseconds. (Only shown if check status is reachable)        |
|MinLatencyInMs     | Minimum latency during the connectivity test in milliseconds. (Only shown if check status is reachable)        |
|MaxLatencyInMs     | Maximum latency during the connectivity test in milliseconds. (Only shown if check status is reachable)        |
|ProbesSent     | Number of probes sent during the check. Max value is 100.        |
|ProbesFailed     | Number of probes that failed during the check. Max value is 100.        |
|Hops     | Hop by hop path from source to destination.        |
|Hops[].Type     | Type of resource. Possible values are **Source**, **VirtualAppliance**, **VnetLocal**, and **Internet**.        |
|Hops[].Id | Unique identifier of the hop.|
|Hops[].Address | IP address of the hop.|
|Hops[].ResourceId | ResourceID of the hop if the hop is an Azure resource. If it is an internet resource, ResourceID is **Internet**. |
|Hops[].NextHopIds | The unique identifier of the next hop taken.|
|Hops[].Issues | A collection of issues that were encountered during the check at that hop. If there were no issues, the value is blank.|
|Hops[].Issues[].Origin | At the current hop, where issue occurred. Possible values are:<br/> **Inbound** - Issue is on the link from the previous hop to the current hop<br/>**Outbound** - Issue is on the link from the current hop to the next hop<br/>**Local** - Issue is on the current hop.|
|Hops[].Issues[].Severity | The severity of the issue detected. Possible values are **Error** and **Warning**. |
|Hops[].Issues[].Type |The type of issue found. Possible values are: <br/>**CPU**<br/>**Memory**<br/>**GuestFirewall**<br/>**DnsResolution**<br/>**NetworkSecurityRule**<br/>**UserDefinedRoute** |
|Hops[].Issues[].Context |Details regarding the issue found.|
|Hops[].Issues[].Context[].key |Key of the key value pair returned.|
|Hops[].Issues[].Context[].value |Value of the key value pair returned.|

The following is an example of an issue found on a hop.

```json
"Issues": [
    {
    	"Origin": "Outbound",
    	"Severity": "Error",
    	"Type": "NetworkSecurityRule",
    	"Context": [
            {
        		"key": "RuleName",
        		"value": "UserRule_Port80"
    	    }
        ]
    }
]
```
## Fault types

The connectivity check returns fault types about the connection. The following table provides a list of the current fault types returned.

|Type  |Description  |
|---------|---------|
|CPU     | High CPU utilization.       |
|Memory     | High Memory utilization.       |
|GuestFirewall     | Traffic is blocked due to a virtual machine firewall configuration.        |
|DNSResolution     | DNS resolution failed for the destination address.        |
|NetworkSecurityRule    | Traffic is blocked by an NSG Rule (Rule is returned)        |
|UserDefinedRoute|Traffic is dropped due to a user defined or system route. |

### Next steps

Learn how to verify connectivity to a resource by visiting: [Test connectivity with Azure Network Watcher](network-watcher-connectivity-powershell.md).

<!--Image references-->
[1]: ./media/network-watcher-next-hop-overview/figure1.png

