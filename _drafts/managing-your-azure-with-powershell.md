scripts: 



('docker', 'dockerinst') | %{  Remove-AzureRmResourceGroup -ResourceGroupName $_  -Confirm}