# ARMs Race

## Recon

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace]
└─$ nc 94.237.120.143 42600
Level 1/50: 370301e3e01804e3651940e38d2902e340224ae3900100e0900200e0341a0ee30a1b48e3da2e02e3522748e3900100e0900200e0f5190fe3b91c4ae3972207e3b0244fe3010040e00200c0e06b1d08e31d1e4ee3532b07e308294ee3000060e2b41206e3321045e3192ca0e3cc2740e3900100e0900200e0d81b05e347184be3f4200fe3f62b42e3900100e0900200e0e01f0de360164be35e2f02e3112343e3010080e00200a0e0481205e30a114ce3252909e3222f4ee3010020e0020020e0d41a05e3cb1e47e3e52102e3452346e3010080e1020080e15b120de3361945e3cb2a02e3b12d4ae3000060e2d61f05e3111c49e3c82802e3962740e3010040e00200c0e0cf1c05e3111342e3322d01e3c12c4ae3010000e0020000e07f1d0be300104ae3fb2603e3d2234ae3010080e00200a0e0e31a09e30a1644e3a62007e34a2c40e3010000e0020000e08b1108e3541c40e3622c0be3762845e3900100e0900200e0de1a09e39f1346e363290de30e204ce3900100e0900200e012170fe3bc1a46e38e2b0be32c2f4ae3010000e0020000e0b71c0ee321104ce3c62b0ae3312f41e3010080e1020080e1bd170ee37a1245e3b5220ce32d2744e3010080e1020080e1f61d06e3461f41e38a260ae334224ce3900100e0900200e0
Register r0:
```

## Analysis

Binary loads values into registers and performs arithmetic operations between them. We're only interested in final value of `r0`. Use CPU emulation.

```py
from pwn import *
from unicorn import *
from unicorn.arm_const import *

HOST = "94.237.120.143"
PORT = 42600
STEPS = 50

ADDRESS = 0x1000

context.log_level = "DEBUG"

io = remote(HOST, PORT)

for _ in range(STEPS):
    io.recvuntil(b"/50: ")
    hex_elf = io.recvline().strip().decode()
    elf_data = bytes.fromhex(hex_elf)

    mu = Uc(UC_ARCH_ARM, UC_MODE_ARM)
    mu.mem_map(ADDRESS, 2 * 1024 * 1024)
    mu.mem_write(ADDRESS, elf_data)
    mu.emu_start(ADDRESS, ADDRESS + len(elf_data))
    r0 = mu.reg_read(UC_ARM_REG_R0)
    r1 = mu.reg_read(UC_ARM_REG_R1)
    r2 = mu.reg_read(UC_ARM_REG_R2)

    io.recv()
    io.sendline(str(r0).encode())

line = io.recvline().strip().decode()
print(line)
```

```plaintext
HTB{un1c0Rn_0R_C4p5T0nE_0R_qeMU_5ubPR0cE55_0r_F0r90TTen_0Ld_R45berry_4NyTH1N9_BUt_n0_M4nu4LLy}
```
