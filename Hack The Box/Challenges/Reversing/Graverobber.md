# Graverobber

## Recon

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace/rev_graverobber]
└─$ file robber                    
robber: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=972b4d1424b8cd4916e26b94ebb6257f970e4cbb, for GNU/Linux 4.4.0, not stripped
```

Run:

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace/rev_graverobber]
└─$ ./robber                     
We took a wrong turning!
```

## Reversing

```c
wchar32 parts[0x20] = "HTB{br34k1n9_d0wn_th3_sysc4ll5}", 0;

int main(void) {
    int iVar1;
    long in_FS_OFFSET;
    uint local_ec;
    stat local_e8;
    char local_58[72] = {0};
    long local_10;

    local_10 = *(long*)(in_FS_OFFSET + 0x28);
    local_ec = 0;
    do {
        if (0x1f < local_ec) {
            puts("We found the treasure! (I hope it's not cursed)");
            iVar1 = 0;
        LAB_00101256:
            if (local_10 != *(long*)(in_FS_OFFSET + 0x28)) {
                __stack_chk_fail();
            }
            return iVar1;
        }
        local_58[(int)(local_ec * 2)] =
            (char)*(undefined4*)(parts + (long)(int)local_ec * 4);
        local_58[(int)(local_ec * 2 + 1)] = '/';
        iVar1 = stat(local_58, &local_e8);
        if (iVar1 != 0) {
            puts("We took a wrong turning!");
            iVar1 = 1;
            goto LAB_00101256;
        }
        local_ec = local_ec + 1;
    } while (true);
}
```

The flag is `HTB{br34k1n9_d0wn_th3_sysc4ll5}`.
