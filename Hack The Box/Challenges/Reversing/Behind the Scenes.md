# Behind the Scenes

## Recon

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace/rev_behindthescenes]
└─$ file behindthescenes 
behindthescenes: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=e60ae4c886619b869178148afd12d0a5428bfe18, for GNU/Linux 3.2.0, not stripped
```

Run:

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace/rev_behindthescenes]
└─$ ./behindthescenes foo
```

## Reversing

```c
void main(void) {
    code* pcVar1;
    long in_FS_OFFSET;
    sigaction local_a8;
    undefined8 local_10;

    local_10 = *(undefined8*)(in_FS_OFFSET + 0x28);
    memset(&local_a8, 0, 0x98);
    sigemptyset(&local_a8.sa_mask);
    local_a8.__sigaction_handler.sa_handler = segill_sigaction;
    local_a8.sa_flags = 4;
    sigaction(4, &local_a8, (sigaction*)0x0);
    pcVar1 = (code*)invalidInstructionException();
    (*pcVar1)();
}
```

Patch the invalid instructions `UD2` with `NOP`.

```c
int main(int param_1, long param_2) {
    code* pcVar1;
    int iVar2;
    size_t sVar3;
    long in_FS_OFFSET;
    sigaction local_a8;
    long local_10;

    local_10 = *(long*)(in_FS_OFFSET + 0x28);
    memset(&local_a8, 0, 0x98);
    sigemptyset(&local_a8.sa_mask);
    local_a8.__sigaction_handler.sa_handler = segill_sigaction;
    local_a8.sa_flags = 4;
    sigaction(4, &local_a8, (sigaction*)0x0);
    if (param_1 != 2) {
        puts("./challenge <password>");
        pcVar1 = (code*)invalidInstructionException();
        (*pcVar1)();
    }
    sVar3 = strlen(*(char**)(param_2 + 8));
    if ((((sVar3 == 0xc) &&
          (iVar2 = strncmp(*(char**)(param_2 + 8), "Itz", 3), iVar2 == 0)) &&
         (iVar2 = strncmp((char*)(*(long*)(param_2 + 8) + 3), "_0n", 3),
          iVar2 == 0)) &&
        ((iVar2 = strncmp((char*)(*(long*)(param_2 + 8) + 6), "Ly_", 3),
          iVar2 == 0 &&
              (iVar2 = strncmp((char*)(*(long*)(param_2 + 8) + 9), "UD2", 3),
               iVar2 == 0)))) {
        printf("> HTB{%s}\n", *(undefined8*)(param_2 + 8));
    }
    if (local_10 == *(long*)(in_FS_OFFSET + 0x28)) {
        return 0;
    }
    __stack_chk_fail();
}
```

The password is `Itz_0nLy_UD2`.

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace/rev_behindthescenes]
└─$ ./behindthescenes Itz_0nLy_UD2
> HTB{Itz_0nLy_UD2}
```
