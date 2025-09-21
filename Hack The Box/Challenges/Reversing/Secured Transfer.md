# Secured Transfer

## Recon

Binary and pcap.

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace/rev_securedtransfer]
└─$ file securetransfer 
securetransfer: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=0457997eda987eb100de85a2954fc8b8fc660a53, for GNU/Linux 3.2.0, stripped
```

## Reversing

```c
int FUN_00101dce(int param_1, long param_2) {
    OPENSSL_init_crypto(2, 0);
    OPENSSL_init_crypto(0xc, 0);
    OPENSSL_init_crypto(0x80, 0);
    if (param_1 == 3) {
        printf("Sending File: %s to %s\n", *(undefined8*)(param_2 + 0x10),
               *(undefined8*)(param_2 + 8));
        FUN_00101835(*(undefined8*)(param_2 + 8),
                     *(undefined8*)(param_2 + 0x10));
    } else if (param_1 == 1) {
        puts("Receiving File");
        FUN_00101b37();
    } else {
        puts("Usage ./securetransfer [<ip> <file>]");
    }
    return 0;
}

int FUN_00101835(char* param_1, char* param_2) {
    int iVar1;
    int iVar2;
    size_t sVar3;
    long in_FS_OFFSET;
    size_t local_50;
    FILE* local_48;
    ulong local_40;
    void* local_38;
    void* local_30;
    sockaddr local_28;
    long local_10;

    local_10 = *(long*)(in_FS_OFFSET + 0x28);
    iVar1 = socket(2, 1, 0);
    if (iVar1 == -1) {
        puts("ERROR: Socket creation failed");
        iVar1 = 0;
    } else {
        memset(&local_28, 0, 0x10);
        local_28.sa_family = 2;
        iVar2 = inet_pton(2, param_1, local_28.sa_data + 2);
        if (iVar2 == 0) {
            printf("ERROR: Invalid input address \'%s\'\n", param_1);
            iVar1 = 0;
        } else {
            local_28.sa_data._0_2_ = htons(0x539);
            iVar2 = connect(iVar1, &local_28, 0x10);
            if (iVar2 == 0) {
                local_48 = fopen(param_2, "rb");
                if (local_48 == (FILE*)0x0) {
                    printf("ERROR: Can\'t open the file \'%s\'\n", param_2);
                    close(iVar1);
                    iVar1 = 0;
                } else {
                    fseek(local_48, 0, 2);
                    local_40 = ftell(local_48);
                    fseek(local_48, 0, 0);
                    if (local_40 < 0x10) {
                        puts("ERROR: File too small");
                        fclose(local_48);
                        close(iVar1);
                        iVar1 = 0;
                    } else if (local_40 < 0x1001) {
                        local_38 = malloc(local_40);
                        local_30 = malloc(local_40 * 2);
                        sVar3 = fread(local_38, 1, local_40, local_48);
                        if (local_40 == sVar3) {
                            iVar2 = FUN_00101529(
                                local_38, local_40 & 0xffffffff, local_30);
                            local_50 = (size_t)iVar2;
                            write(iVar1, &local_50, 8);
                            write(iVar1, local_30, local_50);
                            puts("File send...");
                            free(local_30);
                            free(local_38);
                            fclose(local_48);
                            close(iVar1);
                            iVar1 = 1;
                        } else {
                            puts("ERROR: Failed reading the file");
                            free(local_30);
                            free(local_38);
                            fclose(local_48);
                            close(iVar1);
                            iVar1 = 0;
                        }
                    } else {
                        puts("ERROR: File too large");
                        fclose(local_48);
                        close(iVar1);
                        iVar1 = 0;
                    }
                }
            } else {
                puts("ERROR: Connection failed");
                iVar1 = 0;
            }
        }
    }
    if (local_10 != *(long*)(in_FS_OFFSET + 0x28)) {
        __stack_chk_fail();
    }
    return iVar1;
}

int FUN_00101529(uchar* param_1, int param_2, uchar* param_3) {
    int iVar1;
    EVP_CIPHER* cipher;
    long in_FS_OFFSET;
    int local_50;
    int local_4c;
    uchar* local_48;
    EVP_CIPHER_CTX* local_40;
    uchar local_38[40];
    long local_10;

    local_10 = *(long*)(in_FS_OFFSET + 0x28);
    builtin_memcpy(local_38, "supersecretkeyusedforencryption!", 0x20);
    local_48 = "someinitialvalue";
    local_40 = EVP_CIPHER_CTX_new();
    if (local_40 == (EVP_CIPHER_CTX*)0x0) {
        iVar1 = 0;
    } else {
        cipher = EVP_aes_256_cbc();
        iVar1 = EVP_EncryptInit_ex(local_40, cipher, (ENGINE*)0x0, local_38,
                                   local_48);
        if (iVar1 == 1) {
            iVar1 = EVP_EncryptUpdate(local_40, param_3, &local_50, param_1,
                                      param_2);
            if (iVar1 == 1) {
                local_4c = local_50;
                iVar1 = EVP_EncryptFinal_ex(local_40, param_3 + local_50,
                                            &local_50);
                if (iVar1 == 1) {
                    local_4c = local_4c + local_50;
                    EVP_CIPHER_CTX_free(local_40);
                    iVar1 = local_4c;
                } else {
                    iVar1 = 0;
                }
            } else {
                iVar1 = 0;
            }
        } else {
            iVar1 = 0;
        }
    }
    if (local_10 != *(long*)(in_FS_OFFSET + 0x28)) {
        __stack_chk_fail();
    }
    return iVar1;
}
```

Calling the binary with two arguments `IP` and `FILE` enters client mode. It connects to the IP on port 1337, and sends file size before sending encrypted file contents using AES-256-CBC with key `supersecretkeyusedforencryption!` and IV `someinitialvalue`.

```c

int FUN_00101b37(void) {
    int iVar1;
    ssize_t sVar2;
    size_t sVar3;
    long in_FS_OFFSET;
    socklen_t local_64;
    int local_60;
    int local_5c;
    ulong local_58;
    void* local_50;
    void* local_48;
    long local_40;
    sockaddr local_38;
    sockaddr local_28;
    long local_10;

    local_10 = *(long*)(in_FS_OFFSET + 0x28);
    local_60 = socket(2, 1, 0);
    if (local_60 == -1) {
        puts("ERROR: Socket creation failed");
        iVar1 = 0;
    } else {
        memset(&local_38, 0, 0x10);
        local_38.sa_family = 2;
        local_38.sa_data._2_4_ = htonl(0);
        local_38.sa_data._0_2_ = htons(0x539);
        iVar1 = bind(local_60, &local_38, 0x10);
        if (iVar1 == 0) {
            iVar1 = listen(local_60, 1);
            if (iVar1 == 0) {
                local_64 = 0x10;
                local_5c = accept(local_60, &local_28, &local_64);
                if (local_5c < 0) {
                    puts("ERROR: Accept failed");
                    iVar1 = 0;
                } else {
                    sVar2 = read(local_5c, &local_58, 8);
                    if (sVar2 == 8) {
                        if (local_58 < 0x10) {
                            puts("ERROR: File too small");
                            close(local_60);
                            iVar1 = 0;
                        } else if (local_58 < 0x1001) {
                            local_50 = malloc(local_58);
                            sVar3 = read(local_5c, local_50, local_58);
                            if (sVar3 == local_58) {
                                close(local_60);
                                local_48 = malloc(local_58 + 1);
                                iVar1 = FUN_001016af(
                                    local_50, local_58 & 0xffffffff, local_48);
                                local_40 = (long)iVar1;
                                *(undefined1*)(local_40 + (long)local_48) = 0;
                                printf("File Received...\n%s\n", local_48);
                                free(local_48);
                                free(local_50);
                                iVar1 = 1;
                            } else {
                                puts("ERROR: File send doesn\'t match length");
                                free(local_50);
                                close(local_60);
                                iVar1 = 0;
                            }
                        } else {
                            puts("ERROR: File too large");
                            close(local_60);
                            iVar1 = 0;
                        }
                    } else {
                        puts("ERROR: Reading secret length");
                        close(local_60);
                        iVar1 = 0;
                    }
                }
            } else {
                puts("ERROR: Listen failed");
                iVar1 = 0;
            }
        } else {
            puts("ERROR: Socket bind failed");
            iVar1 = 0;
        }
    }
    if (local_10 != *(long*)(in_FS_OFFSET + 0x28)) {
        __stack_chk_fail();
    }
    return iVar1;
}

int FUN_001016af(uchar* param_1, int param_2, uchar* param_3) {
    int iVar1;
    EVP_CIPHER* cipher;
    long in_FS_OFFSET;
    int local_50;
    int local_4c;
    uchar* local_48;
    EVP_CIPHER_CTX* local_40;
    uchar local_38[40];
    long local_10;

    local_10 = *(long*)(in_FS_OFFSET + 0x28);
    builtin_memcpy(local_38, "supersecretkeyusedforencryption!", 0x20);
    local_48 = "someinitialvalue";
    local_40 = EVP_CIPHER_CTX_new();
    if (local_40 == (EVP_CIPHER_CTX*)0x0) {
        iVar1 = 0;
    } else {
        cipher = EVP_aes_256_cbc();
        iVar1 = EVP_DecryptInit_ex(local_40, cipher, (ENGINE*)0x0, local_38,
                                   local_48);
        if (iVar1 == 1) {
            iVar1 = EVP_DecryptUpdate(local_40, param_3, &local_50, param_1,
                                      param_2);
            if (iVar1 == 1) {
                local_4c = local_50;
                iVar1 = EVP_DecryptFinal_ex(local_40, param_3 + local_50,
                                            &local_50);
                if (iVar1 == 1) {
                    local_4c = local_4c + local_50;
                    EVP_CIPHER_CTX_free(local_40);
                    iVar1 = local_4c;
                } else {
                    iVar1 = 0;
                }
            } else {
                iVar1 = 0;
            }
        } else {
            iVar1 = 0;
        }
    }
    if (local_10 != *(long*)(in_FS_OFFSET + 0x28)) {
        __stack_chk_fail();
    }
    return iVar1;
}
```

Calling the binary with no arguments enters server mode. It opens a socket on port 1337 and waits for the file size to be sent, followed by the encrypted file contents. It then decodes it using AES-256-CBC with key `supersecretkeyusedforencryption!` and IV `someinitialvalue`.

![[Pasted image 20250921105313.png]]

In the PCAP we see that sent file size is 32 bytes.

![[Pasted image 20250921105337.png]]

![[Pasted image 20250921105446.png]]

Flag: `HTB{3ncRyPt3d_F1LE_tr4nSf3r}`
