# TempDB Monitoring – Example Code & Workflow  
_A snippet of how I collect telemetry for the TempDB portion of my SQL Server monitoring dashboard and automated ticketing system._

<img width="411" height="298" alt="image" src="https://github.com/user-attachments/assets/9e101915-6603-4dec-8c04-af9a2afd8b16" />

As a Jr. DBA, I built many of my first telemetry pipelines by combining SQL tables, PowerShell automation, and Power BI visualizations.  
This example shows one of the simplest — yet most useful — pieces of the SQL Big Brother monitoring system: **TempDB health tracking across all SQL Servers**.

This write-up may help other DBAs or system administrators build or enhance their own monitoring workflows.

---

##  1. TempDB Table Structure (Central Dashboard Database)

This table stores TempDB metrics for **all** SQL Servers in the environment.

```sql
CREATE TABLE [dbo].[Dashboard_TempDB](
    [ServerName]           NVARCHAR(128)   NULL,
    [CurrentDateTime]      DATETIME        NULL,
    [FileName]             NVARCHAR(128)   NULL,
    [FileType]             NVARCHAR(60)    NULL,
    [TotalSize_MB]         DECIMAL(18,2)   NULL,
    [UsedSpace_MB]         DECIMAL(18,2)   NULL,
    [FreeSpace_MB]         DECIMAL(18,2)   NULL,
    [FreeSpacePercentage]  DECIMAL(10,2)   NULL
);
GO

```

## 2. Powershell Script - Collecting TempDB Metrics

This script:

Truncates the existing TempDB data (“empty the bucket”).
Loops through every SQL Server in a CSV list (or later, a master server table).
Queries TempDB file usage from each server.
Inserts results into the central dashboard table.

✔️ Script runs under an account with sysadmin rights

✔️ Uses Invoke-Sqlcmd (SQLServer module required)

In my early days as a Jr. DBA I experimented with different ways of feeding the script a server name list to iterate through. In this example, I use a csv list.
In my CPU script I utilize a hardcoded list. 
As i've matured, I've moved my scripts to now use a custome stored procedure to refrence a master table full of servernames and their status.

To keep it simple, I'll show one utilzing a csv. 

```powershell
$Server = "OurServerName"
$Database = "CentralDBName"
$Table = "Dashboard_TempDB"

# Connection string
$ConnectionString = "Server=$Server;Database=$Database;Integrated Security=True"

# Truncate existing data
Invoke-Sqlcmd -Query "
USE [$Database];
TRUNCATE TABLE $Table;
" -ServerInstance $Server

Write-Host "Table truncated" -ForegroundColor Green

# ---------------- Grab new data ----------------

# CSV containing server list
$ServerList = Import-Csv "C:\Powershell\tempdb.csv"

foreach ($server in $ServerList) {

    $ServerName = $server.ServerName

    $Results = Invoke-Sqlcmd -Query "
        SELECT 
            @@SERVERNAME AS ServerName,
            GETDATE() AS CurrentDateTime,
            name AS FileName,
            type_desc AS FileType,
            size * 8 / 1024 AS TotalSize_MB,
            FILEPROPERTY(name, 'SpaceUsed') * 8 / 1024 AS UsedSpace_MB,
            (size - FILEPROPERTY(name, 'SpaceUsed')) * 8 / 1024 AS FreeSpace_MB,
            ((size - FILEPROPERTY(name, 'SpaceUsed')) * 1.0 / size) * 100 AS FreeSpacePercentage
        FROM tempdb.sys.database_files;
    " -ServerInstance $ServerName

    Write-Host "$ServerName data collected" -ForegroundColor Cyan

    # Insert each row into SQL table
    $Results | ForEach-Object {
        $Query = @"
INSERT INTO $Table (ServerName, CurrentDateTime, FileName, FileType, TotalSize_MB, UsedSpace_MB, FreeSpace_MB, FreeSpacePercentage)
VALUES ('$($_.ServerName)', '$($_.CurrentDateTime)', '$($_.FileName)', '$($_.FileType)', '$($_.TotalSize_MB)', '$($_.UsedSpace_MB)', '$($_.FreeSpace_MB)', '$($_.FreeSpacePercentage)');
"@
        Invoke-Sqlcmd -Query $Query -ConnectionString $ConnectionString
    }
}

Write-Host "TempDB table updated successfully." -ForegroundColor Yellow

```
##  3. Power BI Visualization

Once your table is populated, Power BI can consume it to create a TempDB monitoring view.

Recommended Visual Setup

Connect Power BI to the Dashboard_TempDB table.

Use a Gauge visual.

Set Value = Min of FreeSpacePercentage
(shows the worst TempDB case across the entire environment)

Conditional Formatting
Value:	Gradient color

0%	🔴 Red

50%	🟡 Yellow

100%	🟢 Green

This lets DBAs quickly see if any server is nearing TempDB exhaustion — a common root cause of blocking and failed operations.
The same goes for the T: drive OS disk space on the bottom, becuase TEMPDB files usually get stored in T: drive, it is helpful to monitor disk space there.
(Server names hidden)

<img width="1532" height="846" alt="image" src="https://github.com/user-attachments/assets/684fc13d-1fb9-4bb7-85d2-312a0c62ee4b" />

## Summary

This TempDB workflow demonstrates how the SQL Big Brother system gathers, stores, and visualizes telemetry. Even a basic pipeline like this provides:

- Cross-server TempDB visibility
- Early warning signals
- Automated alerting potential
- Dashboards that refresh every 5 minutes
- An oppurtunity for a newcomer like me to learn more about Tempdb and how critical this is to SQL Server.

As I grew from Jr. DBA to more advanced engineering work, these early scripts became the backbone of a much larger telemetry ecosystem.




