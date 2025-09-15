# CryptOfTheUndead

## Recon

Binary and encrypted file.

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace/rev_crypt_of_the_undead]
└─$ file crypt 
crypt: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=5ac213d86cdb95af5f911c357cdc45c66b6bffc1, for GNU/Linux 4.4.0, not stripped
```

## Reversing

```c
int main(int param_1, undefined8* param_2) {
    int iVar1;
    size_t sVar2;
    char* __dest;
    char* pcVar3;
    long in_FS_OFFSET;
    void* local_40;
    size_t local_38;
    long local_30;

    local_30 = *(long*)(in_FS_OFFSET + 0x28);
    if (param_1 < 2) {
        pcVar3 = "crypt";
        if (param_1 == 1) {
            pcVar3 = (char*)*param_2;
        }
        iVar1 = 1;
        printf("Usage: %s file_to_encrypt\n", pcVar3);
    } else {
        pcVar3 = (char*)param_2[1];
        iVar1 = ends_with(pcVar3, ".undead");
        if (iVar1 == 0) {
            sVar2 = strlen(pcVar3);
            __dest = (char*)malloc(sVar2 + 9);
            strncpy(__dest, pcVar3, sVar2 + 9);
            sVar2 = strlen(__dest);
            builtin_strncpy(__dest + sVar2, ".undead", 8);
            iVar1 = read_file(pcVar3, &local_38, &local_40);
            if (iVar1 == 0) {
                encrypt_buf(local_40, local_38,
                            "BRAAAAAAAAAAAAAAAAAAAAAAAAAINS!!");
                iVar1 = rename(pcVar3, __dest);
                if (iVar1 == 0) {
                    iVar1 = open(__dest, 0x201);
                    if (iVar1 < 0) {
                        iVar1 = 5;
                        perror("error opening new file");
                    } else {
                        write(iVar1, local_40, local_38);
                        close(iVar1);
                        puts("successfully zombified your file!");
                        iVar1 = 0;
                    }
                } else {
                    iVar1 = 4;
                    perror("error renaming file");
                }
            } else {
                iVar1 = 3;
                perror("error reading file");
            }
        } else {
            iVar1 = 2;
            puts("error: that which is undead may not be encrypted");
        }
    }
    if (local_30 == *(long*)(in_FS_OFFSET + 0x28)) {
        return iVar1;
    }
    __stack_chk_fail();
}

void encrypt_buf(undefined8 param_1, undefined8 param_2, undefined8 param_3) {
    long in_FS_OFFSET;
    undefined1 auStack_f8[204];
    undefined8 local_2c;
    undefined4 local_24;
    long local_20;

    local_20 = *(long*)(in_FS_OFFSET + 0x28);
    local_2c = 0;
    local_24 = 0;
    chacha20_init_context(auStack_f8, param_3, &local_2c, 0);
    chacha20_xor(auStack_f8, param_1, param_2);
    if (local_20 == *(long*)(in_FS_OFFSET + 0x28)) {
        return;
    }
    __stack_chk_fail();
}
```

The given file contents is encrypted using ChaCha20, key `BRAAAAAAAAAAAAAAAAAAAAAAAAAINS!!`, nonce `0`.

![[Pasted image 20250915141903.png]]

Flag: `HTB{und01ng_th3_curs3_0f_und34th}`
