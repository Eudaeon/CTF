# SpookyPass

## Recon

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace/rev_spookypass]
└─$ file pass                      
pass: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=3008217772cc2426c643d69b80a96c715490dd91, for GNU/Linux 4.4.0, not stripped
```

Run:

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace/rev_spookypass]
└─$ ./pass                       
Welcome to the SPOOKIEST party of the year.
Before we let you in, you'll need to give us the password: foo
You're not a real ghost; clear off!
```

## Reversing

```c
wchar32 parts[0x1a] = "HTB{un0bfu5c4t3d_5tr1ng5}", 0;

int main(void) {
    int iVar1;
    char* pcVar2;
    long in_FS_OFFSET;
    uint local_c4;
    char local_b8[32] = {0};
    char local_98[136];
    long local_10;

    local_10 = *(long*)(in_FS_OFFSET + 0x28);
    puts("Welcome to the \x1b[1;3mSPOOKIEST\x1b[0m party of the year.");
    printf("Before we let you in, you'll need to give us the password: ");
    fgets(local_98, 0x80, stdin);
    pcVar2 = strchr(local_98, 10);
    if (pcVar2 != (char*)0x0) {
        *pcVar2 = '\0';
    }
    iVar1 = strcmp(local_98, "s3cr3t_p455_f0r_gh05t5_4nd_gh0ul5");
    if (iVar1 == 0) {
        puts("Welcome inside!");
        for (local_c4 = 0; local_c4 < 0x1a; local_c4 = local_c4 + 1) {
            local_b8[(int)local_c4] =
                (char)*(undefined4*)(parts + (long)(int)local_c4 * 4);
        }
        puts(local_b8);
    } else {
        puts("You\'re not a real ghost; clear off!");
    }
    if (local_10 != *(long*)(in_FS_OFFSET + 0x28)) {
        __stack_chk_fail();
    }
    return 0;
}
```

The password is `s3cr3t_p455_f0r_gh05t5_4nd_gh0ul5`, the flag is `HTB{un0bfu5c4t3d_5tr1ng5}`.

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace/rev_spookypass]
└─$ ./pass                       
Welcome to the SPOOKIEST party of the year.
Before we let you in, you'll need to give us the password: s3cr3t_p455_f0r_gh05t5_4nd_gh0ul5
Welcome inside!
HTB{un0bfu5c4t3d_5tr1ng5}
```
