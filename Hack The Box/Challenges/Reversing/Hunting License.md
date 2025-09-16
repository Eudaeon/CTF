# Hunting License

## Recon

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace/rev_hunting_license]
└─$ file license 
license: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=5be88c3ed329c1570ab807b55c1875d429a581a7, for GNU/Linux 3.2.0, not stripped
```

Run:

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace/rev_hunting_license]
└─$ ./license 
So, you want to be a relic hunter?
First, you're going to need your license, and for that you need to pass the exam.
It's short, but it's not for the faint of heart. Are you up to the challenge?! (y/n)
y
Okay, first, a warmup - what's the first password? This one's not even hidden: 
```

## Reversing

```c
const uint8_t t2[16] = {0x47, 0x7b, 0x7a, 0x61, 0x77, 0x52, 0x7d, 0x77,
                        0x55, 0x7a, 0x7d, 0x72, 0x7f, 0x32, 0x32, 0x32};

void exam(void) {
    int iVar1;
    char local_38[28] = {0};
    char local_1c[12] = {0};
    char* local_10;

    local_10 = (char*)readline(
        "Okay, first, a warmup - what's the first password? This one's not "
        "even hidden: ");
    iVar1 = strcmp(local_10, "PasswordNumeroUno");
    if (iVar1 != 0) {
        puts("Not even close!");
        exit(-1);
    }
    free(local_10);
    reverse(local_1c, "0wTdr0wss4P", 0xb);
    local_10 = (char*)readline("Getting harder - what's the second password? ");
    iVar1 = strcmp(local_10, local_1c);
    if (iVar1 != 0) {
        puts("You've got it all backwards...");
        exit(-1);
    }
    free(local_10);
    xor(local_38, t2, 0x11, 0x13);
    local_10 = (char*)readline(
        "Your final test - give me the third, and most protected, password: ");
    iVar1 = strcmp(local_10, local_38);
    if (iVar1 != 0) {
        puts("Failed at the final hurdle!");
        exit(-1);
    }
    free(local_10);
    return;
}
```

Passwords:

1. `PasswordNumeroUno`
2. `0wTdr0wss4P` reversed, which is `P4ssw0rdTw0`
3. Hardcoded string XOR'ed with key 19, which is `ThirdAndFinal!!!`

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace/rev_hunting_license]
└─$ nc 83.136.248.250 45187
What is the file format of the executable?
> ELF
[+] Correct!

What is the CPU architecture of the executable?
> x86-64
[+] Correct!

What library is used to read lines for user answers? (`ldd` may help)
> readline
[+] Correct!

What is the address of the `main` function?
> 0x401172
[+] Correct!

How many calls to `puts` are there in `main`? (using a decompiler may help)
> 5
[+] Correct!

What is the first password?
> PasswordNumeroUno
[+] Correct!

What is the reversed form of the second password?
> 0wTdr0wss4P
[+] Correct!

What is the real second password?
> P4ssw0rdTw0
[+] Correct!

What is the XOR key used to encode the third password?
> 19
[+] Correct!

What is the third password?
> ThirdAndFinal!!!
[+] Correct!

[+] Here is the flag: `HTB{l1c3ns3_4cquir3d-hunt1ng_t1m3!}`
```
