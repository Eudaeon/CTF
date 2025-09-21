# Encryption Bot

## Recon

Binary and encrypted file.

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace/rev_encryption_bot]
└─$ file chall                        
chall: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=581371f680358611dc4e8d77b03858bd4c780174, for GNU/Linux 3.2.0, stripped
```

Run:

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace/rev_encryption_bot]
└─$ ./chall                      

 #######                                                            ######
 #       #    #  ####  #####  #   # #####  ##### #  ####  #    #    #     #  ####  #####
 #       ##   # #    # #    #  # #  #    #   #   # #    # ##   #    #     # #    #   #
 #####   # #  # #      #    #   #   #    #   #   # #    # # #  #    ######  #    #   #
 #       #  # # #      #####    #   #####    #   # #    # #  # #    #     # #    #   #
 #       #   ## #    # #   #    #   #        #   # #    # #   ##    #     # #    #   #
 ####### #    #  ####  #    #   #   #        #   #  ####  #    #    ######   ####    #


Enter the text to encrypt : foo

I'm encrypt only specific length of character.
(-_-) Find it (-_-)
```

## Reversing

```c
int FUN_0010163e(void) {
    undefined1 local_38[32];
    FILE* local_18;
    undefined4 local_c;

    FUN_00101628();
    local_c = 0;
    printf("\n\nEnter the text to encrypt : ");
    __isoc99_scanf(&DAT_0010230f, local_38);
    local_18 = fopen("data.dat", "r");
    if (local_18 != (FILE*)0x0) {
        system("rm data.dat");
    }
    putchar(10);
    FUN_001015dc(local_38);
    FUN_0010128a();
    FUN_0010131d(local_38);
    FUN_001014ba();
    system("rm data.dat");
    putchar(10);
    return 0;
}

void FUN_001015dc(char* param_1) {
    size_t sVar1;

    sVar1 = strlen(param_1);
    if ((int)sVar1 != 0x1b) {
        puts("I'm encrypt only specific length of character.");
        puts("(-_-) Find it (-_-)");
        exit(1);
    }
    return;
}
```

Binary first checks input size. It should be 27 characters long.

```c
int FUN_0010131d(char* param_1) {
    size_t sVar1;
    ulong uVar2;
    undefined1 local_868[2000];
    int aiStack_98[31];
    int local_1c;

    local_1c = 0;
    while (true) {
        uVar2 = (ulong)local_1c;
        sVar1 = strlen(param_1);
        if (sVar1 <= uVar2)
            break;
        aiStack_98[local_1c] = (int)param_1[local_1c];
        FUN_001011d9(aiStack_98[local_1c], local_868);
        local_1c = local_1c + 1;
    }
    FUN_0010129f();
    return 0;
}

int FUN_001011d9(int param_1) {
    int local_6c;
    uint auStack_68[20];
    FILE* local_18;
    int local_10;
    int local_c;

    local_18 = fopen("data.dat", "a");
    local_6c = param_1;
    for (local_c = 0; local_c < 8; local_c = local_c + 1) {
        auStack_68[local_c] = local_6c % 2;
        local_6c = local_6c / 2;
    }
    for (local_10 = 7; -1 < local_10; local_10 = local_10 + -1) {
        fprintf(local_18, "%d", (ulong)auStack_68[local_10]);
    }
    fclose(local_18);
    return 0;
}
```

Binary processes each character of input, and writes their bit representation in `data.dat`.

```c
void FUN_001014ba(void) {
    int iVar1;
    int aiStack_668[400];
    int local_28;
    char local_21;
    FILE* local_20;
    int local_18;
    int local_14;
    int local_10;
    int local_c;

    local_20 = fopen("data.dat", "r+");
    FUN_00101291();
    for (local_c = 1; local_c < 0xd9; local_c = local_c + 1) {
        iVar1 = fgetc(local_20);
        local_21 = (char)iVar1;
        if (local_21 == '0') {
            aiStack_668[local_c + -1] = 0;
        } else if (local_21 == '1') {
            aiStack_668[local_c + -1] = 1;
        }
        if (local_c != 0) {
            if (local_c % 6 == 0) {
                local_14 = 0;
                local_10 = local_c;
                for (local_18 = 0; local_10 = local_10 + -1, local_18 < 6;
                     local_18 = local_18 + 1) {
                    local_28 = FUN_001013ab(local_18);
                    local_14 = local_14 + aiStack_668[local_10] * local_28;
                }
                FUN_001013e9(local_14);
            }
        }
    }
    fclose(local_20);
    return;
}

int FUN_001013ab(int param_1) {
    undefined4 local_1c;
    undefined4 local_c;

    local_c = 1;
    for (local_1c = param_1; local_1c != 0; local_1c = local_1c + -1) {
        local_c = local_c * 2;
    }
    FUN_0010129f();
    return local_c;
}

int FUN_001013e9(int param_1) {
    long lVar1;
    undefined8* puVar2;
    char local_198[64];
    undefined8 local_158[42];

    builtin_strncpy(
        local_198,
        "RSTUVWXYZ0123456789ABCDEFGHIJKLMNOPQabcdefghijklmnopqrstuvwxyz", 0x3f);
    local_198[0x3f] = '\0';
    puVar2 = local_158;
    for (lVar1 = 0x2a; lVar1 != 0; lVar1 = lVar1 + -1) {
        *puVar2 = 0;
        puVar2 = puVar2 + 1;
    }
    putchar((int)local_198[param_1]);
    return 0;
}
```

The bitstream in `data.dat` is read in chunks of 6 bits. These are converted to numbers, and used as an offset to obtain a letter.

Reverse with:

```py
IN = "flag.enc"
CHARSET = "RSTUVWXYZ0123456789ABCDEFGHIJKLMNOPQabcdefghijklmnopqrstuvwxyz"

bitstream = ""
with open(IN, "r") as f:
    while char := f.read(1):
        offset = CHARSET.find(char)
        bitstream += format(offset, "06b")

byte_array = bytearray()
for i in range(0, len(bitstream), 8):
    byte = bitstream[i : i + 8]
    byte_array.append(int(byte, 2))
flag = byte_array.decode()
print(flag)
```

```plaintext
HTB{3nCrypT10N_W1tH_B1Ts!!}
```
