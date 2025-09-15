# Industry Secret

## Recon

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace/rev_industrysecret]
└─$ file zephyr.elf                
zephyr.elf: ELF 32-bit LSB executable, ARM, EABI5 version 1 (SYSV), statically linked, with debug_info, not stripped
```

Run on fdsfsd board:

```plaintext
*** Using Zephyr OS v3.7.99-1f8f3dc29142 ***
** Critical Industrial Machinery **
==============================
[1] READ LOGS
[2] EMERGENCY STOP
==============================
```

## Reversing

```c
ds
```

There's a secret `d` command.

```c
d
```
