# Useful

## Reversing

### Renode

Start container:

```plaintext
docker run -it --rm -p 12345:12345 -v /workspace:/workspace antmicro/renode
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
start
```

Interact with binary:

```plaintext
telnet localhost 12345
```
