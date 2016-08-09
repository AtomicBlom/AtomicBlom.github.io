Title: Various useful powershell commands for Windows 8 Server (beta)
Posted: 2012-04-10
Tags: [ 'Powershell', 'Windows 2012 Server', 'Server Core' ]
---
I’ve been setting up a few instances of Windows 8 Server Core and couldn’t be bothered figuring out if CoreConfig works with Win8 so I thought I’d go the hard way and look at what cmdlets are provided for doing the most basic tasks I need such as:

##Renaming a computer
`Rename-Computer -NewName Server001 -DomainCredential Domain01\Admin01 -Restart`
 

##Get a desired network adapter’s interface index

`$adapterIndex = $(Get-NetIPAddress | where { $_.InterfaceAlias -eq "Wired Ethernet Connection" -and $_.AddressFamily -eq "IPv4" }).InterfaceIndex`

##Change the adapter’s IP Address to a static one:

`Remove-NetIPAddress -InterfaceIndex $adapterIndex -AddressFamily IPv4`
`New-NetIPAddress -InterfaceIndex $adapterIndex -IPv4Address 192.168.0.2 -PrefixLength 24 -DefaultGateway 192.168.0.254`

##Change the DNS Settings (I decided to keep the pre-existing IPv6 Settings, using Set-DnsClientServerAddress will wipe out both IPv4 and IPv6 settings.)

`Set-DnsClientServerAddress -InterfaceIndex $adapterIndex -ServerAddresses @("192.168.0.1", "fec0:0:0:ffff::1", "fec0:0:0:ffff::2", "fec0:0:0:ffff::3")`

##Firewall Rules for SQL Server

`New-NetFirewallRule -DisplayName "SQL Server Database" -Profile Domain -Direction Inbound -Action Allow -LocalPort @("1433", "1434", "4022", "135") -Protocol TCP`
 