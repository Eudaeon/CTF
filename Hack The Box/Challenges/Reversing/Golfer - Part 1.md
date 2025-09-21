# Golfer - Part 1

## Recon

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace/rev_golfer]
└─$ file golfer       
golfer: ELF 32-bit (Cell LV2)
```

## Reversing

```plaintext
entry0:
    0x0800004c:     jmp 0x8000127
    0x08000051:     inc bl
    0x08000053:     inc dl
    0x08000055:     mov ecx, 0x800000a
    0x0800005a:     call fcn.0800012f
    0x0800005f:     mov ecx, 0x8000008
    0x08000064:     call fcn.0800012f
    0x08000069:     mov ecx, 0x8000024
    0x0800006e:     call fcn.0800012f
    0x08000073:     mov ecx, 0x800000e
    0x08000078:     call fcn.0800012f
    0x0800007d:     mov ecx, 0x800000c
    0x08000082:     call fcn.0800012f
    0x08000087:     mov ecx, 0x8000023
    0x0800008c:     call fcn.0800012f
    0x08000091:     mov ecx, 0x8000009
    0x08000096:     call fcn.0800012f
    0x0800009b:     mov ecx, 0x8000021
    0x080000a0:     call fcn.0800012f
    0x080000a5:     mov ecx, 0x8000006
    0x080000aa:     call fcn.0800012f
    0x080000af:     mov ecx, 0x800000d
    0x080000b4:     call fcn.0800012f
    0x080000b9:     mov ecx, 0x8000022
    0x080000be:     call fcn.0800012f
    0x080000c3:     mov ecx, 0x8000021
    0x080000c8:      call fcn.0800012f
    0x080000cd:     mov ecx, 0x8000005
    0x080000d2:     call fcn.0800012f
    0x080000d7:     mov ecx, 0x8000021
    0x080000dc:     call fcn.0800012f
    0x080000e1:     mov ecx, 0x8000020
    0x080000e6:     call fcn.0800012f
    0x080000eb:     mov ecx, 0x8000023
    0x080000f0:     call fcn.0800012f
    0x080000f5:     mov ecx, 0x800000f
    0x080000fa:     call fcn.0800012f
    0x080000ff:     mov ecx, 0x8000007
    0x08000104:     call fcn.0800012f
    0x08000109:     mov ecx, 0x8000022
    0x0800010e:     call fcn.0800012f
    0x08000113:     mov ecx, 0x8000025
    0x08000118:     call fcn.0800012f
    0x0800011d:     mov ecx, 0x800000b
    0x08000122:     call fcn.0800012f
    0x08000127:     xor al, al
    0x08000129:     inc al
    0x0800012b:     mov bl, 0x2a
    0x0800012d:     int 0x80

fcn.0800012f:
    0x0800012f:     push ebp
    0x08000130:     mov ebp, esp
    0x08000132:     mov al, 4
    0x08000134:     int 0x80
    0x08000136:     leave
    0x08000137:     ret
```

Binary starts by jumping straight to the end, which triggers a syscall to `exit`.

Without this jump, the binary sets addreses in `ecx` and makes syscalls to `write` to write 1 byte of the value stored at the address in `ecx.`

Flag: `HTB{y0U_4R3_a_g0lf3r}`
