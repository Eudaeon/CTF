# Terrorfryer

## Recon

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace/rev_terrorfryer]
└─$ file fryer 
fryer: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=c3cfff35eca84e95e72a1d9d9095208851e5c260, for GNU/Linux 4.4.0, not stripped
```

Run:

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace/rev_terrorfryer]
└─$ ./fryer                      
Please enter your recipe for frying: foo
got:      `foo`
expected: `1_n3}f3br9Ty{_6_rHnf01fg_14rlbtB60tuarun0c_tr1y3`
This recipe isn't right :(
```

## Reversing

```c
int main(void) {
    int iVar1;
    char* pcVar2;
    long in_FS_OFFSET;
    char acStack_68[72];
    long local_20;

    local_20 = *(long*)(in_FS_OFFSET + 0x28);
    setvbuf(stdout, (char*)0x0, 2, 0);
    printf("Please enter your recipe for frying: ");
    fgets(acStack_68, 0x40, stdin);
    pcVar2 = strchr(acStack_68, 10);
    if (pcVar2 != (char*)0x0) {
        *pcVar2 = '\0';
    }
    fryer(acStack_68);
    printf("got:      `%s`\nexpected: `%s`\n", acStack_68, desired);
    iVar1 = strcmp(desired, acStack_68);
    if (iVar1 == 0) {
        puts("Correct recipe - enjoy your meal!");
    } else {
        puts("This recipe isn\'t right :(");
    }
    if (local_20 == *(long*)(in_FS_OFFSET + 0x28)) {
        return 0;
    }
    __stack_chk_fail();
}

void fryer(char* param_1) {
    char cVar1;
    int iVar2;
    size_t sVar3;
    long lVar4;

    if (init.1 == 0) {
        seed.0 = 0x13377331;
        init.1 = 1;
    }
    sVar3 = strlen(param_1);
    if (1 < sVar3) {
        lVar4 = 0;
        do {
            iVar2 = rand_r(&seed.0);
            cVar1 = param_1[lVar4];
            param_1[lVar4] =
                param_1[(int)((ulong)(long)iVar2 % (sVar3 - lVar4)) +
                        (int)lVar4];
            param_1[(int)((ulong)(long)iVar2 % (sVar3 - lVar4)) + (int)lVar4] =
                cVar1;
            lVar4 = lVar4 + 1;
        } while (lVar4 != sVar3 - 1);
    }
    return;
}
```

Shuffles in-place the characters of the input.

Input: `abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUV`

Output: `TzHLVAnNpJbkdlOsuaCSGwtyIUeojKgcQqmiMhBxRDfErFvP`

Shuffled flag: `1_n3}f3br9Ty{_6_rHnf01fg_14rlbtB60tuarun0c_tr1y3`

```python
INPUT = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUV"
OUTPUT = "TzHLVAnNpJbkdlOsuaCSGwtyIUeojKgcQqmiMhBxRDfErFvP"
SHUFFLED_FLAG = "1_n3}f3br9Ty{_6_rHnf01fg_14rlbtB60tuarun0c_tr1y3"

position_map = [INPUT.index(c) for c in OUTPUT]

flag = [""] * len(SHUFFLED_FLAG)
for i, pos in enumerate(position_map):
    if i < len(SHUFFLED_FLAG):
        flag[pos] = SHUFFLED_FLAG[i]

print("".join(flag))
```

```plaintext
HTB{4_truly_t3rr0r_fry1ng_funct10n_9b3ab6360f11}
```
