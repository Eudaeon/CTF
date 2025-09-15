# Simple Encryptor

## Recon

Binary and encrypted file.

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace/rev_simpleencryptor]
└─$ file encrypt
encrypt: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=0bddc0a794eca6f6e2e9dac0b6190b62f07c4c75, for GNU/Linux 3.2.0, not stripped
```

## Reversing

```c
int main(void) {
    int iVar1;
    time_t tVar2;
    long in_FS_OFFSET;
    uint local_40;
    uint local_3c;
    long local_38;
    FILE* local_30;
    size_t local_28;
    void* local_20;
    FILE* local_18;
    long local_10;

    local_10 = *(long*)(in_FS_OFFSET + 0x28);
    local_30 = fopen("flag", "rb");
    fseek(local_30, 0, 2);
    local_28 = ftell(local_30);
    fseek(local_30, 0, 0);
    local_20 = malloc(local_28);
    fread(local_20, local_28, 1, local_30);
    fclose(local_30);
    tVar2 = time((time_t*)0x0);
    local_40 = (uint)tVar2;
    srand(local_40);
    for (local_38 = 0; local_38 < (long)local_28; local_38 = local_38 + 1) {
        iVar1 = rand();
        *(byte*)((long)local_20 + local_38) =
            *(byte*)((long)local_20 + local_38) ^ (byte)iVar1;
        local_3c = rand();
        local_3c = local_3c & 7;
        *(byte*)((long)local_20 + local_38) =
            *(byte*)((long)local_20 + local_38) << (sbyte)local_3c |
            *(byte*)((long)local_20 + local_38) >> 8 - (sbyte)local_3c;
    }
    local_18 = fopen("flag.enc", "wb");
    fwrite(&local_40, 1, 4, local_18);
    fwrite(local_20, 1, local_28, local_18);
    fclose(local_18);
    if (local_10 != *(long*)(in_FS_OFFSET + 0x28)) {
        __stack_chk_fail();
    }
    return 0;
}
```

The encryption process iterates over each character, XOR's them, and rotates them bitwise to the left using random numbers.

The seed used to generate the random numbers is written at the beginning of the encrypted file (first 4 bytes), so this is reversible.

```python
import struct
import ctypes

IN = "flag.enc"

libc = ctypes.CDLL("libc.so.6")


def rol8(v, s):
    v &= 0xFF
    s &= 7
    return ((v << s) | (v >> (8 - s))) & 0xFF


def ror8(v, s):
    v &= 0xFF
    s &= 7
    return ((v >> s) | ((v << (8 - s)) & 0xFF)) & 0xFF


with open(IN, "rb") as f:
    hdr = f.read(4)
    seed = struct.unpack("<I", hdr)[0]
    encrypted = bytearray(f.read())
    libc.srand(ctypes.c_uint(seed))

    out = bytearray(len(encrypted))
    for i in range(len(encrypted)):
        r1 = libc.rand()
        r2 = libc.rand()
        shift = r2 & 7
        tmp = ror8(encrypted[i], shift)
        orig = tmp ^ (r1 & 0xFF)
        out[i] = orig
print(out.decode())
```

```plaintext
HTB{vRy_s1MplE_F1LE3nCryp0r}
```
