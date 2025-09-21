# Ouija

## Recon

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace/rev_ouija]
└─$ file ouija
ouija: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=53a9e0435f7c7041c557e9d4a8418cb6a916f339, for GNU/Linux 3.2.0, not stripped
```

Run:

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace/rev_ouija]
└─$ ./ouija
Retrieving key.
     ..... done!
Hmm, I don't like that one. Let's pick a new one.
     ..... done!
Yes, 18 will do nicely.
     ..... done!
Let's get ready to start. This might take a while!
     ..... done!
This one's an uppercase letter!
     ..... done!
Okay, let's write down this letter! This is a pretty complex operation, you might want to check back later.
     ..... done!
H
```

## Reversing

```c
const uint32_t key = 0xd;

int main(void) {
    char local_78[44];
    int local_4c;
    char* local_48;
    int local_3c;
    int local_38;
    int local_34;
    int local_30;
    int local_2c;
    int local_28;
    int local_24;
    char* local_20;
    int local_14;
    int local_10;
    int local_c;

    builtin_strncpy(local_78, "ZLT{Kdwhafy_ak_fgl_gtxmkuslagf}", 0x1f);
    setvbuf(stdout, (char*)0x0, 2, 0);
    local_48 = strdup(local_78);
    puts("Retrieving key.");
    sleep(10);
    for (local_c = 1; local_c < 0x1e; local_c = local_c + 1) {
        if (local_c % 5 == 0) {
            printf("\r     ");
        }
        putchar(0x2e);
        sleep(1);
    }
    puts(" done!");
    local_4c = key;
    puts("Hmm, I don\'t like that one. Let\'s pick a new one.");
    sleep(10);
    for (local_10 = 1; local_10 < 0x1e; local_10 = local_10 + 1) {
        if (local_10 % 5 == 0) {
            printf("\r     ");
        }
        putchar(0x2e);
        sleep(1);
    }
    puts(" done!");
    local_4c = local_4c + 5;
    puts("Yes, 18 will do nicely.");
    sleep(10);
    for (local_14 = 1; local_14 < 0x14; local_14 = local_14 + 1) {
        if (local_14 % 5 == 0) {
            printf("\r     ");
        }
        putchar(0x2e);
        sleep(1);
    }
    puts(" done!");
    local_20 = local_48;
    puts("Let\'s get ready to start. This might take a while!");
    sleep(10);
    for (local_24 = 1; local_24 < 0x32; local_24 = local_24 + 1) {
        if (local_24 % 5 == 0) {
            printf("\r     ");
        }
        putchar(0x2e);
        sleep(1);
    }
    puts(" done!");
    for (; *local_20 != '\0'; local_20 = local_20 + 1) {
        if ((*local_20 < 'a') || ('z' < *local_20)) {
            if ((*local_20 < 'A') || ('Z' < *local_20)) {
                puts("We can leave this one alone.");
                sleep(10);
                for (local_38 = 1; local_38 < 10; local_38 = local_38 + 1) {
                    if (local_38 % 5 == 0) {
                        printf("\r     ");
                    }
                    putchar(0x2e);
                    sleep(1);
                }
                puts(" done!");
            } else {
                puts("This one\'s an uppercase letter!");
                sleep(10);
                for (local_30 = 1; local_30 < 0x14; local_30 = local_30 + 1) {
                    if (local_30 % 5 == 0) {
                        printf("\r     ");
                    }
                    putchar(0x2e);
                    sleep(1);
                }
                puts(" done!");
                if (*local_20 - local_4c < 0x41) {
                    puts("Wrapping it round...");
                    sleep(10);
                    for (local_34 = 1; local_34 < 0x32;
                         local_34 = local_34 + 1) {
                        if (local_34 % 5 == 0) {
                            printf("\r     ");
                        }
                        putchar(0x2e);
                        sleep(1);
                    }
                    puts(" done!");
                    *local_20 = *local_20 + '\x1a';
                }
                *local_20 = *local_20 - (char)local_4c;
            }
        } else {
            puts("This one\'s a lowercase letter");
            sleep(10);
            for (local_28 = 1; local_28 < 0x14; local_28 = local_28 + 1) {
                if (local_28 % 5 == 0) {
                    printf("\r     ");
                }
                putchar(0x2e);
                sleep(1);
            }
            puts(" done!");
            if (*local_20 - local_4c < 0x61) {
                puts("Wrapping it round...");
                sleep(10);
                for (local_2c = 1; local_2c < 0x32; local_2c = local_2c + 1) {
                    if (local_2c % 5 == 0) {
                        printf("\r     ");
                    }
                    putchar(0x2e);
                    sleep(1);
                }
                puts(" done!");
                *local_20 = *local_20 + '\x1a';
            }
            *local_20 = *local_20 - (char)local_4c;
        }
        puts(
            "Okay, let\'s write down this letter! This is a pretty complex "
            "operation, you might want to check back later.");
        sleep(10);
        for (local_3c = 1; local_3c < 300; local_3c = local_3c + 1) {
            if (local_3c % 5 == 0) {
                printf("\r     ");
            }
            putchar(0x2e);
            sleep(1);
        }
        puts(" done!");
        printf("%c\n", (ulong)(uint)(int)*local_20);
    }
    puts("You\'re still here?");
    return 0;
}
```

Binary uses a ROT cipher with a left shift of 18 to decrypt the flag.

![[Pasted image 20250921110622.png]]

Flag: `HTB{Sleping_is_not_obfuscation}`
