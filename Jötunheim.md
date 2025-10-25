```
 ▐▄▄▄      ▄▄▄▄▄▄• ▄▌ ▐ ▄  ▄ .▄▄▄▄ .▪  • ▌ ▄ ·. ▄▄▄  
  ·██▪     •██  █▪██▌•█▌▐███▪▐█▀▄.▀·██ ·██ ▐███▪▀▄ █·
▪▄ ██ ▄█▀▄  ▐█.▪█▌▐█▌▐█▐▐▌██▀▐█▐▀▀▪▄▐█·▐█ ▌▐▌▐█·▐▀▀▄ 
▐▌▐█▌▐█▌.▐▌ ▐█▌·▐█▄█▌██▐█▌██▌▐▀▐█▄▄▌▐█▌██ ██▌▐█▌▐█•█▌
 ▀▀▀• ▀█▄▀▪ ▀▀▀  ▀▀▀ ▀▀ █▪▀▀▀ · ▀▀▀ ▀▀▀▀▀  █▪▀▀▀.▀  ▀
```
<center>Privilege Escalation techniques on windows and linux</center>

# Linux privilege escalation

## Enumeration

Enumeration is arguably the first and most important step when we gain any sort of access to a system. In this section we'll be focusing on enumeration techniques for Linux systems.

## Great commands for enumerating a system

- `hostname`
This command will output the hostname of the target, although the output value can easily be changed or produce a meaningless name, sometimes it's useful when attacking large scale corporate systems to help identify the target machine we're on. 

- `uname -a` 
Will print system info giving us any details about the kernel used by the system and could be useful when searching for vulnerabilities.

- /proc/version`
Provides essential information about the processes on the target system.

- /etc/issue
Contains info about the operating system but the file itself can easily be edited, in fact any system information file could be edited to contain misleading info.

- `ps`
Shows all the currently running process on the victim machine. Outputs helpful info such as:
- PID: The process ID (unique to the process)
- TTY: Terminal type used by the user
- Time: Amount of CPU time used by the process (this is NOT the time this process has been running for)
- CMD: The command or executable running (will NOT display any command line parameter)

The “ps” command provides a few useful options.

- `ps -A`: View all running processes
- `ps axjf`: View process tree (see the tree formation until `ps axjf` is run below)

![](https://i.imgur.com/xsbohSd.png)  

- `ps aux`: The `aux` option will show processes for all users (a), display the user that launched the process (u), and show processes that are not attached to a terminal (x). Looking at the ps aux command output, we can have a better understanding of the system and potential vulnerabilities.


- `env`

The `env` command will show environmental variables.

![](https://i.imgur.com/LWdJ8Fw.png)  

The PATH variable may have a compiler or a scripting language (e.g. Python) that could be used to run code on the target system or leveraged for privilege escalation.

- `sudo -l`
Some systems allow users to run commands with root privileges. This command lists all the commands you can run with. `sudo`

- `id`
Provides a general overview of user privilege level and group memberships. Can also be used to ID other users.

- /etc/passwd
Shows you all the users on the system, but some output could be quite redundant such as system or service users. Usually we grep for 'home' as most real users have a home directory under which they store their files.

- `history`
Reveals earlier commands and can give us some insight into the target system.

- `ifconfig`
The target system may be a pivoting point to another network. The `ifconfig` command will give us information about the network interfaces of the system. The example below shows the target system has three interfaces (eth0, tun0, and tun1). Our attacking machine can reach the eth0 interface but can not directly access the two other networks.

![](https://i.imgur.com/hcdZnwK.png)

This can be confirmed using the `ip route` command to see which network routes exist.

![](https://i.imgur.com/PSrmz5O.png)

### netstat

Following an initial check for existing interfaces and network routes, it is worth looking into existing communications. The `netstat` command can be used with several different options to gather information on existing connections.

  

- `netstat -a`: shows all listening ports and established connections.
- `netstat -at` or `netstat -au` can also be used to list TCP or UDP protocols respectively.
- `netstat -l`: list ports in “listening” mode. These ports are open and ready to accept incoming connections. This can be used with the “t” option to list only ports that are listening using the TCP protocol (below)

  

![](https://i.imgur.com/BbLdyrr.png)

  

- `netstat -s`: list network usage statistics by protocol (below) This can also be used with the `-t` or `-u` options to limit the output to a specific protocol.

  

![](https://i.imgur.com/mc8OWP0.png)

  

- `netstat -tp`: list connections with the service name and PID information.

  

![](https://i.imgur.com/fDYQwbW.png)

  

This can also be used with the `-l` option to list listening ports (below)

  

![](https://i.imgur.com/JK7DNv0.png)

  

We can see the “PID/Program name” column is empty as this process is owned by another user.

Below is the same command run with root privileges and reveals this information as 2641/nc (netcat)

![](https://i.imgur.com/FjZHqlY.png)`   `

- `netstat -i`: Shows interface statistics. We see below that “eth0” and “tun0” are more active than “tun1”.

![](https://i.imgur.com/r6IjpmZ.png)

  

  

The `netstat` usage you will probably see most often in blog posts, write-ups, and courses is `netstat -ano` which could be broken down as follows;

- `-a`: Display all sockets
- `-n`: Do not resolve names
- `-o`: Display timers

  

![](https://i.imgur.com/UxzLBRw.png)

  

  

### find Command

Searching the target system for important information and potential privilege escalation vectors can be fruitful. The built-in “find” command is useful and worth keeping in your arsenal.

Below are some useful examples for the “find” command.

**Find files:**

- `find . -name flag1.txt`: find the file named “flag1.txt” in the current directory
- `find /home -name flag1.txt`: find the file names “flag1.txt” in the /home directory
- `find / -type d -name config`: find the directory named config under “/”
- `find / -type f -perm 0777`: find files with the 777 permissions (files readable, writable, and executable by all users)
- `find / -perm a=x`: find executable files
- `find /home -user frank`: find all files for user “frank” under “/home”
- `find / -mtime 10`: find files that were modified in the last 10 days
- `find / -atime 10`: find files that were accessed in the last 10 day
- `find / -cmin -60`: find files changed within the last hour (60 minutes)
- `find / -amin -60`: find files accesses within the last hour (60 minutes)
- `find / -size 50M`: find files with a 50 MB size

This command can also be used with (+) and (-) signs to specify a file that is larger or smaller than the given size.

![](https://i.imgur.com/pSMfoz4.png)

The example above returns files that are larger than 100 MB. It is important to note that the “find” command tends to generate errors which sometimes makes the output hard to read. This is why it would be wise to use the “find” command with “-type f 2>/dev/null” to redirect errors to “/dev/null” and have a cleaner output (below).

![](https://i.imgur.com/UKYSdE3.png)

  

Folders and files that can be written to or executed from:

- `find / -writable -type d 2>/dev/null` : Find world-writeable folders
- `find / -perm -222 -type d 2>/dev/null`: Find world-writeable folders
- `find / -perm -o w -type d 2>/dev/null`: Find world-writeable folders

The reason we see three different “find” commands that could potentially lead to the same result can be seen in the manual document. As you can see below, the perm parameter affects the way “find” works.

![](https://i.imgur.com/qb0klHH.png)  

- `find / -perm -o x -type d 2>/dev/null` : Find world-executable folders

Find development tools and supported languages:

- `find / -name perl*`
- `find / -name python*`
- `find / -name gcc*`

Find specific file permissions:

Below is a short example used to find files that have the SUID bit set. The SUID bit allows the file to run with the privilege level of the account that owns it, rather than the account which runs it. This allows for an interesting privilege escalation path,we will see in more details on task 6. The example below is given to complete the subject on the “find” command.

- `find / -perm -u=s -type f 2>/dev/null`: Find files with the SUID bit, which allows us to run the file with a higher privilege level than the current user.
---
## Automated privesc tools

Several tools can help you save time during the enumeration process. These tools should only be used to save time knowing they may miss some privilege escalation vectors. Below is a list of popular Linux enumeration tools with links to their respective GitHub repositories.

The target system’s environment will influence the tool you will be able to use. For example, you will not be able to run a tool written in Python if it is not installed on the target system. This is why it would be better to be familiar with a few rather than having a single go-to tool.

- **LinPeas**: [https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS)
- **LinEnum:** [https://github.com/rebootuser/LinEnum](https://github.com/rebootuser/LinEnum)[](https://github.com/rebootuser/LinEnum)
- **LES (Linux Exploit Suggester):** [https://github.com/mzet-/linux-exploit-suggester](https://github.com/mzet-/linux-exploit-suggester)
- **Linux Smart Enumeration:** [https://github.com/diego-treitos/linux-smart-enumeration](https://github.com/diego-treitos/linux-smart-enumeration)
- **Linux Priv Checker:** [https://github.com/linted/linuxprivchecker](https://github.com/linted/linuxprivchecker)