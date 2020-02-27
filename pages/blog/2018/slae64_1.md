title: Password Protected Bind Shell Shellcode
date: 2018-04-22
published: true 
summary: My solution to SLAE64 Exam Question 1
tags: shellcode, exploit, assembly

#Password Protected Bind Shell Shellcode

I've been taking the SLAE64 certification class in my spare time, and the exam consists of 7 assembly code challenges, with code and accompanying blog posts. I designed processor hardware and wrote assembly code for that in college, but I'm definitely still a n00b when it comes to reversing or exploit development. I was very happy with the course and thought it was a great opportunity to learn about linux calling conventions, avoiding null bytes in shellcode, and other tricks that definitely weren't covered in my college courses (or I just wasn't paying attention). 

The first exam question was to write a password-protected tcp bind shell. I'll caveat all of the following by noting that Vivek (the instructor) provides a lot of this sample code in the class materials, so a lot of the instructions are reused from that code. This exercise mainly consisted of removing null bytes from the output and adding logic to password protect the shell.  

First, I started by adding a bit of customization to Vivek's code. We were previously binding the shell to ```INADDR_ANY```, or all system interfaces. I changed that to ```INADDR_LOOPBACK```, or only the local interface. This would prevent connections to our shell from any system other than our own. For a real payload this is obviously not very useful, but it was a fun exercise and adds some customization to this shellcode. I used python to get the hex value in network byte order for the ```INADDR_LOOPBACK``` value. This was a pain, as it contained null bytes, which we are trying to avoid. I then xor'd this value with 0x23232323 to get a value which I could include in an instruction without null bytes:

![inaddr_loopback](/static/blog/2018/SLAE64/1/inaddr_loopback.png)

I then added an instruction to decode the value:

![inaddr_decode](/static/blog/2018/SLAE64/1/inaddr_decode.png)

I also changed the bind port to ```31337``` so we would really be hacking. Again, I used the python prompt to find the hex value needed:

![31337_hex](/static/blog/2018/SLAE64/1/31337_hex.png)

The next step was removing all the null bytes from the compiled code. For the most part, this consisted of replacing ```mov``` instructions using 64-bit registers to ```xor``` instructions to zero the target register, then using a ```mov``` with a 8-bit target register. There are other strategies that might save more space, like using ```push``` instructions with immediate byte values to zero the remaining 7 bytes, but this method works fine and keeps my shellcode reasonably small. I made a few other small improvements as well, such as removing superfluous zeroing ```xor``` operations for already zero-d registers or stack space.

I then added simple password checking code - essentially I just read 8 bytes from the already-initialized socket and compare it to a hardcoded password. If they don't match, I jump to the end of my shellcode - I don't exit gracefully or anything like that, the program will just crash. Following is my password protection:

        ; password check code starts here - 7 byte password + newline 
        read_password: ; read syscall - read onto stack ; allocate space on stack sub rsp, 8
                ; make syscall
                ; read(fd, *buf, count)
                xor rax, rax
                xor rdi, rdi
                mov rsi, rsp
                xor rdx, rdx
                mov dl, 8
                syscall

        ;compare password with mrtfsh1
        compare_password:
                mov rbx,0x0a3168736674726d
                mov rdi,[rsp]
                cmp rbx,rdi
                jne exit

        ; execve code starts here - mostly provided in course


After all this, my shellcode came out to 215 bytes, with no null characters. Not bad:

![bytecount_nullcount](/static/blog/2018/SLAE64/1/bytecount_nullcount.png)

I ran it using a simple C program to call the shellcode (this was useful for encoding and printing statements at other points in the course):

![base_sc_runner](/static/blog/2018/SLAE64/1/base_sc_runner.png)

When I entered the correct passcode, the program allowed me to obtain a shell:

![correct](/static/blog/2018/SLAE64/1/correct.png)

When I entered an incorrect passcode, the program simply crashed:

![incorrect](/static/blog/2018/SLAE64/1/incorrect.png)

![incorrect2](/static/blog/2018/SLAE64/1/incorrect2.png)

All code referenced in this blog post is available at: <https://github.com/mrT-F/SLAE64/tree/master/1>

*This blog post has been created for completing the requirements of the SecurityTube Linux Assembly Expert Certification.*

*Certification at: <http://securitytube-training.com/online-courses/securitytube-linux-assembly-expert/>*

*Student ID: SLAE64 - 1546*