
Script to migrate settings from user secrets to azure key vault

dotnet user-secrets list | ? {$_.Contains('Ftp')} | %{$s = $_.Split('='); $k = $s[0].Trim().Replace(':', '--'); $v = $s[1].Trim(); Set-AzureKeyVaultSecret -VaultName 'boojum-dev' -Name $k -SecretValue (ConvertTo-SecureString -AsPlainText -String $v -Force)}