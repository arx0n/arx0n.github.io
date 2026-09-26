# PICKLERICK

## Information

- **Platform:** TryHackMe
- **Difficulty:** Easy
- **System:** Linux
- **IP:** `10.130.186.22`
- **Objective:** Find **3 secret ingredients** hidden throughout the machine

## Reconnaissance

First, we create a directory for the machine. Then, using `mkt`, which is a function I have defined in my `zshrc`, we create a series of directories that help me keep everything organized.

![PICKLERICK](/images/THM/PICKLERICK/mkt.png) 

Once this is set up, we can start working on the machine. First, let's check whether we have connectivity to it before moving on to the reconnaissance phase. We'll send an ICMP request to the target machine.

![PICKLERICK](/images/THM/PICKLERICK/ping.png) 

There are two important things we can take from this. First, we can determine that we are dealing with a Linux machine based on the TTL. Remember: **Linux - TTL = 64 | Windows - TTL = 128**.

When sending the ICMP request, the packet passes through an intermediate node which, among other things, causes the TTL to decrease by one. Based on the proximity, we can determine that we are dealing with a Linux machine. It also tells us that 1 packet was sent and 1 was received, meaning that we have connectivity to the machine.

## Enumeration

``` bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.130.186.22 -oG allPorts
``` 

![PICKLERICK](/images/THM/PICKLERICK/nmap.png) 

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

As we can see, it reports that ports **22** and **88** are open. We can see which service is running on each port; in this case, **22 → SSH** and **88 → HTTP**.

I'd also like to know which version is running, so let's launch another **nmap** scan focused on identifying the version and service running on each port.

```bash
nmap -sCV -p,22,80 10.130.186.22
```

![PICKLERICK](/images/THM/PICKLERICK/nmap2.png) 

<details>
<summary>Command Explanation</summary>

- `-sCV` → Runs basic reconnaissance scripts while also detecting the versions of the services running on the specified ports.
</details>

## Initial Access

Let's start by checking out the web page and seeing what we can find.

![PICKLERICK](/images/THM/PICKLERICK/web.png) 

Since I don't see anything unusual, let's take a look at the page's source code. Besides helping us understand how the website is built, we can sometimes find some interesting things in there...

![PICKLERICK](/images/THM/PICKLERICK/web2.png) 

EUREKA! As you can see, someone left a username commented out. Pretty careless on the developer's part, but hey, we'll take it.

Let's keep investigating to see what we can do with this username and whether we can find the secret ingredients. Before blindly looking around the website, I'm going to perform a **directory discovery** using `gobuster`.

``` bash
gobuster dir -u http://10.130.186.22 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt
```

![PICKLERICK](/images/THM/PICKLERICK/gobuster.png) 

<details>
<summary>Command explanation</summary>

- `dir` → Indicates that we are going to perform directory enumeration.
- `-u` → Specifies the URL we want to scan.
- `-w` → Indicates the wordlist we are going to use for the enumeration.
- `-x` → Allows us to specify the file extensions we want to search for.
</details>

As we can see, there are some interesting things here: a **"login"** panel, an **assets** directory, and the one that catches my attention, **robots.txt**. Let's investigate a little further.

![PICKLERICK](/images/THM/PICKLERICK/robots.png) 

There's some weird text here. I'm not entirely sure what it is, but since I still don't really have a clear direction, let's go straight to the **login** panel and see if we can find anything.

![PICKLERICK](/images/THM/PICKLERICK/login.png) 

Since there doesn't seem to be any obvious vulnerability, let's try working with what little information we have. I'll use the username we found earlier, **R1ckRul3**, and the password from `robots.txt` that we found before, **Wuabbalubbadubdub**.

![PICKLERICK](/images/THM/PICKLERICK/login2.png) 

Not bad! We have access to what appears to be a **remote command execution** interface. Let's see how much we can get from here.

## Exploitation

![PICKLERICK](/images/THM/PICKLERICK/whoami.png) 

As you can see, this confirms that we have access through **remote command execution**, or more technically, **RCE**. So let's try to find the ingredients.

![PICKLERICK](/images/THM/PICKLERICK/pwd.png) 

By running `pwd`, we can see the current directory we're in. Let's run `ls` to list the contents of **/var/www/html**.

![PICKLERICK](/images/THM/PICKLERICK/ls.png) 

We have the first ingredient! Let's try to find out what it is. We can use the `cat` command to read the file.

![PICKLERICK](/images/THM/PICKLERICK/cat.png) 

Well! Looks like they're not going to make this easy for us. Since the file is located in **/var/www/html**, we could try accessing it through the web or use an alternative command to **cat**. I'm going to use `less` because I find it simpler and more convenient.

![PICKLERICK](/images/THM/PICKLERICK/less.png) 

We have the first ingredient!! Two more to go. If you noticed, there were more files when we ran `ls`, so let's see what else we can find.

![PICKLERICK](/images/THM/PICKLERICK/clue.png) 

As you can see, we have another text file called **clue.txt**. This time, I'll access it through the web so we can see another way of doing the same thing.

![PICKLERICK](/images/THM/PICKLERICK/clue2.png) 

It gives us what appears to be a clue, which we're going to use. Let's try to find files on the system containing the word **ingredient**. For that, we'll use `find`.

``` bash
find / -iname "ingredient" 2>/dev/null
```

![PICKLERICK](/images/THM/PICKLERICK/find.png) 

<details>
<summary>Command explanation</summary>

- `find /` → Searches for files and directories starting from the system root.
- `-iname "ingredient"` → Searches for items named `ingredient`, without distinguishing between uppercase and lowercase letters.
- `2>/dev/null` → Redirects permission errors and other errors to the **/dev/null** black hole so they don't appear on screen.

</details>

There we go! We have the second ingredient. Since I don't know what format it's in, I'll use the `strings` command right away so we don't waste any time.

![PICKLERICK](/images/THM/PICKLERICK/2nd.png) 

We now have the second ingredient! Since the `find` command didn't report any more ingredients, I suspect the third ingredient will only be accessible as the **root** user and will have a name we wouldn't expect. So let's go straight into privilege escalation.

## Privilege Escalation

Alright, let's start with some basic commands to see what we can do. I'll start with the classic `sudo -l` to see which commands we can run through sudo and whether a password is required.

![PICKLERICK](/images/THM/PICKLERICK/sudo.png) 

Well, that was easy. As we can see, the user has access to all commands and doesn't need a password. My first thought is to list the contents of **root's** home directory using `sudo ls /root`.

![PICKLERICK](/images/THM/PICKLERICK/lsroot.png) 

And as you can see, I was right. We wouldn't have been able to find the name of the third ingredient using `find` (or maybe we could), so proceeding this way worked out well. Let's list it and that should be it.

![PICKLERICK](/images/THM/PICKLERICK/root.png) 

We now have the third and final ingredient, and we've completed the machine.

## Conclusion

A fairly straightforward machine where we relied on basic web reconnaissance, gained access to a panel that allowed us to execute commands, and managed to find all three secret ingredients.

## CHALLENGE

As you can see, we found all three ingredients through the web, which is not bad, but we never actually became **root** in a more direct way. So here's a challenge for you: try turning the **RCE** into a **Reverse Shell**. This way, we can gain full access to the system instead of just getting what we need to complete the machine. Good luck!

![PICKLERICK](/images/THM/PICKLERICK/reto.png) 

The IP address changed because my connection to the machine dropped while I was writing the writeup, so I had to spawn it again.