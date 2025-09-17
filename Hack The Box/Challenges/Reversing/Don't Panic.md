# Don't Panic

## Recon

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace/rev_dontpanic]
└─$ file dontpanic 
dontpanic: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=ea636c24aa596a8aac8801e4b5e2507007a264aa, for GNU/Linux 3.2.0, with debug_info, not stripped
```

Rnu:

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace/rev_dontpanic]
└─$ ./dontpanic 
🤖💬 < Have you got a message for me? > 🗨️ 🤖: foo
😱😱😱 You made me panic! 😱😱😱
```

## Reversing

```c
void __rustcall src::check_flag(long param_1, long param_2) {
    long lVar1;
    long lVar2;
    long extraout_RAX;
    long extraout_RAX_00;
    undefined1 auVar3[16];
    long lStack_1e0;
    undefined1* puStack_1d8;
    undefined8 uStack_1d0;
    undefined** ppuStack_1c8;
    undefined8 uStack_1c0;
    undefined* puStack_1b8;
    undefined8 uStack_1b0;
    undefined8 uStack_1a8;
    undefined8 uStack_198;
    long lStack_188;
    long local_168[2];
    code* local_158[35];
    undefined8 local_40[6];

    local_158[0] = core::ops::function::FnOnce::call_once;
    local_158[1] = core::ops::function::FnOnce::call_once;
    local_158[2] = core::ops::function::FnOnce::call_once;
    local_158[3] = core::ops::function::FnOnce::call_once;
    local_158[4] = core::ops::function::FnOnce::call_once;
    local_158[5] = core::ops::function::FnOnce::call_once;
    local_158[6] = core::ops::function::FnOnce::call_once;
    local_158[7] = core::ops::function::FnOnce::call_once;
    local_158[8] = core::ops::function::FnOnce::call_once;
    local_158[9] = core::ops::function::FnOnce::call_once;
    local_158[10] = core::ops::function::FnOnce::call_once;
    local_158[0xb] = core::ops::function::FnOnce::call_once;
    local_158[0xc] = core::ops::function::FnOnce::call_once;
    local_158[0xd] = core::ops::function::FnOnce::call_once;
    local_158[0xe] = core::ops::function::FnOnce::call_once;
    local_158[0xf] = core::ops::function::FnOnce::call_once;
    local_158[0x10] = core::ops::function::FnOnce::call_once;
    local_158[0x11] = core::ops::function::FnOnce::call_once;
    local_158[0x12] = core::ops::function::FnOnce::call_once;
    local_158[0x13] = core::ops::function::FnOnce::call_once;
    local_158[0x14] = core::ops::function::FnOnce::call_once;
    local_158[0x15] = core::ops::function::FnOnce::call_once;
    local_158[0x16] = core::ops::function::FnOnce::call_once;
    local_158[0x17] = core::ops::function::FnOnce::call_once;
    local_158[0x18] = core::ops::function::FnOnce::call_once;
    local_158[0x19] = core::ops::function::FnOnce::call_once;
    local_158[0x1a] = core::ops::function::FnOnce::call_once;
    local_158[0x1b] = core::ops::function::FnOnce::call_once;
    local_158[0x1c] = core::ops::function::FnOnce::call_once;
    local_158[0x1d] = core::ops::function::FnOnce::call_once;
    local_158[0x1e] = core::ops::function::FnOnce::call_once;
    local_158[0x1f] = core::ops::function::FnOnce::call_once;
    local_158[0x20] = core::ops::function::FnOnce::call_once;
    local_158[0x21] = core::ops::function::FnOnce::call_once;
    local_158[0x22] = core::ops::function::FnOnce::call_once;
    local_168[1] = 0x23;
    if (param_2 == 0x23) {
        local_168[0] = 0x23;
        lVar2 = 0;
        do {
            lVar1 = lVar2 + 1;
            (*local_158[lVar2])(*(undefined1*)(param_1 + lVar2));
            lVar2 = lVar1;
        } while (lVar1 != 0x23);
        return;
    }
    local_40[0] = 0;
    local_168[0] = param_2;
    core::panicking::assert_failed(local_168, local_168 + 1, local_40);
    lStack_188 = param_1;
    std::panicking::set_hook();
    ppuStack_1c8 = &PTR_DAT_00158250;
    uStack_1c0 = 1;
    puStack_1b8 = &DAT_00147000;
    uStack_1b0 = 0;
    uStack_1a8 = 0;
    std::io::stdio::_print();
    std::io::stdio::stdout();
    std::io::stdio::flush();
    if (extraout_RAX != 0) {
        core::ptr::drop_in_place<>(&lStack_1e0);
    }
    lStack_1e0 = 0;
    puStack_1d8 = &DAT_00000001;
    uStack_1d0 = 0;
    std::io::stdio::stdin();
    std::io::stdio::Stdin::read_line();
    if (extraout_RAX_00 != 0) {
        core::result::unwrap_failed();
    }
    auVar3 = remove_newline(puStack_1d8, uStack_1d0);
    check_flag(auVar3._0_8_, auVar3._8_8_);
    uStack_198 = 0;
    ppuStack_1c8 = &PTR_DAT_00158278;
    uStack_1c0 = 1;
    puStack_1b8 = &DAT_00147000;
    uStack_1b0 = 0;
    uStack_1a8 = 0;
    std::io::stdio::_print();
    if (lStack_1e0 != 0) {
        std::alloc::__default_lib_allocator::__rust_dealloc(puStack_1d8);
    }
    return;
}

undefined8 __rustcall core::ops::function::FnOnce::call_once(byte param_1) {
    undefined8 in_RAX;

    if (param_1 < 0x48) {
        panicking::panic();
    }
    if (param_1 == 0x48) {
        return in_RAX;
    }
    panicking::panic();
}
```

Binary uses closures (anonymous functions) to check each input character, so this creates one function per character.

```py
import angr

BIN = "./dontpanic"
TARGET_ADDR = 0x10936a

proj = angr.Project(BIN, auto_load_libs=False)
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
