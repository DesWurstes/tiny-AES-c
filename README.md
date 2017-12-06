### Tiny AES in C

This is a small and portable implementation of the AES [ECB](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Electronic_Codebook_.28ECB.29), [CTR](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Counter_.28CTR.29) and [CBC](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Cipher_Block_Chaining_.28CBC.29) encryption algorithms written in C.

You can override the default key-size of 128 bit with 192 or 256 bit by defining the symbols AES192 or AES256 in `aes.h`.

The API is very simple and looks like this (I am using C99 `<stdint.h>`-style annotated types):

```C
//Init ctx with
void AES_init_ctx(struct AES_ctx *ctx,const uint8_t* key);
void AES_init_ctx_iv(struct AES_ctx *ctx,const uint8_t* key,const uint8_t* iv);

//or reset iv at random point
void AES_ctx_set_iv(struct AES_ctx *ctx,const uint8_t* iv);

//then do 
void AES_ECB_encrypt(struct AES_ctx *ctx, const uint8_t* buf);
void AES_ECB_decrypt(struct AES_ctx *ctx, const uint8_t* buf);

void AES_CBC_encrypt_buffer(struct AES_ctx *ctx, uint8_t* buf, uint32_t length);
void AES_CBC_decrypt_buffer(struct AES_ctx *ctx, uint8_t* buf, uint32_t length);

/* Same function for encrypting as for decrypting in CTR mode */
void AES_CTR_xcrypt_buffer(struct AES_ctx* ctx, uint8_t* buf, uint32_t length);
```

Note: 
 * We don't provide any padding so all buffers should be mutiple of 16 bytes if you need padding we rocomend https://en.wikipedia.org/wiki/Padding_(cryptography)#PKCS7
 * ECB mode is considered unsafe and is not implemented in streaming mode. If you realy need this just call the function for every block of 16 bytes you need encrypted. See https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Electronic_Codebook_(ECB) for more details

You can choose to use any or all of the modes-of-operations, by defining the symbols CBC, CTR or ECB. See the header file for clarification.

There is no built-in error checking or protection from out-of-bounds memory access errors as a result of malicious input.

The module uses less than 200 bytes of RAM and 1-2K ROM when compiled for ARM, but YMMV depending on which modes are enabled.

It is one of the smallest implementation in C I've seen yet, but do contact me if you know of something smaller (or have improvements to the code here). 

I've successfully used the code on 64bit x86, 32bit ARM and 8 bit AVR platforms.


GCC size output when only CTR mode is compiled for ARM:

    $ arm-none-eabi-gcc -Os -DCBC=0 -DECB=0 -DCTR=1 -c aes.c
    $ size aes.o
       text    data     bss     dec     hex filename
       1203       0       0    1203     4b3 aes.o

.. and when compiling for the THUMB instruction set, we end up just above 1K in code size.

    $ arm-none-eabi-gcc -Os -mthumb -DCBC=0 -DECB=0 -DCTR=1 -c aes.c
    $ size aes.o
       text    data     bss     dec     hex filename
        955       0       0     955     3bb aes.o


I am using the Free Software Foundation, ARM GCC compiler:

    $ arm-none-eabi-gcc --version
    arm-none-eabi-gcc (4.8.4-1+11-1) 4.8.4 20141219 (release)
    Copyright (C) 2013 Free Software Foundation, Inc.
    This is free software; see the source for copying conditions.  There is NO
    warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.



This implementation is verified against the data in:

[National Institute of Standards and Technology Special Publication 800-38A 2001 ED](http://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-38a.pdf) Appendix F: Example Vectors for Modes of Operation of the AES.

The other appendices in the document are valuable for implementation details on e.g. padding, generation of IVs and nonces in CTR-mode etc.


A heartfelt thank-you to all the nice people out there who have contributed to this project.


All material in this repository is in the public domain.
