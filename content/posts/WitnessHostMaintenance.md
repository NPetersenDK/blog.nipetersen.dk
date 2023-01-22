---
title: "Place a VSAN Witness host into maintenance through PowerCLI"
date: 2022-07-24T00:32:00+01:00
draft: false
---

The PowerCLI documentation is actually really good, but sometimes the documentation is just silly aswell. I was trying to make a vSAN Witness host go into Maintenance Mode through PowerCLI and had trouble doing it.

The Set-VMHost command has some parameters you can set for VSAN Data Migration and stuff like that, so i thought maybe it wanted to do something with vSAN even though the GUI is the normal Maintenance Mode dialog box.

![vCenter GUI VSAN MM Mode](/blog/img/vCSA_Witness_MM.png)

When I tried using the Set-VMHost command i got the error A specified parameter was not correct:

<!--more-->

I went into my LogInsight installation and tried to find the error, and sure enough PowerCLI tries to evacuateAllData, see below:

![LogInsight Error MM Witness](/blog/img/vRLi_Witness_MM_Error.png)

What fixed it for me was using the -VsanDataMigrationMode parameter with "NoDataMigration". 

So the full command for me was: **Get-VMHost vsanwitness01.mylab.local | -State Maintenance -VsanDataMigrationMode NoDataMigration**
