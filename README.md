# Active Directory: Bulk Update users properties
This routine executes a bulk update on Active Directory users, by Organizational Unity queries. This was tested in scenario with 1 hundred or user inner domains with quickly miliseconds of execution.

```powershell
Import-Module ActiveDirectory
$logResult = ""
$i = 0
$j = 0
$m =0

##This example has a domain controller name as 'FirstNameDomainController.LastNameDC'
##Mount Active Directory query statament for users inners this domains
$OUs =
  'ou=FirstChildOU, ou=RootOU, dc=FirstNameDomainController, dc=LastNameDC',
  'ou=SecondChildOU, ou=FirstChildOU, ou=RootOU, dc=FirstNameDomainController, dc=LastNameDC'
  
##Get Active Directory users by defined query
$adUsers = $OUs | ForEach { Get-ADUser -Filter {(Enabled -eq $true)} -Properties * -SearchBase $_ }

##At this foreach loop, will update the mobile user numbers, adding '9' in left,
##recently change in all mobile and telecommunications service from Brasil at 2015 year.

foreach ($user in $adUsers){
    $cel = [string]$user.mobile
    $profile = [string]$user.SAMAccountName
    $currentUser = [Environment]::UserName

    If(!([string]::IsNullOrEmpty($cel)) -and $cel.Length -lt 16){

        $mobile = $cel.Replace(") ", ") 9")

        Set-ADUser -Identity $profile -mobile $mobile
        
        #Montando String do Log
        $i++
        $number = "{0:00000}" -f $i
        $logResult += "`r`n"
        $logResult += $number + ": Success to update mobile number of the user: " + $profile
        $j++
     }
     Else{
     
        #Montando String do Log
        $i++
        $number = "{0:00000}" -f $i
        $logResult += "`r`n"
        $logResult += $number + ": The field MOBILE of the user " + $profile + " is empty or had updated."
        $m++
     }

}
$logResult += "`r`n"
$logResult += "Total de usuários atualizados: "$j
$logResult += "`r`n"
$logResult += "Total de usuários inalterados: "$m

New-Item C:\Users\$currentUser\Desktop\ChangeMobileNumber_LOG.txt -type file -force -value $logResultado    
Write-Host "Look at this LOG in C:\Users\"$currentUser"\Desktop\ChangeMobileNumber_LOG.txt"
```
