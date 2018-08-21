---
title: "Powershell: Self signed certificate"
excerpt: "Generate a self signed cert using powershell"
tags: 
    - Powershell
    - Certificate
    - Public Key
    - Private Key
date:   2018-08-05 01:00:00 -0600
#categories: powershell
---

My favourite powershell script to generate Self-signed Certificate.

Please make sure the following fields are updated with appropriate value.
* `C=[country]` - your country, eg `C=United States`
* `O=[organization]` - the company name, eg `O=Microsoft Corporation`
* `CN=[common name]` eg. `C=Microsoft`
* `L=[location]` eg. `C=Redmond`,

```bash
#Set expiry date to be 3 years from today's date
$expiryDate = (Get-Date).AddYears(3) 
$cert = New-SelfSignedCertificate -DnsName "C=[country], O=[organization], CN=[common name], L=[location]" -CertStoreLocation "cert:\LocalMachine\My" -notafter $expiryDate
Write-Host $cert.Thumbprint

#export the public key to a file
Export-Certificate -Cert $cert -FilePath c:\publickey.cer

#export the private key to a file with password protection
$mypwd = ConvertTo-SecureString -String "mysupersecurepassword" -Force -AsPlainText
Export-PfxCertificate -Cert $cert -FilePath c:\privatekey.pfx -password $mypwd

#export the public key a base 64 encoded file
$content = @(
    '-----BEGIN CERTIFICATE-----'
    [System.Convert]::ToBase64String($cert.RawData, 'InsertLineBreaks')
    '-----END CERTIFICATE-----'
)
$content | Out-File -FilePath c:\cert.txt -Encoding ascii
```
