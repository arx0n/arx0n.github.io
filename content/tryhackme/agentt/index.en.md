# Agent T

## Information

- **Platform:** Try Hack Me
- **Difficulty:** Easy
- **System:** Linux
- **IP:** `10.129.138.6`
- **Objective:** Find the **root** user's flag.

## Reconnaissance

First, we create a directory for the machine. Afterwards, using `mkt`, which is a function I have defined in my `zshrc`, we create a series of directories that will help me stay organized.

![AGENTT](/images/THM/AGENTT/mkt.png) 

Once we have this, we can start working on the machine. Let's check whether we have connectivity to it and proceed with the reconnaissance phase. We are going to send an ICMP trace to the target machine.

![AGENTT](/images/THM/AGENTT/ping.png) 

We can get two important things from this. First, we know that we are dealing with a Linux machine based on the TTL. Remember: **Linux - TTL = 64 | Windows - TTL = 128**. 

When tracing the ICMP path, the packet passes through an intermediate node which, among other things, causes the TTL to decrease by one. Based on the proximity, we can determine that we are dealing with a Linux machine. It also tells us that 1 packet was sent and 1 was received, so we have connectivity with the machine.

## Enumeration

``` bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.129.138.6 -oG allPorts
``` 

![AGENTT](/images/THM/AGENTT/nmap.png) 

<details>
<summary>Command explanation</summary>

- `-p-` → Scans the entire port range, all 65535 ports.
- `--open` → Ports can be in different states: open, closed, or filtered. This parameter makes it only consider the ones it sees as open.
- `-sS` → Performs a SYN scan, making the scan much faster while also being aggressive and accurate.
- `--min-rate 5000` → Sets a minimum rate of 5000 packets per second. Combined with the previous parameter, this makes the scan extremely fast.
- `-vvv` → Provides a little more information.
- `-n` → Prevents DNS resolution.
- `-Pn` → Prevents host discovery.
- `-oG` → Exports the scan in grepable format.
</details>

As we can see, port **80** is open. We can see which service is running on it, in this case **80 → HTTP**.

I would also like to know which version is running, so let's run another **nmap** focused on telling me which version and service are running on each port.

```bash
nmap -sCV -p,22,80 10.129.138.6
```

![AGENTT](/images/THM/AGENTT/nmap2.png) 

<details>
<summary>Command explanation</summary>

- `-sCV` → Ejecuta scripts básicos de reconocimiento a la par que detecta las versiones para los puertos que especifiquemos.
</details>

This particular version of **PHP** sounds familiar, and I think there may be something we can take advantage of, but let's investigate the website a little first. 

## Initial Access

Let's start by investigating the website to see what we can find. Before diving in, I'm going to use a command called `whatweb`, which gives us some information about it.

![AGENTT](/images/THM/AGENTT/whatweb.png) 

Como veis nos complementa un poco más las versiones del servidor web, frameworks, cms, lenguajes, librerías... Es bastante útil para tener una idea de como esta montada la web.
Let's take a look at what this machine has to offer.

![AGENTT](/images/THM/AGENTT/web.png) 

Since I don't see anything unusual, let's try directory enumeration with `gobuster`. 

``` bash
gobuster dir -u http://10.129.138.6 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt
```

![AGENTT](/images/THM/AGENTT/web2.png) 

<details>
<summary>Command explanation</summary>

- `dir` → Indicates that we are going to perform directory discovery.
- `-u` → Specifies the URL we want to scan.
- `-w` → Specifies the wordlist we are going to use for the discovery process.
- `-x` → Permite especificar las extensiónes que queremos buscar.
</details>

Interestingly, it reports that all responses returned a 200 status code, so there is no point in trying to discover directories this way.
Let's use a tool called `searchsploit` to see whether **PHP** is vulnerable.

## Exploitation

``` bash
searchsploit php 8.1.0-dev
```

![AGENTT](/images/THM/AGENTT/searchsploit.png) 

As we can see, there is an **RCE** vulnerability, meaning that we can execute commands remotely. Let's download it using the `-m` parameter followed by the exploit number without its extension, in this case `49933`.

![AGENTT](/images/THM/AGENTT/49933.png) 

Let's take a look at the file's code:

![AGENTT](/images/THM/AGENTT/catt.png) 

Este script aprovecha una vulnerabilidad de PHP 8.1.0-dev para ejecutar comandos mediante la cabecera User-Agent y conseguir una pseudo-shell en el servidor. Vamos a ejecutarlo con `python3` y a ver que nos reporta.

## Privilege Escalation

![AGENTT](/images/THM/AGENTT/sploit.png) 

As you can see, this confirms that we have access through **remote command execution**, or, more technically, an **RCE**. I tried to move between directories, but as you can see, we cannot. On the other hand, we are already the `root` user. 

I'm going to try to find a file containing the flag using the `find` command.

``` bash
find / -type f -name "*.txt" 2>/dev/null
```

![AGENTT](/images/THM/AGENTT/find.png) 

<details>
<summary>Command explanation</summary>

- `find /` → Searches for files and directories starting from the root of the system.
- `-type f` → Specifies what we want to search for; in this case, `f` means we want to search for files.
- `-name "*.txt"` → Specifies the file name. In this case, we use `*` to tell it to search for **all** `.txt` files.
- `2>/dev/null` → Redirects permission errors and other errors to the **/dev/null** black hole so they do not appear on screen.

</details>

As you can see, it finds a file called `flag.txt` located in the root directory. We are going to use `cat` to display its contents, and that will be it.

## Conclusion

Máquina bastante sencilla, hemos tirado de reconocimiento web básico, hemos usado la herramienta `searchsploit` para buscar vulnerabilidades y mediante un programa con `python` hemos entrado por todo lo alto como usuario **root**. 

## Comments

{{< comments >}}