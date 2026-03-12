# Threat Hunting with Velociraptor

- Install Velociraptor server in one machine and client in a Windows machine.
- Configure Velociraptor for endpoint monitoring.

![Velociraptor Home Page](<attachments/Velociraptor/Velociraptor Home page.png>)

## Installing Sysmon on Windows Client

1. Download Microsoft Sysinternals Sysmon: [Sysmon v15.15](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)

2. Download Sysmon Config ZIP File: [olafhartong/sysmon-modular](https://github.com/olafhartong/sysmon-modular). Extract the file, ensure the `sysmonconfig.xml` file is there.

3. Transfer all files to the same folder.
    - Open `cmd` as admin, navigate to the folder: `cd C:\Users\[username]\Documents\sysmon-master`
    - Run the command below

```shell
Sysmon.exe -accepteula -i sysmonconfig.xml

####################~OUTPUT~#####################

#System Monitor v15.15 - System activity monitor
#By Mark Russinovich and Thomas Garnier
#Copyright (C) 2014-2024 Microsoft Corporation
#Using libxml2. libxml2 is Copyright (C) 1998-2012 Daniel Veillard. All #Rights Reserved.
#Sysinternals - www.sysinternals.com

#Loading configuration file with schema version 4.90
#Configuration file validated.
#Sysmon installed.
#SysmonDrv installed.
#Starting SysmonDrv.
#SysmonDrv started.
#Starting Sysmon...
#Sysmon started.
```

4. Confirm that it has been installed. Open PowerShell as admin and run

```PowerShell
Get-Service Sysmon

####################~OUTPUT~#####################
#Status   Name               DisplayName
#------   ----               -----------
#Running  Sysmon             Sysmon
```

## Querying Sysmon on Velociraptor Server

1. Query the Sysmon events

```sql
SELECT * FROM parse_evtx( filename='C:/Windows/System32/winevt/logs/Microsoft-Windows-Sysmon%4Operational.evtx' )
```

![Query Sysmon events](<attachments/Velociraptor/Sysmon events Query.png>)

2. Confirm everything is being captured (Sysmon is running all events)

```sql
-- Event ID 1: Process Creation
-- Event ID 3: Network Connection
-- Event ID 7: Image/DLL Loaded
-- Event ID 8: CreateRemoteThread
-- Event ID 10: ProcessAccess
-- Event ID 11: FileCreate
-- Event ID 12-14: Registry Events
-- Event ID 22: DNS Query

SELECT 
  System.EventID.Value AS EventID,
  System.TimeCreated.SystemTime AS Time,
  EventData
FROM parse_evtx(
  filename='C:/Windows/System32/winevt/logs/Microsoft-Windows-Sysmon%4Operational.evtx'
)
WHERE System.EventID.Value IN (1, 3, 7, 11, 22)
```

![Running Second Query](<attachments/Velociraptor/Running Query 2.png>)

![Results of second Query](<attachments/Velociraptor/Query 2 Results.png>)
