Title: Minecraft server on Server Core? Totally Possible
Published: 2012-06-02
Tags: [ 'Server Core', 'Minecraft', 'Tekkit' ]
---
So seeing a couple of youtube videos got me inspired to try out a completely modded Minecraft Server. RedPower 2 looks pretty cool, and the Tekkit Server makes this super easy.

Anyway, I thought to try it out on a Windows Server 2012 Release Candidate VM.

So, two thoughts, you will need to install Java on the VM. I installed 1.7 x64 with no issues.
You will need to open firewall ports:

`PS C:\Windows\system32> New-NetFirewallRule -DisplayName “Minecraft” -Profile Domain -Direction Inbound -Action Allow -LocalPort @(“25565″) -Protocol TCP`

The latest JRE install will add java to the path, so after a restart, you can create a .bat file with the following (appropriate for a machine with 4GB RAM):

`java -Xmx3072M -Xms1536M -jar Tekkit.jar nogui`
