Azure automation.

1. copy staging db to a local one
Using Resource Manager and Powershell make this task almost trivial. 
Login-AzureRMAccount
Select-AzureRmSubscription

2. Azure automation to run your PS scripts in Azure from anywhere. Just create an Automation Account. 
And change Sample script to yours.
Add webhook to allow invoking that script from outside of Azure.

3. Backup / Restore of Azure SQL Database. Good intro: https://mikhail.io/2016/10/azure-sql-databases-backups-disaster-recovery-import-export/

https://blogs.msdn.microsoft.com/azuresqldbsupport/2017/02/07/automate-export-azure-sql-db-to-blob-storage-use-automation-account/


https://github.com/Azure/azure-powershell/issues/4192

yes, you need to add script that updates version of PS modules, publish it and execute, because otherwise you'll get weird errors.