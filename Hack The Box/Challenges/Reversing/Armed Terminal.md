# Armed Terminal

## Recon

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace]
└─$ file armedterminal              
armedterminal: ELF 32-bit LSB executable, ARM, EABI5 version 1 (SYSV), statically linked, stripped
```

Run:

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace]
└─$ qemu-arm ./armedterminal 
foo
Wrong!
```

## Reversing

```c
d
```
