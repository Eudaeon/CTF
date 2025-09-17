# Crushing

## Recon

Binary and compressed file.

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace/rev_crushing]
└─$ file crush 
crush: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=fdbeaffc5798388ef49c3f3716903d37a53e8e03, for GNU/Linux 3.2.0, not stripped
```

Run:

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace/rev_crushing]
└─$ ./crush
foo
```

## Reversing

```c
int main(void) {
    long lVar1;
    undefined8* puVar2;
    undefined8 local_818[256];
    uint local_14;
    long local_10;

    puVar2 = local_818;
    for (lVar1 = 0xff; lVar1 != 0; lVar1 = lVar1 + -1) {
        *puVar2 = 0;
        puVar2 = puVar2 + 1;
    }
    local_10 = 0;
    while (true) {
        local_14 = getchar();
        if (local_14 == 0xffffffff)
            break;
        add_char_to_map(local_818, local_14 & 0xff, local_10);
        local_10 = local_10 + 1;
    }
    serialize_and_output(local_818);
    return 0;
}
void add_char_to_map(long param_1, byte param_2, undefined8 param_3) {
    undefined8* puVar1;
    long local_10;

    local_10 = *(long*)(param_1 + (ulong)param_2 * 8);
    puVar1 = (undefined8*)malloc(0x10);
    *puVar1 = param_3;
    puVar1[1] = 0;
    if (local_10 == 0) {
        *(undefined8**)((ulong)param_2 * 8 + param_1) = puVar1;
    } else {
        for (; *(long*)(local_10 + 8) != 0; local_10 = *(long*)(local_10 + 8)) {
        }
        *(undefined8**)(local_10 + 8) = puVar1;
    }
    return;
}

void serialize_and_output(long param_1) {
    undefined8 local_28;
    undefined8* local_20;
    void* local_18;
    int local_c;

    for (local_c = 0; local_c < 0xff; local_c = local_c + 1) {
        local_20 = (undefined8*)(param_1 + (long)local_c * 8);
        local_28 = list_len(local_20);
        fwrite(&local_28, 8, 1, stdout);
        for (local_18 = (void*)*local_20; local_18 != (void*)0x0;
             local_18 = *(void**)((long)local_18 + 8)) {
            fwrite(local_18, 8, 1, stdout);
        }
    }
    return;
}
void add_char_to_map(long param_1, byte param_2, undefined8 param_3) {
    undefined8* puVar1;
    long local_10;

    local_10 = *(long*)(param_1 + (ulong)param_2 * 8);
    puVar1 = (undefined8*)malloc(0x10);
    *puVar1 = param_3;
    puVar1[1] = 0;
    if (local_10 == 0) {
        *(undefined8**)((ulong)param_2 * 8 + param_1) = puVar1;
    } else {
        for (; *(long*)(local_10 + 8) != 0; local_10 = *(long*)(local_10 + 8)) {
        }
        *(undefined8**)(local_10 + 8) = puVar1;
    }
    return;
}

void serialize_and_output(long param_1) {
    undefined8 local_28;
    undefined8* local_20;
    void* local_18;
    int local_c;

    for (local_c = 0; local_c < 0xff; local_c = local_c + 1) {
        local_20 = (undefined8*)(param_1 + (long)local_c * 8);
        local_28 = list_len(local_20);
        fwrite(&local_28, 8, 1, stdout);
        for (local_18 = (void*)*local_20; local_18 != (void*)0x0;
             local_18 = *(void**)((long)local_18 + 8)) {
            fwrite(local_18, 8, 1, stdout);
        }
    }
    return;
}
```

Binary reads character by character. It uses a hashmap with ASCII characters as keys. Values are linked lists corresponding to the character positions in the input string. The hashmap is serialized when exiting the program.

```py
import struct

IN = "message.txt.cz"

chars = {}
with open(IN, "rb") as f:
    index = 0
    while True:
        num_bytes = f.read(8)
        if not num_bytes:
            break
        (num,) = struct.unpack("<Q", num_bytes)
        values = []
        for _ in range(num):
            value_bytes = f.read(8)
            (value,) = struct.unpack("<Q", value_bytes)
            values.append(value)
        chars[index] = values
        index += 1

max_pos = max(p for pos_list in chars.values() for p in pos_list)
decoded = [""] * (max_pos + 1)
for char_code, positions in chars.items():
    for pos in positions:
        decoded[pos] = chr(char_code)
original_string = "".join(decoded)
print(original_string)
```

```plaintext
Organizer 1: Hey, did you finalize the password for the next... you know?

Organizer 2: Yeah, I did. It's "HTB{4_n0t_s0_g00d_b4d_compr3ss1on_sch3m3}"

Organizer 1: "HTB{4_n0t_s0_g00d_b4d_compr3ss1on_sch3m3}". Okay, got it. Sounds ominous enough to keep things interesting. Where do we spread the word?

Organizer 2: Let's stick to the usual channels: encrypted messages to the leaders and discreetly slip it into the training manuals for the participants.

Organizer 1: Perfect. And let's make sure it's not leaked this time. Last thing we need is an early bird getting the worm.

Organizer 2: Agreed. We can't afford any slip-ups, especially with the stakes so high. The anticipation leading up to it should be palpable.

Organizer 1: Absolutely. The thrill of the unknown is what keeps them coming back for more. "HTB{4_n0t_s0_g00d_b4d_compr3ss1on_sch3m3}" it is then.
```
