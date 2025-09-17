# QuickScan

## Recon

```plaintext
I am about to send you 128 base64-encoded ELF files, which load a value onto the stack. You must send back the loaded value as a hex string
You must analyze them all in under 120 seconds
Let's start with a warmup
ELF:  f0VMRgIBAQAAAAAAAAAAAAIAPgABAAAAeIAECAAAAABAAAAAAAAAAAAAAAAAAAAAAAAAAEAAOAABAAAAAAAAAAEAAAAFAAAAAAAAAAAAAAAAgAQIAAAAAACABAgAAAAAMwMAAAAAAAAzAwAAAAAAAAAQAAAAAAAASIPsGEiNNVwBAABIiee5GAAAAPOkuDwAAAAPBTFLemEvpDOeXbwfFrPi37BELIRPWIBPehbb2Z1yhjIDm7tmaFyxyYo2wE+wF53pJEy/oe6rgKUY3n4NFsNagXFPx1EucD076o4JyTIfdq3Cui1twVMT7UP2HPZceCooHKa4IlR+93yl01AlzEODKCMh0nmZAbTDvPJVD309pTGNKD7YNYO8hYlnDrBuSeABV4i99E9Rc4eEoOVPfqFnUG48bSY/SjwhBcdWdspa7M6W33BHF2HWr389L+UjzY5QMJLzegd2aK3IN/7Gq4QvrMC9AIcFUaWEUvV6kfP3qootltDFFFeMuOjlbVFlXGqNc2Fmal+UDLv8iW8fMMoWt//LHXpxEq1MtvsxBA4cO8CVQMW8ufFxyoraEit8kXsvlGZhXKkiA1dw1nXIIAGncZbXOv8irU27RJYgBUDq6X0JL0CqXzbxney5CVfb7/MESBqzApRG7y4jDg3Ud9+eWlSAKPKW0eaS7VGb97qenT5BAE5SxNU1NCbl3z1K4sxzbcMEPUEWwYzCiDqy2wJh2pP3ZaMfID0xvE9ywWhuLUQRsBPIuW/rjgAnNqLud4SInARfo0H9UUAol7qi1dZezNCYJ/WPjL3ymqS5fC43+CiHgL4/qkh1HiqVmCSqmRVXPb7wVkGAGMz6+HaKfmmbGgl7N9+UlH75wkdSUAKD18WSx3k6FAliGTeoA+1kftzVWWp8mLlAoYFH06ZVFAGVzbAvO/Rp8QU3w9Bn/70Y0qZzsSBGAayRcO4SNxiSuqavSftiwgO+YxVr5mIzta+TiMqGCiXw37B4ddoDukEwg59J++FM4NlvcgVGQfqniK5EprqmV65HZmHuOICjWtw4JrN5UJ6TRicYcUCAHQOJYZURi2bVdVv+svtugJ4GZOqb
Expected bytes: dbeff304481ab3029446ef2e230e0dd477df9e5a548028f2
Bytes?
```

## Analysis

Binary loads value address in `rsi` register, and moves value length in `ecx` register.

```py
from pwn import *
import base64
import r2pipe

HOST = "94.237.61.242"
PORT = 39034
STEPS = 128

context.log_level = "DEBUG"

io = remote(HOST, PORT)
for _ in range(STEPS + 1):
    io.recvuntil(b"ELF:  ")
    base64_elf = io.recvline().strip().decode()
    elf_data = base64.b64decode(base64_elf)
    with open("challenge.elf", "wb") as f:
        f.write(elf_data)
    r2 = r2pipe.open("./challenge.elf")
    r2.cmd("aaa")
    disasm = r2.cmdj("pdfj @entry0")
    for op in disasm.get("ops", []):
        disasm_str = op.get("disasm", "")
        if disasm_str.startswith("lea rsi, [0x"):
            addr = disasm_str.split(", ")[1].split("[")[-1].strip("]")
        if disasm_str.startswith("mov ecx, "):
            try:
                length_tmp = int(disasm_str.split(", ")[1], 16)
                if length_tmp < 0x100:
                    length = length_tmp
            except:
                pass
    value = r2.cmdj(f"pxj {length} @ {addr}")
    hex_value = "".join(f"{b:02x}" for b in value)
    io.sendlineafter(b"Bytes? ", hex_value.encode())
line = io.recvline().strip().decode()
print(line)
```

```plaintext
Wow, you did them all. Here's your flag: HTB{y0u_4n4lyz3d_th3_p4tt3rns!}
```
