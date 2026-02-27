---
name: infrastructure-diagrams
description: Create professional Azure, hybrid, and on-premises infrastructure architecture diagrams using Python's Diagrams library. Use when asked to create architecture diagrams, infrastructure diagrams, cloud diagrams, network diagrams, system architecture visualizations, or data center layouts. Supports Azure (VMs, networking, storage, databases, containers, security), on-premises (servers, databases, networking equipment, monitoring), Kubernetes, and hybrid cloud scenarios. Outputs PNG, SVG, or PDF files.
---

# Infrastructure Diagrams Skill

Generate professional cloud and on-premises infrastructure diagrams using Python's Diagrams library.

## Prerequisites

Install required packages before generating diagrams:

```bash
pip install diagrams --break-system-packages
apt-get update && apt-get install -y graphviz
```

## Quick Start

```python
from diagrams import Diagram, Cluster, Edge
from diagrams.azure.compute import VM
from diagrams.azure.network import VirtualNetworks
from diagrams.onprem.database import PostgreSQL

with Diagram("My Architecture", show=False, filename="architecture", outformat="png"):
    with Cluster("Azure"):
        vm = VM("App Server")
    with Cluster("On-Premises"):
        db = PostgreSQL("Database")
    vm >> Edge(label="VPN") >> db
```

## Core Concepts

### Diagram Parameters

```python
with Diagram(
    name="Diagram Title",      # Title shown on diagram
    show=False,                # Don't auto-open (always use False)
    filename="output",         # Output filename (without extension)
    outformat="png",           # png, svg, pdf, jpg
    direction="LR",            # LR (left-right), TB (top-bottom), RL, BT
    graph_attr={"bgcolor": "white", "pad": "0.5"},  # Graph styling
    node_attr={"fontsize": "12"},                    # Node styling
    edge_attr={"color": "darkgray"}                  # Edge styling
):
    # diagram content
```

### Clusters (Grouping Resources)

```python
with Cluster("Azure Subscription"):
    with Cluster("Resource Group"):
        with Cluster("Virtual Network"):
            vm = VM("Server")
```

### Edges (Connections)

```python
# Basic connections
source >> target           # Left to right flow
source << target           # Right to left flow
source - target            # Bidirectional

# Labeled/styled connections
source >> Edge(label="HTTPS", color="green", style="bold") >> target

# Multiple targets
source >> [target1, target2, target3]
```

## Provider Imports

### Azure Resources

```python
# Compute
from diagrams.azure.compute import VM, VMLinux, VMWindows, VMScaleSet, FunctionApps, KubernetesServices, ContainerInstances, AppServices, BatchAccounts

# Networking
from diagrams.azure.network import VirtualNetworks, Subnets, LoadBalancers, ApplicationGateway, Firewall, VirtualNetworkGateways, ExpressrouteCircuits, DNSZones, TrafficManagerProfiles, FrontDoors, CDNProfiles, PublicIpAddresses, NetworkSecurityGroupsClassic

from diagrams.azure.networking import VirtualNetworks, Bastions, Firewalls, LoadBalancers, ApplicationGateways, VirtualNetworkGateways, ExpressrouteCircuits, NetworkSecurityGroups, PrivateEndpoint, PrivateLinkService

# Storage
from diagrams.azure.storage import StorageAccounts, BlobStorage, DataLakeStorage, FileStorage, QueueStorage, TableStorage

# Database
from diagrams.azure.database import SQLDatabases, SQLServers, CosmosDb, CacheForRedis, DatabaseForPostgresqlServers, DatabaseForMysqlServers

# Identity & Security
from diagrams.azure.identity import ActiveDirectory, ManagedIdentities, ConditionalAccess, Users, Groups
from diagrams.azure.security import KeyVaults, SecurityCenter, Sentinel

# Integration
from diagrams.azure.integration import LogicApps, ServiceBus, EventGridTopics, APIManagement

# DevOps & Monitoring
from diagrams.azure.devops import ApplicationInsights, AzureDevops, Repos, Pipelines
from diagrams.azure.monitor import Monitor, LogAnalyticsWorkspaces, ApplicationInsights

# General
from diagrams.azure.general import Subscriptions, ResourceGroups, ManagementGroups
```

### On-Premises Resources

```python
# Compute
from diagrams.onprem.compute import Server, Nomad
from diagrams.onprem.client import User, Users, Client

# Database
from diagrams.onprem.database import PostgreSQL, MySQL, MSSQL, Oracle, MongoDB, Cassandra, Redis

# Network
from diagrams.onprem.network import Nginx, Apache, HAProxy, Traefik, Internet, Consul, Envoy, CiscoRouter

# Monitoring
from diagrams.onprem.monitoring import Grafana, Prometheus, Datadog, Splunk, Nagios, Zabbix

# Security
from diagrams.onprem.security import Vault, Trivy

# Container/Orchestration
from diagrams.onprem.container import Docker
from diagrams.k8s.compute import Pod, Deployment, StatefulSet
from diagrams.k8s.network import Service, Ingress
```

### Generic Resources

```python
from diagrams.generic.network import Firewall, Router, Switch, VPN
from diagrams.generic.storage import Storage
from diagrams.generic.compute import Rack
from diagrams.generic.os import Windows, Linux
from diagrams.generic.device import Mobile, Tablet
```

### Custom Icons

```python
from diagrams.custom import Custom
from urllib.request import urlretrieve

# Download custom icon
icon_url = "https://example.com/icon.png"
icon_file = "custom_icon.png"
urlretrieve(icon_url, icon_file)

# Use custom icon
custom_node = Custom("Label", icon_file)
```

## Common Patterns

### Hub-and-Spoke Network (Azure)

```python
from diagrams import Diagram, Cluster, Edge
from diagrams.azure.network import VirtualNetworks, VirtualNetworkGateways, Firewall
from diagrams.azure.compute import VM

with Diagram("Hub and Spoke", show=False, direction="TB"):
    with Cluster("Hub VNet"):
        fw = Firewall("Azure Firewall")
        vpn = VirtualNetworkGateways("VPN Gateway")
    
    with Cluster("Spoke 1"):
        spoke1_vm = VM("App Server")
    
    with Cluster("Spoke 2"):
        spoke2_vm = VM("DB Server")
    
    vpn >> fw
    fw >> Edge(label="Peering") >> spoke1_vm
    fw >> Edge(label="Peering") >> spoke2_vm
```

### Hybrid Connectivity

```python
from diagrams import Diagram, Cluster, Edge
from diagrams.azure.network import VirtualNetworks, VirtualNetworkGateways, ExpressrouteCircuits
from diagrams.azure.compute import VM
from diagrams.onprem.compute import Server
from diagrams.onprem.network import CiscoRouter
from diagrams.generic.network import VPN

with Diagram("Hybrid Architecture", show=False, direction="LR"):
    with Cluster("On-Premises Data Center"):
        router = CiscoRouter("Edge Router")
        onprem_server = Server("Legacy System")
        router >> onprem_server
    
    vpn = VPN("Site-to-Site VPN")
    
    with Cluster("Azure"):
        vpn_gw = VirtualNetworkGateways("VPN Gateway")
        with Cluster("Virtual Network"):
            azure_vm = VM("Cloud App")
    
    router >> vpn >> vpn_gw >> azure_vm
```

### Three-Tier Web Application

```python
from diagrams import Diagram, Cluster, Edge
from diagrams.azure.network import ApplicationGateway, LoadBalancers
from diagrams.azure.compute import VM, VMScaleSet
from diagrams.azure.database import SQLDatabases
from diagrams.azure.storage import BlobStorage
from diagrams.onprem.client import Users

with Diagram("Three-Tier Architecture", show=False, direction="TB"):
    users = Users("Users")
    
    with Cluster("Azure"):
        appgw = ApplicationGateway("App Gateway")
        
        with Cluster("Web Tier"):
            web = [VM("Web 1"), VM("Web 2")]
        
        with Cluster("App Tier"):
            app = VMScaleSet("App Servers")
        
        with Cluster("Data Tier"):
            db = SQLDatabases("SQL Database")
            storage = BlobStorage("Blob Storage")
    
    users >> appgw >> web >> app
    app >> db
    app >> storage
```

### Kubernetes on Azure (AKS)

```python
from diagrams import Diagram, Cluster
from diagrams.azure.compute import KubernetesServices
from diagrams.azure.network import LoadBalancers
from diagrams.azure.database import CosmosDb
from diagrams.k8s.compute import Pod, Deployment
from diagrams.k8s.network import Service, Ingress

with Diagram("AKS Architecture", show=False):
    with Cluster("Azure"):
        lb = LoadBalancers("Load Balancer")
        db = CosmosDb("Cosmos DB")
        
        with Cluster("AKS Cluster"):
            ingress = Ingress("Ingress")
            
            with Cluster("Namespace: production"):
                svc = Service("Service")
                with Cluster("Deployment"):
                    pods = [Pod("Pod 1"), Pod("Pod 2"), Pod("Pod 3")]
    
    lb >> ingress >> svc >> pods
    pods >> db
```

## Best Practices

1. **Always use `show=False`** - Prevents auto-opening images in headless environments
2. **Use meaningful names** - Labels appear on the diagram
3. **Group related resources** - Use Clusters for logical grouping
4. **Control direction** - Use `direction` parameter for layout (LR, TB, RL, BT)
5. **Label important connections** - Use Edge() with labels for clarity
6. **Keep diagrams focused** - Create multiple diagrams for complex architectures

## Output

Generated diagrams are saved to the current working directory. Move to outputs for user access:

```bash
cp architecture.png /mnt/user-data/outputs/
```

## Reference Files

For complete node lists, see:
- `references/azure-nodes.md` - All Azure provider nodes
- `references/onprem-nodes.md` - All on-premises provider nodes
- `references/patterns.md` - Common architecture patterns
