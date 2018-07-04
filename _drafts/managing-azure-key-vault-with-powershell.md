---
title: "Managing Azure Key Vault with Powershell"
---

Header
===

.NET Core configuration system allows to use multiple configuration sources simultaneously.
You can 

1. Get-AzureRmKeyVault

You should see list of available vaults in your subscription. Usually we have one vault per

My way of working with configuration settings:
1. Create stuff in `appsettings.json` (or `appsettings.Development.json`). There I can
experiment with settings structure, decide whether I should create a POCO settings like 

{% highlight json %}
{
  "credentials": {
    "login": "username",
    "password": "pwd"
  }
}
{% endhighlight %}

or key-value style would be enough:

{% highlight json %}
{
  "credentialslogin": "username",
  "credentialspassword": "pwd"
}
{% endhighlight %}

```
$k = @{  "AppleCertPwd"= "PASSWORD"; "AppleCertTeamId" =  "YE2G4E3XAY"; "AppleCertPassTypeId" = "pass.steer73.boojum"}

$k.GetEnumerator() | % {dotnet user-secrets set $_.Key $_.Value}

$k.GetEnumerator() | % {Set-AzureKeyVaultSecret -VaultName $v.VaultName -Name $_.Key -SecretValue (ConvertTo-SecureString -String $_.Value -AsPlainText -Force)}

```

Install certificate: https://blogs.technet.microsoft.com/kv/2016/09/26/get-started-with-azure-key-vault-certificates/


Import-AzureKeyVaultCertificate -VaultName $v.VaultName -Name "BoojumPassCertificate" -FilePath .\BoojumPassCertificate.p12 -Password (ConvertTo-SecureString -String "PASSWORD" -AsPlainText -Force)

My failed steps. 
Code where certificate is added via byte array and passsword. Implemented it with file, works fine.
Then tried to add certificate to local storage, encountered issue that password is needed 
when exporting the certificate. Flow was "export from store" -> fill properties of API with
bytes and password. Locally it worked OK. 
Next phase -- make it running in Azure.
First step -- how to add certificate to APp service. Most obvious way is using Azure Key Vault. Adding certificate to Azure Key Vault is super easy with `Import-AzureKeyVaultCertificate`. More details you can find in [internet](https://blogs.technet.microsoft.com/kv/2016/09/26/get-started-with-azure-key-vault-certificates). But the next question is how can you use them in app service? We use ARM and it seemed natural to be able to add certificate from key vault to app service during deployment. And ARM pretends to do [that](https://github.com/Azure/azure-quickstart-templates/blob/73cce4601ed599caa82adfb6b0dfe4f4dbbd6abf/201-web-app-certificate-from-key-vault/README.md). 

But the problems is that at the time of writting ARM supported only certificates [added as
secrets]((https://rahulpnath.com/blog/pfx-certificate-in-azure-key-vault/)
) to Key Vault. And that way has been declared as deprecated, while new shiny first class
citizen Key Vault support of certificates has not been supported by ARM yet.



[install certificate as a Secret]

Another issue is a format of certificate itself (https://peter.orneholm.com/post/155341445983/using-azure-key-vault-with-azure-web-apps-the).

