# FlagCasino

## Recon

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace/rev_flagcasino]
└─$ file casino 
casino: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=7618b017ef4299337610a90a0a6ccb7f9efc44a4, for GNU/Linux 3.2.0, not stripped
```

Run:

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace/rev_flagcasino]
└─$ ./casino    
[ ** WELCOME TO ROBO CASINO **]
     ,     ,
    (\____/)
     (_oo_)
       (O)
     __||__    \)
  []/______\[] /
  / \______/ \/
 /    /__\
(\   /____\
---------------------
[*** PLEASE PLACE YOUR BETS ***]
> foo
[ * INCORRECT * ]
[ *** ACTIVATING SECURITY SYSTEM - PLEASE VACATE *** ]
```

## Reversing

```c
const uint8_t check[120] = {
    0xbe, 0x28, 0x4b, 0x24, 0x05, 0x78, 0xf7, 0x0a, 0x17, 0xfc, 0x0d, 0x11,
    0xa1, 0xc3, 0xaf, 0x07, 0x33, 0xc5, 0xfe, 0x6a, 0xa2, 0x59, 0xd6, 0x4e,
    0xb0, 0xd4, 0xc5, 0x33, 0xb8, 0x82, 0x65, 0x28, 0x20, 0x37, 0x38, 0x43,
    0xfc, 0x14, 0x5a, 0x05, 0x9f, 0x5f, 0x19, 0x19, 0x20, 0x37, 0x38, 0x43,
    0x9f, 0x5f, 0x19, 0x19, 0x5e, 0x9c, 0x7c, 0x74, 0x37, 0xa2, 0x3d, 0x0f,
    0x99, 0xb2, 0x5a, 0x61, 0x33, 0xc5, 0xfe, 0x6a, 0x20, 0x37, 0x38, 0x43,
    0x37, 0xa2, 0x3d, 0x0f, 0x33, 0xc5, 0xfe, 0x6a, 0x99, 0xb2, 0x5a, 0x61,
    0xb8, 0x82, 0x65, 0x28, 0xfc, 0x14, 0x5a, 0x05, 0x94, 0x49, 0xe4, 0x3a,
    0xe9, 0xdf, 0xd7, 0x06, 0xa2, 0x59, 0xd6, 0x4e, 0xcd, 0x4a, 0xcd, 0x0c,
    0x64, 0xed, 0xd8, 0x57, 0x99, 0xb2, 0x5a, 0x61, 0x2a, 0xbc, 0xe9, 0x22};

int main(void) {
    int iVar1;
    char local_d;
    uint local_c;

    puts("[ ** WELCOME TO ROBO CASINO **]");
    puts(
        "     ,     ,\n    (\\____/)\n     (_oo_)\n       (O)\n     __||__    "
        "\\)\n  []/______\\[] /\n   / \\______/ \\/\n /    /__\\\n(\\   "
        "/____\\\n---------------------");
    puts("[*** PLEASE PLACE YOUR BETS ***]");
    local_c = 0;
    while (true) {
        if (0x1d < local_c) {
            puts("[ ** HOUSE BALANCE $0 - PLEASE COME BACK LATER ** ]");
            return 0;
        }
        printf("> ");
        iVar1 = __isoc99_scanf(" %c", &local_d);
        if (iVar1 != 1)
            break;
        srand((int)local_d);
        iVar1 = rand();
        if (iVar1 != *(int*)(check + (long)(int)local_c * 4)) {
            puts("[ * INCORRECT * ]");
            puts("[ *** ACTIVATING SECURITY SYSTEM - PLEASE VACATE *** ]");
            exit(-2);
        }
        puts("[ * CORRECT *]");
        local_c = local_c + 1;
    }
    exit(-1);
}
```

Input is read character by character. Each character is used as a seed, and a random number is generated. This number is compared against a hardcoded one.

Script to recover each character by bruteforcing:

```py

```
