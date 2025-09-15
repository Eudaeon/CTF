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
const uint32_t check[30] = {
    0x244b28be, 0x0af77805, 0x110dfc17, 0x07afc3a1, 0x6afec533, 0x4ed659a2,
    0x33c5d4b0, 0x286582b8, 0x43383720, 0x055a14fc, 0x19195f9f, 0x43383720,
    0x19195f9f, 0x747c9c5e, 0x0f3da237, 0x615ab299, 0x6afec533, 0x43383720,
    0x0f3da237, 0x6afec533, 0x615ab299, 0x286582b8, 0x055a14fc, 0x3ae44994,
    0x06d7dfe9, 0x4ed659a2, 0x0ccd4acd, 0x57d8ed64, 0x615ab299, 0x22e9bc2a};

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

Script to recover each character by bruteforcing (possible as seed is between 0 and 255):

```py
import ctypes

CHECK = [
    0x244B28BE,
    0x0AF77805,
    0x110DFC17,
    0x07AFC3A1,
    0x6AFEC533,
    0x4ED659A2,
    0x33C5D4B0,
    0x286582B8,
    0x43383720,
    0x055A14FC,
    0x19195F9F,
    0x43383720,
    0x19195F9F,
    0x747C9C5E,
    0x0F3DA237,
    0x615AB299,
    0x6AFEC533,
    0x43383720,
    0x0F3DA237,
    0x6AFEC533,
    0x615AB299,
    0x286582B8,
    0x055A14FC,
    0x3AE44994,
    0x06D7DFE9,
    0x4ED659A2,
    0x0CCD4ACD,
    0x57D8ED64,
    0x615AB299,
    0x22E9BC2A,
]

libc = ctypes.CDLL("libc.so.6")
mapping = {}
for seed in range(256):
    libc.srand(seed)
    mapping[libc.rand()] = seed

out = bytes(mapping[val] for val in CHECK)
print(out.decode())
```

```plaintext
HTB{r4nd_1s_sup3r_pr3d1ct4bl3}
```

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
> HTB{r4nd_1s_sup3r_pr3d1ct4bl3}
[ * CORRECT *]
> [ * CORRECT *]
> [ * CORRECT *]
> [ * CORRECT *]
> [ * CORRECT *]
> [ * CORRECT *]
> [ * CORRECT *]
> [ * CORRECT *]
> [ * CORRECT *]
> [ * CORRECT *]
> [ * CORRECT *]
> [ * CORRECT *]
> [ * CORRECT *]
> [ * CORRECT *]
> [ * CORRECT *]
> [ * CORRECT *]
> [ * CORRECT *]
> [ * CORRECT *]
> [ * CORRECT *]
> [ * CORRECT *]
> [ * CORRECT *]
> [ * CORRECT *]
> [ * CORRECT *]
> [ * CORRECT *]
> [ * CORRECT *]
> [ * CORRECT *]
> [ * CORRECT *]
> [ * CORRECT *]
> [ * CORRECT *]
> [ * CORRECT *]
[ ** HOUSE BALANCE $0 - PLEASE COME BACK LATER ** ]
```
