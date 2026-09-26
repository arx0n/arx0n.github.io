# CAP

## Information

- **Platform:** Hack The Box
- **Difficulty:** Easy
- **System:** Linux
- **IP:** `10.129.22.41`
- **Objective:** Discover the **usuario** and **root** flags

## Reconnaissance

First, we create a folder for the machine. Afterwards, using `mkt`, which is a function I have defined in my `.zshrc`, we create a series of directories that will help me keep everything organized.

![CAP](/images/HTB/CAP/mkt.png) 

Once we have this, we can start working on the machine. Let's check if we have connectivity with it so we can proceed to the reconnaissance phase. We are going to send an ICMP trace to the target machine.

![CAP](/images/HTB/CAP/ping.png) 

From this, we can get two important things. The first is that we know we are dealing with a Linux machine based on the TTL. Remember that **Linux - TTL = 64 | Windows - TTL = 128**.

When tracing the ICMP packet, it passes through an intermediate node which, among other things, causes the TTL to decrease by one. Based on its proximity, we can determine that we are dealing with a Linux machine. Apart from that, it tells us that `1` packet was sent and `1` was received, so we have connectivity with the machine.

## Enumeration

``` bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.129.22.41 -oG allPorts

``` 

![CAP](/images/HTB/CAP/nmap.png) 

<details>
<summary>Explanation of the command</summary>

- `-p-` → Scans the entire port range, all 65535 ports.
- `--open` → Ports can be in different states: open, closed, filtered. This parameter makes it only consider the ones it finds open.
- `-sS` → Performs a SYN scan, making the scan much faster, while also being aggressive and precise.
- `--min-rate 5000` → Sends packets at a minimum rate of 5000 packets per second. Combined with the previous parameter, this makes the scan run very quickly.
- `-vvv` → Gives me a little more information.
- `-n` → Prevents DNS resolution.
- `-Pn` → Prevents host discovery.
- `-oG` → Exports the scan in greppable format.

</details>


As we can see, it reports that ports **21**, **22**, and **88** are open. We can see which service is running on each port: in this case, **21 → FTP**, **22 → SSH**, and **88 → HTTP**.

I would also like to know which version is running, so we are going to launch another **Nmap** scan that will be more focused on telling me which version and service are running on each port.

```bash
nmap -sCV -p21,22,80 10.129.22.41
```
![CAP](/images/HTB/CAP/nmap2.png) 

<details>
<summary>Explanation of the command</summary>

- `-sCV` → Runs basic reconnaissance scripts while detecting the versions of the services running on the specified ports.

</details>

## Initial Access

Since I don't see anything that particularly catches my attention, I'm going to try connecting via **FTP** using basic credentials such as `admin-admin`, `admin-1234`, etc.

![CAP](/images/HTB/CAP/ftp.png) 

But nothing seems to work, so let's go directly to the web service and see what it is and what we can exploit from it.

![CAP](/images/HTB/CAP/web.png) 

As you can see, the website is some kind of cybersecurity dashboard where monitoring and security analysis seem to be taking place. We can see the user **Nathan** at the top. Other than that, I don't see anything unusual, so let's have a look around the dashboard menu.

![CAP](/images/HTB/CAP/webb2.png) 

It looks like a dashboard where you can download packet data that has been stored. I'm going to try downloading it and see what we get.

![CAP](/images/HTB/CAP/web3.png) 

It downloads a **.pcap** file with a number as its name. If you look closely, I'm in the `data` directory and there is another directory inside it, in this case **1**. I've been trying different numbers, and the one that caught my attention the most is this one:

![CAP](/images/HTB/CAP/web4.png) 

As you can see, using **/0** there is a significant number of packets. It's no longer just 1, so I'm going to download the file and see if there's anything interesting in it.

## Exploitation

![CAP](/images/HTB/CAP/pcap.png) 

I moved the file from the **Downloads** directory to my current working directory and, as you can see, it is the same file that can be downloaded from the website. Let's try to analyze its contents using `Tshark`.

![CAP](/images/HTB/CAP/tshark.png) 

As you can see, it contains quite a lot of packet information. If you look closely, at one point we can see a credential, so as soon as I noticed it, I decided to filter the output to avoid wasting time, and this is the result:

![CAP](/images/HTB/CAP/tshark2.png) 

<details>
<summary>Explanation of the command</summary>

- `tshark` → Analyzes network traffic through the terminal.
- `-r` → This parameter is used to specify the file we want to analyze.
- `grep` → Searches for text within an input or file.
- `-Ei (x)` → Allows us to use extended regular expressions, meaning that we can search for the text we want while ignoring uppercase and lowercase letters.

</details>

We have credentials! We found a credential that appears to belong to the **nathan** user, the same user we saw on the website. Let's try connecting via **SSH** and see if we can gain access to the machine.

![CAP](/images/HTB/CAP/ssh.png) 

With `sshpass`, we can provide the password directly. As we can see, the credentials are valid and we successfully gain access to the machine. I'm going to check which directory I'm currently in and retrieve the first **flag**.

![CAP](/images/HTB/CAP/txt1.png) 

As we can see, we now have the first flag. Let's see if we can escalate our privileges and become root.

## Privilege Escalation

I've been checking **SUID** permissions and haven't found anything, so, since the machine is called **Cap**, I tried looking for a **capability**.

![CAP](/images/HTB/CAP/capabi.png) 

As you can see, `python3.8` has a capability, so we can escalate our privileges by importing the **os** library. We would change the **UID** to that of root and that would be it.

![CAP](/images/HTB/CAP/root1.png) 

As I mentioned, by importing the **os** library we can execute system-level commands and, once we change the **UID** to that of **root**, we open a **bash** shell and have successfully escalated our privileges.

## Conclusion

A fairly straightforward machine where we gained initial access by extracting credentials from a **.pcap** file and subsequently escalated our privileges by taking advantage of a **capability** assigned to `python3.8`.