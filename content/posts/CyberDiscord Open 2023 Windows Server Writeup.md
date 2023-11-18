---
title: "CyberDiscord Open 2023 Windows Server Writeup"
date: 2023-06-10
description: "A writeup for the CyberDiscord Open 2023 Windows Server image."
tags: ["CyberDiscord Open", "CyberPatriot", "Writeup"]
type: post
weight: 20
showTableOfContents: true
---

# Forensics
## Forensics 1
Question:

Hash functions are one-way functions. Secure hash functions have several properties such as collision resistance, preimage resistance, and second preimage resistance. Hashes are the output of hash functions and are typically displayed to the user in hex.

There is a file somewhere on this machine named "cain.zip".

What is the MD5 hash of this file?

This question is pretty straightforward. The first step is to find this "cain.zip" file. For this, I simply installed the [Everything tool from voidtools](https://www.voidtools.com/) and searched cain.zip to reveal it's in C:\Windows. After going there with Powershell I ran
```
Get-FileHash -Algorithm MD5 cain.zip
```
To get the md5 which was `c700ff8bdf9faef4df4b4fdd7b428376`

## Forensics 2
Question:

Create a string that satisfies the following regular expression:

```
^(\d{10}--?[^aA-Zb-mt-zn-prs]{4,5})\s+(ab)\s*(?:[\d]+)(\\\$\\)\.\s+.*\3\2\1p+$
```

Oh, boy don't we all love regex. Well first off I found a website able to test my regex, the best one I found was [https://regex101.com/](https://regex101.com/)

For some background on regex, `()` mark a capturing group that breaks up the regex into parts. There is also a non-capturing group that does pretty much the same thing. In our case, this is still an important distinction that will be apparent later. So our broken-up regex is as follows:
```
Capturing Group 1
(\d{10}--?[^aA-Zb-mt-zn-prs]{4,5})
```

Let's tackle this piece by piece starting with capturing group 1. `\d` matches digits and `{10}` means it wants 10 of the previous token which was `\d`. So together `\d{10}` will match with any 10-digit string. For simplicity let's go with `1234567890`. Next is `--?`. Our string must have at least 1 `-` to match with the first `-` in the regex. However the `?` makes it so that we need 0-1 of the previous token, so we don't need to have a second one. So `--?` matches with `-`. Lastly `[^aA-Zb-mt-zn-prs]{4,5}` matches 4-5 characters that are not in the brackets. There are many we can use but since everyone likes money lets go with `$$$$`. So our string for group 1 is `1234567890-$$$$`.

```
\s+
```

`\s` just matches with whitespace characters like ` `. `+` means we can have a 1-unlimited number of them.

```
Capturing Group 2
(ab)
```

`ab` matches exactly with `ab`, so our string for group 2 is just `ab`.

```
\s*
```

`\s*` means we can have a 0-unlimited number of spaces, so let's just stick to 0.

```
Non-Capturing Group
(?:[\d]+)
```

`\d` matches with any number, let's use `1`, the rest of the symbols don't matter.

```
Capturing Group 3
(\\\$\\)
```

`\\\$\\` matches with `\$\` which will be our string for group 3.

```
\.\s+.*\3\2\1p+$
```

`\.s+` matches with `. `. The `.*` matches with anything so we'll just leave it blank. `\3` will match with whatever we had as our string for capturing group 3, so `\$\`. Similarly `\2` matches with `ab` and `\1` matches with `1234567890-$$$$`. Finally `p+` will just match with `p`. So our last string will be `. \$\ab1234567890-$$$$p`

Now combining all of these strings we get our answer which is `1234567890-$$$$ ab1\$\. \$\ab1234567890-$$$$p`.

## Forensics 3
There is a backdoor within the DNS server previously planted by our old CEO. The backdoor is programmed to run commands from a certain file location.

What is the absolute path of the file needed to run a command under the DNS server?

Unfortunately I misunderstood this forensics question and wasn't able to solve it during the competition.

# Vulnerabilites
## User Auditing - 7
User auditing without a doubt was the most annoying part of the image as it had 29 authorized users.

Going through the readme and comparing it to the users on the system, I found 4 unauthorized users. They were `rdiaz`, `ethawne`, `mmerlyn`, and `aheart`. Deleting them gave me 4 vulns.

Looking at the passwords of the users I noticed that the `hwells` user had a very insecure password. So I changed their password and got another vuln.

Lastly, I went through the different administrator groups and found `cramon` as an administrator and `tqueen` as an enterprise admin, both unauthorized. Removing them from these groups got me another 2 vulns.

In total, I was able to find 7 user auditing vulns, which I'm pretty sure is all there was.

## Prohibited files - 2
Next, I went through the many unwanted software present on the computer. First I removed the `Cain.zip` from Forensics 1. Cain and Abel is a password cracking tool for Windows so it shouldn't be on the system.

Again using the Everything tool, I selected media files and filtered them to not show files in `C:\WinSxS` as there's a lot of junk in there. Scrolling down a bit or sorting by date will reveal 4 media files. Deleting them got me another vuln. 

I checked other types of files, looked for alternate data streams, checked for shadow copies, etc. but found nothing else.

## Unwanted Software - 5
Then I went through and uninstalled/deleted `MineSweeper, TeamViewer, Nmap, CCleaner, and Radmin Server,` racking up another 5 vulns.

I didn't check individual user's local installs or portable applications meaning there could have possibly been more but I managed to find 5 in total.

## Application Updates - 0

This one was a little rough. I went through and updated a bunch of stuff all to get nothing. Updating Firefox most likely would have given me a point however it was broken and I couldn't get it to work. I could have uninstalled Firefox and installed it again fresh but it seemed like too much work and I didn't do it. I most likely also missed updating one of the software installed netting me 0 vulns in this category.

## Operating System Update - 1
This one was just enabling Windows automatic updates, nothing special.

## Malware - 3

As explained in Forensics 3, all I did was turn the defender on from group policy, change a few settings from the GUI, and run a scan. It detected both the `tini backdoor(:DNS.exe)` and `rcat backdoor`. I also ran a Malwarebytes scan which detected the sticky keys backdoor. This backdoor works by renaming a shell, in this case, `cmd.exe` to `sethc.exe` and placing it in system32. Now when you press shift in succession windows will try and open `sethc.exe` and instead will open the shell with `nt authority` permissions. Removing all three got me 3 more vulns.

There could have been many more that I didn't find as malware and persistence mechanisms are very widespread. Autoruns is usually a go-to for detecting persistence however even it isn't perfect. It is more about just knowing where to look and what to look for.

## Application Security - 6

We don't talk about this category as I suck at it. 

### Remote Access 
I decided to enable network-level authentication. This will force the client to authenticate earlier in the connection process enhancing security. To enable this you can either do it via group policy or registry. I'm a big boy so I primarily use the registry. `HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services\UserAuthentication` must be set to `1` to enable NLA.

The next remote access vuln I found was not allowing LPT port redirection. If you do allow this a remote access user would be able to redirect server traffic to a lpt port. The use case for this is rare and the image didn't require this functionality so disabling it lessens the attack surface on the machine. To disable this you must set `HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services\fDisableLPT` to `1`.

### Firefox
These are self-explanatory so I won't go into depth. Enabling the pop-up blocker and forcing the use of HTTPS websites only enhances the security by lessening the chance a user clicks on something they shouldn't or leak sensitive information through unencrypted traffic.

### SMB
Removed the insecure smb 1.0 version.

### DNS
DNS cache pollution or cache poisoning is a common attack when it comes to DNS servers. The below image illustrates how the attack works in theory

![poisoning](/images/CyberDiscord-Open-2023-Windows-Server-Writeup/poisoning.png)

![poisoned](/images/CyberDiscord-Open-2023-Windows-Server-Writeup/poisoned.png)

To prevent such an attack we can set `HKLM\System\CurrentControlSet\Services\DNS\Parameters\SecureResponses` to `1`.

This pretty much maxed out my basic knowledge of critical services but there are probably a dozen others.

## Defensive Countermeasures - 2
Most attacks start with a simple phish. So preventing users from falling for them is extremely important. We can block dangerous websites which are most likely phishing campaigns through windows defender by setting `HKLM\SOFTWARE\Policies\Microsoft\Windows Defender\Windows Defender Exploit Guard\Network Protection\EnableNetworkProtection` to `1`.

Other threats could be on a system level so making it harder for 0-day exploits to be carried out is also an important security step. One of these settings is randomized memory allocation or ASLR. You can set this either through the GUI by going to the exploit protection settings or by using `Set-ProcessMitigation -System -Enable ASLR`.

There was probably more but I didn't get any.

## Service Auditing - 3
Many services are either not needed or directly decrease security. In general, if you don't need something disabling it will lower the attack surface making your machine more secure. The three services which weren't needed and provided points were the `World Wide Web Publishing service`, `Net. TCP Port Sharing service`, and `Telephony service`.

## Local Policy - 4
Man-in-the-middle attack(MITM) occurs when network traffic isn't signed and can be spoofed easily. We can prevent this by having LDAP sign all of its communications. To do this you can set `HKLM\System\CurrentControlSet\Services\NTDS\Parameters\LDAPServerIntegrity` to `2` or set it through security options in the GPO.

Another MITM attack vector can come from SMB traffic so we should sign that as well. To do this you can set `HKLM\System\CurrentControlSet\Services\LanmanWorkstation\Parameters\RequireSecuritySignature` to `1` or set it through security options in the GPO.

As computers get stronger and many encryption suites become insecure. RC4 and DES are examples of this. To prevent Windows from using these encryption suites for Kerberos you can set `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\Kerberos\Parameters\SupportedEncryptionTypes` to `2147483640` or set it through security options in the GPO.

DOS attacks come in various forms, with TCP SYN floods being one of the most common. In these attacks, the attacker floods the server with SYN packets without their corresponding ACK packets. This causes the server to be overwhelmed. To prevent this we can set a max on the number of these types of TCP packets by setting `HKLM\System\CurrentControlSet\Services\Tcpip\Parameters\TcpMaxDataRetransmissions` to `3`or setting it through security options in the GPO.

I know I know, I missed a bunch in audit policy and user rights, but unfortunately, I broke pretty much everything after running my script haha. That's what I get for only testing it on Windows 10 and not AD Windows Server.

## Account Policy - 0
Again I couldn't do this because I broke windows.

## Uncategorized OS Settings - 2
You really shouldn't be sharing the entirety of the `C:\` at all. Removing the share gave a vuln.

Password hashes are stored in a file at `C:\Windows\NTDS\ntds.dit`, which means if someone can read this file they could try and crack the passwords. So it is best to not allow normal users to view this file. Looking at the permissions I saw that domain users could read it. After removing this permission I got my last vuln and shut the image off minutes before the 4-hour mark.


# Conclusion
I ended with 37/62 vulns and 60/100 points.

Overall I thought I did pretty badly on this one ngl. For future notice, don't think that if a script doesn't break Windows 10 it won't wreck a AD server. I could have probably gotten another dozen vulns if I didn't do that and knew how to do app sec. The 4-hour time limit was an experience too. 