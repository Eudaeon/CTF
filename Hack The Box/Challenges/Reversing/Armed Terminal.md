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

Binary is obfuscated with control flow flattening.

```plaintext
setup_sigill_handler:
    0x10314:     push    {lr}
    0x10318:     sub     sp, sp, #148
    0x1031c:     str     r0, [sp, #4]
    0x10320:     mov     r2, #0
    0x10324:     str     r2, [sp, #12]
    0x10328:     str     r2, [sp, #8]
    0x1032c:     mov     r0, #4
    0x10330:     str     r0, [sp, #136]
    0x10334:     add     r1, sp, r0
    0x10338:     mov     r7, #67 @ 0x43
    0x1033c:     svc     0x00000000
    0x10340:     add     sp, sp, #148
    0x10344:     pop     {pc}
```

Binary starts by setting up a signal handler for SIGILL using a syscall to `sigaction`. These will be handled by jumping to the value in `r1`, which contains a stack address with value `0x10240`. It then jumps back to the start of the program.

```plaintext
start:
    0x10054:     ldr     r0, [pc, #764]
    0x10058:     bl      0x10314
    0x1005c:     udf     #0
```

This triggers a SIGILL with value 0.

SIGILLs store the current sigframe onto the stack.

```plaintext
retrieve_value_jump_table:
    0x10244:     ldr     r4, [r3]
    0x10248:     lsr     r0, r4, #4
    0x1024c:     and     r3, r4, #15
    0x10250:     and     r0, r0, #240
    0x10254:     orr     r0, r0, r3
    0x10258:     and     r3, r4, #240
    0x1025c:     cmp     r3, #240
    0x10260:     uxtbne  r0, r4
    0x10264:     add     r2, pc, #36
    0x10268:     ldr     r3, [r2, r0, lsl #2]
    0x1026c:     bic     r2, r3, #1
    0x10270:     str     r2, [sp, #92]
    0x10274:     ldr     r2, [sp, #96]
    0x10278:     bic     r2, r2, #32
    0x1027c:     lsl     r3, r3, #5
    0x10280:     and     r3, r3, #32
    0x10284:     orr     r3, r3, r2
    0x10288:     str     r3, [sp, #96]
    0x1028c:     bx      lr
```

This retrieves an address in the jump table, the offset being the SIGILL value. This address is then stored at `$sp + 92` (`pc` register in sigframe).

```plaintext
handle_sigreturn:
    0x40801481:  movs    r7, #119
    0x40801483:  svc     0
```

This makes a `sigreturn` syscall, which restores the environment from the sigframe. This will pop the address stored in the jump table into `pc`, jumping to it.

```plaintext
save_state:
    0x10224:     push    {r1, r2, r7}
    0x10228:     mov     r0, #0
    0x1022c:     push    {r0}
    0x10230:     mov     r2, #1
    0x10234:     mov     r1, sp
    0x10238:     udf     #20
```

```plaintext
read_char:
    0x100e8:     mov     r0, #1
    0x100ec:     mov     r7, #3
    0x100f0:     svc     0x00000000
    0x100f4:     udf     #21
```

```plaintext
restore_state:
    0x10124:     pop     {r0}
    0x10128:     pop     {r1, r2, r7}
    0x1012c:     bx      lr
    0x10130:     mov     r0, #0
```

These three functions are called to read a character from stdin, which is stored in `r0`.

Get address in jump table from offset:

```py
import sys
from pwn import *

JUMP_TABLE = 0x10290


def parse_offset(s):
    try:
        if s.startswith(("0x", "0X")):
            return int(s, 16)
        return int(s, 10)
    except Exception:
        raise ValueError(f"Invalid offset: {s}")


offset = parse_offset(sys.argv[1])

binary = ELF("./armedterminal")

addr = JUMP_TABLE + offset * 4
data = binary.read(addr, 4)

if len(data) == 4:
    value = u32(data, endian="little")
    print(f"Address: 0x{addr:08x}")
    print(f"Bytes  : {data.hex()}")
    print(f"Value  : {value} (0x{value:08x})")
```

The first 4 characters are checked directly:

```plaintext
check_first_char:
    0x100a0:     bl      0x10224
    0x100a4:     cmp     r0, #72
    0x100a8:     bne     0x101ac
    0x100ac:     udf     #6

check_second_char:
    0x10084:     bl      0x10224
    0x10088:     cmp     r0, #84
    0x1008c:     bne     0x101ac
    0x10090:     udf     #7

check_third_char:
    0x1014c:     bl      0x10224
    0x10150:     cmp     r0, #66
    0x10154:     bne     0x101ac
    0x10158:     udf     #8

check_fourth_char:
    0x100b0:     bl      0x10224
    0x100b4:     cmp     r0, #123
    0x100b8:     bne     0x101ac
    0x100bc:     udf     #9
```

The remaining ones are concatenated to form 8 bytes and then compared with computed values.

```plaintext
get_fifth_sixth_seven_eighth_chars_and_join_fifth_with_eighth:
    0x100c0:     bl      0x10224
    0x100c4:     mov     r4, r0
    0x100c8:     bl      0x10224
    0x100cc:     mov     r6, r0
    0x100d0:     bl      0x10224
    0x100d4:     uxtb    r4, r4
    0x100d8:     mov     r5, r0
    0x100dc:     bl      0x10224
    0x100e0:     orr     r4, r4, r0, lsl #24
    0x100e4:     udf     #13

join_sixth_char_with_fifth_and_eight:
    0x100f8:     lsls    r0, r6, #8
    0x100fc:     uxth    r0, r0
    0x10100:     orrs    r4, r4, r0
    0x10104:     udf     #17

join_seventh_char_with_fifth_sixth_and_eight:
    0x10108:     lsls    r0, r5, #16
    0x1010c:     and     r0, r0, #16711680
    0x10110:     orrs    r0, r0, r4
    0x10114:     udf     #14

load_key_and_xor_chars:
    0x10118:     ldr     r2, [pc, #576]
    0x1011c:     eor     r0, r0, r2
    0x10120:     udf     #22

check_chars:
    0x10190:     ldr     r1, [pc, #468]
    0x10194:     cmp     r0, r1
    0x10198:     bne     0x101ac
    0x1019c:     udf     #10
```

Characters from 5 to 8 (reversed) are XOR'ed with `\xba\xad\xf0\x0d` and compared against `\xc0\xc0\x82\x39`.

![[Pasted image 20250919120824.png]]

This should be `4rmz`.

```plaintext
get_ninth_tenth_eleventh_twelvth_chars:
    0x10060:     bl      0x10224
    0x10064:     mov     r6, r0
    0x10068:     bl      0x10224
    0x1006c:     mov     r5, r0
    0x10070:     bl      0x10224
    0x10074:     mov     r4, r0
    0x10078:     bl      0x10224
    0x1007c:     uxtb    r3, r0
    0x10080:     udf     #11

join_ninth_char_with_twelvth:
    0x1016c:     lsls    r0, r5, #16
    0x10170:     lsls    r4, r4, #8
    0x10174:     orr     r3, r3, r6, lsl #24
    0x10178:     and     r0, r0, #16711680
    0x1017c:     udf     #18

join_tenth_eleventh_chars_with_ninth_twelvth:
    0x10180:     orrs    r0, r0, r3
    0x10184:     uxth    r4, r4
    0x10188:     orrs    r0, r0, r4
    0x1018c:     udf     #12

load_key_and_xor_chars:
    0x101a0:     ldr     r2, [pc, #456]
    0x101a4:     eor     r0, r0, r2
    0x101a8:     udf     #15

check_chars:
    0x1015c:     ldr     r1, [pc, #516]
    0x10160:     cmp     r0, r1
    0x10164:     bne     0x101ac
    0x10168:     udf     #3
```

Characters from 9 to 12 are XOR'ed with `\xf0\x00\xba\xaa` and compared against `\xaf\x34\xf4\xee`.

![[Pasted image 20250919121849.png]]

This should be `_4ND`.
