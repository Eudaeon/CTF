# Industry Secret

## Recon

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace/rev_industrysecret]
└─$ file zephyr.elf                
zephyr.elf: ELF 32-bit LSB executable, ARM, EABI5 version 1 (SYSV), statically linked, with debug_info, not stripped
```

Run on nRF board:

```plaintext
*** Using Zephyr OS v3.7.99-1f8f3dc29142 ***
** Critical Industrial Machinery **
==============================
[1] READ LOGS
[2] EMERGENCY STOP
==============================
```

## Reversing

```c
int main(void){
  undefined1 uVar1;
  undefined4 *puVar2;
  _Bool _Var3;
  _Bool _Var4;
  uchar uVar5;
  int iVar6;
  char *buf;
  size_t sVar7;
  undefined4 uVar8;
  undefined4 *puVar9;
  undefined4 *puVar10;
  _Bool _Var11;
  _Bool _Var12;
  _Bool _Var13;
  uchar *puVar14;
  uchar cmpbuf [21];
  char buffer [64];
  
  _Var4 = z_impl_device_is_ready(&__device_dts_ord_113);
  if (_Var4) {
    uarte_nrfx_irq_callback_set
              (&__device_dts_ord_113,(uart_irq_callback_user_data_t)0x111,(void *)0x0);
    uarte_nrfx_irq_rx_enable(&__device_dts_ord_113);
    print_uart("** Critical Industrial Machinery **\r\n");
    print_uart("==============================\r\n");
    print_uart("[1] READ LOGS\r\n");
    print_uart("[2] EMERGENCY STOP\r\n");
    print_uart("==============================\r\n");
    _Var13 = false;
    _Var3 = false;
    _Var11 = false;
    while (_Var12 = _Var3,
          iVar6 = z_impl_k_msgq_get(&uart_msg_queue,buffer,(k_timeout_t)0xffffffffffffffff),
          iVar6 == 0) {
      print_uart("\r");
      iVar6 = 0x3f;
      do {
        print_uart(" ");
        iVar6 = iVar6 + -1;
      } while (iVar6 != 0);
      print_uart("\r");
      iVar6 = memcmp(buffer,"1",2);
      if (iVar6 == 0) {
        if (_Var13 == false) {
          print_uart("100%% Operational\r\n");
          buf = "All Systems Online\r\n";
        }
        else {
          print_uart("0%% Operational\r\n");
          buf = "All Systems Offline\r\n";
        }
LAB_00000334:
        print_uart(buf);
LAB_00000338:
        print_uart("==============================\r\n");
        _Var3 = _Var12;
      }
      else {
        iVar6 = memcmp(buffer,"2",2);
        if (iVar6 == 0) {
          if (_Var11 == false) {
            print_uart("Remote Emergency Stop\r\n");
            buf = "Is DISABLED\r\n";
            goto LAB_00000334;
          }
          print_uart("System Shutdown...\r\n");
          _Var13 = _Var11;
          goto LAB_00000338;
        }
        iVar6 = memcmp(buffer,"secretpassword",0xf);
        _Var3 = _Var4;
        if (((iVar6 != 0) && (_Var3 = _Var12, _Var12 != false)) &&
           (sVar7 = strlen(buffer), sVar7 == 0x14)) {
          iVar6 = 0;
          puVar14 = (uchar *)buffer;
          do {
            uVar5 = encode(*puVar14,iVar6);
            iVar6 = iVar6 + 1;
            *puVar14 = uVar5;
            puVar14 = puVar14 + 1;
          } while (iVar6 != 0x14);
          puVar14 = cmpbuf;
          puVar2 = &DAT_00005a04;
          do {
            puVar10 = puVar2;
            puVar9 = (undefined4 *)puVar14;
            uVar8 = puVar10[1];
            *puVar9 = *puVar10;
            puVar9[1] = uVar8;
            puVar14 = (uchar *)(puVar9 + 2);
            puVar2 = puVar10 + 2;
          } while (puVar10 + 2 != &DAT_00005a14);
          uVar1 = *(undefined1 *)(puVar10 + 3);
          puVar9[2] = 0xeb0bb429;
          *(undefined1 *)(puVar9 + 3) = uVar1;
          iVar6 = memcmp(buffer,cmpbuf,0x15);
          if (iVar6 == 0) {
            print_uart("Backdoor enabled\r\n");
            print_uart("==============================\r\n");
            _Var11 = _Var12;
          }
        }
      }
    }
  }
  return 0;
}
```

There's a secret `secretpassword <PASSWORD>` command. The whole line should be 20 characters long, and is processed by the binary before being compared against a hardcoded string.

```c
const char DAT_00005a1c[4] = {0x00, 0x01, 0x03, 0x02}
const char DAT_00005a18[4] = {0x06, 0x03, 0x05, 0x04} 

uchar encode(uchar o,int index){
  byte bVar1;
  uint uVar2;
  int extraout_r1;
  int iVar3;
  uint uVar4;
  uint unaff_r4;
  uint uVar5;
  
  bVar1 = shuffle(o);
  uVar2 = (uint)bVar1;
  uVar5 = 5;
  iVar3 = extraout_r1 % 5;
  if (iVar3 == 0) {
    unaff_r4 = 2;
  }
  else if (iVar3 - 1U < 4) {
    unaff_r4 = (uint)(byte)(&DAT_00005a1c)[iVar3 - 1U];
    uVar5 = (uint)(byte)(&DAT_00005a18)[iVar3 - 1U];
  }
  else {
    uVar5 = 0;
  }
  uVar4 = 1 << (unaff_r4 & 0xff);
  uVar2 = (((uVar4 & uVar2) >> (unaff_r4 & 0xff)) << uVar5 |
          ((1 << uVar5 & uVar2) >> uVar5) << (unaff_r4 & 0xff)) & 0xff |
          uVar2 & ~(uVar4 | 1 << uVar5);
  if (extraout_r1 % 0xb == 0) {
    uVar2 = uVar2 - 0x62;
  }
  else {
    switch(extraout_r1 % 0xb) {
    case 1:
      uVar2 = uVar2 - 2;
      break;
    case 2:
      uVar2 = uVar2 - 0x56;
      break;
    case 3:
      uVar2 = uVar2 - 0xf6;
      break;
    case 4:
      uVar2 = uVar2 - 0x46;
      break;
    case 5:
      uVar2 = uVar2 - 0xc;
      break;
    case 6:
      uVar2 = uVar2 - 0x99;
      break;
    case 7:
      uVar2 = uVar2 - 0x14;
      break;
    case 8:
      uVar2 = uVar2 - 0x85;
      break;
    case 9:
      uVar2 = uVar2 - 0xab;
      break;
    case 10:
      uVar2 = uVar2 - 0xba;
      break;
    default:
      goto switchD_00000252_default;
    }
  }
  if ((int)uVar2 < 0) {
    uVar2 = uVar2 + 0x100;
  }
switchD_00000252_default:
  return (uchar)uVar2;
}


uchar shuffle(uchar x){
  if ((byte)(x - 0x21) < 0x5e) {
    x = "B2yO^?3[1$g5*,JKI4zsowp~HWT(9)ZUtLk\\7\";rxb|RQnP]!G`mD+e'>{Yhc/a0CN8q=}A<vSX&f@i_u6.-E#Vlj:%FdM"
        [(byte)(x - 0x21)];
  }
  return x;
}
```

The binary iterates over each character of the command. Each character is substituted, and the resulting value is modified depending on the index of the character in the string.

Reverse the process with:

```py
CIPHERTEXT = "f46bf017204bd2629ab18bd177f1661f29b40beb"

DAT_00005a1c = [0x00, 0x01, 0x03, 0x02]
DAT_00005a18 = [0x06, 0x03, 0x05, 0x04]

shuffle_string = "B2yO^?3[1$g5*,JKI4zsowp~HWT(9)ZUtLk\\7\";rxb|RQnP]!G`mD+e'>{Yhc/a0CN8q=}A<vSX&f@i_u6.-E#Vlj:%FdM"
inv_shuffle = {ord(c): i + 0x21 for i, c in enumerate(shuffle_string)}

reverse_cases = {
    0: 0x62,
    1: 2,
    2: 0x56,
    3: 0xF6,
    4: 0x46,
    5: 0xC,
    6: 0x99,
    7: 0x14,
    8: 0x85,
    9: 0xAB,
    10: 0xBA,
}

encoded_bytes = bytes.fromhex(CIPHERTEXT)
decoded_chars = []

for i, c in enumerate(encoded_bytes):
    uVar2 = (c + reverse_cases[i % 0xB]) & 0xFF

    iVar3 = i % 5
    if iVar3 == 0:
        unaff_r4, uVar5 = 2, 5
    else:
        unaff_r4, uVar5 = DAT_00005a1c[iVar3 - 1], DAT_00005a18[iVar3 - 1]

    uVar4 = 1 << unaff_r4
    uVar2 = (
        ((uVar4 & uVar2) >> unaff_r4) << uVar5
        | ((1 << uVar5 & uVar2) >> uVar5) << unaff_r4
    ) & 0xFF | uVar2 & ~(uVar4 | 1 << uVar5)

    decoded_char = inv_shuffle.get(uVar2, uVar2)
    decoded_chars.append(chr(decoded_char))

decoded_string = "".join(decoded_chars)
print(decoded_string)
```

```plaintext
HTB{H4CKED_H4RDWARE}
```
