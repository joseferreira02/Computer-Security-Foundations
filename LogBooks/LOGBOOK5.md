# Logbook 5 - Buffer Overflow

## Task 1 - Getting familiar with shellcode


There are two builds because 32‑bit and 64‑bit targets use different ABIs — having both lets us practice exploiting each. The `-m32` option makes gcc produce a 32‑bit binary; `-z` execstack makes the stack executable so the shellcode on the stack can run (bypassing NX). The shellcode calls execve("/bin/sh", ...), which spawns a shell by placing the string and pointers on the stack and invoking the kernel. When I compiled and ran the binaries they behaved as expected. The 32‑bit binary runs on my x86_64 machine because modern systems support 32‑bit compatibility mode, so either shellcode will run depending on which binary you execute.

![Logbook 5 — Compilation and execution of shellcode](/images/logbook5/task1ExecuteSh.png)
_Figure 1 — Execute the shellcode_

## Task 2 - Understanding the Vulnerable Program

In this task the vulnerable program reads `517` bytes from a user-controlled file (`badfile`) into a local buffer of size `BUF_SIZE` (default `100`) inside `bof()` using `strcpy()` with no bounds checking. This allows an attacker to overflow the buffer and overwrite stack control data — most importantly the saved return address. Because the binary is compiled with protections disabled (`-fno-stack-protector` and `-z execstack`) and installed set‑UID root, a successful exploit that redirects execution to injected shellcode can spawn a root shell. Our objective is therefore to place shellcode in memory, overwrite the saved return address with an appropriate target address, and then test the payload.

To set up the lab we adjust the buffer length constant `L1` using the formula `L1 = 100 + 8*G`, where `G` is our group number. For our class (`G = 3`) this yields `L1 = 124`. After updating `L1` we compile the program using the `-stack1` target in the `Makefile`. Note that the compilation flags used here are the same as those used to compile the shellcode in Task 1.

![Logbook 5 — Compilation of code.c](/images/logbook5/task1ExecuteSh.png)
_Figure 2 - Compilation of the code stack.c_

## Task 3 - Launching Attack on 32-bit Program

This task was the most challenging in the lab because it required several careful steps to carry out the exploit. We began the attack inside the debugger to obtain precise addresses. First, we launched the program under GDB with `gdb stack-L1-gdb`. We then set a breakpoint at `bof` (the vulnerable function) and stepped into it. Our goal was to find two critical values: the current value of `$ebp` and the offset between `$ebp` and the local buffer (`&buffer`). Knowing these lets us compute the location of the saved return address (which is stored at `$ebp + 4` in a 32-bit stack frame) and choose the exact address to overwrite it with. We overwrite the saved return address with a pointer that lands slightly above our injected payload—into a `\x90` NOP sled—so execution slides into our shellcode.


#### The exploit script 
Our script has the following structure:

```py
#!/usr/bin/python3
import sys
shellcode= (
"" # I Need to change I
).encode(’latin-1’)
# Fill the content with NOP’s
content = bytearray(0x90 for i in range(517))
##################################################################
# Put the shellcode somewhere in the payload
start = 0 # I Need to change I
content[start:start + len(shellcode)] = shellcode
# Decide the return address value
# and put it somewhere in the payload
ret = 0x00 # I Need to change I
offset = 0 # I Need to change I
L = 4 # Use 4 for 32-bit address and 8 for 64-bit address
content[offset:offset + L] = (ret).to_bytes(L,byteorder=’little’)
##################################################################
```
The shellcode we will change to our our shellcode which is a series of bytes that will instruct the program to execute our code in our case open the shell in 32bytes to obtain the results of task1 :

***Shellcode Insertion***

The shellcode used in this exercise is a 32-byte sequence designed to spawn a ‘/bin/sh‘ shell:

```py
"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f"
"\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31"
"\xd2\x31\xc0\xb0\x0b\xcd\x80"
```

This sequence of bytes is embedded within the NOP sled. The variable start dictates the placement of the shellcode. We chose to place the shellcode as high as possible in the buffer to minimize potential addressing issues, specifically at the location: 500−length(shellcode). Given the presence of the NOP sled, the exact starting position within the sled is not critical for execution success in this controlled scenario.

***Return Address Overwrite***

A critical step in a stack-based buffer overflow is overwriting the saved Return Address on the stack.
- ***offset:*** This variable represents the byte difference between the start of the vulnerable `buffer` and the location of `$EBP`. Determining this value, typically through a debugger like GDB, is essential for precise overwriting.
- ***ret (New Return Address)***: Instead of pointing directly to the start of the shellcode, the new return address is calculated to point into the middle of the NOP sled increasing the chance of hitting the shellcode. The address chosen was $EBP+16, which guarantees a safe landing within the NOP region just above the return address location.
The address size L was set to 4 bytes because the target uses a 32‑bit architecture.

The final payload construction successfully redirects the program's execution flow. When launched within the GDB debugger, the exploit executes the shellcode, resulting in the successful spawning of a Zsh shell. 

![Logbook 5 — Exploit.py for gdb](/images/logbook5/explotBeforeENVCHECK.png)
_Figure 3 - Exploit.py for gdb_


![Logbook 5 — Attack on gdb](/images/logbook5/gdbExploitBEforeEnv.png)
_Figure 4 - Attack on gdb_

***The problem is that when we run this payload outside of the gdb what we find is that we hit a segmentation fault:***
![Logbook 5 — Attack for gdb on normal code](/images/logbook5/beforeEnvSegmentFault.png)
_Figure 5 - Attack for gdb on normal code_

The unexpected behavior is caused by the debugger environment. When we run the program under `gdb`, the debugger itself sets environment variables and allocates additional memory for its internal use. That added memory shifts the addresses on the stack compared with a normal run, so an exploit that works inside the debugger can fail (or vice versa) because the target return address changes.

To avoid this interference we launched the debugger with a clean environment using `env - gdb stack-L1-dbg` so that no extra environment variables are inherited. Inside the debugger we used `show env` to list environment variables created by the debugger; this revealed two entries, `LINES` and `COLUMNS`. These variables consume stack or environment space and can change the layout of memory used by the vulnerable program, which explains the differing addresses between normal execution and a `gdb` session.

![Logbook 5 — Disable extra variables](/images/logbook5/disableEnvsGDB.png)
_Figure 6 - Disable extra variables_

To obtain the correct memory addresses for the exploit, we start GDB on the `stack-L1-dbg` binary and ensure any debugger-added environment variables are removed (`LINES` and `COLUMNS` as seen before). With the debugger running in the same environment as normal execution,all we do is repeat the earlier exploit steps: locate and record the frame pointer (`$ebp`) and measure the distance between the `buffer` and `$ebp`. Incorporate those measured values into the `exploit` script and rebuild the `payload` so it overwrites the `saved return address` with the correct target address.

![Logbook 5 — Final exploit.py](/images/logbook5/rightExploit.png)
_Figure 7 - Final exploit.py_

![Logbook 5 — Buffer Overflow](/images/logbook5/BUFFEROVERFLOWN.png)
_Figure 8 - Buffer Overflow_

Note: we ran the program with its full path rather than `./stack-L1`. That makes `argv[0]` longer and shifts the memory layout a little. We could’ve found the best variant by trial and error, but we picked the full path on purpose to get a consistent change.



## Address Visualization

Has we explained above we can run the gdb and inspect the first 20 addresses to see exacly where we are hitting the address we want.

![Logbook 5 — Buffer Overflow](/images/logbook5/address%20overflow.png)
_Figure 9 - Address of Buffer Overflow_

As we can see we above our the addresses above the return register have been replaced with 0x90 which is a nop slide. in the what should be the return address for BOF we actually choose an addresses slightly above it(`0xFFFFCA18`) to hit the NOP Slide eventually hitting the shellcode that will be in 500-len(shellcode)