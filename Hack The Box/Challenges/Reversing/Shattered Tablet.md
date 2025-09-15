# Shattered Tablet

## Recon

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace/rev_shattered_tablet]
└─$ file tablet 
tablet: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=71ad3ff9f7e5fbf0edc75446337a0a68deb7ecd6, for GNU/Linux 3.2.0, not s
```

Run:

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace/rev_shattered_tablet]
└─$ ./tablet 
Hmmmm... I think the tablet says: foo
No... not that
```

## Reversing

```c
int main(void) {
    char local_48[64] = {0};

    printf("Hmmmm... I think the tablet says: ");
    fgets(local_48, 0x40, stdin);
    if (((((local_48[0x22] == '4') && (local_48[0x14] == '3')) &&
          (local_48[0x24] == 'r')) &&
         ((((local_48[1] == 'T' && (local_48[0x15] == 'v')) &&
            ((local_48[6] == '0' &&
              ((local_48[0x27] == '}' && (local_48[0x26] == 'd')))))) &&
           (local_48[0x1f] == 'r')))) &&
        ((((((local_48[0x1d] == '3' && (local_48[8] == '3')) &&
             (local_48[0x16] == 'e')) &&
            ((local_48[0x23] == '1' && (local_48[5] == 'r')))) &&
           ((local_48[0] == 'H' &&
             ((local_48[0x20] == '3' && (local_48[0x12] == '.')))))) &&
          (((((local_48[0xd] == '4' &&
               (((((local_48[3] == '{' && (local_48[10] == '_')) &&
                   (local_48[0x10] == '.')) &&
                  ((local_48[4] == 'b' && (local_48[7] == 'k')))) &&
                 (local_48[0xf] == 't')))) &&
              (((local_48[0xe] == 'r' && (local_48[0x13] == 'n')) &&
                ((local_48[0x19] == 't' &&
                  (((local_48[0x11] == '.' && (local_48[9] == 'n')) &&
                    (local_48[0x1e] == '_')))))))) &&
             (((local_48[0x1a] == '0' && (local_48[0x18] == '_')) &&
               (local_48[0xc] == 'p')))) &&
            ((((local_48[0x17] == 'r' && (local_48[0x1c] == 'b')) &&
               ((local_48[0x21] == 'p' &&
                 (((local_48[2] == 'B' && (local_48[0x1b] == '_')) &&
                   (local_48[0xb] == '4')))))) &&
              (local_48[0x25] == '3')))))))) {
        puts("Yes! That\'s right!");
    } else {
        puts("No... not that");
    }
    return 0;
}
```

The flag is compared character by character.

```py
import angr

TARGET_ADDR = 0x401371

proj = angr.Project("./tablet", auto_load_libs=False)
start_state = proj.factory.entry_state()
simgr = proj.factory.simulation_manager(start_state)
simgr.explore(find=TARGET_ADDR)

if simgr.found:
    st = simgr.found[0]
    input = st.posix.dumps(0)
    print(input.strip(b"\x00").decode())
```

```plaintext
HTB{br0k3n_4p4rt...n3ver_t0_b3_r3p41r3d}
```
