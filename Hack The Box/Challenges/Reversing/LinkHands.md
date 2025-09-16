# LinkHands

## Recon

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace/rev_linkhands]
└─$ file link                         
link: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=998c1a511ccf0a80eec3ed30dab3331ce2cf9f7f, for GNU/Linux 4.4.0, stripped
```

Run:

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace/rev_linkhands]
└─$ ./link                       
The cultists look expectantly to you - who will you link hands with? foo
You fail to grasp their hands - they look at you with suspicious...
```

## Reversing

```c
int FUN_00401196(void) {
    int iVar1;
    char* pcVar2;
    undefined** ppuVar3;
    long in_FS_OFFSET;
    undefined8* local_68;
    undefined8 local_60;
    char local_58[72];
    long local_10;

    local_10 = *(long*)(in_FS_OFFSET + 0x28);
    printf(
        "The cultists look expectantly to you - who will you link hands "
        "with? ");
    fgets(local_58, 0x40, stdin);
    pcVar2 = strchr(local_58, 10);
    if (pcVar2 != (char*)0x0) {
        *pcVar2 = '\0';
    }
    iVar1 = __isoc99_sscanf(local_58, "%p %p", &local_68, &local_60);
    if (iVar1 == 2) {
        *local_68 = local_60;
        ppuVar3 = &PTR_PTR_00404190;
        do {
            putchar((int)*(char*)(ppuVar3 + 1));
            ppuVar3 = (undefined**)*ppuVar3;
        } while (ppuVar3 != (undefined**)0x0);
        putc(10, stdout);
        iVar1 = 0;
    } else {
        puts(
            "You fail to grasp their hands - they look at you with "
            "suspicious...");
        iVar1 = 1;
    }
    if (local_10 == *(long*)(in_FS_OFFSET + 0x28)) {
        return iVar1;
    }
    __stack_chk_fail();
}
```

Binary reads two addresses. The value at the first address is set to the second address. It then starts printing values at `0x404190` with offset 8, moving to the address pointed to by the first 8 bytes: this is a linked list.

Binary actually defines two linked lists. First one starts at `0x404190` and links until `0x404060`, the second at `0x404070`. Should link `0x404060` to `0x404070`.

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace/rev_linkhands]
└─$ ./link
The cultists look expectantly to you - who will you link hands with? 0x404060 0x404070
HTB{4_br34k_1n_th3_ch41n_0e343f537ebc}
```
