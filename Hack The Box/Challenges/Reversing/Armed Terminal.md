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

This program starts by setting up a signal handler for SIGILL using a syscall to `sigaction`. These will be handled by jumping to the value in `r1`, which contains a stack address with value `0x10240`. It then jumps back to the start of the program.

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

This retrieves the opcode value of the SIGILL that triggered the jump to this, and the value stored at the SIGILL offset in the jump table. This value is then stored at `$sp + 92` (`pc` register in sigframe). It also sets the `r7` register in sigframe with the syscall number for `sigaction`.

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

This saves a few registers onto the stack, and triggers a SIGILL with value 20, which goes back to `0x10244`.

```plaintext
read_char:
    0x100e8:     mov     r0, #1
    0x100ec:     mov     r7, #3
    0x100f0:     svc     0x00000000
    0x100f4:     udf     #21
```

The function at offset 20 is used to read a character from stdin.

The jump table should then return the address of the character-checking function.

```plaintext
restore_state:
    0x10124:     pop     {r0}
    0x10128:     pop     {r1, r2, r7}
    0x1012c:     bx      lr
    0x10130:     mov     r0, #0
```

Stores in `r0` character read.

save_state => read_char => restore_state
