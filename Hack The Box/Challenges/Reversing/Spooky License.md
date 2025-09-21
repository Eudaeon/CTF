# Spooky License

## Recon

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace/rev_spookylicence]
└─$ file spookylicence 
spookylicence: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=b4f93c6f8f1c73bbd5f9c2b6862be023abed1b47, for GNU/Linux 3.2.0, stripped
```

Run:

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace/rev_spookylicence]
└─$ ./spookylicence foo
Invalid License Format
```

## Reversing

```c
int FUN_00101169(int param_1, long param_2) {
    char* pcVar1;
    int iVar2;
    size_t sVar3;

    if (param_1 == 2) {
        sVar3 = strlen(*(char**)(param_2 + 8));
        if (sVar3 == 0x20) {
            pcVar1 = *(char**)(param_2 + 8);
            if ((((((((pcVar1[0x1d] == (char)((pcVar1[5] - pcVar1[3]) + 'F')) &&
                      ((char)(pcVar1[2] + pcVar1[0x16]) ==
                       (char)(pcVar1[0xd] + '{'))) &&
                     ((char)(pcVar1[0xc] + pcVar1[4]) ==
                      (char)(pcVar1[5] + '\x1c'))) &&
                    ((((char)(pcVar1[0x19] * pcVar1[0x17]) ==
                           (char)(*pcVar1 + pcVar1[0x11] + '\x17') &&
                       ((char)(pcVar1[0x1b] * pcVar1[1]) ==
                        (char)(pcVar1[5] + pcVar1[0x16] + -0x15))) &&
                      (((char)(pcVar1[9] * pcVar1[0xd]) ==
                            (char)(pcVar1[0x1c] * pcVar1[3] + -9) &&
                        ((pcVar1[9] == 'p' &&
                          ((char)(pcVar1[0x13] + pcVar1[0x15]) ==
                           (char)(pcVar1[6] + -0x80))))))))) &&
                   (pcVar1[0x10] ==
                    (char)((pcVar1[0xf] - pcVar1[0xb]) + '0'))) &&
                  (((((((char)(pcVar1[7] * pcVar1[0x1b]) ==
                            (char)(pcVar1[1] * pcVar1[0xd] + '-') &&
                        (pcVar1[0xd] ==
                         (char)(pcVar1[0x12] + pcVar1[0xd] + -0x65))) &&
                       ((char)(pcVar1[0x14] - pcVar1[8]) ==
                        (char)(pcVar1[9] + '|'))) &&
                      ((pcVar1[0x1f] ==
                            (char)((pcVar1[8] - pcVar1[0x1f]) + -0x79) &&
                        ((char)(pcVar1[0x14] * pcVar1[0x1f]) ==
                         (char)(pcVar1[0x14] + '\x04'))))) &&
                     ((char)(pcVar1[0x18] - pcVar1[0x11]) ==
                      (char)(pcVar1[0x15] + pcVar1[8] + -0x17))) &&
                    ((((char)(pcVar1[7] + pcVar1[5]) ==
                           (char)(pcVar1[5] + pcVar1[0x1d] + ',') &&
                       ((char)(pcVar1[0xc] * pcVar1[10]) ==
                        (char)((pcVar1[1] - pcVar1[0xb]) + -0x24))) &&
                      ((((char)(pcVar1[0x1f] * *pcVar1) ==
                             (char)(pcVar1[0x1a] + -0x1b) &&
                         ((((char)(pcVar1[1] + pcVar1[0x14]) ==
                                (char)(pcVar1[10] + -0x7d) &&
                            (pcVar1[0x12] ==
                             (char)(pcVar1[0x1b] + pcVar1[0xe] + '\x02'))) &&
                           ((char)(pcVar1[0x1e] * pcVar1[0xb]) ==
                            (char)(pcVar1[0x15] + 'D'))))) &&
                        ((((char)(pcVar1[5] * pcVar1[0x13]) ==
                               (char)(pcVar1[1] + -0x2c) &&
                           ((char)(pcVar1[0xd] - pcVar1[0x1a]) ==
                            (char)(pcVar1[0x15] + -0x7f))) &&
                          (pcVar1[0x17] ==
                           (char)((pcVar1[0x1d] - *pcVar1) + 'X'))))))))))) &&
                 (((pcVar1[0x13] == (char)(pcVar1[8] * pcVar1[0xd] + -0x17) &&
                    ((char)(pcVar1[6] + pcVar1[0x16]) ==
                     (char)(pcVar1[3] + 'S'))) &&
                   ((pcVar1[0xc] == (char)(pcVar1[0x1a] + pcVar1[7] + -0x72) &&
                     (((pcVar1[0x10] ==
                            (char)((pcVar1[0x12] - pcVar1[5]) + '3') &&
                        ((char)(pcVar1[0x1e] - pcVar1[8]) ==
                         (char)(pcVar1[0x1d] + -0x4d))) &&
                       ((char)(pcVar1[0x14] - pcVar1[0xb]) ==
                        (char)(pcVar1[3] + -0x4c))))))))) &&
                (((char)(pcVar1[0x10] - pcVar1[7]) ==
                      (char)(pcVar1[0x11] + 'f') &&
                  ((char)(pcVar1[1] + pcVar1[0x15]) ==
                   (char)(pcVar1[0xb] + pcVar1[0x12] + '+'))))) {
                puts("License Correct");
                iVar2 = 0;
            } else {
                puts("License Invalid");
                iVar2 = -1;
            }
        } else {
            puts("Invalid License Format");
            iVar2 = -1;
        }
    } else {
        puts("./spookylicence <license>");
        iVar2 = -1;
    }
    return iVar2;
}
```

Binary compares argument character by character.

```py
import angr
import claripy

BIN = "./spookylicence"
TARGET_ADDR = 0x40187D
FLAG_LENGTH = 32

arg = claripy.BVS("arg", FLAG_LENGTH * 8)
argv = [BIN, arg]

proj = angr.Project(BIN, auto_load_libs=False)
start_state = proj.factory.entry_state(args=argv)
simgr = proj.factory.simulation_manager(start_state)
simgr.explore(find=TARGET_ADDR)

if simgr.found:
    st = simgr.found[0]
    input = st.solver.eval(arg, cast_to=bytes)
    print(input.strip(b"\x00").decode())
```

```plaintext
HTB{The_sp0000000key_liC3nC3K3Y}
```
