# Failover-Clustering-Workgroup
Steps to deploy Failover Clustering in Workgroup on Windows Server 2016 + Storage Spaces Direct (S2D)

## Create Local admin account ##
Create a local admin account on each node (must have same name, password and member of Local Administrator group)

```
net user CLUADMIN Pa$$w0rd /add
net localgroup administrators CLUADMIN /add
```

## Change Registry Key ##
This registry key will affect User Account Control (UAC). You must change this key on each node

```
new-itemproperty -path HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System -Name LocalAccountTokenFilterPolicy -Value 1
```

## Enable Remote Commands in Windows PowerShell ##
This registry key allow the execution of remote comannds in PowerShell

```
Set-Item WSMan:\localhost\Client\TrustedHosts -Value node1,node2,node3
```

## Install Features on each node ##
Install the following features on each note remotely: Failover Clustering, File Server and management tools. You must replace value "node" to the name of your servers

```
$Servers = 'node1','node2','node3' 
$Servers | ForEach { Install-WindowsFeature -ComputerName $_ -Name Failover-Clustering,FS-FileServer -IncludeManagementTools -restart -verbose}
```

## Check the installation of the features
Execute the command bellow to check the installation. Must be performed for each server

```
get-windowsfeature -computername node2
```

## Check if the nodes can support Storage Spaces Direct

```
Test-cluster -node node1, node2, node3 -Include "Storage Spaces Direct",Inventory,Network,"System Configuration"
```

## Create Failover Clustering
This command bellow create the cluster without a shared storage and uses a static IP Address

```
new-cluster -name democluster -node node1, node2, node3 -nostorage -staticaddress 192.168.1.130 -AdministrativeAccessPoint DNS -verbose
```

## Enable Storage Spaces Direct ##
Enable S2D to be used by Cluster. Ensure that you have enough extra disks on each node (same quantity and size)

```
enable-clusters2d -verbose
```

## Create a Storage Pool using S2D ##
This command will create your first Storage Pool using disks from S2D

```
New-Volume -StoragePoolFriendlyName S2D* -FriendlyName MultiResilient -FileSystem CSVFS_REFS -StorageTierFriendlyName Capacity -StorageTierSizes 10GB
```

## Check installation ##
- Open Failover Cluster Manager and select the properties of "democluster"
- List disks
- Open the folder c:\ClusterStorage to check if the CSV was enabled
- Execute the following PowerShell command to check disks
```
Get-ClusterAvailableDisk | Add-ClusterDisk
```
