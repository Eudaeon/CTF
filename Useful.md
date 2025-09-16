# Useful

## Reversing

### Renode

Start container:

```plaintext
docker run -it --rm -p 12345:12345 -p 33333:33333 -v /workspace:/workspace antmicro/renode
```

Load binary:

```plaintext
using sysbus
mach create
machine LoadPlatformDescription @platforms/cpus/nrf52840.repl
sysbus LoadELF @/workspace/binary.elf
showAnalyzer uart0
emulation CreateServerSocketTerminal 12345 "term"
connector Connect sysbus.uart0 term
machine StartGdbServer 33333
```

Connect to GDB server:

```plaintext
target remote :33333
```

Interact with binary:

```plaintext
telnet localhost 12345
```
