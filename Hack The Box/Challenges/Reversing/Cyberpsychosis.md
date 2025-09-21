# Cyberpsychosis

## Recon

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace/rev_cyberpsychosis]
└─$ file diamorphine.ko 
diamorphine.ko: ELF 64-bit LSB relocatable, x86-64, version 1 (SYSV), BuildID[sha1]=e6a635e5bd8219ae93d2bc26574fff42dc4e1105, with debug_info, not stripped
```

Given README file mentions "Victor N. Ramos Mello". We find [GitHub repo with same name as binary](https://github.com/m0nad/Diamorphine), this is a rootkit (kernel module).

Features:

- Module starts invisible
- Hide/unhide any process by sending a signal (31 by default)
- Sending a signal (63 by default) to any PID makes the module become (in)visible
- Sending a signal (64 by default) to any PID makes the given user become root
- Files or directories starting with the `MAGIC_PREFIX` (`diamorphine_secret` by default) become invisible

## Reversing

Build project from source and compare with given binary.

`MAGIC_PREFIX` is stored at start of `.rodata.str1.1` segment. It's `psychosis` for given binary.

```plaintext
00000000: 70 73 79 63 68 6f 73 69 73                       psychosis
```

Function `hacked_kill` contains a `switch` statement with the different signal values.

```c
int hacked_kill(pt_regs* pt_regs) {
    undefined1* puVar1;
    list_head* plVar2;
    int iVar3;
    long lVar4;
    undefined* puVar5;

    plVar2 = module_previous;
    iVar3 = (int)pt_regs->si;
    if (iVar3 == 0x2e) {
        if (module_hidden != 0) {
            __this_module.list.next = module_previous->next;
            (__this_module.list.next)->prev = &__this_module.list;
            __this_module.list.prev = plVar2;
            module_hidden = 0;
            plVar2->next = (list_head*)0x1010c8;
            return 0;
        }
        iVar3 = 0;
        (__this_module.list.next)->prev = __this_module.list.prev;
        module_previous = __this_module.list.prev;
        (__this_module.list.prev)->next = __this_module.list.next;
        __this_module.list.next = (list_head*)0xdead000000000100;
        __this_module.list.prev = (list_head*)0xdead000000000122;
        module_hidden = 1;
    } else if (iVar3 == 0x40) {
        lVar4 = prepare_creds();
        iVar3 = 0;
        if (lVar4 != 0) {
            *(undefined8*)(lVar4 + 4) = 0;
            *(undefined8*)(lVar4 + 0xc) = 0;
            *(undefined8*)(lVar4 + 0x14) = 0;
            *(undefined8*)(lVar4 + 0x1c) = 0;
            commit_creds(lVar4);
            return iVar3;
        }
    } else {
        puVar5 = &init_task;
        if (iVar3 == 0x1f) {
            do {
                puVar1 = *(undefined1**)(puVar5 + 0x8b8);
                puVar5 = puVar1 + -0x8b8;
                if (puVar1 == &DAT_001028f0) {
                    return -3;
                }
            } while ((int)pt_regs->di != *(int*)(puVar1 + 0x108));
            if (puVar5 == (undefined*)0x0) {
                return -3;
            }
            *(uint*)(puVar1 + -0x88c) = *(uint*)(puVar1 + -0x88c) ^ 0x10000000;
            return 0;
        }
        lVar4 = (*orig_kill)(pt_regs);
        iVar3 = (int)lVar4;
    }
    return iVar3;
}
```

- Signal 46 to make the module become (in)visible
- Signal 64 to make the given user become root
- Signal 31 to hide/unhide any process

```plaintext
~ $ kill -64 1
kill -64 1

/root # lsmod
lsmod
diamorphine 16384 0 - Live 0xffffffffc03c0000 (OE)

rmmod diamorphine

find / -name '*psychosis*' 2>/dev/null
/opt/psychosis

/opt/psychosis # cat flag.txt
cat flag.txt
HTB{N0w_Y0u_C4n_S33_m3_4nd_th3_r00tk1t_h4s_b33n_sUcc3ssfully_d3f34t3d!!}
```
