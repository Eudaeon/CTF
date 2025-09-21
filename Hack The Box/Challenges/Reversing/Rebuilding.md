# Rebuilding

## Recon

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace/rev_rebuilding]
└─$ file rebuilding 
rebuilding: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=c7a145f3a4b213cf895a735e2b26adffc044c190, not stripped
```

Run:

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace/rev_rebuilding]
└─$ ./rebuilding foo
Preparing secret keys
Password length is incorrect
```

## Reversing

```c
const uint8_t data[34] = {0x29, 0x38, 0x2b, 0x1e, 0x06, 0x42, 0x05, 0x5d, 0x07,
                          0x02, 0x31, 0x10, 0x51, 0x08, 0x5a, 0x16, 0x31, 0x42,
                          0x0f, 0x33, 0x0a, 0x55, 0x00, 0x00, 0x15, 0x1e, 0x1c,
                          0x06, 0x1a, 0x43, 0x13, 0x59, 0x14, 0x00};

const char key[0x7] = "humans";

int main(int param_1, long param_2) {
    int iVar1;
    size_t sVar2;
    int local_14;
    int local_10;
    int local_c;

    if (param_1 != 2) {
        puts("Missing required argument");
        exit(-1);
    }
    local_14 = 0;
    sVar2 = strlen(*(char**)(param_2 + 8));
    if (sVar2 == 0x20) {
        for (local_10 = 0; local_10 < 0x20; local_10 = local_10 + 1) {
            printf("\rCalculating");
            for (local_c = 0; local_c < 6; local_c = local_c + 1) {
                if (local_c == local_10 % 6) {
                    iVar1 = 0x2e;
                } else {
                    iVar1 = 0x20;
                }
                putchar(iVar1);
            }
            fflush(stdout);
            local_14 = local_14 +
                       (uint)((byte)(encrypted[local_10] ^ key[local_10 % 6]) ==
                              *(byte*)((long)local_10 + *(long*)(param_2 + 8)));
            usleep(200000);
        }
        puts("");
        if (local_14 == 0x20) {
            puts("The password is correct");
            iVar1 = 0;
        } else {
            puts("The password is incorrect");
            iVar1 = -1;
        }
    } else {
        puts("Password length is incorrect");
        iVar1 = -1;
    }
    return iVar1;
}
```

Binary decrypts flag using XOR with key `humans`, and compares it with input argument.

```c
void _INIT_1(void) {
    puts("Preparing secret keys");
    builtin_strncpy(key, "aliens", 6);
    return;
}
```

Key is modified to `aliens` at runtime.

![[Pasted image 20250921115730.png]]

Flag: `HTB{h1d1ng_c0d3s_1n_c0nstruct0r5}`
