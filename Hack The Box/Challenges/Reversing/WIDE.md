# WIDE

## Recon

Binary and database file.

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace/rev_wide]
└─$ file wide  
wide: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=13869bb7ce2c22f474b95ba21c9d7e9ff74ecc3f, not stripped
```

Run:

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace/rev_wide]
└─$ ./wide db.ex 
[*] Welcome user: kr4eq4L2$12xb, to the Widely Inflated Dimension Editor [*]
[*]    Serving your pocket dimension storage needs since 14,012.5 B      [*]
[*]                       Displaying Dimensions....                      [*]
[*]       Name       |              Code                |   Encrypted    [*]
[X] Primus           | people breathe variety practice  |                [*]
[X] Cheagaz          | scene control river importance   |                [*]
[X] Byenoovia        | fighting cast it parallel        |                [*]
[X] Cloteprea        | facing motor unusual heavy       |                [*]
[X] Maraqa           | stomach motion sale valuable     |                [*]
[X] Aidor            | feathers stream sides gate       |                [*]
[X] Flaggle Alpha    | admin secret power hidden        |       *        [*]
Which dimension would you like to examine? 6    
[X] That entry is encrypted - please enter your WIDE decryption key:
```

## Reversing

```c
void menu(long param_1, int param_2) {
    int iVar1;
    long lVar2;
    undefined8* puVar3;
    long in_FS_OFFSET;
    uint local_1d4;
    wchar_t local_1c8[16];
    ...
    char local_c8[16];
    char local_b8[32] = {0};
    ...

    local_10 = *(undefined8*)(in_FS_OFFSET + 0x28);
    do {
        while (true) {
            while (true) {
                printf("Which dimension would you like to examine? ");
                fgets(local_b8, 0x20, stdin);
                lVar2 = strtol(local_b8, (char**)0x0, 10);
                iVar1 = (int)lVar2;
                if ((-1 < iVar1) && (iVar1 < param_2))
                    break;
                puts("That option was invalid.");
            }
            puVar3 = (undefined8*)(param_1 + (long)iVar1 * 0xb4);
            ...
            if ((int)local_188 != 0)
                break;
            puts((char*)&uStack_154);
        }
        ...
        printf(
            "[X] That entry is encrypted - please enter your WIDE decryption "
            "key: ");
        fgets(local_c8, 0x10, stdin);
        mbstowcs(local_1c8, local_c8, 0x10);
        iVar1 = wcscmp(local_1c8, L"sup3rs3cr3tw1d3");
        if (iVar1 == 0) {
            for (local_1d4 = 0;
                 (local_1d4 < 0x80 &&
                  (*(char*)((long)&local_98 + (long)(int)local_1d4) != '\0'));
                 local_1d4 = local_1d4 + 1) {
                *(byte*)((long)&local_98 + (long)(int)local_1d4) =
                    *(byte*)((long)&local_98 + (long)(int)local_1d4) ^
                    (char)(local_1d4 * 0x1b) +
                        (char)((int)(local_1d4 * 0x1b) / 0xff);
            }
            puts((char*)&local_98);
        } else {
            puts(
                "[X]                          Key was incorrect                "
                "           [X]");
        }
    } while (true);
}
```

The password for the encrypted entry is `sup3rs3cr3tw1d3`.

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace/rev_wide]
└─$ ./wide db.ex 
[*] Welcome user: kr4eq4L2$12xb, to the Widely Inflated Dimension Editor [*]
[*]    Serving your pocket dimension storage needs since 14,012.5 B      [*]
[*]                       Displaying Dimensions....                      [*]
[*]       Name       |              Code                |   Encrypted    [*]
[X] Primus           | people breathe variety practice  |                [*]
[X] Cheagaz          | scene control river importance   |                [*]
[X] Byenoovia        | fighting cast it parallel        |                [*]
[X] Cloteprea        | facing motor unusual heavy       |                [*]
[X] Maraqa           | stomach motion sale valuable     |                [*]
[X] Aidor            | feathers stream sides gate       |                [*]
[X] Flaggle Alpha    | admin secret power hidden        |       *        [*]
Which dimension would you like to examine? 6    
[X] That entry is encrypted - please enter your WIDE decryption key: sup3rs3cr3tw1d3
HTB{som3_str1ng5_4r3_w1d3}
```
